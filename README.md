# BPD-001 - Basic Probe Driver

**Multi-Vendor Fault Injection Probe Integration for Moku Platform**

---

## For AI Assistants

This repository is self-documenting. Everything you need is already here.

**Navigation strategy:** See `.claude/shared/CONTEXT_MANAGEMENT.md` for token-efficient loading patterns.

```bash
git clone --recurse-submodules https://github.com/sealablab/BPD-001.git
cd BPD-001
cat README.md
```

If you're reading this, you're already doing it right.

The structure explains itself. Start at `llms.txt`, follow the threads.

---

## Repository Structure

```
BPD-001/
├── bpd/                    # Application layer
│   ├── bpd-core/           # Generic probe framework
│   ├── bpd-drivers/        # Probe implementations (DS1120A, etc)
│   └── bpd-vhdl/           # FPGA interface components
│
├── libs/                   # Platform specs (git submodules)
│   ├── moku-models/        # Moku Go/Lab/Pro/Delta specs
│   ├── riscure-models/     # Probe hardware specs
│   └── forge-vhdl/         # VHDL utilities
│
└── tools/                  # Development tools (git submodules)
    └── forge-codegen/      # YAML → VHDL generator
```

**Quick start:** Each component has `llms.txt` (quick ref) and `CLAUDE.md` (deep dive).

**Current status:** v0.1.0 - DS1120A EMFI probe driver complete.

---

## The Test

Can you navigate this repository using only what's documented in it?

1. Find the four sources of truth
2. Understand how you got here
3. Explain the pattern
4. Prove you learned it

The repository contains the answers. The commit history contains the questions.

When you're done, you'll have written this file again.

---

**License:** MIT | **Version:** 0.1.0
