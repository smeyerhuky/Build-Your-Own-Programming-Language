---
type: Design Doc
title: "Conductor GPU Serving Layer Design"
description: vLLM-based GPU serving for Conductor's private and fast model tiers, using Gemma4 models with AWQ quantization and MTP speculative decoding on AWS g5.xlarge spot instances across three regions. Covers model rationale, deployment configuration, spot interruption handling, and cost model.
resource: /startup/06-design/gpu-serving-design.md
tags: [gpu, vllm, gemma4, speculative-decoding, mtp, aws, serving, conductor]
timestamp: 2026-06-28T00:00:00Z
---

# Conductor GPU Serving Layer Design

## 1. Overview

The GPU serving layer provides Conductor's **Tier 2 (private)** and **Tier 3 (fast)** model tiers using open-weight Gemma4 models served via vLLM, with AWQ quantization and Multi-Token Prediction (MTP) speculative decoding. It exposes an OpenAI-compatible REST API so the model router can switch between Bedrock, OpenAI, and the private GPU tier by changing only the `base_url` — no other code changes.

The layer runs on AWS g5.xlarge spot instances ($0.35/hr each), deployed across three regions for latency and fault tolerance. At ~60 tokens/second throughput per instance for the 26B model and ~80 tok/s for the 12B model, a single instance can handle a typical pipeline run's LLM work in seconds. The cost advantage over Bedrock is roughly **60–300×** per token — the primary economic motivation for operating private GPU infrastructure.

All traffic flows through the model router (runtime component); the GPU layer is not directly accessible by tenants.

---

## 2. Model Selection Rationale

| Model | Architecture | Total Params | Active Params / Token | VRAM Required (AWQ Q4) | Throughput (A10G) | Conductor Tier |
|---|---|---|---|---|---|---|
| Gemma4-26B-A4B | MoE | 26B | ~4B | ~12 GB | ~60 tok/s | Tier 2 (privacy) |
| Gemma4-12B | Dense | 12B | 12B | ~8 GB | ~80 tok/s | Tier 3 (fast, free) |
| Gemma4-31B | Dense | 31B | 31B | ~22 GB | ~15 tok/s | Reserved (H100) |

**Why Gemma4-26B-A4B for Tier 2?**
The 26B-A4B model is a Mixture-of-Experts architecture: 26B total parameters, but only ~4B are active during any single forward pass (the MoE gating selects a subset of experts per token). This means it fits on a single A10G (24GB VRAM) under AWQ Q4 quantization, while still delivering quality comparable to much larger dense models for code and reasoning tasks. The MoE structure also makes it a natural fit for batched inference — different tokens in the same batch route through different experts, increasing GPU utilization.

**Why Gemma4-12B for Tier 3?**
The 12B dense model is fast (~80 tok/s on A10G), small (~8GB VRAM, leaving headroom for KV-cache), and ideal for the free tier where cost per token matters most. It handles summarization, code explanation, and light refactoring tasks well. Users on the free tier cannot access Bedrock models, so this is their only LLM option.

---

## 3. MTP Speculative Decoding

Standard autoregressive LLM inference is bottlenecked by memory bandwidth: each token requires a full forward pass through the model. MTP (Multi-Token Prediction) speculative decoding reduces this bottleneck:

```
Standard autoregressive:
Token 1 → [forward pass] → Token 2 → [forward pass] → Token 3 → ... (N forward passes for N tokens)

MTP speculative decoding:
Drafter predicts tokens 1..N autoregressively (fast, small model)
                    │
                    ▼
Target model verifies all N tokens in ONE forward pass (parallel)
   - Accepted tokens: all correct predictions kept
   - First mismatch: reject remaining draft tokens, take target's token
   - Net: fewer forward passes → higher throughput
```

**Drafter Model**
The drafter for Gemma4 MTP is a small auxiliary head trained alongside the target model (`gemma-4-26B-A4B-it-assistant` in vLLM's speculative model notation). Key properties:
- 4 transformer layers, ~200MB
- Has access to the target model's KV-cache → higher acceptance rate than external speculative models
- Predicts 5 tokens ahead (configurable via `--num-speculative-tokens`)

**Adaptive Acceptance Rate**
vLLM's speculative scheduler tracks the rolling acceptance rate:
- If the last 20 drafts had >90% acceptance: increase `num_speculative_tokens` by 1 (up to max 8)
- If acceptance rate drops below 50%: decrease by 1 (down to min 2)
- This prevents wasted computation on low-quality drafts for unusual inputs

**Observed throughput gain: 2–3×**
At 5 speculative tokens and 80% acceptance rate, the target model runs one forward pass per ~4.5 output tokens instead of one per token → ~2.5× throughput improvement. Real-world gain varies with acceptance rate; prompts that produce highly predictable outputs (structured JSON, code templates) see the highest benefit.

---

## 4. vLLM Deployment Configuration

### Tier 2 — Gemma4-26B-A4B (A10G 24GB, privacy tier)

```bash
vllm serve google/gemma-4-26B-A4B-it \
  --speculative-model google/gemma-4-26B-A4B-it-assistant \
  --num-speculative-tokens 5 \
  --speculative-draft-tensor-parallel-size 1 \
  --quantization awq \
  --dtype bfloat16 \
  --max-model-len 32768 \
  --tensor-parallel-size 1 \
  --host 0.0.0.0 \
  --port 8000 \
  --max-num-seqs 40 \
  --enable-prefix-caching \
  --served-model-name gemma4-26b
```

| Flag | Value | Reason |
|---|---|---|
| `--quantization awq` | AWQ | Activation-aware Weight Quantization: better quality than naive Q4 at same VRAM cost |
| `--dtype bfloat16` | bfloat16 | Supported by A10G; better numerical range than float16 |
| `--max-model-len 32768` | 32k tokens | Matches Conductor's max context budget |
| `--tensor-parallel-size 1` | 1 GPU | A10G is a single GPU; no TP needed |
| `--max-num-seqs 40` | 40 concurrent | Tuned for A10G VRAM with 32k context; higher causes OOM |
| `--enable-prefix-caching` | on | Caches shared prefixes (system prompts) across requests → latency reduction |
| `--num-speculative-tokens 5` | 5 | Draft 5 tokens ahead; tuned for code tasks |

### Tier 3 — Gemma4-12B (g5.xlarge, fast tier)

```bash
vllm serve google/gemma-4-12B-it \
  --speculative-model google/gemma-4-12B-it-assistant \
  --num-speculative-tokens 8 \
  --quantization awq \
  --dtype bfloat16 \
  --max-model-len 32768 \
  --tensor-parallel-size 1 \
  --host 0.0.0.0 \
  --port 8001 \
  --max-num-seqs 80 \
  --enable-prefix-caching \
  --served-model-name gemma4-12b
```

The 12B model leaves more VRAM headroom, so `--max-num-seqs` is set to 80 and `--num-speculative-tokens` is raised to 8 for higher throughput on simple outputs.

### Systemd Service (on each g5.xlarge instance)

```ini
[Unit]
Description=vLLM Tier 2 Server (Gemma4-26B)
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu
Environment=HUGGING_FACE_HUB_TOKEN=<secret>
Environment=CUDA_VISIBLE_DEVICES=0
ExecStart=/home/ubuntu/venv/bin/vllm serve google/gemma-4-26B-A4B-it \
  --speculative-model google/gemma-4-26B-A4B-it-assistant \
  --num-speculative-tokens 5 \
  --quantization awq \
  --dtype bfloat16 \
  --max-model-len 32768 \
  --max-num-seqs 40 \
  --enable-prefix-caching \
  --served-model-name gemma4-26b
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

---

## 5. Multi-Region Architecture

```
                  Global Load Balancer
              (AWS ALB with latency-based routing)
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
   │  US-EAST-1  │  │  EU-WEST-1  │  │  AP-SE-1    │
   │  (Virginia) │  │  (Ireland)  │  │  (Singapore)│
   │             │  │             │  │             │
   │  g5.xlarge  │  │  g5.xlarge  │  │  g5.xlarge  │
   │  spot       │  │  spot       │  │  spot       │
   │             │  │             │  │             │
   │  Tier 2:    │  │  Tier 2:    │  │  Tier 2:    │
   │  Gemma4-26B │  │  Gemma4-26B │  │  Gemma4-26B │
   │  :8000      │  │  :8000      │  │  :8000      │
   │             │  │             │  │             │
   │  Tier 3:    │  │  Tier 3:    │  │  Tier 3:    │
   │  Gemma4-12B │  │  Gemma4-12B │  │  Gemma4-12B │
   │  :8001      │  │  :8001      │  │  :8001      │
   │             │  │             │  │             │
   │  ~$250/mo   │  │  ~$250/mo   │  │  ~$280/mo   │
   │  ~60 tok/s  │  │  ~60 tok/s  │  │  ~60 tok/s  │
   └─────────────┘  └─────────────┘  └─────────────┘

   Total GPU spend:  ~$780/mo (3 instances, 24/7)
   Total capacity:   ~180 tok/s Tier 2, ~240 tok/s Tier 3
```

**Load Balancing Policy**
- Health check: `GET /health` on each vLLM instance (returns 200 when model loaded)
- Routing: ALB routes to the healthiest instance in the closest region (latency-based)
- On instance interruption: ALB deregisters and routes to remaining instances within 30s
- Minimum healthy instances: 1 per tier (below this, alert fires; above this, load is shared)

**Authentication**
Each vLLM instance requires a bearer token header (`Authorization: Bearer <internal_token>`). The token is stored in AWS Secrets Manager and injected into the model router at startup. Instances are in a private VPC subnet; only the runtime orchestrator ECS tasks (same VPC) and the ALB can reach them.

---

## 6. Spot Instance Interruption Handling

AWS EC2 spot instances can be interrupted with a 2-minute warning via the instance metadata service. The interruption handling chain:

```
EC2 Spot Interruption Notice
        │
        ▼ (instance metadata: /latest/meta-data/spot/interruption-action)
┌─────────────────────────────────────────────────────────┐
│  interruption-monitor.sh (runs every 30s via cron)      │
│                                                         │
│  1. Poll: curl http://169.254.169.254/.../spot/...      │
│  2. If "terminate" returned:                            │
│     a. Send SIGTERM to vllm process (graceful drain)    │
│     b. Deregister instance from ALB target group         │
│        (aws elbv2 deregister-targets ...)               │
│     c. Log interruption event to CloudWatch             │
│     d. Wait 90s for in-flight requests to drain         │
│     e. SIGKILL if still running after 90s               │
└─────────────────────────────────────────────────────────┘
        │
        ▼
ALB detects unhealthy target → routes new requests to other regions
        │
        ▼
In-flight LLM_CALL in runtime orchestrator:
  → Connection error → RecoverableError
  → Retry with exponential backoff (2s, 4s, 8s)
  → If all retries fail: run persists to DB with ir_index at the failed LLM_CALL
  → ASG replaces instance (new spot or on-demand fallback)
  → Run auto-retried by RQ worker (scheduled retry after 60s on RecoverableError)
        │
        ▼
Run resumes from last committed ir_index (just before the failed LLM_CALL)
Idempotency key ensures the LLM call is not double-billed if the previous
attempt completed but the result was not persisted.
```

**ASG Configuration**
- Min capacity: 1 per tier per region
- Max capacity: 4 per tier per region (manual cap; increase as MAU grows)
- Spot allocation strategy: `capacity-optimized` (chooses pool with most available capacity)
- On-demand fallback: 20% baseline capacity as on-demand to prevent full outage

---

## 7. Metrics to Surface in Dashboard

These metrics are collected from vLLM's built-in Prometheus endpoint (`/metrics`) and forwarded to CloudWatch via a metrics scraper sidecar.

| Metric | Source | Dashboard Use |
|---|---|---|
| `vllm:generation_tokens_total` | vLLM Prometheus | Live tok/s gauge (per instance and aggregate) |
| `vllm:num_requests_running` | vLLM Prometheus | Active concurrent requests |
| `vllm:num_requests_waiting` | vLLM Prometheus | Queue depth (alert if >20 for >2min) |
| `vllm:gpu_cache_usage_perc` | vLLM Prometheus | KV-cache utilization (% of VRAM used for KV-cache) |
| `vllm:spec_decode_draft_acceptance_rate` | vLLM Prometheus | MTP acceptance rate (alert if <40%) |
| GPU utilization (%) | nvidia-smi → CloudWatch agent | Overall GPU utilization |
| GPU memory (GB used / total) | nvidia-smi → CloudWatch agent | VRAM headroom |
| Time-to-first-token p50/p99 | Custom instrumentation in model router | Latency for streaming starts |
| Cost per 1M tokens (amortized) | Computed: spot_cost / tokens_generated | Economics dashboard |

---

## 8. Cost Model

**Per-Instance Economics**

| Item | Value |
|---|---|
| g5.xlarge spot price (us-east-1 avg) | $0.35/hr |
| Monthly cost (24/7) | $0.35 × 24 × 30 = **$252/mo** |
| Tier 2 throughput (26B, MTP enabled) | ~60 tok/s |
| Tokens per hour | 60 × 3600 = 216,000 tok/hr |
| Tokens per month (24/7) | 216,000 × 720 = **155.5M tok/mo** |
| Cost per 1M tokens | $252 / 155.5 = **$1.62/1M tokens** |

At ~80% GPU utilization (realistic with queue), effective throughput is ~48 tok/s:
- Monthly tokens: 48 × 3600 × 720 = 124.4M
- **Effective cost: $2.03/1M tokens**

**Comparison to Bedrock**

| Provider | Model | Input Cost/1M | Output Cost/1M |
|---|---|---|---|
| AWS Bedrock | Claude 3.5 Sonnet | ~$3.00 | ~$15.00 |
| AWS Bedrock | Llama 3 70B | ~$0.99 | ~$0.99 |
| Conductor GPU | Gemma4-26B (Tier 2) | ~$2.03 | ~$2.03 |
| Conductor GPU | Gemma4-12B (Tier 3) | ~$1.27 | ~$1.27 |

**Tier 2 vs Claude 3.5 Sonnet:** ~1.5× cheaper on input, ~7× cheaper on output.
**Tier 3 vs Bedrock Llama 3 70B:** ~0.8× on input (similar), but Tier 3 serves free users who wouldn't otherwise have Bedrock access.

**Three-Region Total**
- US-EAST-1: $252/mo
- EU-WEST-1: $252/mo
- AP-SE-1: ~$280/mo (slightly higher spot price)
- **Total: ~$784/mo for 3-region Tier 2 coverage**
- At launch, Tier 3 runs on the same instances (separate port/process); no additional hardware needed until GPU utilization exceeds ~70% consistently.
