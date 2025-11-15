# Implementation Plan: Long-Only Crypto Strategy

**Branch**: `001-crypto-long-strategy` | **Date**: 2025-11-15 | **Spec**: [spec.md](./spec.md)  
**Input**: Feature specification from `/specs/001-crypto-long-strategy/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

Deliver a Pine Script v6 strategy that trades long-only BTC-EUR (and cloneable pairs) across 15m–8h charts using a configurable EMA/MACD trend filter, RSI/StochRSI momentum gate, ATR+swing hybrid stop-loss, and fixed 3–6% take-profit. The script must auto-adjust tiers per timeframe, enforce a 15% drawdown ceiling, emit deterministic alerts, and remain deployable/testable entirely inside TradingView.

## Technical Context

<!--
  ACTION REQUIRED: Replace the content in this section with the technical details
  for the project. The structure here is presented in advisory capacity to guide
  the iteration process.
-->

**Language/Version**: Pine Script v6 (`//@version=6`)  
**Primary Dependencies**: TradingView built-ins (`ta.ema`, `ta.macd`, `ta.rsi`, `ta.stochrsi`, `ta.atr`, `ta.bb`, `request.security`, `alertcondition`, `strategy.*` APIs)  
**Storage**: In-memory TradingView series; no external persistence  
**Testing**: TradingView Strategy Tester (backtests + forward replay), alert log validation, exported performance reports compared to spec criteria  
**Target Platform**: TradingView web/desktop charts (spot BTC-EUR plus cloneable crypto pairs)
**Project Type**: Single-script TradingView strategy (pine/strategies directory)  
**Performance Goals**: <100 ms execution per bar, alerts issued within the closing bar, positive rolling 14-day PnL, zero drawdown-limit breaches  
**Constraints**: Long-only, EUR 1,000 fixed allocation, no pyramiding/averaging down, drawdown hard stop at 15%, take-profit 3–6% range, deterministic `barstate.isconfirmed` signals  
**Scale/Scope**: Supports BTC-EUR primary plus at least two additional crypto spot pairs using identical parameter sets across 15m, 30m, 1h, 4h, and 8h charts

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

1. **Pine Script v6 Compliance** – PASS. Strategy header will declare `//@version=6`; all helper functions call only documented APIs and cite manual URLs inside comments when using `request.security`, `ta.bb`, etc.
2. **Modular Strategy Functions** – PASS. Plan defines dedicated helpers: `entryCondition()`, `trendFilter()`, `momentumCheck()`, `stopLossLogic()`, `takeProfitLogic()`, `riskGuards()`, `alertPayload()`; each user story maps to these modules.
3. **Deterministic Signals & Alerts** – PASS. Signals evaluate inside `if barstate.isconfirmed` blocks; entry/exit alerts share the payload schema `pair|timeframe|signal|reason` and are tied to specific conditions to prevent duplicate firing.
4. **User Inputs & Cross-Pair Parity** – PASS. Input panel will expose timeframe tier selector, indicator choices (EMA vs MACD, RSI vs StochRSI), ATR multipliers, take-profit %, drawdown cap, Bollinger toggle, and per-pair enable with identical defaults/tooltips.
5. **Performance & Reliability Guardrails** – PASS. Indicator computations cached per bar (store ATR, EMAs, MACD, RSI, Bollinger) and reused; risk ledger uses lightweight accumulators; plan mandates profiler checks ensuring <100 ms/bar and alert latency auditing during QA.

## Project Structure

### Documentation (this feature)

```text
specs/001-crypto-long-strategy/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── tech-stack.md
├── contracts/
│   └── alerts.md
└── tasks.md (future /speckit.tasks)
```

### Source Code (repository root)
<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for this feature. Delete unused options and expand the chosen structure with
  real paths (e.g., apps/admin, packages/something). The delivered plan must
  not include Option labels.
-->

```text
pine/
└── strategies/
  └── long_only_crypto_strategy.pine   # Primary script

docs/
└── tradingview/
  └── alert_payloads.md                # Generated from contracts/alerts.md for publish-ready docs

tests/
└── backtests/
  └── btc_eur_regression.report        # Exported CSV/HTML from TradingView to validate SC metrics
```

**Structure Decision**: Single TradingView strategy kept in `pine/strategies/`; documentation and regression exports live under `docs/tradingview` and `tests/backtests` for versioning of alert schema and benchmark evidence.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| _None_ | – | – |

## Phase 0 – Research & KPI Alignment

**Goals**: Confirm indicator stack choices, timeframe tier parameters, and performance profiling approach so no `NEEDS CLARIFICATION` remains.

- Document decisions in `research.md` covering (1) EMA vs MACD selection logic per timeframe, (2) RSI vs StochRSI momentum thresholds, (3) alert payload schema and delivery guarantees.  
- Capture benchmarking approach: use TradingView profiler to verify <100 ms per bar and log methodology for SC-007/SC-008.  
- Identify data requirements (OHLCV, spread, ATR lookback) and gaps; ensure cloning to ETH-EUR/third pair has same parameters.

**Exit Criteria**: Research doc finalized, no outstanding “NEEDS CLARIFICATION”, indicator/alert decisions locked for design.

## Phase 1 – Design, Data Model & Contracts

**Goals**: Translate research into concrete architecture, data/state definitions, and user-facing documentation.

- `data-model.md`: Detail Strategy Profile, Risk Ledger, Trade Ticket, Market Snapshot, Alert Event structures (fields, relationships, state transitions).  
- `contracts/alerts.md`: Define `pair|timeframe|signal|reason` payload schema, reason codes, and alertcondition names.  
- `quickstart.md`: Provide step-by-step instructions for installing the script, configuring inputs (timeframe tier, trend filter, momentum option, ATR multiplier, Bollinger toggle), and validating alerts.  
- Update agent context via `.specify/scripts/powershell/update-agent-context.ps1 -AgentType copilot` so future commands know Pine Script v6 stack decisions.  
- Re-run Constitution Check after design to ensure helper breakdown, inputs, and performance plans still satisfy gates.

**Exit Criteria**: All design docs authored, agent context updated, Constitution Check reaffirmed (document in plan addendum if anything changes).

## Phase 2 – Implementation Strategy (Stop After Planning)

**Shared Foundations**

1. Create `pine/strategies/long_only_crypto_strategy.pine` with header, inputs, reusable helper stubs, and logging utilities.  
2. Build risk ledger module (rolling 14-day profitability, drawdown tracking, EUR 1,000 allocation enforcement) and attach to strategy performance variables.  
3. Implement caching layer for indicators (EMAs, MACD, RSI, StochRSI, ATR, Bollinger) to avoid redundant calls; include profiler instrumentation comments referencing Pine manual sections.
4. Build a timeframe-tier controller that reloads the preset matrix (see spec Timeframe Tier Presets) whenever chart resolution or the tier input changes, applying new parameters before the next confirmed candle.
5. When a timeframe change occurs while a trade is open, reuse the controller to recompute stops/alerts using the new tier, update exposure before the next confirmed candle, and log the adjustment without emitting duplicate entries.

**User Story 1 – BTC-EUR Long Execution (P1)**

- Implement `trendFilter()` supporting EMA crossover (default) and MACD fallback; add inputs for fast/slow EMA lengths and MACD signal smoothing.  
- Implement `momentumCheck()` with RSI default thresholds (e.g., >55) and StochRSI alternative; expose inputs with tooltips referencing manual sections.  
- Complete `entryCondition()` combining trend + momentum + Bollinger optional gate; ensure `barstate.isconfirmed`.  
- Implement `stopLossLogic()` (ATR multiple vs swing low) and `takeProfitLogic()` (fixed %).  
- Wire `strategy.entry`, `strategy.exit`, and alertconditions (“LONG_ENTRY”, “EXIT_TP”, “EXIT_SL”, “EXIT_DRAWDOWN”) with `pair|timeframe|signal|reason` payload.  
- Add backtest validation steps: TradingView Strategy Tester runs for 30 days BTC-EUR per timeframe tier, verifying SC-001–SC-003.

**User Story 2 – Cross-Pair Reuse (P2)**

- Externalize pair symbol via `input.symbol` + enable toggle; ensure cloning retains identical parameter defaults.  
- Validate spread/liquidity guard (reject if `request.security` spread > configured max).  
- Provide instructions in quickstart for duplicating chart setups; confirm logs differentiate pairs.  
- Run ETH-EUR + third pair regression exports showing parity with BTC-EUR config.

**User Story 3 – Rolling Performance Monitor (P3)**

- Implement rolling 14-day profitability accumulator (using `ta.change(strategy.netprofit)` + custom window).  
- Build drawdown monitor referencing starting capital constant; block new entries and log/alert `ROLLING_LOSS` when thresholds breached.  
- Surface monitoring info via `strategy.risk.allow_entry` toggles, `label.new` or `table` summaries, and ensure alerts fire when suspension toggles change.  
- Document recovery procedure in quickstart (i.e., re-enable once profitability positive) and ensure state persists per chart.

**Testing & Validation Plan**

- Backtest matrix: BTC-EUR across 15m, 1h, 4h; ETH-EUR 1h; third pair 4h. Capture profits/drawdown/export to `tests/backtests/`.  
- Alert QA: Configure webhook endpoints to log payloads; verify schema and <1 bar latency (SC-008).
- Alert Latency QA: Replay alert logs for BTC-EUR and ETH-EUR, compute timestamp deltas from candle close to receipt at the external webhook endpoint (operator notification), and document proof that alerts reach operators within 1 minute (SC-005) in `tests/backtests/alert_latency.md`. Explicitly measure end-to-end latency including external delivery, not just TradingView internal alert generation.
- Timeframe Tier QA: Script or manually trigger timeframe switches on live charts, confirm the preset parameters apply before the next candle via log output, and store evidence in `tests/backtests/timeframe_switch.log` (SC-004 / FR-002).
- Open-Trade Timeframe QA: Include at least one scenario where a position remains open during a timeframe switch, capture how stops and alerts are recalculated, and append the evidence to `tests/backtests/timeframe_switch.log`.
- Installation QA: Install the Pine script on 15m, 1h, and 4h charts, capture compilation/backtest screenshots or HTML exports, and archive them under `tests/backtests/install/` to satisfy SC-006.
- Rolling PnL QA: Run 90-day backtests per timeframe tier, compute rolling 14-day windows, and store the calculations in `tests/backtests/rolling_pnl_90d.log` to satisfy SC-001 while confirming FR-006 suspensions when windows turn non-positive.
- Performance QA: Use Pine profiler to capture execution time; include screenshot/reference in docs demonstrating <100 ms per bar.  
- Governance QA: Peer review checklist references constitution principles I–V plus stack alignment (trend, momentum, Bollinger).

**Handover**

- Once phases above are ready, invoke `/speckit.tasks` using this plan to derive executable tasks per user story and ensure independence per MVP slices.
