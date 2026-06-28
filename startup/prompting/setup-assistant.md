---
type: Setup Prompt Pattern
title: Setup assistant — install and verify failures
description: Prompt templates for getting LLM help when a tool install or verify step fails.
tags: [prompting, install, verify, setup, troubleshoot]
timestamp: 2026-06-27T00:00:00Z
---

# Setup assistant

Use these templates when a `--version` check fails, an installer
exits with an error, or `make` can't find a tool.

---

## Template: tool not found

```
I am setting up the J0 compiler course on <PLATFORM>.
I installed <TOOL> using <COMMAND> but running `<TOOL> --version` gives:
  <PASTE EXACT ERROR>

My PATH is: <paste: echo $PATH>
My shell is: <paste: echo $SHELL>

What is the most likely cause, and what is the shortest fix?
```

**Example:**
```
I am setting up the J0 compiler course on Ubuntu 22.04.
I installed JFlex using `sudo apt-get install jflex` but running
`jflex --version` gives:
  -bash: jflex: command not found

My PATH is: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
My shell is: /bin/bash

What is the most likely cause, and what is the shortest fix?
```

---

## Template: wrong version

```
I am setting up the J0 compiler course on <PLATFORM>.
`<TOOL> --version` reports <ACTUAL VERSION> but the course requires <REQUIRED VERSION>.

Is <ACTUAL VERSION> compatible? If not, what is the recommended upgrade path?
```

---

## Template: Java version mismatch

```
`java -version` reports:  <PASTE>
`javac -version` reports: <PASTE>

The major versions do not match. How do I make them use the same JDK?
Platform: <PLATFORM>.
```

---

## Template: PATH confusion (macOS + Homebrew)

```
I installed <TOOL> with Homebrew on macOS.
`brew info <TOOL>` says: <PASTE>
But `<TOOL>` is not found in a new terminal.
My shell: <zsh|bash>. Config file I loaded: <~/.zshrc|~/.bashrc>.
How do I make Homebrew's bin directory visible in new terminals?
```
