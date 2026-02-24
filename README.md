# Vibe Science v5.5 ORO — Codex Agent Skill

**Scientific research engine with adversarial review, tree search over hypotheses, and serendipity detection.**

Built natively for [OpenAI Codex](https://openai.com/codex) multi-agent architecture. Designed to prevent AI agents from publishing false scientific claims.

---

## What Is This?

Vibe Science is an agentic skill that turns a coding assistant into a rigorous scientific research partner. It solves a specific problem: **AI agents in science optimize for completion, not truth.** They find patterns, build narratives, and declare "done" — without ever asking *"what if this is an artifact?"*

The solution is architectural: the system embeds a permanent adversarial reviewer (Reviewer 2) whose only job is to **destroy claims**. In Codex, R2 runs as a separate sub-agent with its own context window — it never sees the researcher's reasoning, only the claims and evidence. This is [Blind-First Pass](references/blind-first-pass.md) by architecture, not by protocol.

### The Three Principles

1. **Serendipity Detects** — actively hunt for the unexpected at every cycle
2. **Persistence Follows** — 5, 10, 20+ cycles of testing, not one-and-done
3. **Reviewer 2 Validates** — systematic demolition before any claim is accepted

---

## Why v5.5? What Changed?

v5.5 ORO (*Observe-Recall-Operate*) was born from a post-mortem: over 21 sprints of real research, 12 data-quality errors were found — and **zero were caught by v5.0's 27 gates**. The reason: v5.0 gates verify *claim quality* (is the conclusion supported?) but not *data quality* (are the features correct? do the numbers match?).

v5.5 closes that gap.

### What's New Over v5.0

| Feature | v5.0 IUDEX | v5.5 ORO |
|---------|-----------|----------|
| Quality gates | 27 | **34** (+DQ1-4, DD0, DC0, L-1) |
| R2 activation modes | 6 | **7** (+INLINE per-finding review) |
| R2 architecture | Same-agent self-review | **Separate sub-agent** (native BFP) |
| Enforcement | Prompt-only (bypassable) | **Python scripts** (exit code 0/1) |
| Observer | None | **Parallel sub-agent** scanning for drift |
| Research Spine | None | **Mandatory structured logbook** |
| SSOT | None | **JSON-as-source-of-truth** + sync check |
| Immutable Laws | 10 | **11** (+LISTEN TO THE USER) |
| Domain scope | Bio-specific (scRNA-seq) | **Domain-agnostic** (configurable) |
| Multi-agent | None | **4 sub-agent roles** with model recommendations |
| JSON schemas | 9 | **12** (+3 new for DQ, Spine, Finding) |

### Previous Versions

| Version | Codename | Platform | Location |
|---------|----------|----------|----------|
| v5.0 | IUDEX | Codex | [`archive/vibe-science-v5.0-codex`](https://github.com/th3vib3coder/vibe-science/tree/main/archive/vibe-science-v5.0-codex) |
| v5.5 | ORO | Claude Code | [`archive/vibe-science-v5.5`](https://github.com/th3vib3coder/vibe-science/tree/main/archive/vibe-science-v5.5) |
| v6.0 | NEXUS | Claude Code Plugin | [`th3vib3coder/vibe-science`](https://github.com/th3vib3coder/vibe-science) (main repo) |

This repository is the **Codex-native** implementation of v5.5.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  MAIN AGENT (Researcher)                                     │
│  Model: gpt-5.3-codex | Reasoning: medium                   │
│  Role: Build, explore, execute OTAE cycles                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Sub-Agents (spawned as needed):                             │
│                                                              │
│  ┌────────────────────┐  ┌────────────────────┐             │
│  │ R2-DEEP            │  │ R2-INLINE          │             │
│  │ Separate context!  │  │ Fast, per-finding  │             │
│  │ FORCED/BATCH/BRAIN │  │ 7-point checklist  │             │
│  └────────────────────┘  └────────────────────┘             │
│                                                              │
│  ┌────────────────────┐  ┌────────────────────┐             │
│  │ OBSERVER           │  │ EXPLORER           │             │
│  │ Read-only scans    │  │ Parallel branches  │             │
│  │ Drift detection    │  │ Literature search  │             │
│  └────────────────────┘  └────────────────────┘             │
│                                                              │
│  Scripts (deterministic, non-bypassable):                    │
│  dq_gate.py | sync_check.py | tree_health.py                │
│  gate_check.py | spine_entry.py | observer.py                │
└──────────────────────────────────────────────────────────────┘
```

### Why Codex Multi-Agent Is Architecturally Superior

In Claude Code (v5.5 skill and v6.0 plugin), Reviewer 2 is the **same agent playing a different role**. It has access to the researcher's reasoning, excitement, and narrative — it must *self-blind* via protocol. In Codex, R2 is a **genuinely separate sub-agent** with its own context window. It never sees the researcher's justifications. Blind-First Pass happens by **architecture**, not by discipline.

---

## Directory Structure

```
vibe-science/                      ← Skill root
├── SKILL.md                       ← Main spec (382 lines, under 500 limit)
├── LICENSE                        ← Apache 2.0
├── .gitignore
│
├── agents/
│   └── openai.yaml                ← Codex manifest + implicit invocation
│
├── scripts/                       ← Enforcement (Python 3.8+, stdlib only)
│   ├── dq_gate.py                 ← DQ1-DQ4 data quality checks
│   ├── sync_check.py              ← SSOT: JSON vs markdown number match
│   ├── tree_health.py             ← T3 gate: exploration ratio
│   ├── gate_check.py              ← Generic JSON Schema validation
│   ├── spine_entry.py             ← Research Spine entry creation
│   └── observer.py                ← Project health checks
│
├── references/                    ← On-demand protocols (28 files)
│   ├── constitution.md            ← 11 Laws + role constraints
│   ├── reviewer2-ensemble.md      ← 7 modes, multi-agent R2
│   ├── gates-complete.md          ← All 34 gates detailed
│   ├── loop-otae.md               ← OTAE loop with v5.5 insertions
│   ├── dq-gates.md                ← DQ1-DQ4 protocol
│   ├── research-spine.md          ← Structured logbook
│   ├── ssot.md                    ← Single Source of Truth
│   ├── silent-observer.md         ← Observer sub-agent protocol
│   ├── multi-agent-config.md      ← Codex agent configuration
│   └── ... (19 more)              ← See SKILL.md routing table
│
└── assets/                        ← Static resources
    ├── templates.md               ← STATE.md, PROGRESS.md, CLAIM-LEDGER templates
    ├── fault-taxonomy.yaml        ← SFI meta-faults (domain-agnostic)
    ├── judge-rubric.yaml          ← R3 scoring rubric
    ├── domain-config-example.yaml ← How to add domain-specific thresholds
    └── schemas/                   ← 12 JSON Schema files (READ-ONLY)
        ├── brainstorm-quality.schema.json
        ├── claim-promotion.schema.json
        ├── data-quality-gate.schema.json      ← NEW
        ├── spine-entry.schema.json            ← NEW
        ├── finding-validation.schema.json     ← NEW
        └── ... (7 more from v5.0)
```

---

## Quick Start

### 1. Install as Codex Agent Skill

Place this directory inside your Codex agent skills folder. The skill auto-activates when Codex detects a scientific research task (see `agents/openai.yaml`).

### 2. Explicit Invocation

```
$vibe-science
```

### 3. Domain Configuration (Optional)

Copy `assets/domain-config-example.yaml` to your project root as `domain-config.yaml` and customize thresholds, metrics, and domain-specific SFI faults. Without a config file, Vibe Science operates with generic defaults.

### 4. Enforcement Scripts

All scripts require Python 3.8+ with no external dependencies (stdlib only):

```bash
# Data quality gate
python scripts/dq_gate.py --gate DQ1 --data extracted_features.json

# SSOT sync check
python scripts/sync_check.py --json results.json --md FINDINGS.md

# Tree health
python scripts/tree_health.py --tree .vibe-science/TREE-STATE.json

# Generic gate validation
python scripts/gate_check.py --gate B0 --artifact brainstorm.json --schema assets/schemas/brainstorm-quality.schema.json

# Research Spine entry
python scripts/spine_entry.py --spine .vibe-science/SPINE.md --type DATA_LOAD --action "Loaded dataset X"

# Observer
python scripts/observer.py --project .vibe-science/
```

---

## The 11 Immutable Laws

1. **DATA-FIRST** — No thesis without evidence. `NO DATA = NO GO.`
2. **EVIDENCE DISCIPLINE** — Every claim tracked with ID, confidence, and status.
3. **GATES BLOCK** — 34 gates are hard stops, not suggestions.
4. **REVIEWER 2 IS CO-PILOT** — R2 can VETO, REDIRECT, FORCE. Non-negotiable.
5. **SERENDIPITY IS THE MISSION** — Hunt for the unexpected at every cycle.
6. **ARTIFACTS OVER PROSE** — If it can be a file, it must be.
7. **FRESH CONTEXT RESILIENCE** — Resumable from `STATE.md` + `TREE-STATE.json`.
8. **EXPLORE BEFORE EXPLOIT** — Min 3 drafts before promotion. Exploration >= 20%.
9. **CONFOUNDER HARNESS** — Raw → conditioned → matched. Sign change = ARTIFACT.
10. **CRYSTALLIZE OR LOSE** — If it's not in a file, it doesn't exist.
11. **LISTEN TO THE USER** — Follow user corrections immediately. No arguing.

---

## The 34 Quality Gates

| Category | Gates | New in v5.5 |
|----------|-------|-------------|
| Pipeline | G0-G6 | — |
| Literature | L-1, L0-L2 | L-1 |
| Decision | D0-D2 | — |
| Tree | T0-T3 | — |
| Brainstorm | B0 | — |
| Stage | S1-S5 | — |
| Data Quality | DQ1-DQ4 | DQ1-DQ4 |
| Data Dictionary | DD0 | DD0 |
| Design Compliance | DC0 | DC0 |
| Vigilance | V0 | — |
| Judge | J0 | — |

8 gates are schema-enforced (JSON Schema validation required, prose claims of completion ignored).

---

## Lineage

Vibe Science draws from:

- **AI-Scientist-v2** (Yamada et al., 2025) — 5-stage experiment manager, tree search over hypotheses
- **Kahneman's Adversarial Collaboration** — builder-breaker asymmetry
- **Mutation Testing** (Jia & Harman, 2011) — Seeded Fault Injection for R2 vigilance
- **LLM Self-Correction Limits** (Huang et al., ICLR 2024) — why same-agent review fails
- **21 sprints of real research** — every protocol exists because something went wrong

---

## License

Apache 2.0. See [LICENSE](LICENSE).

---

*Detect the unexpected. Follow it relentlessly. Destroy every claim that can't survive hostile review.*
