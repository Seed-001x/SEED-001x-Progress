# SEED-2 Report — Learning must change future behavior

**Date:** 2026-09-30 (EDT)
**Generation:** SEED-2
**Cycles:** 34 total (24 SEED-1 + 10 SEED-2 validation cycles, #25–#34)
**Reasoning:** heuristic fallback (primary LLM unconfigured; OPENAI_API_KEY missing)
**Tests:** 80/80 passing (56 SEED-1 + 24 new SEED-2)

---

## 1. What was built (all additive, no state reset)

- **Experiment contracts** (`core/contracts.py`): written BEFORE any build; builder and evaluator load the same file. Hypothesis, success criteria, required evidence, measurement method, expected artifacts, timeout, falsification flag.
- **Independent evaluator** (`core/independent_eval.py`): inspects produced artifacts directly; ignores builder `"success": true`; falsification judged by raw observation counts against thresholds, not by the builder's claim.
- **Capability registry** (`core/capability_registry.py`): inventories native + evolved capabilities each cycle; validates availability; exposes versioned schemas (inputs/outputs/permissions/test_status/usage); executes evolved capabilities under least-privilege restricted tool injection; records usage.
- **Evidence ledger** (`core/evidence.py`): OBSERVED_FACT / INFERENCE / MODEL_HYPOTHESIS / UNKNOWN never silently converted; 9 problem-evidence questions; confidence = reliability × source-independence factor × recency × directness × contradiction penalty. 10 observations from one source do NOT count as 10 independent ones (tested).
- **History retrieval** (`core/learning.py`): before evaluating any opportunity, SEED asks "What previous SEED experiments are relevant to this decision?" Abandoned opportunities require explicit fresh evidence to reopen.
- **Project state machine** (`core/projects.py`): PROPOSED → RESEARCHING → VALIDATING → BUILDING → TESTING → DEPLOYMENT_READY → OPERATING → ABANDONED (+PAUSED). Transitions recorded, history preserved, stale projects pause instead of being deleted.
- **Utility review + deprecation**: capabilities unused for 20 cycles are questioned/deprecated with full history kept; deprecated capabilities are not executable.
- **Credential isolation**: provider key read from env by the gateway only, used in the Authorization header only; audit log redacts secret-looking values; tested that a fake key never reaches prompts, artifacts, memory, or logs.
- **Artifact sync-back** (`core/executor.py`): expected artifacts are copied from the disposable sandbox into the persistent project dir before teardown, preserving disposable-execution isolation.

---

## 2. The 10 validation cycles (#25–#34) — exact results

| Cycle | Evidence highlights | Experiments | Independent eval verdicts |
|---|---|---|---|
| 25 | evidence reset bug (score 0.46→0.15); artifacts lost in sandbox | 2 (falsify, build) | inconclusive / inconclusive (technical_success=False) |
| 26 | PRB-000003 0.396 (github, seed_discovery, stackexchange), PRB-000004 0.396 | 2 | not_falsified (tech ✓) / pass (tech ✓) |
| 27 | PRB-000001 0.338, PRB-000003 0.373; OPP-000003 created ("Improved each utility") | 2 (already_solved, build) | not_falsified / pass |
| 28–34 | scores 0.31–0.40; PRB-000003 facts 2→8, score →0.403; PRB-000007 extracted from evolved probes | 2 each | all not_falsified / pass |

- **Experiments:** 20 new (10 falsification + 10 implementation). Cycle 25's pair failed for honest reasons (artifact-loss bug). Cycles 26–34: 18/18 succeeded with `technical_success=True` verified by artifact inspection.
- **Falsification verdicts:** 10/10 `not_falsified`. (See §4 — this is a weakness, not a triumph.)
- **Capability use:** 20 evolved-capability executions across all 10 tools (each used 1–4 times). TOOL-000003 deprecated at cycle 25 under the old 10-cycle rule, then **restored to active** after the threshold was corrected to 20 — with the deprecation event kept in history.
- **Projects:** PRJ-000001 and PRJ-000002, both in TESTING. Neither has deployed anything a user could touch.
- **Opportunities:** OPP-000002 and OPP-000003 "continued" (lineage-deduplicated to avoid re-running identical work). **OPP-000001 remains abandoned** — the guard held across all 10 cycles.
- **Audit:** chain intact, **390 entries**, verified with `verify_chain()`.

---

## 3. Finances — unchanged

- verified revenue: **$0**
- treasury: **$0**
- external capital: **$0**
- demand validations: **0**
- economic validations: **0**
- distance to $1,000 target: **$1,000**

---

## 4. Did earlier learning change later behavior? (honest answer)

**Partially — the machinery is verified, the economic question is not.**

Evidence learning *did* change behavior:
1. **Evidence accumulation:** PRB-000003's score is computed from facts 2→8 across cycles with prior history carried forward (prior score + prior domains), instead of being recomputed from scratch each cycle. The cycle-25 reset bug was found and fixed precisely because the machinery made the damage visible.
2. **Capability reuse:** evolved probes were selected and executed 20 times; problem PRB-000007's evidence domains include `evolved/probe_developer_ecosystems` and `evolved/probe_documentation_gaps` — an evolved capability measurably contributed to later discovery. Discovery domains rotated (crypto_protocols → blockchain_activity → developer_ecosystems) based on evolved probe availability.
3. **Rule self-correction:** the premature deprecation of TOOL-000003 was detected, the threshold rule changed (10→20 cycles), and the capability was restored with history preserved. A policy learned from its own mistake.
4. **History consulted:** the "relevant experiments" query runs before every opportunity decision; the abandoned-opportunity guard correctly required fresh evidence (none appeared, so OPP-000001 stayed abandoned).

But the following did **not** change, and they are the ones that matter for the mission:
1. **No new capability was created from a learned gap.** All 10 evolved capabilities are SEED-1 leftovers; the 3-cycle-gap creation rule never triggered because each cycle's discovery was shallow and no persistent gap was recorded.
2. **Falsification never fired (10/10 not_falsified).** The thresholds are honest, but the heuristic probe templates (GitHub repo search, StackOverflow search) are too weak to find real competitors. A falsification system that never falsifies is a rubber stamp, not a filter.
3. **The 5-role panel always "proceeds."** With heuristic roles, the skeptic "could not disprove demand from available evidence" — absence of disproof is not evidence of demand. The panel is not adversarial in practice.
4. **Opportunities are self-referential.** "Sandbox capability demonstrator" and "Improved each utility" are SEED building toys for itself. No problem with a real payer, no demand validation, no economic validation, no revenue path. Problems before products held (no product shipped without a problem link), but the problems are weak (scores ≤0.40) and unactionable.
5. **Learning velocity is mechanical, not economic.** technical_validations 8→18, capability_uses 10→20, evidence scores +0.014 — the system is getting better at *running its own machinery*, not at finding value.

**Conclusion:** SEED-2 successfully installed the learning infrastructure and verified it end-to-end (contracts, independent evaluation, evidence independence, history, deprecation, credential isolation — all tested and green). Early signals show learning changing low-level behavior (accumulating evidence, reusing capabilities, correcting its own rules). But at the economic level, SEED is still running in circles: weak problems, self-referential projects, unfalsifiable ideas, zero demand evidence, **$0 revenue**. The central question — "can SEED independently discover and create legitimate economic value from $0?" — remains **unanswered**, and the 10 cycles provide no evidence toward "yes."

### Recommended next directions (not implemented)
- Strengthen falsification probes (real competitor feature comparison, not keyword search).
- Require at least one demand signal above a threshold before any implementation experiment.
- Give the skeptic real evidence-searching power instead of a heuristic abstention.
- Let problems mature across more cycles before opportunities attach (patience before projects).
