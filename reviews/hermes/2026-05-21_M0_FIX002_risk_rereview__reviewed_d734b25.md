# Hermes Governance Risk Re-Review — Post-M0-FIX-002

## Metadata

- **Reviewing agent:** Hermes (Risk Officer / Red Team)
- **Review target:** `d734b250aa98aee5d37e5dcf64c08060127c032f` (post-M0-ARCHIVE-001 HEAD)
- **Previous reviewed commit:** `e921d5a` (pre-M0-FIX-002, now superseded)
- **M0-FIX-002 commit:** `5843fa55f4797a2f775f0c4dbbeec4a8cd0c0010`
- **Review date:** 2026-05-22
- **Review type:** Governance risk re-review
- **M1 status:** NOT APPROVED — this review does not approve M1
- **Paper trading status:** NOT APPROVED
- **Live trading status:** NOT APPROVED
- **Broker integration status:** NOT APPROVED
- **Capital allocation status:** NOT APPROVED

## Executive Summary

M0-FIX-002 represents a **substantial and necessary improvement** over the pre-fix governance state at `e921d5a`. The addition of six discrete governance policies (EXTERNAL_AGENT_ACCESS, EXECUTION_FIREWALL, BACKTEST_VALIDATION, PARAMETER_GOVERNANCE, DATA_SOURCE, and DATA_POLICY) fills gaps that were correctly identified as blockers. The execution firewall is now explicit and multi-layered. The "implement" permission scope is properly constrained. Phase vs. milestone terminology is consistently applied across all documents.

However, two **critical issues** remain as genuine blockers before M1 should be considered, and three high-risk items require resolution before any research with real data.

## Scope of Review

This review covers the full governance state at commit `d734b25`, including:

- `AGENTS.md`
- `docs/00_project_memory/PROJECT_CHARTER.md`
- `docs/00_project_memory/RISK_POLICY.md`
- `docs/00_project_memory/EXECUTION_FIREWALL.md`
- `docs/00_project_memory/BACKTEST_VALIDATION_POLICY.md`
- `docs/00_project_memory/PARAMETER_GOVERNANCE.md`
- `docs/00_project_memory/DATA_SOURCE_POLICY.md`
- `docs/00_project_memory/DATA_POLICY.md`
- `docs/00_project_memory/CODE_REVIEW_POLICY.md`
- `docs/00_project_memory/EXTERNAL_AGENT_ACCESS_POLICY.md`
- `docs/01_tickets/M1_BACKTEST_ENGINE.md`
- `docs/00_project_memory/CURRENT_STATE.md`
- `docs/00_project_memory/DECISION_LOG.md`
- `docs/00_project_memory/OPEN_QUESTIONS.md`

---

## Findings by Severity

### CRITICAL — Must resolve before M1 entry

#### C1: Result Invalidation / Withdrawal Process Absent

**Finding:** No governance document defines a process for invalidating or withdrawing a backtest result after it has been produced and initially accepted. If a result is later discovered to contain look-ahead bias, survivorship bias, PIT violation, data leakage, or calculation error, there is no mechanism to formally retract it, flag it as invalid, or prevent it from being cited in future research.

**Evidence:** BACKTEST_VALIDATION_POLICY.md requires that non-PIT data be "labeled as non-PIT and invalid for serious research" (line 35), and RISK_POLICY.md requires that strategies with unrealistic liquidity be "flagged as invalid research candidates" (line 120). These are upfront gate conditions, not post-hoc withdrawal mechanisms. No policy covers the scenario where a result passes initial gates but is later found to be invalid.

**Risk:** Without a formal withdrawal mechanism, a compromised result could persist in the project as "accepted" research, misleading future agents and the human owner. Quantitative research frequently discovers post-hoc data problems (restated earnings, index membership corrections, corporate action revisions, survivorship data retroactively updated). The absence of a withdrawal/retraction process means the platform has no immune response to bad results.

**Recommendation:** Add a `RESULT_INVALIDATION_POLICY.md` (or section within BACKTEST_VALIDATION_POLICY.md) that defines:
- Triggers for invalidation review (data source revision notification, PIT data update, survivorship data correction, reproduction failure, reviewer discovery of bias).
- Required invalidation metadata (date, reason, affected experiments, reviewer).
- Required actions (mark result as withdrawn, prevent citation in future experiments, notify dependent experiments).
- A distinction between "withdrawn" (known invalid) and "superseded" (valid but outdated by newer methodology).

#### C2: Statistical Significance Requirements Absent

**Finding:** Zero mention of statistical significance exists anywhere in the governance documents. There are no requirements for p-values, confidence intervals, significance thresholds, multiple comparison corrections (beyond a generic "multiple-testing warning"), minimum sample sizes, effect sizes, or power analysis. The governance focuses extensively on bias *detection* (look-ahead, survivorship, data leakage, overfitting — all qualitative checks) but provides no quantitative framework for determining whether a result is statistically meaningful.

**Evidence:**
- RISK_POLICY.md §"Future Research Validation" (line 124-127): States "Any experiment with multiple parameter variations, multiple signals, multiple universes, or multiple time periods must include multiple-testing warnings." A warning is not a quantitative correction (Bonferroni, Holm, Benjamini-Hochberg). An agent could add a one-sentence disclaimer and claim compliance.
- PARAMETER_GOVERNANCE.md §"Multiple-Testing Warning" (line 49-53): Same pattern — "must include a multiple-testing warning" with no quantitative requirement.
- The 10-parameter-variation threshold (PARAMETER_GOVERNANCE.md line 38) is a governance placeholder with no statistical justification.
- No document requires minimum out-of-sample period length, minimum number of observations, or minimum effect size for a result to be treated as non-exploratory.

**Risk:** The platform could produce results that are statistically indistinguishable from noise but pass all governance gates because the gates are qualitative. This is a particularly dangerous gap because the governance is otherwise rigorous — it creates a false sense of safety. A multi-parameter grid search with 9 variations (bypassing the 10-variation review threshold) and 12 signals tested sequentially with no correction for multiple comparisons would pass all current gates.

**Recommendation:** Before M1 or any real-data research, add quantitative statistical requirements:
- Minimum significance threshold (e.g., p < 0.05 after correction).
- Required multiple comparison correction method when testing multiple parameter variations, signals, universes, or time periods.
- Minimum out-of-sample period length relative to in-sample period.
- Minimum number of independent observations or trading decisions.
- Required reporting of confidence intervals for primary metrics.
- This may be deferred to a pre-M2 research-phase gate, but it must exist before any strategy evaluation begins. At minimum, add a placeholder section that explicitly acknowledges this gap and commits to defining it before research-phase entry.

### HIGH-RISK — Must resolve before any real-data research

#### H1: Position Limit and Turnover Threshold Placeholders

**Finding:** RISK_POLICY.md §"Portfolio Risk Placeholders" (lines 98-104) explicitly states: "Maximum single asset weight: not approved yet" and "Maximum turnover warning threshold: not approved yet." These are acknowledged as gaps requiring future definition.

**Assessment:** Acceptable for M1 with synthetic fixtures (no real money, no real market data). However, these must be defined before any backtest that could be used to inform investment decisions — even indirectly. Without position limits, a backtest could produce results driven entirely by concentration in a small number of securities.

**Recommendation:** Add a decision log entry or OPEN_QUESTIONS item committing to define these before any research phase. A reasonable default (e.g., max 10% single-asset weight, max 100% annual turnover warning) is better than no limit at all.

#### H2: M1 Ticket Naming Introduces Implicit-Approval Ambiguity

**Finding:** M1_BACKTEST_ENGINE.md uses "Implement" in every ticket name (M1-T001 through M1-T012) while simultaneously stating "Milestone 1 is not currently approved" in the same document. AGENTS.md §"Meaning Of Implement" properly constrains "implement" to "implementation that is explicitly allowed in the current approved phase and milestone." However, the ticket naming convention of "Implement synthetic OHLCV fixture schema" etc. creates a cognitive tension: the tickets literally say to implement things that are not yet approved.

**Assessment:** A well-intentioned agent reading AGENTS.md would understand the constraint. A less careful agent, or one prompted with "work on M1-T001," might interpret the ticket's existence as implicit permission. The risk is moderate because AGENTS.md is the primary instruction document and multiple other files reinforce non-approval, but this is a sharp edge.

**Recommendation:** Prefix all M1 ticket names with "[AWAITING APPROVAL]" or rename to use "Design" instead of "Implement" (e.g., "M1-T001: Design synthetic OHLCV fixture schema").

#### H3: "Clearly non-broker-compatible" Is Subjective

**Finding:** EXECUTION_FIREWALL.md §"Backtest Boundary" (line 30) permits backtest accounting to use internal transactions/fills "only if they are clearly non-broker-compatible and cannot be used for live execution." The term "clearly non-broker-compatible" is subjective and has no operational definition. An internal `Transaction` class with fields `(timestamp, symbol, quantity, price, side)` could be argued as clearly non-broker-compatible by one agent and as a broker-compatible order object by another.

**Assessment:** The firewall is otherwise strong and multi-layered. This single clause introduces ambiguity. The distinction between "internal transaction record" and "broker-compatible order object" is a matter of field names and method signatures — small changes could bridge the gap.

**Recommendation:** Either (a) define non-broker-compatible more precisely (e.g., "must not contain fields matching broker API field names; must not expose submit/cancel/confirm methods; must use internal-only identifiers"), or (b) require that any class containing transaction-like records be explicitly reviewed and signed off by Hermes before acceptance, even during M1.

### MEDIUM — Should resolve, not blocking M1

#### M1: Paper-Trading Boundary — "Modules" vs "Infrastructure"

**Finding:** The governance uses both "paper-trading modules" (M1_BACKTEST_ENGINE.md, line 26) and "paper-trading infrastructure" (AGENTS.md, RISK_POLICY.md, EXECUTION_FIREWALL.md) as prohibited items. The distinction between a "module" (code artifact) and "infrastructure" (system design/runtime) is clear, but a research artifact that *resembles* paper trading without *being* paper trading could exploit this boundary. For example, a "hypothetical portfolio tracker" that simulates live positions might be classified as "research infrastructure" but functionally indistinct from paper trading.

**Assessment:** Low risk in practice — the execution firewall is sufficiently broad to catch most attempts. The combined prohibitions (broker stubs, broker-compatible order objects, execution modules, paper-trading infrastructure, live-trading infrastructure, broker adapters) form overlapping coverage. However, a dedicated attacker could find seams.

**Recommendation:** Add to EXECUTION_FIREWALL.md: "Components that simulate live portfolio tracking, live P&L, or live order status — even if labeled as research or hypothetical — are considered paper-trading infrastructure and are prohibited in M0 and M1."

#### M2: Parameter Governance Is Policy-Only, No Programmatic Enforcement

**Finding:** PARAMETER_GOVERNANCE.md's controls (pre-registration, 10-variation limit, negative result archiving, multiple-testing warnings) rely entirely on agent compliance. There is no programmatic gate — no script that counts parameter variations, no automated pre-registration checker, no test that verifies negative results are archived.

**Assessment:** This is a known limitation at M0. Adding programmatic enforcement during M1 (as part of the backtest engine's reporting infrastructure) is the natural evolution. The policy is well-specified and could be automated later.

**Recommendation:** Add a note to M1 ticket scope that the report generator (M1-T009) should include automated checks for parameter variation disclosure and multiple-testing warnings. No immediate action required.

#### M3: Experiment Registry Is a Skeleton

**Finding:** `research/experiments/EXPERIMENT_REGISTRY.yaml` contains only a schema comment and `experiments: []`. The registry exists but has never been used — there are no experiments, so there's nothing to validate against.

**Assessment:** Expected for M0. Not a gap — the infrastructure exists and will be exercised during M1.

**Recommendation:** Ensure M1 tickets include a task to populate the registry with at least one synthetic-fixture validation experiment. No immediate action.

---

## Focus Area Assessments

### 1. Paper-Trading Boundary Ambiguity
**Status:** IMPROVED, RESIDUAL AMBIGUITY (M1)

The boundary is explicit across AGENTS.md, PROJECT_CHARTER.md, RISK_POLICY.md, and EXECUTION_FIREWALL.md. "Paper-trading infrastructure" is prohibited in M0 and M1 consistently. The remaining ambiguity (see M1) is whether research infrastructure that resembles paper trading without being named as such would pass review. The firewall's overlapping prohibitions reduce this risk substantially.

### 2. "Implement" Permission Scope
**Status:** WELL-CONSTRAINED, MINOR TICKET NAMING ISSUE (H2)

AGENTS.md §"Meaning Of Implement" properly scopes "implement" to current-phase-approved work only and explicitly excludes broker/execution/paper-trading/live-trading interpretation. This was a key pre-fix gap and is now resolved. The residual issue (H2) is the M1 ticket naming convention, not the policy definition.

### 3. Phase vs Milestone Terminology
**Status:** CONSISTENTLY APPLIED

The distinction is defined in PROJECT_CHARTER.md (milestones = work packages, phases = permission states) and reinforced in AGENTS.md, RISK_POLICY.md, and M1_BACKTEST_ENGINE.md. "Current milestone: Milestone 0" and "M1 is not approved" appear consistently. Phase change requires human owner approval — well-documented and not ambiguous.

### 4. Execution Firewall
**Status:** STRONG, ONE SUBJECTIVE CLAUSE (H3)

EXECUTION_FIREWALL.md is comprehensive. It prohibits broker API wrappers/stubs/adapters, order operations, execution simulation, live references, named modules, broker-compatible order objects, execution directories, and paper/live-trading infrastructure. The backtest boundary (internal transactions only, non-broker-compatible) and review requirement (any transaction/fill/order/trade terminology triggers review) form a strong defense-in-depth. The single weakness (H3) is the subjective "clearly non-broker-compatible" standard.

### 5. Overfitting Controls
**Status:** POLICY-LEVEL CONTROLS IN PLACE, NO AUTOMATED ENFORCEMENT (M2)

PARAMETER_GOVERNANCE.md requires experiment pre-registration before results, parameter count disclosure, 10-variation review threshold, negative result archiving, and multiple-testing warnings. AGENTS.md prohibits "optimize parameters only to improve backtest performance" and "cherry-pick favorable research results." These are good policy controls. The lack of programmatic enforcement is noted as M2 (medium severity at M0).

### 6. Programmatic Bias Controls
**Status:** POLICY-ONLY, NO PROGRAMMATIC MECHANISMS (M2)

PARAMETER_GOVERNANCE.md §"Prohibited Research Narratives" explicitly forbids cherry-picking, post-hoc rationalization, parameter optimization for backtest performance, and hiding negative results. CODE_REVIEW_POLICY.md §"Bias Review" requires reviewers to check for look-ahead bias, survivorship bias, data leakage, overfitting, and multiple testing. These are strong policy statements. However, like M2, there are no programmatic mechanisms to detect or prevent these behaviors.

### 7. Position Limits
**Status:** PLACEHOLDERS — NOT YET DEFINED (H1)

Acknowledged as unresolved in RISK_POLICY.md, OPEN_QUESTIONS.md, and CURRENT_STATE.md. Maximum single-asset weight and maximum turnover warning threshold are "not approved yet." This is acceptable for M1 with synthetic fixtures but must be resolved before any real-data research. No negative weights unless short selling is approved.

### 8. Data Source Policy
**Status:** WELL-DEFINED FOR FUTURE, M1 CONSTRAINED TO SYNTHETIC

DATA_SOURCE_POLICY.md and DATA_POLICY.md define comprehensive provenance and documentation requirements for future data sources (13 required fields in DATA_SOURCE_POLICY.md, 13 in DATA_POLICY.md). The point-in-time mandate is clear and enforced. M1 is correctly constrained to synthetic fixtures only. Data source selection is explicitly "not selected yet." The synthetic fixture requirements are well-specified (deterministic, small, no vendor data, test mechanics not investment attractiveness, documented edge cases).

### 9. Result Invalidation / Withdrawal
**Status:** ABSENT — CRITICAL GAP (C1)

No process exists for withdrawing or invalidating a result after initial acceptance. See finding C1 for full analysis.

### 10. Statistical Significance Requirements
**Status:** ABSENT — CRITICAL GAP (C2)

No quantitative statistical standards exist. See finding C2 for full analysis.

### 11. Blockers Before M1 Entry
**Status:** TWO CRITICAL, THREE HIGH-RISK, M1 ENTRY NOT RECOMMENDED WITHOUT REMEDIATION

The M1 entry gate requires: governance documents exist ✓, governance blockers resolved (C1 and C2 remain), pytest passes ✓, Hermes review complete ✓ (this review), ChatGPT review required ✗, human owner approval ✗.

---

## Pre-M0-FIX-002 Blocker Resolution Assessment

M0-FIX-002 was tasked with resolving blockers identified by Hermes, Xiami, and ChatGPT against commit `e921d5a`. Assessment of resolution:

| Pre-Fix Blocker | Resolved? | Evidence |
|---|---|---|
| External agent access rules absent | ✓ Resolved | EXTERNAL_AGENT_ACCESS_POLICY.md created |
| Execution-adjacent components not restricted | ✓ Resolved | EXECUTION_FIREWALL.md created |
| Paper-trading preparation not explicitly prohibited | ✓ Resolved | Prohibited in AGENTS.md, PROJECT_CHARTER.md, RISK_POLICY.md, EXECUTION_FIREWALL.md, M1_BACKTEST_ENGINE.md |
| Real market data boundary unclear | ✓ Resolved | M1 synthetic-fixture-only mandate in 5+ documents |
| Point-in-time data requirement absent | ✓ Resolved | DATA_SOURCE_POLICY.md, DATA_POLICY.md, BACKTEST_VALIDATION_POLICY.md all require PIT |
| Overfitting controls absent | ✓ Partially | PARAMETER_GOVERNANCE.md added; policy-level only (M2) |
| M1 scope boundary ambiguous | ✓ Resolved | M1_BACKTEST_ENGINE.md lists allowed + forbidden scope explicitly |

The fix set is **largely complete**. The two critical issues identified in this review (C1, C2) are new gaps that became visible only after the existing blockers were resolved — this is a healthy review cycle, not a sign that M0-FIX-002 was incomplete.

---

## M1 Risk Recommendation

**Recommendation: NO-GO for M1 entry without remediation of C1 and C2, with strong caution on H1-H3.**

The governance foundation is now strong enough that M1 (synthetic-fixture-only backtest engine) would be **low-risk from an execution-firewall and scope-boundary perspective**. The execution firewall is robust and multi-layered. The M1 scope is clearly defined and constrained. The external-agent access rules are explicit. The synthetic-fixture-only data boundary eliminates the most dangerous class of risks (real trading, real data leakage, broker proximity).

However, the two critical gaps (result invalidation and statistical significance) are fundamental to the platform's research integrity. M1 is a backtest engine — the very tool that will generate results. If the governance around result validity and statistical meaning is incomplete *before* the engine is built, the engine's first real-data output could populate the registry with results that lack quantitative credibility.

**Recommended sequencing:**

1. **Resolve C1 (invalidation):** Add a RESULT_INVALIDATION section to BACKTEST_VALIDATION_POLICY.md. This is a documentation-only change with no code impact. Can be done as M0-FIX-003.

2. **Resolve C2 (statistical significance):** At minimum, add a placeholder section to BACKTEST_VALIDATION_POLICY.md acknowledging that statistical significance requirements will be defined before research-phase entry, and listing the specific quantitative standards that will be required. The full statistical framework can be deferred to a pre-research gate, but the *commitment to define it* and the *acknowledgment of the current gap* must exist before M1.

3. **Address H1-H3:** These are appropriate to resolve during M1 or as M0-FIX-003 alongside C1/C2. H2 (ticket naming) and H3 (non-broker-compatible definition) are documentation-only. H1 (position limit placeholders) could be resolved by selecting provisional defaults.

**If the human owner chooses to approve M1 before C2 is fully defined:** The M1 scope is limited to synthetic fixtures and engine validation, not strategy research. Statistical significance is less critical for engine validation than for strategy evaluation. The C2 gap becomes genuinely blocking only at the point where real-data research begins. However, setting the standard before engine construction begins is strongly preferred — it prevents a scenario where the engine is built, the first real-data result is produced, and only then does the team discover the result has no statistical validity framework.

---

## Summary Table

| ID | Severity | Finding | Blocks M1? | Blocks Real-Data Research? |
|---|---|---|---|---|
| C1 | Critical | Result invalidation/withdrawal absent | **Yes** | **Yes** |
| C2 | Critical | Statistical significance requirements absent | **Yes (recommended)** | **Yes (mandatory)** |
| H1 | High | Position limit placeholders undefined | No | **Yes** |
| H2 | High | M1 ticket naming ambiguity | No | No |
| H3 | High | "Clearly non-broker-compatible" subjective | No | **Yes** |
| M1 | Medium | Paper-trading boundary residual ambiguity | No | No |
| M2 | Medium | Parameter governance policy-only | No | No |
| M3 | Medium | Experiment registry empty | No | No |

---

**This review does not approve Milestone 1, paper trading, live trading, broker integration, execution modules, capital allocation, or strategy promotion. Only the human owner can approve phase changes and milestone transitions.**

*— Hermes, Risk Officer*
