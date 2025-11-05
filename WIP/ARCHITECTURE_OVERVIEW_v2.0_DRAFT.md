# Composable Architecture Overview

**Version:** 2.0
**Purpose:** Understanding the elegant flat composable architecture
**Audience:** Humans and AI agents learning this system

---

## The Elegant Architecture

This repository demonstrates a **flat, composable architecture** with **self-contained authoritative submodules** that integrate through explicit patterns, supported by a **3-tier documentation system** optimized for token-efficient AI context loading.

---

## 1. Git Repository Structure (Flat Composition)

```
BPD-001/                                       # Root: Probe driver framework
│
├── bpd/                                       # Application code (your work)
│   ├── bpd-core/                              # Generic probe framework (Python)
│   │   ├── llms.txt                           # Tier 1: Quick ref
│   │   ├── CLAUDE.md                          # Tier 2: Design rationale
│   │   └── src/bpd_core/                      # Framework implementation
│   │
│   ├── bpd-drivers/                           # Probe-specific drivers (Python)
│   │   ├── llms.txt                           # Tier 1: Driver catalog
│   │   ├── CLAUDE.md                          # Tier 2: Driver development guide
│   │   └── src/bpd_drivers/                   # DS1120A, future probes
│   │
│   └── bpd-vhdl/                              # Vendor-agnostic VHDL interface
│       ├── llms.txt                           # Tier 1: Component catalog
│       ├── CLAUDE.md                          # Tier 2: FSM architecture
│       └── vhdl/                              # Generic probe control FSM
│
├── libs/                                      # Authoritative submodules (git submodules)
│   ├── moku-models/                           # Platform specs (Pydantic models)
│   │   ├── llms.txt                           # Tier 1: Platform quick ref
│   │   ├── CLAUDE.md                          # Tier 2: Integration patterns
│   │   └── moku_models/                       # MOKU_GO_PLATFORM, etc.
│   │
│   ├── riscure-models/                        # Probe hardware specs (Pydantic models)
│   │   ├── llms.txt                           # Tier 1: Probe specs
│   │   ├── CLAUDE.md                          # Tier 2: Safety patterns
│   │   └── riscure_models/                    # DS1120A_PLATFORM
│   │
│   └── forge-vhdl/                            # Reusable VHDL utilities
│       ├── llms.txt                           # Tier 1: Component catalog
│       └── vhdl/                              # Clock dividers, utilities
│
├── tools/                                     # Development tools (git submodules)
│   └── forge-codegen/                         # YAML → VHDL generator
│
├── llms.txt                                   # Tier 1: Root navigation
├── CLAUDE.md                                  # Tier 2: BPD-001 overview
└── .claude/                                   # AI agent system
    └── shared/
        └── CONTEXT_MANAGEMENT.md              # Token optimization strategy
```

---

## 2. The Elegant Properties

### Property 1: Self-Contained Authoritative Components

Each component is a **self-contained authority** for its domain:

**moku-models** (Platform Specs Authority)
```
libs/moku-models/
├── llms.txt           # "Moku:Go = 125MHz, Moku:Lab = 500MHz..."
├── CLAUDE.md          # "Platform integration patterns, I/O specs..."
└── moku_models/
    ├── platforms.py   # Pydantic: MOKU_GO_PLATFORM (AUTHORITATIVE)
    ├── routing.py     # I/O routing models
    └── deployment.py  # MokuConfig structures
```

- **Authority:** Clock frequencies, voltage ranges, I/O configurations
- **Zero dependencies** (except Pydantic) - pure data models
- **No deployment logic** - no Moku API calls
- **Standalone** - works outside BPD-001

**riscure-models** (Probe Hardware Authority)
```
libs/riscure-models/
├── llms.txt           # "DS1120A voltage ranges: 0-3.3V TTL..."
├── CLAUDE.md          # "Wiring safety, voltage validation patterns..."
└── riscure_models/
    ├── probes.py      # Pydantic: DS1120A_PLATFORM (AUTHORITATIVE)
    └── validation.py  # is_voltage_compatible()
```

- **Authority:** Probe electrical specs, voltage safety, port definitions
- **Zero dependencies** (except Pydantic) - pure data models
- **Composable** with moku-models for wiring validation
- **Standalone** - works outside BPD-001

**bpd-core** (Framework Authority)
```
bpd/bpd-core/
├── llms.txt           # "Generic FIProbeInterface protocol..."
├── CLAUDE.md          # "Framework design, validation patterns..."
└── src/bpd_core/
    ├── interface.py   # FIProbeInterface (AUTHORITATIVE)
    ├── registry.py    # Driver discovery
    └── validation.py  # validate_probe_moku_compatibility()
```

- **Authority:** Probe driver interface, safety validation patterns
- **Minimal dependencies** (Pydantic for models)
- **Framework-level** - composes moku-models + riscure-models
- **Extensible** - add new probe types without changing core

### Property 2: Composability Without Coupling

Components are composable but don't depend on each other:

```python
# Each component knows only its domain
from moku_models import MOKU_GO_PLATFORM
from riscure_models import DS1120A_PLATFORM
from bpd_core import validate_probe_moku_compatibility

# Libraries are independent (no imports between them)
# moku-models doesn't know about riscure-models
# riscure-models doesn't know about moku-models

# Compose at framework layer for cross-validation
from bpd_drivers import DS1120ADriver

driver = DS1120ADriver()  # Uses riscure-models specs
validate_probe_moku_compatibility(driver, MOKU_GO_PLATFORM)
# → bpd-core orchestrates validation using both authoritative sources

# Each is authoritative for its domain, bpd-core composes them
```

**Key insight:**
- **libs/** components don't import each other - they're standalone
- **bpd/bpd-core/** composes them for integration validation
- Integration patterns documented in component CLAUDE.md files

### Property 3: Three-Tier Documentation System

Every authoritative component follows the pattern:

**Tier 1: llms.txt** (~150 lines, ~500 tokens)
- Quick facts
- Core exports (table format)
- Basic usage example
- Pointers to Tier 2

**Tier 2: CLAUDE.md** (~250-600 lines, ~2-5k tokens)
- Design rationale
- Complete specifications
- Integration patterns
- Development workflows

**Tier 3: Source Code** (~variable, 5-10k tokens per file)
- Implementation details
- Pydantic models
- Tests

**AI Agent Strategy:**
```
Quick question? → Load Tier 1 (llms.txt)
Design question? → Load Tier 2 (CLAUDE.md)
Implementation? → Load Tier 3 (source code)
```

### Property 4: Never Guess, Always Trust

Components are **authoritative sources of truth**:

```python
# ❌ BAD: AI Agent guesses
"I think Moku:Go runs at 100MHz..."
"The probe probably accepts 5V..."
"DS1120A likely has configurable pulse width..."

# ✅ GOOD: AI Agent reads authority
AI loads: libs/moku-models/llms.txt
→ "Moku:Go = 125 MHz (authoritative)"

AI loads: libs/riscure-models/llms.txt
→ "DS1120A digital_glitch = 0-3.3V TTL only"
→ "DS1120A pulse width = 50ns (FIXED, not configurable)"

AI loads: bpd/bpd-core/llms.txt
→ "FIProbeInterface protocol defines: arm(), trigger(), disarm()"
```

**The principle:**
- **Never infer** specs from similar hardware
- **Always read** the authoritative component's documentation
- **Start with Tier 1** (llms.txt) for quick facts
- **Escalate to Tier 2** (CLAUDE.md) for design context

### Property 5: Token-Efficient Context Loading

```
Quick lookup:
  Load 3× llms.txt (450 lines total, ~1k tokens)
  Budget used: 0.5%

Design work:
  Load llms.txt + MODELS_INDEX.md + 1× CLAUDE.md (~4k tokens)
  Budget used: 2%

Deep implementation:
  Load llms.txt + CLAUDE.md + source files (~10k tokens)
  Budget used: 5%

Still have 190k tokens available (95%)!
```

---

## 3. Documentation Ecosystem Map

### Root Level (BPD-001)
```
BPD-001/
├── llms.txt                        # Tier 1: Entry point, navigation
├── CLAUDE.md                       # Tier 2: BPD-001 overview
├── README.md                       # Human-friendly project intro
└── .claude/
    └── shared/
        ├── ARCHITECTURE_OVERVIEW.md    # This file
        └── CONTEXT_MANAGEMENT.md       # Token optimization strategy
```

### Application Layer (bpd/)
```
bpd/
├── bpd-core/                       # Framework component
│   ├── llms.txt                    # Tier 1: Framework quick ref
│   ├── CLAUDE.md                   # Tier 2: Design rationale
│   └── src/bpd_core/               # Tier 3: Source code
│
├── bpd-drivers/                    # Driver component
│   ├── llms.txt                    # Tier 1: Driver catalog
│   ├── CLAUDE.md                   # Tier 2: Driver development guide
│   └── src/bpd_drivers/            # Tier 3: DS1120A, future drivers
│
└── bpd-vhdl/                       # VHDL component
    ├── llms.txt                    # Tier 1: Component catalog
    ├── CLAUDE.md                   # Tier 2: FSM architecture
    └── vhdl/                       # Tier 3: VHDL source
```

### Library Layer (libs/) - Authoritative Submodules
```
libs/
├── moku-models/                    # Platform specs (git submodule)
│   ├── llms.txt                    # Tier 1: Platform quick ref
│   ├── CLAUDE.md                   # Tier 2: Integration patterns
│   └── moku_models/                # Tier 3: Pydantic models
│
├── riscure-models/                 # Probe hardware (git submodule)
│   ├── llms.txt                    # Tier 1: Probe specs
│   ├── CLAUDE.md                   # Tier 2: Safety patterns
│   └── riscure_models/             # Tier 3: Pydantic models
│
└── forge-vhdl/                     # VHDL utilities (git submodule)
    ├── llms.txt                    # Tier 1: Component catalog
    └── vhdl/                       # Tier 3: VHDL packages
```

---

## 4. Information Flow: The Truth Cascade

**[Examples to be added - placeholder for BPD-001 specific workflows]**

This section will contain concrete examples showing:
- Quick probe capability lookup
- Wiring safety validation workflow
- Driver discovery patterns
- VHDL FSM debugging scenarios

---

## 5. Component Integration Patterns

### Pattern 1: Platform ← Probe Wiring Safety

```python
# Validate safe connection between Moku output and probe input
from moku_models import MOKU_GO_PLATFORM
from riscure_models import DS1120A_PLATFORM
from bpd_core import validate_probe_moku_compatibility

# Each library is authoritative for its domain
platform = MOKU_GO_PLATFORM  # Platform specs from moku-models
probe_specs = DS1120A_PLATFORM  # Probe specs from riscure-models

# bpd-core composes them for safety validation
from bpd_drivers import DS1120ADriver
driver = DS1120ADriver()

# Cross-component validation
validate_probe_moku_compatibility(driver, platform, output_id='OUT1')
# → Checks: Moku OUT1 voltage range vs DS1120A input limits
# → Result: "✓ Safe connection (use TTL mode, not raw DAC)"
```

**Components involved:**
- **moku-models** (authoritative for platform I/O specs)
- **riscure-models** (authoritative for probe electrical limits)
- **bpd-core** (orchestrates validation)

### Pattern 2: Driver ← Probe Specs

```python
# Driver implementation uses probe specs as source of truth
from riscure_models import DS1120A_PLATFORM
from bpd_core import FIProbeInterface, ProbeCapabilities

class DS1120ADriver(FIProbeInterface):
    def __init__(self):
        # Read authoritative specs from riscure-models
        self._probe_specs = DS1120A_PLATFORM

    @property
    def capabilities(self) -> ProbeCapabilities:
        # Return capabilities from authoritative source
        port = self._probe_specs.get_port_by_id('digital_glitch')
        return ProbeCapabilities(
            min_voltage_v=port.voltage_min,
            max_voltage_v=port.voltage_max,
            min_pulse_width_ns=50,  # DS1120A fixed
            max_pulse_width_ns=50
        )
```

**Components involved:**
- **riscure-models** (authoritative for DS1120A specs)
- **bpd-core** (defines FIProbeInterface protocol)
- **bpd-drivers** (implements driver using authoritative specs)

### Pattern 3: Framework ← VHDL Interface

```python
# Python framework controls VHDL FSM through Moku registers
from moku_models import MOKU_GO_PLATFORM
from bpd_core import FIProbeInterface

class GenericProbeDriver(FIProbeInterface):
    def arm(self):
        # Write to Moku control register
        # VHDL FSM: IDLE → ARMED
        self._moku_device.set_control_register('arm', 1)

    def trigger(self):
        # Write to Moku control register
        # VHDL FSM: ARMED → PULSE_ACTIVE → COOLDOWN
        self._moku_device.set_control_register('trigger', 1)
```

**Components involved:**
- **bpd-core** (Python framework API)
- **bpd-vhdl** (VHDL FSM implementation)
- **moku-models** (platform register interface)

---

## 6. Git Submodule Workflow

### The Structure

BPD-001 uses **flat composition** - submodules in `libs/` and `tools/`:

```
BPD-001/ (git repo)
  ├── libs/moku-models/ (git submodule)
  ├── libs/riscure-models/ (git submodule)
  ├── libs/forge-vhdl/ (git submodule)
  └── tools/forge-codegen/ (git submodule)
```

**Key insight:** No nested submodules - all at same level for discoverability

### The Pattern

**Modifying a foundational library (e.g., basic-app-datatypes):**

```bash
# 1. Navigate to submodule
cd forge/libs/basic-app-datatypes

# 2. Make changes IN THE SUBMODULE
git checkout -b feat/add-10v-type
# ... edit code ...
git commit -m "feat: Add voltage_output_10v_s16 type"

# 3. Push THE SUBMODULE (to its own repo)
git push origin feat/add-10v-type

# 4. Create PR in submodule repo, merge it

# 5. Return to parent and update reference
cd ../../..
git add forge/libs/basic-app-datatypes
git commit -m "chore: Update basic-app-datatypes"
git push
```

**Key rules:**
- Always commit in submodule FIRST
- Then commit submodule reference update in parent
- Each submodule has its own repo, issues, PRs

---

## 8. Workspace Architecture (Option A)

**Everything in one place:** `forge/apps/<probe_name>/`

```
forge/apps/DS1140_PD/
├── DS1140_PD.yaml                    # Source specification (YAML)
├── DS1140_PD_custom_inst_shim.vhd    # Auto-generated (DO NOT EDIT)
├── DS1140_PD_custom_inst_main.vhd    # User implementation
├── manifest.json                      # Package contract (register mappings)
├── control_registers.json             # Default CR values
└── README.md                          # Probe documentation
```

**Why Option A?**
- Simple mental model: "everything in one place"
- YAML + generated VHDL + implementation colocated
- Easy to navigate: one directory per probe
- Works with forge as-is (no modifications needed)

**Workflow:**
```bash
/init-probe DS1180_LASER        # Creates forge/apps/DS1180_LASER/
# Edit: forge/apps/DS1180_LASER/DS1180_LASER.yaml
/generate forge/apps/DS1180_LASER/DS1180_LASER.yaml
# Edit: forge/apps/DS1180_LASER/*_main.vhd
/deploy DS1180_LASER --device 192.168.1.100
```

---

## 9. Design Principles

### 1. Composability
- Submodules are standalone
- Monorepo orchestrates
- Integration patterns explicit

### 2. Tiered Documentation
- llms.txt (quick ref) → CLAUDE.md (deep dive) → source code
- Load minimally, expand as needed
- Token budget optimization

### 3. Agent Delegation
- Monorepo coordinates
- Forge executes
- Libraries validate

### 4. Single Source of Truth
- Types → basic-app-datatypes (authoritative)
- Platforms → moku-models (authoritative)
- Probes → riscure-models (authoritative)
- Never guess, always read

### 5. Context Efficiency
- Start with ~1k tokens (Tier 1)
- Expand to ~4k tokens (Tier 2)
- Deep dive to ~12k tokens (Tier 3)
- Reserve 188k tokens (94% of budget)

---

## 10. The Elegant Summary

This system achieves:

✅ **4-level git submodule hierarchy** (monorepo → forge → libs → nested)
✅ **3 self-contained authoritative Pydantic model libraries**
✅ **3-tier documentation system** (llms.txt → CLAUDE.md → source)
✅ **Token-efficient AI context loading** (start ~1k, expand as needed)
✅ **Composability without coupling** (libraries don't import each other)
✅ **Never guess, always trust** (authoritative sources of truth)
✅ **Hierarchical agent delegation** (monorepo → forge → libraries)
✅ **Simple workspace model** (forge/apps/ everything in one place)

**The magic:** Each layer is independently meaningful, yet they compose elegantly. AI agents navigate with minimal tokens. Humans understand the structure intuitively. The system scales without complexity explosion.

---

**Last Updated:** 2025-11-03
**Maintained By:** moku-instrument-forge team
**Version:** 1.0
