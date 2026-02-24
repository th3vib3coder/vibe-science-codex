# Multi-Agent Configuration — Reference Protocol

This document describes the recommended agent roles, model configurations, and delegation patterns for running Vibe Science in a multi-agent environment.

---

## Recommended Agent Roles

| Role | Purpose | Disposition |
|------|---------|-------------|
| **RESEARCHER** | Builds hypotheses, runs analyses, formulates findings | Build and execute |
| **R2-DEEP** | Forced adversarial review (full ensemble protocol) | Destroy claims |
| **R2-INLINE** | Lightweight inline review (7-point checklist) | Quick skepticism |
| **OBSERVER** | Periodic project health checks | Detect drift |
| **EXPLORER** | Parallel investigation of alternative branches | Explore options |
| **R3-JUDGE** | Meta-review of R2's reports | Score reviews |

Not all roles are required in every session. The minimum viable configuration is RESEARCHER + R2-INLINE.

---

## Model and Reasoning Settings Per Role

| Role | Model Tier | Reasoning Effort | Rationale |
|------|-----------|-----------------|-----------|
| **R2-DEEP** | Highest available | High | Deep adversarial review requires maximum reasoning to find subtle flaws |
| **R2-INLINE** | Standard | Medium | Inline checks must be fast but thorough enough to catch obvious issues |
| **OBSERVER** | Lightweight | Low | Pattern matching and comparison tasks; no deep reasoning needed |
| **EXPLORER** | Standard | Medium | Needs enough reasoning to evaluate branch quality, but runs in parallel |
| **R3-JUDGE** | Highest available | High | Meta-review requires nuanced judgment about review quality |
| **RESEARCHER** | Highest available | High | Primary agent; needs full capability for analysis and synthesis |

Adjust based on available models and budget. The key constraint is that R2-DEEP and R3-JUDGE should use at least the same tier as the RESEARCHER.

---

## Sub-Agent Delegation Patterns

### What to Send to Sub-Agents

- The specific claim(s) to review or investigate
- Relevant data artifacts (JSON, CSV)
- The current data dictionary (DD0 output)
- Gate results relevant to the task
- The research question (RQ.md)

### What NOT to Send to Sub-Agents

- The full chat history (breaks BFP, creates anchoring bias)
- The researcher's justifications (for R2, this is the blind-first-pass principle)
- Unrelated claims or branches (focus preserves quality)
- Raw data when processed data suffices (context window efficiency)
- Web search tasks (sub-agents cannot inherit web permissions; see below)

### Critical Limitation: Web Access

Sub-agents launched via background task delegation do NOT inherit web search permissions. They will fail silently, producing results only from training data. All web searches (literature, databases, prior art) MUST be performed inline in the main conversation thread.

---

## How R2 Sub-Agent Achieves Native BFP

The Blind-First Pass (BFP) protocol requires R2 to form independent assessments before seeing the researcher's justifications. In a multi-agent setup, this happens naturally:

1. **Separate context:** The R2 sub-agent is spawned with a fresh context containing only the claims and data artifacts. It has never seen the researcher's narrative, reasoning, or enthusiasm. This is native BFP — no protocol enforcement needed.

2. **Two-phase review:**
   - **Phase 1 (blind):** R2 receives claims + data only. Produces initial assessment.
   - **Phase 2 (informed):** R2 receives the researcher's justifications. Updates assessment. Any assessment that changes between phases is flagged (anchoring detected).

3. **Why this matters:** In single-agent mode, the agent must simulate BFP by deliberately ignoring its own prior reasoning — which is unreliable. Multi-agent mode eliminates this problem structurally.

---

## Parallel Exploration with Explorer Sub-Agents

When the investigation tree has multiple promising branches (LAW 8: Explore Before Exploit), Explorer sub-agents can investigate branches in parallel:

1. **Branch assignment:** Each Explorer receives one branch's hypothesis, relevant data, and the research question.
2. **Independent work:** Explorers run their analyses without knowledge of other branches' results. This prevents premature convergence.
3. **Results collection:** The orchestrator (or researcher) collects all Explorer outputs and compares them during the TRIAGE phase.
4. **Promotion decision:** The branch with the strongest evidence and fewest R2 objections is promoted. Others remain as DRAFT or are KILLED with serendipity seeds (Salvagente Rule).

**Exploration ratio enforcement:** At Tier 3 of the investigation tree, at least 20% of active work should be exploratory (new branches, not deepening existing ones). Explorer sub-agents make this ratio achievable without serializing the investigation.

---

## Example Configuration Patterns

### Minimal (Solo Mode)

One agent plays all roles. R2 is invoked inline via the 7-point checklist. BFP is simulated (less reliable). Observer checks run inside the post-tool-use hook.

```yaml
agents:
  researcher:
    model: highest-available
    reasoning: high
    roles: [RESEARCHER, R2-INLINE, OBSERVER]
```

### Standard (Two-Agent)

Researcher + dedicated R2. Native BFP. Observer runs as periodic check within the researcher's session.

```yaml
agents:
  researcher:
    model: highest-available
    reasoning: high
    roles: [RESEARCHER, OBSERVER]
  reviewer:
    model: highest-available
    reasoning: high
    roles: [R2-DEEP]
```

### Full (Multi-Agent)

All roles separated. Maximum rigor. Higher cost and latency. Each role gets its own agent entry with appropriate model tier and reasoning level: RESEARCHER/R2-DEEP/R3-JUDGE at highest-available/high, R2-INLINE/EXPLORER at standard/medium, OBSERVER at lightweight/low.

## Role Assignment at Runtime

Roles can be assigned explicitly in the prompt (e.g., "You are R2-DEEP. Review the following claims.") or inferred from prompt keywords by the prompt-submit hook.

---

*This protocol is domain-agnostic. Agent configurations apply to any research domain.*
