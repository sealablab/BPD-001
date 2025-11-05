# Template Quick-Start Guide

**How to use this template for your own project**

---

## Overview

This template provides a composable monorepo architecture with:
- ✅ Git submodules for clean component separation
- ✅ 3-tier documentation (AI-optimized for token efficiency)
- ✅ Example tools and libraries you can keep, remove, or replace
- ✅ AI-assisted customization support

**Time to customize:** 15-30 minutes

---

## Step 1: Create Your Repository

### Option A: Use GitHub Template (Recommended)

1. Go to: https://github.com/sealablab/moku-instrument-forge-mono-repo
2. Click **"Use this template"** button (top right)
3. Name your repository (e.g., `my-instrument-dev-kit`)
4. Choose public or private
5. Click **"Create repository"**

**Benefits:**
- Fresh git history (no template commits)
- Clean slate for your project
- Already set up as template on GitHub

### Option B: Fork

1. Fork the repository on GitHub
2. Rename if desired

**Benefits:**
- Can pull updates from original template
- Maintains connection to upstream

---

## Step 2: Clone with Submodules

```bash
# Clone your new repository
git clone --recurse-submodules <your-repo-url>
cd your-repo

# Verify submodules initialized
git submodule status --recursive
```

**Expected output:**
```
 <hash> libs/forge-vhdl (tag)
 <hash> libs/moku-models (tag)
 <hash> libs/riscure-models (tag)
 <hash> tools/forge-codegen (tag)
```

---

## Step 3: Understand What You're Customizing

**IMPORTANT:** This template is for **Moku platform + probe integration**. You're NOT choosing a platform—you're adding YOUR probes to Moku.

### Core Components (Always Keep)

✅ **`libs/moku-models/`** - **REQUIRED** - Moku platform specifications (Go/Lab/Pro/Delta)
  - This is the foundation - DON'T remove this!

✅ **`tools/forge-codegen/`** - YAML → VHDL code generator
  - Keep unless doing Python-only work (rare)

✅ **`libs/forge-vhdl/`** - VHDL component library
  - Keep unless doing Python-only work (rare)

### Probe Models (Customize Here)

This is the MAIN customization step:

**Option A: Using Riscure EMFI probes**
  - ✅ Keep `libs/riscure-models/` - Already configured!
  - No changes needed

**Option B: Using different probes (laser, RF, custom) - MOST COMMON**
  - ⚠️ Keep `libs/riscure-models/` as reference (recommended)
    - Use it as template for YOUR probe models
    - Shows voltage safety validation patterns
    - Example of 3-tier documentation
  - ➕ Add `libs/YOUR-probe-models/` - Create your own probe specs
    - Pydantic models for your probe hardware
    - Voltage compatibility validation
    - Following same documentation pattern

**Option C: Multiple probe types**
  - ✅ Keep `libs/riscure-models/` (if using Riscure)
  - ➕ Add `libs/laser-models/` (if using laser)
  - ➕ Add `libs/rf-models/` (if using RF)
  - Multiple probe models coexist happily!

### Rare Cases

**Python-only (no VHDL/FPGA):**
  - Keep: `libs/moku-models/` + your probe models
  - Remove: `tools/forge-codegen/`, `libs/forge-vhdl/`
  - ⚠️ Unusual for Moku development (most work involves FPGA)

**Extending Moku platform models:**
  - Fork `libs/moku-models/` to add custom platforms
  - Update `.gitmodules` to point to your fork

### ⛔ Not Using Moku?

This template is **Moku-specific**. If you want to use Red Pitaya, PicoScope, or other platforms, you'll need to:
1. Fork this repository
2. Replace ALL Moku-specific code
3. Essentially create a new template

**Recommendation:** Create a platform-specific template from scratch instead.

---

## Step 4: Remove Unnecessary Submodules

For each submodule you don't need:

```bash
# 1. Remove the submodule
git rm libs/unwanted-module/

# 2. Update workspace members in pyproject.toml
# Edit pyproject.toml and remove from [tool.uv.workspace] members list

# 3. Update workspace
uv sync

# 4. Commit the removal
git commit -m "chore: Remove unwanted-module (not needed for our use case)"
```

**Example: Removing riscure-models if you're not using Riscure probes**
```bash
# Remove git submodule
git rm libs/riscure-models/

# Edit pyproject.toml - remove "libs/riscure-models" from workspace members
# Then sync workspace
uv sync

# Commit
git commit -m "chore: Remove riscure-models (not using Riscure probes)"
```

**Don't forget to update documentation:**
- Remove from `[tool.uv.workspace] members` in `pyproject.toml` ⚠️ **CRITICAL**
- Remove entry from `llms.txt`
- Remove section from `CLAUDE.md`
- Remove entry from `.claude/manifest.json`

---

## Step 5: Add Your Own Submodules

For each new component you need:

### 5a. Create the Submodule Repository

```bash
# Create new repo on GitHub
# Example: my-platform-models

# Clone it locally
git clone https://github.com/yourusername/my-platform-models.git
cd my-platform-models

# Create 3-tier documentation structure
touch llms.txt CLAUDE.md README.md
mkdir -p your_module_name/

# Write your code and docs
# Follow the pattern from existing submodules
```

**3-Tier Documentation Pattern:**

**llms.txt** (~500-1000 tokens):
```markdown
# your-module-name

> One-line description

## What is this?
[Brief explanation]

## Core Exports
- Main classes/functions
- Key types

## Basic Usage
[Quick example]
```

**CLAUDE.md** (~3-5k tokens):
```markdown
# CLAUDE.md

## Project Overview
[Complete description, design rationale]

## Quick Start
[Installation, setup]

## Architecture
[Design decisions]

## Integration
[How to use with other libraries]
```

### 5b. Add as Submodule to Your Monorepo

```bash
# In your monorepo
git submodule add https://github.com/yourusername/my-platform-models.git libs/my-platform-models/
git submodule update --init libs/my-platform-models/

# Add to workspace members in pyproject.toml
# Edit [tool.uv.workspace] members = [...] to include "libs/my-platform-models"

# Update workspace
uv sync
```

### 5c. Update Monorepo Documentation

**pyproject.toml** - Add to workspace members:
```toml
[tool.uv.workspace]
members = [
    "libs/forge-vhdl",
    "libs/moku-models",
    "libs/my-platform-models",  # Add your new module
    "tools/forge-codegen",
]
```

**llms.txt** - Add entry:
```markdown
| [my-platform-models](libs/my-platform-models/) | Platform specs for XYZ | [llms.txt](libs/my-platform-models/llms.txt) |
```

**CLAUDE.md** - Add section:
```markdown
**[my-platform-models](libs/my-platform-models/)**
- **Purpose:** Platform specifications for XYZ platform
- **Authority:** Platform specs, timing, voltage ranges
- **Quick Ref:** [libs/my-platform-models/llms.txt](libs/my-platform-models/llms.txt)
```

**.claude/manifest.json** - Add entry:
```json
{
  "name": "my-platform-models",
  "path": "libs/my-platform-models",
  "type": "library",
  "required": true,
  "description": "Platform specs for XYZ",
  "documentation": {
    "llms_txt": "libs/my-platform-models/llms.txt",
    "claude_md": "libs/my-platform-models/CLAUDE.md"
  }
}
```

### 5d. Commit

```bash
git add .gitmodules libs/my-platform-models/ llms.txt CLAUDE.md .claude/manifest.json
git commit -m "feat: Add my-platform-models submodule"
```

---

## Step 6: Update Repository Metadata

### 6a. Update README.md

**Change the title and description:**
```markdown
# my-instrument-dev-kit

**Custom Instrument Development Environment for XYZ Platform**

This monorepo contains tools and libraries for developing custom instruments on the XYZ platform.
```

**Update the submodule list** to reflect your components.

### 6b. Update CLAUDE.md

**Update "Current Architecture" section** to list your actual submodules.

**Update integration examples** to match your use case.

### 6c. Update llms.txt

**Update the component catalog** to reflect your actual structure.

### 6d. Update .claude/manifest.json

**Update metadata:**
```json
{
  "name": "my-instrument-dev-kit",
  "description": "Your description here",
  "repository": "https://github.com/yourusername/my-instrument-dev-kit"
}
```

---

## Step 7: Customize AI Commands (Optional)

### Remove Template Commands

If you don't need the template customization command:
```bash
git rm .claude/commands/customize-monorepo.md
git commit -m "chore: Remove template customization command"
```

### Add Your Own Commands

Create `.claude/commands/your-command.md`:
```markdown
# Your Custom Command

Description of what it does

## Usage
/your-command
```

---

## Step 8: Set Up Python Environment

This monorepo uses **uv workspace mode**, which means:
- Root `pyproject.toml` is a pure workspace container (no package at root)
- Each submodule is a workspace member with its own `[build-system]`
- Shared dependencies defined at root level
- No dummy package code required

```bash
# Install dependencies (workspace mode - no build at root)
uv sync

# Run tests across all workspace members
pytest

# Verify imports work
python scripts/setup_forge_path.py  # If using forge-codegen
```

### Understanding Workspace Mode

**What happens when you run `uv sync`:**
1. uv reads root `pyproject.toml`
2. Finds workspace members in `[tool.uv.workspace]`
3. Creates unified dependency graph
4. Installs all dependencies in single `.venv/`
5. Makes all workspace members importable

**When you add/remove submodules:**
- Update `[tool.uv.workspace] members = [...]` in root `pyproject.toml`
- Run `uv sync` to update workspace

**Example: After removing riscure-models:**
```toml
[tool.uv.workspace]
members = [
    "libs/forge-vhdl",
    "libs/moku-models",
    # "libs/riscure-models",  # Removed
    "tools/forge-codegen",
]
```

Then run: `uv sync`

---

## Step 9: Test Everything

```bash
# 1. Verify submodules
git submodule status --recursive

# 2. Check git status is clean
git status

# 3. Test AI navigation
# Open in Claude Code and try:
# - /customize-monorepo (should work)
# - Check that AI can read llms.txt files
# - Verify 3-tier documentation navigation works

# 4. Run any tests
pytest
```

---

## Step 10: First Commit

```bash
git add -A
git commit -m "feat: Initial customization from template

- Removed: [list removed submodules]
- Added: [list new submodules]
- Updated: Documentation to reflect our use case"

git push
```

---

## Common Customization Scenarios

### Scenario 1: Moku + Riscure EMFI (Template As-Is)

**Goal:** Using Moku with Riscure DS1120A/DS1140A probes

**Actions:**
- ✅ **No customization needed!**
- Clone, install, start developing

**Keep everything:**
- `libs/moku-models/` ✅
- `libs/riscure-models/` ✅
- `tools/forge-codegen/` ✅
- `libs/forge-vhdl/` ✅

---

### Scenario 2: Moku + Custom Laser Probes (MOST COMMON)

**Goal:** Using Moku with custom laser probes

**Keep (Core):**
- `libs/moku-models/` ✅ (REQUIRED)
- `tools/forge-codegen/` ✅
- `libs/forge-vhdl/` ✅

**Keep (Reference - Recommended):**
- `libs/riscure-models/` ⚠️ (use as template)

**Add:**
- `libs/laser-models/` ➕ (your custom probe specs)

**Steps:**
1. Create `laser-models` repository with Pydantic models
2. Add as submodule: `git submodule add <url> libs/laser-models/`
3. Update `pyproject.toml` workspace members
4. Run `uv sync`
5. Update documentation (llms.txt, CLAUDE.md)

---

### Scenario 3: Moku + Multiple Probe Types

**Goal:** Supporting Riscure + laser + RF probes simultaneously

**Keep:**
- `libs/moku-models/` ✅ (REQUIRED)
- `libs/riscure-models/` ✅
- `tools/forge-codegen/` ✅
- `libs/forge-vhdl/` ✅

**Add:**
- `libs/laser-models/` ➕
- `libs/rf-analyzer-models/` ➕

All probe models coexist in `libs/` as independent submodules.

---

### Scenario 4: Python-Only (No VHDL) - RARE

**Goal:** Only need Python models, no FPGA development

**Keep:**
- `libs/moku-models/` ✅ (REQUIRED)
- Your probe models ✅

**Remove:**
- `tools/forge-codegen/` ❌
- `libs/forge-vhdl/` ❌

**Note:** Unusual for Moku custom instrument work (most involves FPGA)

---

### Scenario 5: Fork & Extend Moku Models

**Goal:** Add custom/unreleased Moku platforms

**When:** You want to customize moku-models (e.g., add custom platforms)

**Steps:**
1. Fork the submodule on GitHub
2. Update `.gitmodules` URL to your fork
3. Run `git submodule sync`
4. Make your changes in the fork
5. Update submodule reference in parent

**See:** [CLAUDE.md - Forking a Submodule](CLAUDE.md#forking-a-submodule) for detailed instructions

---

## After Customization

### Share Your Work

If your customization creates a useful pattern:
- Make your repo public
- Add a good README
- Share with the community
- Consider contributing patterns back to the original template

### Keep Template Benefits

Even after customization, maintain:
- ✅ 3-tier documentation pattern
- ✅ Self-contained submodules
- ✅ Islands of authority (no duplication)
- ✅ AI-navigable structure

---

## Getting Help

### AI Assistance

```bash
# In Claude Code
/customize-monorepo
```

The AI will guide you through the customization process interactively.

### Documentation

- **CLAUDE.md** - Complete architecture guide with composability patterns
- **.claude/manifest.json** - Programmatic structure definition
- **Individual submodule docs** - Each has llms.txt → CLAUDE.md → source

### Community

- **Original Template Issues:** https://github.com/sealablab/moku-instrument-forge-mono-repo/issues
- **Original Template Discussions:** https://github.com/sealablab/moku-instrument-forge-mono-repo/discussions

---

## Troubleshooting

### "Submodules are empty after cloning"

```bash
git submodule update --init --recursive
```

### "I removed a submodule but it's still showing"

```bash
git rm --cached libs/module-name/
rm -rf .git/modules/libs/module-name/
```

### "Submodule is in detached HEAD state"

```bash
cd libs/module-name/
git checkout main  # or appropriate branch
cd ../..
```

### "Can't push submodule changes"

You need to push submodule first, then parent:
```bash
cd libs/your-module/
git push origin main
cd ../..
git add libs/your-module/
git commit -m "Update submodule reference"
git push
```

---

## Next Steps

After customization:

1. **Set up CI/CD** for your specific needs
2. **Create project board** for your work
3. **Invite collaborators** if working with a team
4. **Start building!** The template is now yours

---

**Remember:** This template is a starting point. Adapt it freely to your needs. The architecture pattern (composable submodules + 3-tier docs) is more important than the specific components.

**Happy building!** 🚀
