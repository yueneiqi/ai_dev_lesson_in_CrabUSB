# CrabUSB Prompt Patterns Summary

This document summarizes the prompt patterns extracted from 60+ development sessions (Codex, Claude Code, OpenCode) for the CrabUSB USB driver project.

---

## Overview

| Category | Frequency | Description |
|----------|-----------|-------------|
| **Task/Goal** | ~25% | Goal-oriented task initiation |
| **Debug/Log** | ~30% | Hardware log analysis and debugging |
| **Git Ops** | ~15% | Commit, organize, PR generation |
| **Refactor** | ~10% | Code cleanup and quality |
| **Investigation** | ~10% | Code explanation and analysis |
| **Fix** | ~10% | Error fixing and troubleshooting |

---

## 1. Task Initiation Prompts

### Goal-Oriented Tasks

**Pattern**: Define clear goal with success criteria

```
Goal:
even without `usb start` command in uboot, port1 can be enabled and connected like in qemu:
```
💡 23.974s    [crab_usb::backend::xhci:250] Port 1: Enabled: true, Connected: true, Speed 4, Power true
```

- Enable port1 (USB3_1) which connects to GL3523 hub
- Ignore port0, since it needs more work to support FUSB302 Type-C controller and I2C driver.
```

### Reference-Based Tasks

**Pattern**: Point to external file containing task definition

```
do task in @status.md
```

```
finish task and achieve the goal in @status.md
```

### Migration/Porting Tasks

**Pattern**: Specify source code to migrate with file reference

```
now try to migrate uboot 中 usb start 有关 xhci 的逻辑
XHCI DWC3 Driver (u-boot/drivers/usb/host/xhci-dwc3.c:158)
```

---

## 2. Investigation & Analysis Prompts

### Code Explanation

**Pattern**: Ask about specific function/behavior

```
explain in uboot what does xhci_dwc3_reset_init(dev, plat) do?
```

```
how does cargo test -p crab-usb --test test --target aarch64-unknown-none-softfloat -- --show-output work?
```

### Specification Writing

**Pattern**: Read multiple sources, synthesize into spec

```
read usb driver related files in u-boot/ and linux-orangepi and rk3588_doc_md, write a usb driver spec for orangepi5plus
```

### Comparison Analysis

**Pattern**: Compare behavior across environments (Chinese)

```
在CrabUSB中，在qemu上测试通过；在搭载rk3588的orangepi5plus上测试失败
```

Translation: CrabUSB passes on QEMU but fails on Orange Pi 5 Plus with RK3588

---

## 3. Debugging Prompts

### Log Analysis

**Pattern**: Direct log file reference

```
read log/20251205_202722_crabusb.log
```

```
it works, read 20251205_210252_crabusb.log
```

### Error Investigation

**Pattern**: Include full error message

```
find out why when run cargo test -p crab-usb --test test --target aarch64-unknown-none-softfloat -- --show-output, it output: kernel panic: no xhci found
```

```
result of script/run_crabusb_uboot.sh: kernel panic
```

### Dependency Analysis

**Pattern**: Describe working vs non-working conditions (Chinese)

```
目前，在uboot启动命令中加入usb start后，CrabUSB的测试可顺利连接USB
```

Translation: Currently, USB connects successfully when adding `usb start` to U-Boot boot command

---

## 4. Code Quality Prompts

### Refactoring with Verification

**Pattern**: Specify commit range + quality goal + verification steps

```
review code between 786d04cf6967d03d49f8ebbaaf5f6eec8c69ea8f and 826fff0248b34de270f180c0226c1e297caf320d, make it production-ready, remember do incremental refactor, break big refactor into small steps, and verify it before do next refactor. Validation steps:
- Build both configurations:
  - cargo check -p crab-usb
  - cargo check -p crab-usb --no-default-features
- Then run the hardware flows (per AGENTS.md):
  - bash $SCRIPT_PATH/run_crabusb_uboot.sh | tee $LOG_PATH/<ts>_crabusb.log
```

### Dead Code Cleanup

**Pattern**: Review recent commits, remove unused code

```
review latest three commits, remove dead code
```

```
review latest three commits, remove dead code, only keep dead code that really needed
```

### Test Cleanup

**Pattern**: Make test code production-ready

```
clean up CrabUSB/usb-host/tests/test.rs, make it production-ready
```

---

## 5. Git Operations Prompts

### Simple Commits

**Pattern**: Minimal instruction

```
git add and commit
```

### Commit Organization

**Pattern**: Specify range, request consolidation (Chinese)

```
从 786d04cf 到 a7ccd1c2, commits 太多了，整理commits
```

Translation: Too many commits from 786d04cf to a7ccd1c2, organize commits

### PR Generation

**Pattern**: Generate PR metadata from commits

```
based on the latest 4 commits, give me a pr title in chinese
```

---

## 6. Build System Prompts

### Script Creation

**Pattern**: Reference existing implementation as template

```
based on the making uimg logic in arceos, add a shell script in CrabUSB/ to make a uimg
```

### Fix Scripts

**Pattern**: Include full error message

```
write a shell script to fix OSError: [Errno 16] Device or resource busy: '/dev/ttyUSB0'
```

---

## 7. OpenSpec/Change Management Prompts

**Pattern**: Apply predefined change specification

```
apply the OpenSpec change of openspec/changes/remove-uboot-usb-start-dependency to CrabUSB
```

---

## 8. Mode-Specific Prompts (with prefixes)

### Search Mode

**Pattern**: Prefix with `[search-mode]` for exhaustive search

```
[search-mode]
MAXIMIZE SEARCH EFFORT. Launch multiple background agents IN PARALLEL:
- explore agents (codebase patterns, file structures, ast-grep)
- librarian agents (remote repos, official docs, GitHub examples)
Plus direct tools: Grep, ripgrep (rg), ast-grep (sg)
NEVER stop at first result - be exhaustive.

<actual task here>
```

### Analyze Mode

**Pattern**: Prefix with `[analyze-mode]` for deep analysis

```
[analyze-mode]
ANALYSIS MODE. Gather context before diving deep:

CONTEXT GATHERING (parallel):
- 1-2 explore agents (codebase patterns, implementations)
- 1-2 librarian agents (if external library involved)
- Direct tools: Grep, AST-grep, LSP for targeted searches

IF COMPLEX (architecture, multi-system, debugging after 2+ failures):
- Consult oracle for strategic guidance

SYNTHESIZE findings before proceeding.

<actual task here>
```

### Ralph Loop (Autonomous Execution)

**Pattern**: Self-referential loop until completion

```
<command-instruction>
You are starting a Ralph Loop - a self-referential development loop that runs until task completion.

## How Ralph Loop Works

1. You will work on the task continuously
2. When you believe the task is FULLY complete, output: `<promise>{{COMPLETION_PROMISE}}</promise>`
3. If you don't output the promise, the loop will automatically inject another prompt to continue
4. Maximum iterations: Configurable (default 100)

## Rules

- Focus on completing the task fully, not partially
- Don't output the completion promise until the task is truly done
- Each iteration should make meaningful progress toward the goal
- If stuck, try different approaches
- Use todos to track your progress

## Exit Conditions

1. **Completion**: Output `<promise>DONE</promise>` (or custom promise text) when fully complete
2. **Max Iterations**: Loop stops automatically at limit
3. **Cancel**: User runs `/cancel-ralph` command
</command-instruction>

<user-task>
finish task and achieve the goal in @status.md 
</user-task>
```

---

## Typical Prompt Examples by Session Type

### Hardware Debugging Session

```
# Start
read log/20251206_103358_crabusb.log

# After analysis
Ports disconnected without `usb start`, analyzing log/20251206_112524_crabusb.log

# After fix attempt
it works, read 20251205_210252_crabusb.log

# Wrap up
git add and commit
```

### Code Migration Session

```
# Start
do task in @status.md

# During work (implicit - agent works autonomously)

# Wrap up
git add and commit
```

### Refactoring Session

```
# Start
review code between <commit1> and <commit2>, make it production-ready

# Cleanup
review latest three commits, remove dead code

# Wrap up
git add and commit
```

---

## Key Observations

1. **Bilingual**: Mix of Chinese and English prompts based on context
2. **Reference-heavy**: Frequent use of `@file.md` file references
3. **Log-driven debugging**: Hardware logs are primary debugging input
4. **Incremental**: Tasks broken into small, verifiable steps
5. **Mode prefixes**: `[search-mode]`, `[analyze-mode]` control agent behavior
6. **Commit discipline**: Frequent `git add and commit` after changes
7. **Goal clarity**: Success criteria often include expected log output

---

## Anti-Patterns (What NOT to do)

| Anti-Pattern | Better Alternative |
|--------------|-------------------|
| `fix the bug` | `fix: Port 1 shows CCS=0, expected CCS=1 after PHY init` |
| `make it work` | `Goal: Port 1 Enabled: true, Connected: true, Speed 4` |
| `clean up the code` | `review commits X-Y, remove dead code, verify with cargo check` |
| `debug this` | `read log/20251206_*.log, find why PLS=5 instead of PLS=0` |

---

## Data Sources

- **Codex**: 30+ sessions (Nov 21 - Dec 23, 2025)
- **Claude Code**: 17 sessions (Dec 22, 2025 - Jan 9, 2026)
- **OpenCode**: 15+ sessions (Jan 8-13, 2026)

Total: 60+ sessions, 5000+ messages analyzed
