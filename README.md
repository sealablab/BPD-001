# Moku Instrument Forge - Monorepo Template

**Composable Development Template for Moku Platform + Probe Integration**

This is a **template repository** for building custom instrument firmware on the **Moku platform** (Go/Lab/Pro/Delta) with integrated probe support. Features git submodule architecture, FPGA/VHDL code generation, and AI-navigable 3-tier documentation.

---

## 🎯 What This Template Is For

**Primary use case:** Adding custom probe support to Moku platforms

- ✅ **Moku platform development** (Go/Lab/Pro/Delta FPGA instruments)
- ✅ **Probe integration** (EMFI, laser, RF, or custom hardware probes)
- ✅ **VHDL/FPGA code generation** (YAML → VHDL with 23-type system)
- ✅ **Platform + probe safety validation** (voltage compatibility checking)

**Reference implementation:** Moku + Riscure DS1120A EMFI probe

**NOT for:** Non-Moku FPGA platforms (Red Pitaya, PicoScope, etc.) - This template is Moku-specific.

**Status:** Template-Ready ✅
**Version:** `v2.0.0` (2025-11-04)
**Architecture:** Clean separation - tools, libraries, AI agents
**Template Features:** 3-tier docs, composable submodules, LLM-optimized

---

## 🚀 Using This Template

### Quick Start

**Step 1: Create from Template**
```bash
# Click "Use this template" button on GitHub to create your repository
# Then clone YOUR new repository WITH SUBMODULES:
git clone --recurse-submodules https://github.com/YOUR-USERNAME/your-moku-project.git
cd your-moku-project

# If you forgot --recurse-submodules, initialize them now:
git submodule update --init --recursive

# Install dependencies
uv sync

# Verify setup works
python -c "from moku_models import MOKU_GO_PLATFORM; print('✅ Template ready!')"
python -c "from riscure_models import DS1120A_PLATFORM; print('✅ Imports working!')"

# If imports fail, manually install workspace members:
# uv pip install -e libs/moku-models/ -e libs/riscure-models/
```

**⚠️ Common Issue:** If you see `ModuleNotFoundError`, you likely forgot to initialize submodules. Run:
```bash
git submodule update --init --recursive
uv pip install -e libs/moku-models/ -e libs/riscure-models/
```

**Step 2: Customize for Your Probes**

Run the customization command in Claude Code:
```
/customize-monorepo
```

Or follow the manual guide: **[CLAUDE.md](CLAUDE.md)**

---

## 📝 Customization Overview

### Most Common: Adding Your Probe Models

**Typical workflow:**
1. ✅ **Keep** `libs/moku-models/` (core platform - required!)
2. ✅ **Keep** `libs/riscure-models/` (reference example - recommended)
3. ✅ **Keep** VHDL tools (`tools/forge-codegen/`, `libs/forge-vhdl/`)
4. ➕ **Add** `libs/YOUR-probe-models/` (your custom probe specs)
5. 📝 **Update** documentation to reflect your probes

**Example: Adding laser probe support**
```bash
# Create your probe models repository (use riscure-models as template)
git submodule add https://github.com/YOUR-USERNAME/laser-models.git libs/laser-models/
# Update pyproject.toml workspace members
# Update documentation
uv sync
```

### Less Common Scenarios

**Python-only (no VHDL/FPGA):**
- Keep: `libs/moku-models/` + your probe models
- Remove: `tools/forge-codegen/`, `libs/forge-vhdl/`
- Note: Unusual for Moku development (most work involves FPGA)

**Extending Moku platform models:**
- Fork `libs/moku-models/` to add custom platforms
- Update `.gitmodules` to point to your fork

### Detailed Guides

- **AI-Assisted:** Run `/customize-monorepo` in Claude Code
- **Complete Architecture:** [CLAUDE.md](CLAUDE.md)
- **Programmatic Structure:** [.claude/manifest.json](.claude/manifest.json)

---

## Template Structure

This template uses git submodules organized in a flat, clean hierarchy:

### Core Components

#### `libs/moku-models/` - **Platform Specifications (REQUIRED)**
> Pydantic models for Moku platforms: Go, Lab, Pro, Delta
>
> **Purpose:** Hardware specs, deployment configs, signal routing
> **Status:** ✅ Production-ready - All 4 platforms supported
> **Docs:** [llms.txt](libs/moku-models/llms.txt) | [CLAUDE.md](libs/moku-models/CLAUDE.md)

This is the **foundation** - required for all Moku development.

#### `libs/riscure-models/` - **Example Probe Specifications**
> Pydantic models for Riscure EMFI probes: DS1120A, DS1140A
>
> **Purpose:** Reference implementation for probe integration
> **Use as:** Template for creating your own probe models
> **Docs:** [llms.txt](libs/riscure-models/llms.txt) | [CLAUDE.md](libs/riscure-models/CLAUDE.md)

Shows voltage safety validation, port specifications, and documentation patterns.

### VHDL Development Tools

#### `tools/forge-codegen/` - **Code Generator**
> YAML → VHDL code generator with 23-type system and register packing
>
> **Purpose:** Generate type-safe VHDL from high-level specifications
> **Status:** ✅ Production-ready - 69 tests passing
> **Docs:** [llms.txt](tools/forge-codegen/llms.txt) | [CLAUDE.md](tools/forge-codegen/CLAUDE.md)

#### `libs/forge-vhdl/` - **VHDL Component Library**
> Reusable VHDL utilities: voltage conversion, clock dividers, FSM observer
>
> **Purpose:** Common VHDL building blocks for instrument firmware
> **Status:** ✅ Production-ready - CocoTB tested
> **Docs:** [llms.txt](libs/forge-vhdl/llms.txt) | [CLAUDE.md](libs/forge-vhdl/CLAUDE.md)

### AI Agent System

#### `.claude/` - **AI Development Assistance**
> Slash commands and agents for monorepo orchestration
>
> **Key command:** `/customize-monorepo` - Interactive template customization
> **Agents:** Deployment orchestration, hardware debugging

---

## Directory Layout

```
moku-instrument-forge-mono-repo/
│
├── tools/                              # Development tools
│   └── forge-codegen/                  # Submodule: YAML → VHDL generator
│       ├── forge_codegen/
│       │   ├── basic_serialized_datatypes/  # Type system (internal)
│       │   ├── generator/              # Code generation engine
│       │   ├── models/                 # Pydantic models
│       │   ├── templates/              # Jinja2 VHDL templates
│       │   └── vhdl/                   # Frozen type packages
│       ├── tests/                      # 69 tests
│       └── docs/                       # High-quality documentation
│
├── libs/                               # Foundational libraries
│   ├── forge-vhdl/                     # Submodule: VHDL utilities
│   ├── moku-models/                    # Submodule: Platform specs
│   ├── riscure-models/                 # Submodule: Probe specs
│   └── platform/                       # Platform-specific VHDL
│
├── .claude/                            # AI agent configurations
│   ├── agents/                         # Specialized agents
│   ├── commands/                       # Slash commands
│   ├── workflows/                      # Reusable workflows
│   └── shared/                         # Architecture docs
│
├── docs/                               # Monorepo documentation
├── scripts/                            # Helper scripts
└── pyproject.toml                      # Python workspace config
```

**Architecture v2.0:** Clean separation
- **tools/** - Code generation (forge-codegen)
- **libs/** - Foundational libraries (flat, no nesting)

---

## Quick Start

### Clone with Submodules

```bash
git clone --recurse-submodules https://github.com/sealablab/moku-instrument-forge-mono-repo.git
cd moku-instrument-forge-mono-repo
```

Already cloned? Initialize all submodules:

```bash
git submodule update --init --recursive
```

Verify all submodules are properly initialized:

```bash
git submodule status --recursive
```

Expected output:
```
 <commit-hash> libs/forge-vhdl (tag)
 <commit-hash> libs/moku-models (tag)
 <commit-hash> libs/riscure-models (tag)
 <commit-hash> tools/forge-codegen (tag)
```

### Setup Python Environment

This template uses **uv workspace mode** - the root is a pure workspace container.

```bash
# Install dependencies
uv sync

# Verify imports work
python -c "from moku_models import MOKU_GO_PLATFORM; print('✅ Setup complete')"

# Run tests across all workspace members
pytest
```

**About workspace mode:**
- Root `pyproject.toml` declares workspace members (your submodules)
- No build system at root - each submodule has its own
- Shared dependencies defined at root, available to all members
- Cross-submodule imports work seamlessly
- When you add/remove submodules, update `[tool.uv.workspace]` members list

---

## Documentation

Each submodule follows a **3-tier documentation system** optimized for AI navigation:

| Component | Purpose | Quick Ref | Full Guide |
|-----------|---------|-----------|------------|
| `tools/forge-codegen/` | YAML → VHDL generator | [llms.txt](tools/forge-codegen/llms.txt) | [CLAUDE.md](tools/forge-codegen/CLAUDE.md) |
| `libs/moku-models/` | Platform specs (CORE) | [llms.txt](libs/moku-models/llms.txt) | [CLAUDE.md](libs/moku-models/CLAUDE.md) |
| `libs/riscure-models/` | Probe specs (example) | [llms.txt](libs/riscure-models/llms.txt) | [CLAUDE.md](libs/riscure-models/CLAUDE.md) |
| `libs/forge-vhdl/` | VHDL utilities | [llms.txt](libs/forge-vhdl/llms.txt) | [CLAUDE.md](libs/forge-vhdl/CLAUDE.md) |

**Progressive disclosure strategy:**
- **Tier 1 (llms.txt):** Quick reference (~500-1000 tokens) - Load first
- **Tier 2 (CLAUDE.md):** Complete guide (~3-5k tokens) - Load when designing
- **Tier 3 (source code):** Implementation details - Load when coding

**For your custom probe models:** Follow the same pattern! Use `libs/riscure-models/` as a template.

---

## Architecture

**Version 2.0 - Clean Separation**

This template uses a flat, modular architecture:

```
your-moku-project/
├── tools/              # Code generation tools
│   └── forge-codegen/  # YAML → VHDL generator
├── libs/               # Foundational libraries (flat, no nesting)
│   ├── moku-models/    # Platform specs (REQUIRED)
│   ├── riscure-models/ # Probe specs (example)
│   ├── forge-vhdl/     # VHDL utilities
│   └── YOUR-models/    # Your custom probe models (add here)
├── .claude/            # AI agents and commands
├── docs/               # Your project documentation
└── pyproject.toml      # Workspace configuration
```

**Key principles:**
- **Clean separation:** tools/ vs libs/ (no nested submodules)
- **Islands of authority:** Each submodule is authoritative for its domain
- **Self-contained modules:** Each works standalone, independently versioned
- **3-tier documentation:** llms.txt → CLAUDE.md → source

See [CLAUDE.md](CLAUDE.md) for complete architecture details.

---

## Development Workflow

### Using forge-codegen

```bash
# Generate VHDL from YAML spec
python -m forge_codegen.generator.codegen your-spec.yaml --output-dir generated/

# See tools/forge-codegen/ for complete documentation
```

### Managing Submodules

```bash
# Update a submodule to latest version
cd libs/your-custom-probe-models/
git pull origin main
cd ../..
git add libs/your-custom-probe-models/
git commit -m "chore: Update probe models"

# Update workspace after adding/removing submodules
# Edit pyproject.toml [tool.uv.workspace] members, then:
uv sync
```

### Testing

```bash
pytest              # All tests
pytest libs/        # Library tests only
pytest tools/       # Tool tests
pytest -n auto      # Parallel execution
```

Configuration in `pyproject.toml`.

---

## Architecture Documentation

- **Complete Guide:** [CLAUDE.md](CLAUDE.md) - Full architecture, integration patterns
- **AI Navigation:** [llms.txt](llms.txt) - Quick component catalog
- **Customization:** Run `/customize-monorepo` in Claude Code
- **Programmatic:** [.claude/manifest.json](.claude/manifest.json) - Machine-readable structure

---

## Contributing

This is a **template repository**. When you use it:

1. **Create your own repository** from this template (click "Use this template")
2. **Make it yours** - customize for your specific Moku + probe integration
3. **Share your improvements** - If you create generally useful patterns, consider:
   - Documenting them for others
   - Contributing back enhancements to the template structure
   - Sharing your custom probe models (if not proprietary)

**For submodule development:**
1. Make changes in appropriate submodule repository
2. Write CocoTB tests for VHDL changes
3. Validate with `pytest`
4. Commit to submodule, then update monorepo reference
5. Run `uv sync` after workspace changes

---

## License

MIT License - See [LICENSE](LICENSE)

---

## Resources

**Template Repository:** Fork this to create your own Moku development environment
**Documentation:** [CLAUDE.md](CLAUDE.md) - Complete architecture guide
**AI Assistance:** Run `/customize-monorepo` in Claude Code
**Issues/Questions:** Use GitHub Issues in your forked repository

**Happy Moku development! 🚀**
