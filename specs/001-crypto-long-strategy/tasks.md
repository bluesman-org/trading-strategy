---
description: "Task plan for Long-Only Crypto Strategy implementation"
---

# Tasks: Long-Only Crypto Strategy

**Input**: Design documents from `/specs/001-crypto-long-strategy/`
**Prerequisites**: `plan.md`, `spec.md`; supplemental context from `research.md`, `data-model.md`, `contracts/alerts.md`, `quickstart.md`
**Tests**: Backtest exports and alert log checks are defined inside relevant story tasks (tests remain optional per spec but operational verification is required).
**Organization**: Tasks are grouped by user story so every slice stays independently implementable and testable.

> **Constitution Guardrails**: All code must declare `//@version=6`, encapsulate logic in helpers (`entryCondition()`, `trendFilter()`, `momentumCheck()`, `stopLossLogic()`, `takeProfitLogic()`, `riskGuards()`, `alertPayload()`), emit signals only on `barstate.isconfirmed`, expose consistent `input.*` controls, reuse cached indicators to stay under 100 ms per bar, and deliver deterministic `pair|timeframe|signal|reason` alerts.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Task can run in parallel (different files, zero unmet dependencies)
- **[Story]**: User story label (`US1`, `US2`, `US3`)
- Include exact file paths in each description

## Path Conventions

- Strategy source lives in `pine/strategies/long_only_crypto_strategy.pine`
- TradingView-facing docs live in `docs/tradingview/`
- Regression evidence lives in `tests/backtests/`
- Feature documentation remains under `specs/001-crypto-long-strategy/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Establish repo structure and base files so later phases have stable targets.

- [X] T001 Create `pine/strategies/`, `docs/tradingview/`, and `tests/backtests/` directories per plan structure in the repo root.
- [X] T002 Scaffold `pine/strategies/long_only_crypto_strategy.pine` with `//@version=6`, `strategy()` declaration, and stub helper signatures matching constitution modules.
- [X] T003 [P] Create `docs/tradingview/alert_payloads.md` and `tests/backtests/README.md` placeholders summarizing upcoming alert schema references and regression export locations.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Implement shared structures, inputs, and utilities that every story depends on.

- [X] T004 Define Strategy Profile, Trade Ticket, Risk Ledger, and Market Snapshot state containers plus initializers inside `pine/strategies/long_only_crypto_strategy.pine` per `data-model.md`.
- [X] T005 [P] Implement indicator caching helpers (EMA tiers, MACD histogram, RSI, StochRSI, ATR, Bollinger) inside `pine/strategies/long_only_crypto_strategy.pine` so each value is computed once per bar.
- [X] T006 Wire the unified input panel (timeframe tier presets, trend filter selector, momentum selector, ATR multiplier, take-profit slider, Bollinger toggle, drawdown cap, pair enable) in `pine/strategies/long_only_crypto_strategy.pine` with tooltips referencing the Pine v6 manual.
- [X] T007 [P] Add `alertPayload()` helper plus reason code constants in `pine/strategies/long_only_crypto_strategy.pine`, then mirror the schema and codes in `docs/tradingview/alert_payloads.md` based on `contracts/alerts.md`.
- [X] T008 Implement baseline `riskGuards()` scaffolding in `pine/strategies/long_only_crypto_strategy.pine` to track starting capital, EUR 1,000 allocation enforcement, and placeholders for drawdown/rolling stats.

**Checkpoint**: Shared helpers, inputs, and docs exist—user story work can begin.

---

## Phase 3: User Story 1 - Execute BTC-EUR Long Setup (Priority: P1) 🎯 MVP

**Goal**: Produce valid BTC-EUR long entries across 15m–8h tiers with adaptive stops, 3–6% TP, and drawdown-safe capital usage.

**Independent Test**: Replay 30 days of BTC-EUR data; confirm EUR 1,000 trades respect drawdown cap, apply adaptive stops/TP, and log deterministic alerts without relying on other stories.

### Implementation for User Story 1

- [X] T009 [P] [US1] Implement `trendFilter()` in `pine/strategies/long_only_crypto_strategy.pine` with EMA crossover defaults and MACD option tied to timeframe tiers.
- [X] T010 [P] [US1] Implement `momentumCheck()` in `pine/strategies/long_only_crypto_strategy.pine` to evaluate RSI or StochRSI thresholds with cached values.
- [X] T011 [US1] Compose `entryCondition()` in `pine/strategies/long_only_crypto_strategy.pine` that merges trend, momentum, and optional Bollinger confirmations, returns reason codes, and only fires on `barstate.isconfirmed`.
- [X] T012 [US1] Implement `stopLossLogic()` in `pine/strategies/long_only_crypto_strategy.pine` that fuses ATR multiples with swing-low buffers and enforces exchange tick minimums.
- [X] T013 [US1] Implement `takeProfitLogic()` in `pine/strategies/long_only_crypto_strategy.pine` for configurable 3–6% exits defined via the input slider.
- [X] T014 [US1] Wire `strategy.entry`/`strategy.exit` blocks inside `pine/strategies/long_only_crypto_strategy.pine` to allocate exactly EUR 1,000, forbid pyramiding, and preload stops/TP from the helper outputs.
- [X] T015 [US1] Extend `riskGuards()` in `pine/strategies/long_only_crypto_strategy.pine` to enforce the 15% drawdown ceiling and persist Trade Ticket logs with reasons per FR-007/FR-009.
 - [X] T016 [US1] Register runtime alerts in `pine/strategies/long_only_crypto_strategy.pine` that output the `pair|timeframe|signal|reason` payloads using `alert()` (strategies should call `alert()`; `alertcondition()` applies to indicators).
- [ ] T017 [P] [US1] Export a 30-day BTC-EUR Strategy Tester report to `tests/backtests/btc_eur_30d.report` and update validation steps within `specs/001-crypto-long-strategy/quickstart.md` with observed metrics.
- [ ] T032 [US1] Simulate timeframe tier switches (including an active-position scenario) in `pine/strategies/long_only_crypto_strategy.pine`, log preset reload + stop recalculations, and store the transcript in `tests/backtests/timeframe_switch.log` to prove parameters update before the next confirmed candle.

**Checkpoint**: BTC-EUR workflow is live, alert-complete, and independently testable.

---

## Phase 4: User Story 2 - Reuse Parameters on Other Pairs (Priority: P2)

**Goal**: Clone the BTC-EUR template to ETH-EUR and another approved pair with identical risk settings and guardrails.

**Independent Test**: Run historical ETH-EUR data with the shared template; strategy must trade with the same allocation, enforce spread/liquidity checks, and log pair-specific performance without BTC-EUR involvement.

### Implementation for User Story 2

- [ ] T018 [US2] Add pair enable toggles and symbol presets to `pine/strategies/long_only_crypto_strategy.pine`, ensuring Strategy Profiles are generated per symbol with identical defaults.
- [ ] T019 [P] [US2] Implement spread/liquidity guards in `pine/strategies/long_only_crypto_strategy.pine` using `request.security()` data to block non-compliant symbols before activation.
- [ ] T020 [US2] Extend Trade Ticket logging in `pine/strategies/long_only_crypto_strategy.pine` to capture pair/timeframe metadata and verify parity across Strategy Profiles.
- [ ] T021 [US2] Update cloning instructions and warnings in `specs/001-crypto-long-strategy/quickstart.md` so users can safely duplicate the script for ETH-EUR and a third pair.
- [ ] T022 [P] [US2] Export ETH-EUR and third-pair reports (`tests/backtests/eth_eur_30d.report`, `tests/backtests/alt_pair_30d.report`) demonstrating identical risk behavior.

**Checkpoint**: Multi-pair parity verified, each profile independently testable.

---

## Phase 5: User Story 3 - Monitor Rolling Performance (Priority: P3)

**Goal**: Track rolling 14-day profitability/drawdown and auto-suspend trading with deterministic alerts when thresholds fail.

**Independent Test**: Feed two weeks of trade outcomes; verify the rolling PnL monitor toggles trading and emits alerts without relying on other stories.

### Implementation for User Story 3

- [ ] T023 [US3] Implement rolling 14-day PnL accumulation inside `pine/strategies/long_only_crypto_strategy.pine`, storing metrics on the Risk Ledger structure.
 - [ ] T024 [US3] Extend `riskGuards()` in `pine/strategies/long_only_crypto_strategy.pine` to suspend/resume entries by setting `riskLedger.drawdownLock`/`riskLedger.rollingLossLock`. The script will gate `strategy.entry()` with those flags instead of using a non-existent boolean API.
- [ ] T025 [US3] Render an on-chart monitoring table/labels in `pine/strategies/long_only_crypto_strategy.pine` showing enable state, drawdown %, rolling PnL, and last alert reason per FR-010.
- [ ] T026 [US3] Add `SUSPEND_ROLLING_LOSS` and `RESUME_TRADING` alertconditions in `pine/strategies/long_only_crypto_strategy.pine` and extend `docs/tradingview/alert_payloads.md` with the monitoring payload examples.
- [ ] T027 [US3] Document suspension/recovery procedures plus monitoring interpretation guidance inside `specs/001-crypto-long-strategy/quickstart.md`.
- [ ] T028 [P] [US3] Capture monitoring validation evidence (alert log summary + notes) in `tests/backtests/rolling_monitoring_notes.md`.
- [ ] T033 [US3] Replay BTC-EUR and ETH-EUR alert logs, compute candle-close-to-alert latency, and document proof of ≤1 minute delivery inside `tests/backtests/alert_latency.md`.

**Checkpoint**: Rolling-performance safeguards operate independently and emit deterministic alerts.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Finalize performance, governance, and documentation across the feature.

- [ ] T029 [P] Profile `pine/strategies/long_only_crypto_strategy.pine` with the Pine profiler, then record <100 ms/bar metrics and remediation notes in `docs/tradingview/performance.md`.
- [ ] T030 Update `specs/001-crypto-long-strategy/checklists/requirements.md` with pass/fail evidence for FR/SC coverage derived from backtests and monitoring logs.
- [ ] T031 Run the full quickstart validation loop, capture final QA notes in `tests/backtests/final_validation.report`, and ensure `specs/001-crypto-long-strategy/quickstart.md` references the results.
- [ ] T034 [P] Execute installation/backtest QA on 15m, 1h, and 4h charts, store screenshots or HTML exports in `tests/backtests/install/`, and reference the artifacts in `docs/tradingview/performance.md` to satisfy SC-006.
- [ ] T035 Generate 90-day backtests per timeframe tier, compute rolling 14-day net PnL windows, and archive the calculations plus suspension events in `tests/backtests/rolling_pnl_90d.log` to satisfy SC-001 and cross-check FR-006 behavior.

---

## Dependencies & Execution Order

- **Phase 1 → Phase 2**: File structure and stubs must exist before shared helpers are coded.
- **Phase 2 → Stories**: Trend/momentum helpers, inputs, and alert payload scaffolding are prerequisites for every story.
- **User Story Order**: Default delivery is P1 → P2 → P3, but P2 and P3 can start in parallel once Phase 2 completes if their respective dependencies (e.g., US1 drawdown guard) are stubbed.
- **Exports & Docs**: Reporting/documentation tasks (T017, T022, T028, T031) depend on their story implementations but can run parallel with polish work once data is available.

## Parallel Execution Examples

**User Story 1 (P1)**
- T009 and T010 can run concurrently because trend and momentum helpers touch different logic blocks in `pine/strategies/long_only_crypto_strategy.pine`.
- T017 (BTC-EUR backtest export) can execute in parallel with T015–T016 once the entry/exit wiring compiles.

**User Story 2 (P2)**
- T019 (spread/liquidity guard) and T022 (regression exports) can proceed simultaneously after T018 establishes symbol presets.
- Documentation update T021 can happen in parallel with code task T020 once guard behavior is defined.

**User Story 3 (P3)**
- T023 (rolling PnL accumulator) and T025 (monitoring table) can run concurrently because they read the same data but write to different sections of `pine/strategies/long_only_crypto_strategy.pine`.
- T028 (validation evidence) can be captured in parallel with T026 once suspension/resume alerts function.

## Implementation Strategy

1. **MVP First**: Complete Phases 1–2, then finish User Story 1 (P1) to unlock a BTC-EUR-only deployable slice; validate via T017 before proceeding.
2. **Incremental Expansion**: Layer User Story 2 (multi-pair parity) and User Story 3 (rolling monitor) sequentially or in parallel, ensuring each phase’s checkpoint is met and independently testable.
3. **Evidence & Polish**: After all targeted stories pass their acceptance tests, execute Phase 6 to capture profiler data, finalize documentation, and record QA artifacts for governance.
