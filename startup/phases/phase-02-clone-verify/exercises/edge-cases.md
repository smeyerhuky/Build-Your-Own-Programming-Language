---
type: Edge Case Catalog
title: "Phase 2 — Edge cases (negative examples)"
description: Common clone and verify failures with CoT traces and fixes.
tags: [edge-cases, clone, verify, negative-examples, debugging]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Phase 2: Clone & verify

## 1. JFlex reports encoding error on `javalex.l`

### Symptom
```
$ jflex javalex.l
Error in file "javalex.l" (line 1):
  Illegal character: <FEFF>
```

### CoT trace
1. JFlex sees a BOM (byte-order mark, `0xFEFF`) at the start of the
   file.
2. The repo was cloned on Windows with `core.autocrlf=true`, and a
   text editor saved `javalex.l` with a UTF-8 BOM.
3. JFlex expects plain ASCII / UTF-8 without BOM.

### Fix
```bash
# Remove BOM (Linux/macOS):
sed -i '1s/^\xEF\xBB\xBF//' javalex.l

# Or reconfigure Git on Windows:
git config --global core.autocrlf input
git checkout -- ch3/javalex.l    # re-checkout clean version
```

### Lesson
`.l` and `.y` files are sensitive to invisible characters. Always
verify the file's encoding after any editor touch on Windows.

---

## 2. BYacc produces `y.tab.c` instead of `parser.java`

### Symptom
```
$ byacc -J j0gram.y
$ ls
y.tab.c   # C output, not parser.java
```

### CoT trace
1. `byacc -J` should produce `parser.java`, not `y.tab.c`.
2. Some BYacc distributions use `-j` (lowercase) instead of `-J`.
3. Or the `-J` flag was given but a very old build of BYacc silently
   ignored it and fell back to C output.

### Fix
Try lowercase `-j`:
```bash
byacc -j j0gram.y
ls *.java    # should now include parser.java
```
Or upgrade BYacc to 20210808 or later.

### Lesson
Flag case sensitivity varies across BYacc builds. Check `byacc -h`
for the exact flag syntax in your version.

---

## 3. `make` reports "No rule to make target 'all'"

### Symptom
```
$ make
make: *** No rule to make target 'all'.  Stop.
```

### CoT trace
1. `make` cannot find a `makefile` or `Makefile` in the current
   directory.
2. The current directory is the repo root, not a chapter directory.

### Fix
```bash
cd ch3
make
```

### Lesson
Each chapter has its own `makefile`. Always `cd` into a chapter
directory before running `make`.

---

## 4. `git clone` produces a shallow clone warning

### Symptom
```
$ git clone --depth=1 https://github.com/...
warning: You are in 'detached HEAD' state.
```

### CoT trace
1. A shallow clone (`--depth=1`) omits history. All source files are
   present, but `git log` shows only one commit.
2. This is harmless for building the compiler, but some scripts and
   editors display misleading state.

### Fix
Clone without `--depth`:
```bash
git clone https://github.com/smeyerhuky/Build-Your-Own-Programming-Language.git
```
Or unshallow:
```bash
git fetch --unshallow
```

### Lesson
For a course repo you intend to modify and study, a full clone is
preferred over a shallow one.
