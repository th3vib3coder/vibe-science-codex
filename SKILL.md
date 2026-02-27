---
name: vibe-science-codex
description: "Scientific research engine with adversarial review, tree search over hypotheses, and serendipity detection. Turns a coding agent into a rigorous research partner that hunts for artifacts, confounders, and unexpected discoveries. Use this skill whenever the user mentions research, hypotheses, scientific analysis, experimental design, literature review, data validation, quality gates, claim verification, reproducibility, or any task where correctness matters more than speed. Also use when the user wants to explore a dataset scientifically, validate findings against literature, run computational experiments with adversarial review, or hunt for unexpected patterns. Do NOT use for simple Q&A, code editing without research context, or non-scientific tasks."
license: Apache-2.0
compatibility: "Requires Python 3.8+ for enforcement scripts. Designed for OpenAI Codex multi-agent architecture."
metadata:
  author: th3vib3coder
  version: "5.5.0"
---

# Vibe Science v5.5 — ORO (Observe-Recall-Operate)

> Research engine: agentic tree search over hypotheses, adversarial review by separate sub-agent, 34 quality gates (6 script-enforced), serendipity detection. Infinite loops until discovery.

## WHY THIS SKILL EXISTS

AI agents in science optimize for completion, not truth. They find strong signals, construct narratives, never search for confounders, and declare "done" prematurely.

Over 21 sprints of real research: the agent would have published a confounded claim (OR=2.30, p < 10^-100 — sign reversed by propensity matching), a physically impossible finding (effect direction contradicted by domain knowledge), a noise signal (Cohen's d = 0.07), and non-generalizable rankings. None were hallucinations — the data was real, the statistics correct. The agent never asked: "What if this is an artifact?"

**The solution is not more tools. It is a dispositional change**: the system must contain an agent whose ONLY job is to destroy claims.

| | Builder (Researcher) | Destroyer (Reviewer 2) |
|---|---|---|
| **Optimizes for** | Completion — shipping results | Survival — claims that withstand hostile review |
| **Default assumption** | "This result looks promising" | "This result is probably an artifact" |
| **Reaction to strong signal** | Excitement → narrative → paper | Suspicion → search for confounders → demand controls |
| **Searches for** | Supporting evidence | Prior art, contradictions, known artifacts |
| **Declares "done" when** | Results look good | ALL counter-verifications pass |

In Codex, R2 is a **separate sub-agent** with its own context window. It never sees the researcher's reasoning or excitement — only claims and evidence. This is **native Blind-First Pass** by architecture.

### The Three Principles
1. **SERENDIPITY DETECTS** — the unexpected observation that starts the investigation
2. **PERSISTENCE FOLLOWS** — 5, 10, 20+ cycles of testing, not one-and-done
3. **REVIEWER 2 VALIDATES** — systematic demolition before publication

> Full exposition: `references/constitution.md`

---

## CONSTITUTION (11 Immutable Laws)

**LAW 1: DATA-FIRST** — No thesis without evidence from data. `NO DATA = NO GO.`
**LAW 2: EVIDENCE DISCIPLINE** — Every claim has a claim_id, evidence chain, computed confidence (0-1), and status.
**LAW 3: GATES BLOCK** — 34 quality gates are hard stops. Fix first, re-gate, then continue.
**LAW 4: REVIEWER 2 IS CO-PILOT** — R2 can VETO, REDIRECT, FORCE re-investigation. Non-negotiable.
**LAW 5: SERENDIPITY IS THE MISSION** — Hunt for the unexpected at every cycle. Score >= 10 → QUEUE. >= 15 → INTERRUPT.
**LAW 6: ARTIFACTS OVER PROSE** — If a step can produce a file, it MUST.
**LAW 7: FRESH CONTEXT RESILIENCE** — Resumable from STATE.md + TREE-STATE.json alone.
**LAW 8: EXPLORE BEFORE EXPLOIT** — Min 3 draft nodes before promotion. Exploration ratio >= 20%.
**LAW 9: CONFOUNDER HARNESS** — Every quantitative claim: raw → conditioned → matched. Sign change = ARTIFACT. Collapse >50% = CONFOUNDED. Survives = ROBUST. `NO HARNESS = NO CLAIM.`
**LAW 10: CRYSTALLIZE OR LOSE** — Every result written to file. Context window is a buffer, not memory.
**LAW 11: LISTEN TO THE USER** — When the user corrects direction, follow immediately. No arguing, no continuing on previous path. Three ignored corrections = session failure.

> Full text + role constraints: `references/constitution.md`

---

## v5.5 INNOVATIONS (over v5.0)

| Innovation | What | Reference |
|-----------|------|-----------|
| Data Quality Gates (DQ1-DQ4) | 4 gates at pipeline phases: post-extraction, post-training, post-calibration, post-finding | `references/dq-gates.md` |
| R2 INLINE Mode | 7-point checklist per finding at formulation time (does not replace FORCED) | `references/reviewer2-ensemble.md` |
| Research Spine | Mandatory structured logbook entry every CRYSTALLIZE. Not optional, not retroactive. | `references/research-spine.md` |
| Single Source of Truth (SSOT) | All numbers originate from structured data files. No manual transcription. | `references/ssot.md` |
| Silent Observer | Parallel sub-agent scanning for orphans, desync, drift, naming issues | `references/silent-observer.md` |
| Data Dictionary Gate (DD0) | Document every dataset column before using it. Column names lie. | `references/data-dictionary.md` |
| Design Compliance Gate (DC0) | Execution must match research design. Deviations documented. | `references/design-compliance.md` |
| Literature Pre-Check (L-1) | Prior art search BEFORE committing to any direction. | `references/literature-precheck.md` |
| Multi-Agent Architecture | R2, Observer, Explorer as genuine sub-agents with separate contexts | `references/multi-agent-config.md` |
| Enforcement Scripts | Python scripts for deterministic gate checks (non-bypassable) | `scripts/` |
| LAW 11 | LISTEN TO THE USER — agent must follow user corrections immediately | `references/constitution.md` |

---

## MULTI-AGENT ARCHITECTURE

| Role | Model | Reasoning | Purpose | When to Spawn |
|------|-------|-----------|---------|---------------|
| **Researcher** | gpt-5.3-codex | medium | Build, explore, execute OTAE cycles | Main agent (always active) |
| **R2-DEEP** | gpt-5.3-codex | high | FORCED/BATCH/BRAINSTORM reviews. Separate context = native BFP. | Major finding, stage transition, confidence explosion |
| **R2-INLINE** | gpt-5.3-codex-spark | medium | 7-point checklist per finding. Fast, lightweight. | Every finding formulation |
| **OBSERVER** | gpt-5.3-codex-spark | low | Read-only scans: orphans, desync, drift, naming | Every 5 cycles or on demand |
| **EXPLORER** | gpt-5.3-codex-spark | medium | Parallel tree branches, literature search | When branching exploration needed |

R2-DEEP as sub-agent means it has NO access to the researcher's reasoning. It sees ONLY claims and evidence. This is architecturally superior to same-agent role-play.

> Full config: `references/multi-agent-config.md`

---

## SESSION INITIALIZATION

### Banner
```
VIBE SCIENCE v5.5 ORO — Observe · Recall · Operate
SFI → BFP → R2 ENSEMBLE → V0/J0 → GATES (34 total, 8 schema-enforced)
SERENDIPITY RADAR · RESEARCH SPINE · OBSERVER · DQ1-DQ4
Detect · Persist · Demolish · Discover
```

### If `.vibe-science/` exists → RESUME
1. Read STATE.md, TREE-STATE.json, last 20 lines of PROGRESS.md
2. Read CLAIM-LEDGER.md frontmatter, SPINE.md last entry
3. Check pending: R2 demands, gate failures, debug nodes, Observer alerts
4. Resume from "Next Action" in STATE.md
5. Announce: "Resuming RQ-XXX, cycle N, stage S. Tree: X nodes (Y good). Next: [Z]."

### If `.vibe-science/` does NOT exist → INITIALIZE
1. → Phase 0: SCIENTIFIC BRAINSTORM (mandatory)
2. Gate B0 must PASS before any OTAE cycle
3. Create folder structure, populate STATE.md, PROGRESS.md, TREE-STATE.json, SPINE.md

---

## PHASE 0: SCIENTIFIC BRAINSTORM (Before Everything)

Not optional. Not skippable.

1. **UNDERSTAND** — Domain, interests, constraints (ask user, one question at a time)
2. **LANDSCAPE** — Rapid literature scan (last 3-5 years), field mapping, open debates
3. **GAPS** — Blue ocean hunting: cross-domain analogies, assumption reversal, scale shifting, contradiction hunting
4. **DATA** — Reality check: does data exist? Score DATA_AVAILABLE (0-1). LAW 1: `NO DATA = NO GO`
5. **HYPOTHESES** — Generate 3-5 testable, falsifiable hypotheses with null hypotheses and predictions
6. **TRIAGE** — Score: impact x feasibility x novelty x data readiness x serendipity potential (/25)
7. **R2 REVIEW** — Reviewer 2 challenges direction (BLOCKING: must WEAK_ACCEPT)
8. **COMMIT** — Lock RQ.md with: question, hypothesis, predictions, success/kill conditions

**Gate B0**: 3+ gaps with evidence, data confirmed (>= 0.5), falsifiable hypothesis, R2 WEAK_ACCEPT, user approved.

> Full protocol: `references/brainstorm-engine.md`

---

## OTAE-TREE LOOP

```
OBSERVE → THINK → ACT → EVALUATE → CHECKPOINT → CRYSTALLIZE → loop
```

Each cycle: ONE meaningful action. Each tree node = one OTAE cycle.

| Phase | Actions | v5.5 Insertions |
|-------|---------|-----------------|
| **OBSERVE** | Read STATE.md + TREE-STATE.json. Check pending gates, R2 demands, debug nodes. | Check Observer alerts. Check SPINE.md last entry. |
| **THINK** | Select next node or action. Plan: search, analyze, extract, compute, experiment. | **[DD0]** If new data: document all columns before use. **[L-1]** If new direction: literature pre-check. |
| **ACT** | Execute planned action. Produce artifacts. Debug if buggy (max 3, then prune). | **[DQ1]** After extraction. **[DQ2]** After training. **[DQ3]** After calibration. |
| **EVALUATE** | Extract claims → CLAIM-LEDGER. Score confidence. Parse metrics. Detect serendipity. | **[DQ4]** Every finding: numbers match source. **[R2 INLINE]** 7-point checklist per finding. |
| **CHECKPOINT** | Stage gate (S1-S5). R2 co-pilot (FORCED/BATCH/SHADOW). Serendipity radar. Stop conditions. | **[DC0]** At stage transitions: design compliance check. |
| **CRYSTALLIZE** | Update STATE.md, TREE-STATE.json, PROGRESS.md, CLAIM-LEDGER.md. | **[SPINE]** Mandatory structured entry. **[SSOT]** Run `sync_check.py`. |

### v5.0 FORCED Review Path
SFI injection → BFP Phase 1 (blind) → Full review Phase 2 → V0 gate → R3/J0 gate → Schema validation → Normal gate evaluation.

### Tree Structure
Tree modes: **LINEAR** (literature), **BRANCHING** (experiments), **HYBRID** (both). Tree search selects next node by confidence + metrics. Each node = one OTAE cycle.

> Full protocol: `references/loop-otae.md` · Tree search: `references/tree-search.md`

---

## 5-STAGE EXPERIMENT MANAGER

| Stage | Name | Goal | Max Iter | Gate |
|-------|------|------|----------|------|
| **1** | Preliminary Investigation | First working experiment or initial scan | 20 | S1: >= 1 good node |
| **2** | Hyperparameter Tuning | Optimize best approach | 12 | S2: metric improved, 2+ configs |
| **3** | Research Agenda | Explore creative variants | 12 | S3: all sub-experiments attempted |
| **4** | Ablation & Validation | Validate each component + multi-seed | 18 | S4: all ablated, contributions quantified |
| **5** | Synthesis & Review | Final R2 ensemble + conclusion | 5 | S5: R2 ACCEPT + D2 PASS + all VERIFIED |

> Full protocol: `references/experiment-manager.md`

---

## REVIEWER 2 CO-PILOT

4 domain-agnostic reviewers: **R2-Methods**, **R2-Stats**, **R2-Domain**, **R2-Engineering**.

7 activation modes:

| Mode | Trigger | Blocking? | Sub-Agent? |
|------|---------|-----------|------------|
| **BRAINSTORM** | Phase 0 completion | YES — must WEAK_ACCEPT | R2-DEEP |
| **FORCED** | Major finding, stage transition, pivot, confidence explosion (>0.30/2cyc) | YES | R2-DEEP (SFI+BFP+V0+J0) |
| **BATCH** | 3 minor findings accumulated | YES | R2-DEEP |
| **SHADOW** | Every 3 cycles automatically | NO — can ESCALATE to FORCED | R2-DEEP |
| **VETO** | R2 spots fatal flaw | YES — cannot be overridden except by human | R2-DEEP |
| **REDIRECT** | R2 identifies better direction | Soft — user chooses | R2-DEEP |
| **INLINE** | Every finding at formulation time | NO — advisory, but logged | R2-INLINE (spark) |

### R2 INLINE 7-Point Checklist (v5.5)
For every finding, before recording in CLAIM-LEDGER:
1. Numbers match source data? (SSOT)
2. Sample size adequate and reported?
3. Alternative explanations considered?
4. Prior art checked? (not rediscovering known result)
5. Confounder risk identified? (even if full harness not yet run)
6. Reproducible? (seed, parameters, data path documented)
7. Terminology consistent across documents?

### R2 Behavioral Requirements
- **ASSUME** every claim is wrong
- **SEARCH** for prior art, contradictions, artifacts
- **DEMAND** confounder harness for every quantitative claim (LAW 9)
- **REFUSE** premature closure — minimum 3 falsification attempts per major claim
- **ESCALATE**, never soften — each pass MORE demanding
- **SALVAGENTE**: When killing a claim, R2 MUST produce a serendipity seed

> Full ensemble protocol: `references/reviewer2-ensemble.md`

---

## SERENDIPITY RADAR

Three-part process: DETECTION → PERSISTENCE → VALIDATION.

**Detection** (every EVALUATE): 5 scans — anomalies, cross-branch patterns, contradictions, assumption drift, unexpected metrics.

**Response**: Score >= 10 → QUEUE. Score >= 15 → INTERRUPT (create serendipity node). Unaddressed flag after 5 cycles → ESCALATED.

**Salvagente** (v5.0): When R2 kills a claim (INSUFFICIENT/CONFOUNDED/PREMATURE), R2 MUST produce a serendipity seed (schema-validated).

> Full protocol: `references/serendipity-engine.md`

---

## GATES (34 Total)

| Category | Gates | Count | Schema-Enforced |
|----------|-------|-------|-----------------|
| Pipeline | G0-G6 | 7 | — |
| Literature | L-1, L0-L2 | 4 | L0 (source-validity) |
| Decision | D0-D2 | 3 | D1 (claim-promotion), D2 (rq-conclusion) |
| Tree | T0-T3 | 4 | — |
| Brainstorm | B0 | 1 | B0 (brainstorm-quality) |
| Stage | S1-S5 | 5 | S4 (stage4-exit), S5 (stage5-exit) |
| Data Quality | DQ1-DQ4 | 4 | — |
| Data Dictionary | DD0 | 1 | — |
| Design Compliance | DC0 | 1 | — |
| Vigilance | V0 | 1 | V0 (vigilance-check) |
| Judge | J0 | 1 | — |
| **Total** | | **34** | **8 schema-enforced** |

### Key Gate Summaries

- **G0**: Input sanity — data exists, format correct, no corruption
- **G1**: Schema compliance — data schema matches expectation
- **DQ1**: Post-extraction — no zero-variance, no leakage, cross-checks match
- **DQ2**: Post-training — outperforms baseline, no single-feature dominance, stable folds
- **DQ3**: Post-calibration — plausible range, not suspiciously perfect, adequate sample
- **DQ4**: Post-finding — numbers match source JSON, sample size reported, alternatives listed
- **DD0**: Data dictionary — all columns documented before use
- **DC0**: Design compliance — execution matches research design
- **L-1**: Literature pre-check — prior art searched before committing direction
- **V0**: Vigilance — SFI faults caught (RMS >= 0.80, FAR <= 0.10)
- **J0**: Judge — R3 meta-review score >= 12/18, no dimension = 0

> Full gate definitions: `references/gates-complete.md`
> DQ gate protocol: `references/dq-gates.md`

---

## ENFORCEMENT SCRIPTS

Python scripts for deterministic checks. Exit code 0 = PASS, non-zero = FAIL. Non-bypassable.

| Script | Purpose | CLI Example |
|--------|---------|-------------|
| `dq_gate.py` | DQ1-DQ4 data quality checks | `python scripts/dq_gate.py --gate DQ1 --data data.json` |
| `sync_check.py` | SSOT: numbers in markdown match JSON source | `python scripts/sync_check.py --json results.json --md FINDINGS.md` |
| `tree_health.py` | T3 gate: exploration ratio, good/total ratio | `python scripts/tree_health.py --tree TREE-STATE.json` |
| `gate_check.py` | Generic gate: validate artifact against JSON Schema | `python scripts/gate_check.py --gate B0 --artifact out.json --schema schemas/brainstorm-quality.schema.json` |
| `spine_entry.py` | Create/validate Research Spine entries | `python scripts/spine_entry.py --spine SPINE.md --type DATA_LOAD --action "Loaded dataset"` |
| `observer.py` | Observer checks: orphans, desync, drift, naming | `python scripts/observer.py --project .vibe-science/` |

All scripts: Python 3.8+, stdlib only (no external dependencies). Domain-configurable via `--config domain-config.yaml`.

---

## FOLDER STRUCTURE

```
.vibe-science/
├── STATE.md                    # Current state (max 100 lines, rewritten each cycle)
├── PROGRESS.md                 # Append-only log
├── CLAIM-LEDGER.md             # All claims with evidence + confidence
├── SPINE.md                    # Research Spine (structured logbook)
├── ASSUMPTION-REGISTER.md      # All assumptions with risk
├── SERENDIPITY.md              # Unexpected discovery log
├── TREE-STATE.json             # Full tree serialization
├── KNOWLEDGE/                  # Cross-RQ accumulated knowledge
└── RQ-001-[slug]/              # Per Research Question
    ├── RQ.md                   # Question, hypothesis, criteria, kill conditions
    ├── 00-brainstorm/          # Phase 0 outputs
    ├── 01-discovery/           # Literature phase
    ├── 02-analysis/            # Analysis phase
    ├── 03-data/                # Data extraction + validation
    ├── 04-validation/          # Numerical validation
    ├── 05-reviewer2/           # R2 reviews
    ├── 06-runs/                # Run bundles
    ├── 07-audit/               # Decision log + snapshots
    ├── 08-tree/                # Tree search artifacts
    └── 09-writeup/             # Paper drafting
```

---

## STOP CONDITIONS (checked every cycle)

1. **SUCCESS** — All criteria satisfied + all findings R2-approved → Stage 5 → Final R2 → EXIT
2. **NEGATIVE RESULT** — Hypothesis disproven or data unavailable → EXIT with documented negative
3. **SERENDIPITY PIVOT** — Score >= 15 → triage → create new RQ or queue
4. **DIMINISHING RETURNS** — cycles > 15 AND new_finding_rate < 1/3 → WARN → 3 targeted cycles or pivot
5. **DEAD END** — All avenues exhausted → EXIT with what was learned
6. **TREE COLLAPSE** — T3 fails AND no pending debug → R2 emergency review → pivot or conclude

---

## RESOURCE ROUTING TABLE

Load ONLY when needed. Never load all at once.

| Resource | Path | When to Load |
|----------|------|-------------|
| Constitution | `references/constitution.md` | Full law text needed |
| Brainstorm Engine | `references/brainstorm-engine.md` | Phase 0 |
| OTAE Loop | `references/loop-otae.md` | First cycle or complex routing |
| Tree Search | `references/tree-search.md` | THINK-experiment / tree init |
| Experiment Manager | `references/experiment-manager.md` | Stage transitions |
| Auto-Experiment | `references/auto-experiment.md` | ACT-experiment |
| Evidence Engine | `references/evidence-engine.md` | EVALUATE phase |
| R2 Ensemble | `references/reviewer2-ensemble.md` | CHECKPOINT-r2 |
| Search Protocol | `references/search-protocol.md` | ACT-search |
| Serendipity Engine | `references/serendipity-engine.md` | THINK-brainstorm / CHECKPOINT |
| Knowledge Base | `references/knowledge-base.md` | Session init / RQ conclusion |
| Data Extraction | `references/data-extraction.md` | ACT-extract |
| Writeup Engine | `references/writeup-engine.md` | Stage 5 |
| Audit | `references/audit-reproducibility.md` | Run manifests |
| All Gates | `references/gates-complete.md` | EVALUATE phase |
| DQ Gates | `references/dq-gates.md` | DQ1-DQ4 checks |
| Data Dictionary | `references/data-dictionary.md` | DD0 — new data |
| Design Compliance | `references/design-compliance.md` | DC0 — stage transitions |
| Literature Pre-Check | `references/literature-precheck.md` | L-1 — new directions |
| Research Spine | `references/research-spine.md` | CRYSTALLIZE |
| SSOT Protocol | `references/ssot.md` | CRYSTALLIZE |
| Silent Observer | `references/silent-observer.md` | Observer checks |
| Multi-Agent Config | `references/multi-agent-config.md` | Session init (Codex) |
| SFI Protocol | `references/seeded-fault-injection.md` | FORCED R2 reviews |
| Judge Agent | `references/judge-agent.md` | J0 gate |
| BFP Protocol | `references/blind-first-pass.md` | FORCED R2 reviews |
| Schema Validation | `references/schema-validation.md` | Gate validation |
| Circuit Breaker | `references/circuit-breaker.md` | R2 deadlocks |
| Node Schema | `assets/node-schema.md` | Tree mode init |
| Stage Prompts | `assets/stage-prompts.md` | Stage-specific generation |
| Metric Parser | `assets/metric-parser.md` | ACT-experiment |
| Templates | `assets/templates.md` | CRYSTALLIZE / session init |
| Domain Config | `assets/domain-config-example.yaml` | Domain-specific thresholds |
| Schemas | `assets/schemas/*.schema.json` | Gate validation |

---

## DEVIATION RULES

| Situation | Action |
|-----------|--------|
| Search query typo | AUTO-FIX silently, log |
| Missing database in search | ADD database, log, continue |
| Minor finding | ACCUMULATE — batch review at 3 |
| Major finding | GATE — stop → verification → R2 FORCED |
| Serendipity observation | LOG+TRIAGE → serendipity-engine |
| Cross-branch pattern | SERENDIPITY — score → if >= 12: create node |
| Dead end on current path | PIVOT — document → try alternative → escalate if none |
| No data available | **STOP** — LAW 1: NO DATA = NO GO |
| Confidence explosion (>0.30/2cyc) | **FORCED R2** — possible confirmation bias |
| Node buggy 3 times | **PRUNE** — mark pruned, select next |
| Tree health T3 fails | **EMERGENCY** — R2 review → strategy revision |
| Stage gate fails | **BLOCK** — fix, re-gate, advance |
| User corrects direction | **OBEY** — LAW 11: follow immediately, no argument |
| Architectural change needed | **ASK HUMAN** — strategic decisions need human input |
