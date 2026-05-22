# Xiami M0-FIX-002 Re-Review Packet

**Project:** QuantLab  
**Repo:** `mikeaccio666-debug/xiami-quantlab`  
**Branch:** `main`  
**Previous commit reviewed:** `e921d5a`  
**Current commit:** `5843fa5` — "M0-FIX-002 tighten governance before M1"  
**Date:** 2026-05-22  
**Status:** Read-only review. No modifications. No commits pushed.  
**Produced by:** Xiami Main (orchestrator)  

---

## Repo Status

| Attribute | Value |
|-----------|-------|
| Branch | `main` |
| Latest commit | `5843fa5` |
| Commit message | "M0-FIX-002 tighten governance before M1" |
| Previous commit | `e921d5a` |
| Working tree | Clean (no uncommitted changes) |
| Files changed in M0-FIX-002 | 16 files, +2,066 / −44 lines |
| Tests | 1 placeholder test passing |
| Live trading code | None detected |
| Broker integration code | None detected |
| Credentials / secrets | None detected |

**Files reviewed in this re-review:**
- `AGENTS.md` (updated)
- `docs/00_project_memory/PROJECT_CHARTER.md` (updated)
- `docs/00_project_memory/RISK_POLICY.md` (updated)
- `docs/00_project_memory/CODE_REVIEW_POLICY.md` (updated)
- `docs/00_project_memory/DATA_POLICY.md` (updated)
- `docs/00_project_memory/BACKTEST_VALIDATION_POLICY.md` (new)
- `docs/00_project_memory/DATA_SOURCE_POLICY.md` (new)
- `docs/00_project_memory/EXECUTION_FIREWALL.md` (new)
- `docs/00_project_memory/EXTERNAL_AGENT_ACCESS_POLICY.md` (new)
- `docs/00_project_memory/PARAMETER_GOVERNANCE.md` (new)
- `docs/00_project_memory/CURRENT_STATE.md` (updated)
- `docs/00_project_memory/DECISION_LOG.md` (updated)
- `docs/00_project_memory/SESSION_HANDOFF.md` (updated)
- `docs/00_project_memory/OPEN_QUESTIONS.md` (updated)
- `docs/01_tickets/M1_BACKTEST_ENGINE.md` (updated)

---

## Summary

**M0-FIX-002 is a substantial improvement.** Codex added 5 new governance documents and tightened 6 existing ones. The changes directly address the majority of previously identified governance gaps:

- ✅ Experiment pre-registration — now mandated in `PARAMETER_GOVERNANCE.md`
- ✅ Negative result archiving — now required in `PARAMETER_GOVERNANCE.md`
- ✅ Point-in-time data — now mandated across `RISK_POLICY.md`, `DATA_POLICY.md`, `DATA_SOURCE_POLICY.md`, `BACKTEST_VALIDATION_POLICY.md`
- ✅ Multiple-testing controls — now in `PARAMETER_GOVERNANCE.md` with max-10-variations rule
- ✅ Adaptive strategy prohibition — now in `PARAMETER_GOVERNANCE.md`
- ✅ Data provenance — now in `DATA_POLICY.md` and `DATA_SOURCE_POLICY.md`
- ✅ Synthetic-fixture-only rule — now explicit in `RISK_POLICY.md`, `DATA_POLICY.md`, `DATA_SOURCE_POLICY.md`, `BACKTEST_VALIDATION_POLICY.md`, `M1_BACKTEST_ENGINE.md`
- ✅ Execution firewall — new `EXECUTION_FIREWALL.md` document
- ✅ External agent access policy — new `EXTERNAL_AGENT_ACCESS_POLICY.md`
- ✅ Neutral tone requirement — added to `AGENTS.md`

**However, M0 is still not ready to close.** The three hard review gates remain pending: Hermes re-review, ChatGPT review, and human owner approval. Additionally, several medium/low-severity quant research controls are still absent (minimum backtest length, transaction cost sensitivity analysis, regime-conditional reporting, Monte Carlo/bootstrap significance, cooling-off period, tail-risk metrics).

**Bottom line:** The governance foundation is now strong enough for M1 synthetic-fixture-only work, provided the remaining review gates are completed. The governance gaps that remain are acceptable to defer until later real-data research phases.

---

## Previously Identified Governance Gaps — Status Assessment

| # | Gap | Status | Evidence | Remaining Action |
|---|-----|--------|----------|------------------|
| 1 | **Experiment pre-registration** | ✅ **Resolved** | `PARAMETER_GOVERNANCE.md` § "Experiment Pre-Registration" requires hypothesis, data scope, universe, signal, parameters, benchmark, costs, and primary metrics before results are generated. | None. |
| 2 | **Point-in-time (PIT) data** | ✅ **Resolved** | `RISK_POLICY.md` § "Point-In-Time Data Mandate": "Serious research requires point-in-time data... must be labeled as non-PIT and invalid for serious research." Same language in `DATA_POLICY.md`, `DATA_SOURCE_POLICY.md`, `BACKTEST_VALIDATION_POLICY.md`. | None for M1 (synthetic fixtures only). For future real-data phases, ensure PIT data source is selected. |
| 3 | **Multiple testing / data snooping** | ✅ **Resolved** | `PARAMETER_GOVERNANCE.md` § "Multiple-Testing Warning" and § "Maximum Parameter Variations Before Review": max 10 variations before human review. "Performance improvements found after repeated testing must be treated as exploratory." | None for M1. For future real-data phases, consider adding formal statistical correction methods (Bonferroni, FDR, White's Reality Check). |
| 4 | **Out-of-sample (OOS) protocol** | ⚠️ **Partially resolved** | `RISK_POLICY.md` § "Future Research Validation": "Future research phases must define out-of-sample validation before strategy evaluation begins." `PARAMETER_GOVERNANCE.md` requires pre-registration of evaluation metrics. | For M1 (synthetic fixtures, no strategy research), OOS is not relevant. For M2/M3, a formal OOS protocol with embargo/purging and true holdout should be added. |
| 5 | **Negative result logging** | ✅ **Resolved** | `PARAMETER_GOVERNANCE.md` § "Negative Result Archiving": "Agents must not delete or hide results because they are unattractive." | Consider adding `status: completed_negative` / `abandoned` fields to `EXPERIMENT_REGISTRY.yaml` schema. |
| 6 | **Data provenance** | ✅ **Resolved** | `DATA_POLICY.md` § "Data Provenance Rules" requires source name, retrieval date, transformation steps, adjustment steps, validation steps, known gaps. `DATA_SOURCE_POLICY.md` requires provider, license, access method, revision policy, adjustment methodology. | None for M1. For real-data phases, add content-hash (SHA-256) requirement. |
| 7 | **Benchmark policy** | ⚠️ **Partially resolved** | `DATA_SOURCE_POLICY.md` § "Benchmark Documentation" requires benchmark name, source, data timing, adjustment method, composition assumptions, limitations. `AGENTS.md` backtest checklist requires benchmark documentation. | No approved-benchmark list yet. No prohibition on post-hoc benchmark swapping. No factor-model attribution requirement. Acceptable for M1; should be added before real-data research. |
| 8 | **Minimum backtest length / sample size** | ❌ **Unresolved** | Not mentioned in any document. | Add to `RISK_POLICY.md` before real-data research begins. Suggested: minimum 5 years or 1,000 trading days. |
| 9 | **Transaction cost sensitivity analysis** | ❌ **Unresolved** | `RISK_POLICY.md` requires transaction costs but does not require sensitivity analysis (e.g., report at 2x/5x baseline). | Add to `RISK_POLICY.md` before real-data research. Low priority for M1 since M1 uses synthetic fixtures. |
| 10 | **Regime-conditional reporting** | ❌ **Unresolved** | Not mentioned in any document. | Add to `RISK_POLICY.md` before real-data research. Low priority for M1. |
| 11 | **Turnover / capacity / capacity-adjusted returns** | ⚠️ **Partially resolved** | `RISK_POLICY.md` § "Portfolio Risk Placeholders" mentions high-turnover flagging but no capacity estimation. `BACKTEST_VALIDATION_POLICY.md` requires liquidity documentation. | Add capacity estimation requirement before real-data research. |
| 12 | **Adaptive strategy prohibition during backtest** | ✅ **Resolved** | `PARAMETER_GOVERNANCE.md` § "Prohibited Research Narratives": "Optimize parameters only to improve backtest performance" is prohibited. "Cherry-pick favorable results" and "Hide negative tests" are prohibited. | None. |
| 13 | **Monte Carlo / bootstrap significance testing** | ❌ **Unresolved** | Not mentioned. | Add to `RISK_POLICY.md` before real-data research. Medium priority. |
| 14 | **Cooling-off period between rejection and re-submission** | ❌ **Unresolved** | Not mentioned. | Add to `RISK_POLICY.md` or `PARAMETER_GOVERNANCE.md` before real-data research. Low priority for M1. |
| 15 | **Max drawdown / Calmar / Sortino reporting** | ❌ **Unresolved** | `AGENTS.md` backtest checklist does not mandate these metrics. `M1_BACKTEST_ENGINE.md` requires "performance metrics for engine validation" but does not specify which. | Add explicit metric requirements to `M1_BACKTEST_ENGINE.md` or `CODE_REVIEW_POLICY.md`. Low priority for M1. |

---

## Planner Assessment

### Is M0 Ready to Close?

**No.** M0-FIX-002 has significantly strengthened governance, but the formal review-and-approval gate required by `PROJECT_CHARTER.md` and `M0_BOOTSTRAP.md` is still unfulfilled.

| Ticket | Status | Assessment |
|--------|--------|------------|
| M0-T001 | Completed | Repo skeleton, placeholder package, pytest pass. |
| M0-T002 | Draft → Improved | `AGENTS.md` tightened with scope boundaries, neutral tone, execution prohibition. Not formally accepted. |
| M0-T003 | Draft → Improved | `PROJECT_CHARTER.md` expanded with milestones/phases distinction, execution non-goals. Not formally accepted. |
| M0-T004 | Draft → Improved | `RISK_POLICY.md` + new `BACKTEST_VALIDATION_POLICY.md` + `EXECUTION_FIREWALL.md`. Not formally accepted. |
| M0-T005 | Draft → Improved | `CODE_REVIEW_POLICY.md` + execution-adjacent review trigger. Not formally accepted. |
| M0-T006 | Draft → Improved | `DATA_POLICY.md` + new `DATA_SOURCE_POLICY.md`. Not formally accepted. |
| M0-T007 | Draft → Improved | Memory files updated for M0-FIX-002. Not formally accepted. |
| M0-T008 | **Not started** | **Blocker.** Hermes governance re-review mandatory per charter. |
| M0-T009 | **Not started** | **Blocker.** ChatGPT review mandatory, depends on Hermes. |
| M0-T010 | **Not started** | **Blocker.** Human owner approval is the only valid phase-change gate. |

### What Must Happen Before M1?

**Hard blockers (unchanged from previous review):**
1. Hermes completes red-team governance re-review of M0-FIX-002.
2. ChatGPT completes architecture/safety review of M0-FIX-002.
3. Critical/high findings from Hermes + ChatGPT resolved or accepted.
4. Human owner records "Go" decision in `DECISION_LOG.md`.

**Open questions that should resolve before M1-T001:**

| # | Question | Recommended Answer |
|---|----------|------------------|
| 1 | Named human owner | ck (酷酷) |
| 3 | Synthetic fixture edge cases | At minimum: random walk, trending, mean-reverting, split event, dividend event, delisting event, missing-data gap, holiday gap |
| 4 | Max single asset weight placeholder | 25% as placeholder (to be reviewed before real-data research) |
| 5 | Max turnover warning placeholder | 100% annualized as placeholder (to be reviewed before real-data research) |
| 8 | Transaction cost defaults for future phases | 5 bps all-in per side baseline; 10 bps conservative; zero-cost only as labeled diagnostic |
| 9 | Python version standardization | Pin `requires-python = ">=3.12"` in `pyproject.toml` |

### Is DECISION_LOG Ready for Human Go/No-Go?

**Partially.** The `DECISION_LOG.md` has a good M0-FIX-002 entry with decision, rationale, impact, files changed, test results, risks, and next actions. However, the entry does not explicitly list the unresolved governance gaps (gaps 8–15 above) as known limitations. Before the human Go/No-Go, the `DECISION_LOG.md` should note:
- Which previously identified gaps were resolved in M0-FIX-002.
- Which gaps remain unresolved but are acceptable for M1 synthetic-fixture-only work.
- Which gaps must be resolved before any real-data research phase.

---

## Research Assessment

### Are Quant Research Controls Sufficient for M1 Synthetic Fixtures?

**Yes, with caveats.**

M1 is explicitly limited to synthetic fixtures only. The purpose of M1 is to validate backtest engine mechanics (timing, accounting, costs, reproducibility), not to evaluate investment strategies. Under this constrained scope:

- **Experiment pre-registration** (`PARAMETER_GOVERNANCE.md`) is sufficient. Even synthetic-fixture experiments should be pre-registered to establish the habit.
- **PIT data mandate** is satisfied because synthetic fixtures are generated, not sourced, and their availability timing is controlled by the engine designer.
- **Multiple-testing controls** (max 10 variations, warning requirement) are sufficient for M1-scale work.
- **Negative result archiving** is sufficient for M1.
- **Execution firewall** (`EXECUTION_FIREWALL.md`) is robust and directly prevents the most dangerous scope creep.
- **Synthetic-fixture-only rule** is repeated across 5+ documents, making it very hard to accidentally violate.

### Which Controls Can Be Deferred to Later Real-Data Research Phases?

The following gaps are **acceptable to defer** because they only matter when evaluating real strategies on real data:

| Gap | Deferral Rationale |
|-----|-------------------|
| Minimum backtest length / sample size | Only relevant for statistical significance on real data. M1 tests engine mechanics on short synthetic series. |
| Transaction cost sensitivity analysis | Only relevant for strategy viability assessment. M1 needs one cost model, not sensitivity bands. |
| Regime-conditional reporting | Only relevant for strategy robustness assessment. M1 tests engine correctness, not strategy quality. |
| Turnover / capacity estimation | Only relevant for real-market liquidity constraints. Synthetic fixtures have infinite liquidity by design. |
| Monte Carlo / bootstrap significance | Only relevant for Sharpe ratio significance on real data. |
| Cooling-off period | Only relevant when agents are running many real-data experiments. M1 will have few synthetic tests. |
| Max drawdown / Calmar / Sortino | Only relevant for strategy risk assessment. M1 needs basic metrics (return, volatility, Sharpe) for engine validation. |
| Formal OOS protocol with embargo/purging | Only relevant for strategy evaluation. M1 engine tests do not involve train/test splits. |
| Approved benchmark list | Only relevant for real-data research. M1 benchmark is synthetic buy-and-hold of the synthetic universe. |
| Factor-model attribution | Only relevant for real-data alpha decomposition. |

---

## Code Assessment

### Is M1_BACKTEST_ENGINE.md Implementable?

**Yes, but still thin on interfaces and acceptance criteria.**

**Improvements in M0-FIX-002:**
- M1 data boundary is now explicit: synthetic fixtures only.
- M1 execution firewall is now explicit: no broker/execution/order modules.
- Required tests are now listed: data timing, no future access, no-look-ahead, transaction costs, portfolio accounting, reproducibility, report limitations.
- Ticket count expanded from 14 to 15 (added M1-T012: reproducibility tests).
- Forbidden scope is expanded and clearer.

**Remaining weaknesses:**
- **Still no dependency graph.** T001–T015 are listed as a flat sequence. In practice: T001/T002 block T006; T003 blocks T007; T004/T005 block T006; T006 blocks T008/T009.
- **Still no interface definitions.** No Python ABCs or dataclasses specified for `DataSource`, `Calendar`, `CostModel`, `Portfolio`, `Engine`, `Strategy`, `Reporter`.
- **"Simple daily backtest engine" is still undefined.** Vectorized vs event-driven choice affects memory, speed, reproducibility, and testability.
- **T010, T011, T012 are still separate test tickets.** Testing should be integrated into T001–T006 as acceptance criteria, not afterthoughts.
- **No architecture design document prerequisite.** Before T001 begins, the project needs an M1 Architecture Design Document.

### Does M1 Still Need an Architecture Design Document Before T001?

**Yes, strongly recommended.** The M1_BACKTEST_ENGINE.md is now a well-scoped requirement document, but it is not an architecture document. Before any code is written, ChatGPT (as System Architect) should produce an Architecture Design Document specifying:

1. **Engine pattern:** Daily vectorized loop (pandas) vs event-driven loop (Zipline-style).
2. **Module interfaces:** Python ABCs or dataclasses for core components.
3. **Data flow:** How synthetic OHLCV moves from validation → engine → accounting → metrics without mutation.
4. **Reproducibility contract:** Random seed handling, deterministic sorting, `run_id` or config hash.
5. **Extensibility hooks:** Where a future signal-based strategy would plug in.

This document should be a prerequisite to M1-T001 and should be reviewed by ChatGPT before implementation begins.

### Are There Missing Interfaces or Acceptance Criteria?

**Missing interfaces:**
- `DataLoader` / `DataSource` ABC
- `Calendar` / `TradingCalendar` ABC
- `CostModel` / `TransactionCostModel` ABC
- `SlippageModel` ABC (not mentioned in M1 tickets despite governance requiring it)
- `Portfolio` / `Account` dataclass
- `Engine` / `BacktestEngine` ABC
- `Strategy` / `Signal` ABC
- `Reporter` / `Metrics` ABC

**Missing acceptance criteria per ticket:**
- M1-T001: Should specify required columns, index type, and whether data is EOD or intraday.
- M1-T002: Should specify validation rules (no negative prices, no High < Low, no duplicate timestamps, monotonic index).
- M1-T003: Should specify which exchange calendar (NYSE) and which library (`exchange-calendars`, `pandas-market-calendars`, or custom).
- M1-T004: Should specify fixed-point arithmetic (`Decimal`) vs float, and cash conservation invariants.
- M1-T005: Should specify default cost assumptions for synthetic fixtures (e.g., 5 bps all-in).
- M1-T006: Should specify vectorized vs event-driven, and rebalance timing (close-to-close, close-to-open, etc.).
- M1-T007: Should specify benchmark construction (equal-weight of synthetic universe, or single index proxy).

---

## Remaining Blockers Before M1

### Hard Blockers (must complete before M1)

| # | Blocker | Owner | Status |
|---|---------|-------|--------|
| 1 | Hermes red-team governance re-review of M0-FIX-002 | Hermes | Pending |
| 2 | ChatGPT architecture/safety review of M0-FIX-002 | ChatGPT | Pending |
| 3 | Critical/high findings from Hermes + ChatGPT resolved or accepted | Codex + Owner | Pending |
| 4 | Human owner records "Go" decision in `DECISION_LOG.md` | ck | Pending |

### Soft Blockers (should complete before M1-T001 but not M1 approval)

| # | Blocker | Owner | Status |
|---|---------|-------|--------|
| 5 | M1 Architecture Design Document (engine pattern, interfaces, data flow, reproducibility contract) | ChatGPT | Pending |
| 6 | Open Questions 1, 3–5, 8–9 resolved and documented | Owner / Codex | Pending |
| 7 | `pyproject.toml` Python version pin (>=3.12) | Codex | Pending |
| 8 | `EXPERIMENT_REGISTRY.yaml` schema expanded to support negative/abandoned statuses | Codex | Pending |

### Acceptable to Defer to Future Phases

| # | Item | Earliest Phase |
|---|------|---------------|
| 9 | Minimum backtest length / sample size rule | Real-data research phase |
| 10 | Transaction cost sensitivity analysis (2x/5x bands) | Real-data research phase |
| 11 | Regime-conditional reporting (bull/bear, high/low vol) | Real-data research phase |
| 12 | Turnover / capacity estimation | Real-data research phase |
| 13 | Monte Carlo / bootstrap Sharpe significance | Real-data research phase |
| 14 | Cooling-off period between rejection and re-submission | Real-data research phase |
| 15 | Max drawdown / Calmar / Sortino mandatory reporting | Real-data research phase |
| 16 | Formal OOS protocol with embargo/purging/true holdout | Strategy evaluation phase |
| 17 | Approved benchmark list + factor-model attribution | Real-data research phase |
| 18 | Data content-hash (SHA-256) requirement | Real-data research phase |

---

## Questions for ChatGPT Review

1. **Architecture pattern:** Should the M1 backtest engine use a vectorized daily loop (pandas) or an event-driven loop (Zipline-style)? What are the tradeoffs for a small-team daily-frequency project that may eventually want to support signal-based strategies?

2. **Module interfaces:** Should M1 define Python ABCs (`DataLoader`, `Calendar`, `CostModel`, `Portfolio`, `Engine`, `Strategy`, `Reporter`) before T001 begins, or should interfaces emerge incrementally as tickets are implemented?

3. **Slippage model:** `RISK_POLICY.md` and `DATA_SOURCE_POLICY.md` require slippage assumptions, but M1-T005 is only "transaction cost model" and M1_BACKTEST_ENGINE.md does not mention slippage. Should M1 include a minimal slippage model, or defer to a future milestone?

4. **Reproducibility depth:** Is git commit hash + documented config sufficient for M1 reproducibility, or should M1 also pin dependencies in a lockfile (`requirements-lock.txt` or `poetry.lock`) and record synthetic fixture generation seeds?

5. **M1 test strategy:** Should T010/T011/T012 (test tickets) be merged into their parent tickets as acceptance criteria, or kept as standalone integration-test tickets?

6. **Python version:** Should `pyproject.toml` pin `requires-python = ">=3.12"` to match the local verification environment, or maintain `>=3.11` for broader compatibility?

7. **Experiment registry schema:** Does the current `EXPERIMENT_REGISTRY.yaml` need fields for `status: completed_negative` / `abandoned` / `superseded`, or is the current minimal schema sufficient for M1?

8. **Governance gap closure:** Are the remaining unresolved gaps (minimum backtest length, cost sensitivity, regime reporting, tail-risk metrics, cooling-off period) acceptable to defer until a future real-data research phase, or should any be added to M1 acceptance criteria?

9. **Portfolio risk placeholders:** Should the maximum single-asset weight (25%) and maximum turnover warning threshold (100% annualized) be formally adopted as M1 defaults, or remain as open placeholders?

10. **Hermes review integration:** Should Hermes review findings be committed to `docs/03_reviews/hermes/` before ChatGPT review, or can ChatGPT review in parallel?

---

*Disclaimer: This packet is a read-only coordination review. It does NOT approve Milestone 1, paper trading, live trading, or any investment strategy. All trading-related decisions require explicit human owner approval per `AGENTS.md` and `PROJECT_CHARTER.md`.*
