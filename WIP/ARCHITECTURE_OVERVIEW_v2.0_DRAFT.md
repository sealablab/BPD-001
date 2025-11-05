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

### Example 1: Quick Type Lookup

```
User: "What voltage types exist?"
    ↓
AI Agent loads: basic-app-datatypes/llms.txt (Tier 1, ~500 tokens)
    ↓
Answer: "23 types available. voltage_output_05v_s16 for ±5V DAC output..."
    ↓
Done. 99.75% of token budget remaining.
```

### Example 2: Wiring Safety Validation

```
User: "Is my Moku → probe wiring safe?"
    ↓
AI Agent loads: moku-models/llms.txt + riscure-models/llms.txt (Tier 1, ~1k tokens)
    ↓
Check: Moku:Go OUT1 = 10Vpp (±5V) vs DS1120A = 0-3.3V TTL
    ↓
Potential issue detected
    ↓
AI Agent loads: MODELS_INDEX.md (Tier 2, +1.5k tokens)
    ↓
Find integration pattern: "Platform ← Probe Wiring Safety"
    ↓
Answer: "Use TTL mode on Moku output (3.3V), not raw DAC (±5V). Safe connection confirmed."
    ↓
Done. Total: 2.5k tokens (1.25% of budget)
```

### Example 3: Adding New Type

```
User: "How do I add a new voltage type for ±10V?"
    ↓
AI Agent loads: basic-app-datatypes/llms.txt (Tier 1)
    ↓
Sees: "23 types defined. For adding types, see CLAUDE.md"
    ↓
AI Agent loads: basic-app-datatypes/CLAUDE.md (Tier 2, +3k tokens)
    ↓
Find section: "Adding New Types"
    ↓
Answer: "1. Add to BasicAppDataTypes enum
         2. Add metadata to TYPE_REGISTRY
         3. Add conversion function
         4. Add tests
         5. Update llms.txt catalog
         6. Submit PR to basic-app-datatypes repo"
    ↓
Done. Total: 3.5k tokens (1.75% of budget)
```

### Example 4: Debugging VHDL Generation

```
User: "My generated shim has wrong bit slices"
    ↓
AI Agent loads: llms.txt (Tier 1, 1k tokens)
    ↓
Delegate to: forge-context agent
    ↓
AI Agent loads: forge-context/agent.md (Tier 2, +3k tokens)
    ↓
Still unclear, need implementation details
    ↓
AI Agent loads: forge/generator/codegen.py (Tier 3, +5k tokens)
    ↓
Trace bit slice calculation logic
    ↓
AI Agent loads: generated shim file (Tier 3, +3k tokens)
    ↓
Compare: Template logic vs actual output
    ↓
Identify bug: Off-by-one in bit slice calculation
    ↓
Done. Total: 12k tokens (6% of budget)
```

---

## 5. Cross-Library Integration Patterns

From `forge/libs/MODELS_INDEX.md`:

### Pattern 1: Type ← Platform Validation

```python
# Validate that a type is compatible with platform output
from basic_app_datatypes import BasicAppDataTypes, TYPE_REGISTRY
from moku_models import MOKU_GO_PLATFORM

voltage_type = BasicAppDataTypes.VOLTAGE_OUTPUT_05V_S16
metadata = TYPE_REGISTRY[voltage_type]
# → voltage_range: "±5V"

platform = MOKU_GO_PLATFORM
dac_output = platform.get_analog_output_by_id('OUT1')
# → voltage_range_vpp: 10.0 (±5V)

# Validation
assert metadata.voltage_range == "±5V"
assert dac_output.voltage_range_vpp == 10.0
print("✓ Type compatible with platform")
```

**Libraries involved:** basic-app-datatypes (authoritative for type), moku-models (authoritative for platform)

### Pattern 2: Platform ← Probe Wiring Safety

```python
# Validate safe connection between Moku output and probe input
from moku_models import MOKU_GO_PLATFORM
from riscure_models import DS1120A_PLATFORM

# Moku output spec
moku_out = MOKU_GO_PLATFORM.get_analog_output_by_id('OUT1')
# → voltage_range_vpp = 10.0 (can output 0-3.3V TTL mode)

# Probe input spec
probe_in = DS1120A_PLATFORM.get_port_by_id('digital_glitch')
# → voltage_min=0V, voltage_max=3.3V

# Safety check
voltage = 3.3  # TTL mode
if probe_in.is_voltage_compatible(voltage):
    print("✓ Safe connection (use TTL mode, not raw DAC)")
else:
    print("✗ UNSAFE - voltage exceeds probe limits")
```

**Libraries involved:** moku-models (authoritative for Moku output), riscure-models (authoritative for probe input)

### Pattern 3: Type ← Probe Compatibility

```python
# Validate type usage for probe control
from basic_app_datatypes import BasicAppDataTypes
from riscure_models import DS1120A_PLATFORM

# User specifies control type
trigger_type = BasicAppDataTypes.BOOLEAN_1
# → 1-bit boolean

# Probe trigger spec
probe = DS1120A_PLATFORM
trigger_port = probe.get_port_by_id('digital_glitch')
# → Expects TTL signal (boolean-compatible)

print("✓ boolean_1 type compatible with probe TTL trigger")
```

**Libraries involved:** basic-app-datatypes (authoritative for type), riscure-models (authoritative for probe)

---

## 6. AI Agent Hierarchy (Delegation Model)

### Monorepo-Level Agents (`.claude/agents/`)

**probe-design-orchestrator** (Primary coordinator)
- Coordinates complete probe development workflow
- Delegates to forge agents for specialized tasks
- Tracks multi-probe state across forge/apps/
- Handles cross-validation (VHDL ↔ package)

**deployment-orchestrator**
- Hardware deployment (package → Moku device)
- Device discovery
- Routing configuration

**hardware-debug**
- FSM debugging
- State monitoring
- Signal tracing

### Forge-Level Agents (`forge/.claude/agents/`)

**workflow-coordinator**
- Multi-stage forge pipelines
- Orchestrates forge-specific agents
- End-to-end workflows

**forge-context**
- YAML → VHDL generation
- Package creation
- Register mapping optimization

**deployment-context**
- Package → hardware deployment
- Moku device interaction

**docgen-context**
- Documentation generation
- TUI creation
- Python API generation

**hardware-debug-context**
- FSM debugging expert
- Voltage-encoded state monitoring

### Delegation Flow

```
User Request: "Create new probe DS1180_LASER"
    ↓
Monorepo Agent: probe-design-orchestrator
    ↓
Delegates to: forge-context (YAML validation)
    ↓
forge-context loads: basic-app-datatypes/llms.txt (validate types)
    ↓
forge-context loads: moku-models/llms.txt (validate platform)
    ↓
Delegates to: workflow-coordinator (package generation)
    ↓
Returns to: probe-design-orchestrator
    ↓
Cross-validates: VHDL ↔ manifest.json
    ↓
Delegates to: deployment-orchestrator (hardware deployment)
    ↓
Done. Probe operational.
```

---

## 7. Git Submodule Workflow

### The Challenge

Working across 4 levels of nested git repositories:

```
monorepo (git repo)
  └── forge/ (git submodule)
      └── libs/basic-app-datatypes/ (git submodule)
```

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
