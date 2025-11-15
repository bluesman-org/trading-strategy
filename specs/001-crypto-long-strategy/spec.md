# Feature Specification: Long-Only Crypto Strategy

**Feature Branch**: `001-crypto-long-strategy`  
**Created**: 2025-11-15  
**Status**: Draft  
**Input**: User description: "Develop a long-only trading strategy for crypto spot markets. Primary target pair: BTC-EUR, but usable across other crypto pairs with the same parameters. Operates on timeframes between 15 minutes and 8 hours, with auto-adjusted parameters depending on timeframe. Strategy must remain profitable over a rolling two-week period and never exceed 15% drawdown of original capital. Fixed trade allocation: EUR 1,000 per position. Drawdown constraint: Max 15% of starting capital. Position sizing: No pyramiding, no averaging down. Stop-loss placement: Adaptive to volatility and structure. Take profit: Configurable input: 3 to 6 percent."

## Clarifications

### Session 2025-11-15

- Q: How should the strategy be packaged so users can install and test it inside TradingView? → A: Deliver it as a Pine Script v6 strategy so traders can add it to a chart and run TradingView’s native backtester directly.

### User Story 1 - Execute BTC-EUR Long Setup (Priority: P1)

A quantitative trader wants the system to produce valid BTC-EUR long entries on any supported timeframe (15m–8h) while respecting capital, drawdown, and risk limits.

**Why this priority**: Without a compliant BTC-EUR flow the strategy delivers no core value, so this slice must ship first.

**Independent Test**: Replay 30 days of BTC-EUR data, verify trades trigger with EUR 1,000 size, adaptive stops, configured take-profit, and never breach drawdown; this can be tested without other stories.

**Acceptance Scenarios**:

1. **Given** the strategy is enabled on the 1-hour BTC-EUR chart with default risk settings, **When** price action meets the long-only entry rules, **Then** a single EUR 1,000 position is opened with an adaptive stop-loss and take-profit between 3–6% set at order time.
2. **Given** cumulative closed and open PnL implies a 12% drawdown on starting capital, **When** the next entry signal appears, **Then** the trade is skipped and a drawdown warning is logged because the enforced limit is 15%.

---

### User Story 2 - Reuse Parameters on Other Pairs (Priority: P2)

A portfolio manager wants to run the exact strategy template on ETH-EUR or other liquid spot pairs without redefining rules, only selecting the new symbol.

**Why this priority**: Multi-pair portability is the main scaling lever after BTC-EUR is stable.

**Independent Test**: Point the configuration at historical ETH-EUR data; if it executes trades using the same risk profile and reports pair-specific performance the story passes without BTC-EUR involvement.

**Acceptance Scenarios**:

1. **Given** the strategy template is cloned for ETH-EUR on a 4-hour chart, **When** volatility conditions, drawdown, and capital checks pass, **Then** the system opens a single EUR 1,000 long ETH-EUR position with the same adaptive stop and configurable take-profit range as BTC-EUR.
2. **Given** a new symbol exceeds the maximum supported spread or liquidity threshold, **When** the user attempts to enable the strategy, **Then** activation is blocked with guidance to select a compliant market.

---

### User Story 3 - Monitor Rolling Performance (Priority: P3)

Risk management wants automated tracking of rolling two-week profitability so the strategy pauses itself when profitability falls to zero or negative.

**Why this priority**: Continuous validation protects capital once signals are live; it is less critical than generating trades but essential before production rollout.

**Independent Test**: Feed two weeks of trade outcomes; verify the system derives net PnL, compares it to zero, and toggles trading plus alerts with no dependency on other stories.

**Acceptance Scenarios**:

1. **Given** the rolling 14-day net PnL is positive, **When** the window is recomputed at end-of-day, **Then** trading remains enabled and the monitoring dashboard displays profitability metrics.
2. **Given** the rolling 14-day net PnL turns negative, **When** the monitor runs, **Then** new signals are suspended until profitability is restored and a notification is sent to operations.

---

### Edge Cases

- Timeframe switch while a trade is active; the strategy MUST keep the position open, immediately recalculate tier presets (EMA/ATR/momentum) for the new timeframe, and update stops/alerts before the next confirmed candle without issuing duplicate entries.
- Price gaps that jump past the adaptive stop before execution; system must record slippage and recalc drawdown instantly.
- Consecutive signals within the same candle; only the first compliant trade is allowed because pyramiding is forbidden.
- Exchange downtime or missing candles in the 14-day window; profitability calculations must fall back to the last complete data set and flag data quality issues.
- Volatility collapse that would place the adaptive stop closer than exchange minimum tick; enforce minimum stop distance to avoid immediate liquidation.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Strategy MUST support only long entries on spot markets within the 15-minute to 8-hour timeframe range and allow the user to choose one active timeframe per deployment.
- **FR-002**: System MUST auto-adjust signal sensitivity, stop distances, and lookback windows based on the selected timeframe using the parameter tiers defined in **Timeframe Tier Presets** while keeping the core indicator stack identical across pairs.
- **FR-003**: Each trade MUST allocate exactly EUR 1,000 notional, forbid pyramiding, and prevent averaging down by rejecting subsequent entries while a position in the same pair is open.
- **FR-004**: Stop-loss MUST be computed before order placement using a hybrid of ATR-based multiples plus the most recent swing low (buffered by at least one tick), retaining whichever distance is wider.
- **FR-005**: Take-profit MUST be configurable between 3% and 6% in 0.5% increments and applied at order time as a limit or conditional exit.
- **FR-006**: Rolling 14-day net PnL MUST remain strictly positive; otherwise the system suspends new trades, alerts operators, and logs the breach for review.
- **FR-007**: Strategy MUST track drawdown relative to original starting capital and hard-stop all new entries once cumulative drawdown reaches 15% until capital is replenished.
- **FR-008**: Users MUST be able to clone the BTC-EUR configuration to other approved pairs, inheriting identical risk settings while allowing pair-specific enable/disable switches.
- **FR-009**: Every trade execution, skip, or suspension event MUST be logged with timestamp, pair, timeframe, capital usage, stop distance, take-profit, and reason codes for auditability.
- **FR-010**: The system MUST expose a monitoring view or export showing current enablement state, time since last profitable window, open positions, and compliance with capital/drawdown caps.
- **FR-011**: Strategy logic MUST be implemented as a TradingView Pine Script v6 strategy that users can install, configure (timeframe tier, TP range, risk toggles), and backtest directly within TradingView without external tooling.
- **FR-012**: Every script MUST start with `//@version=6`, rely only on Pine Script v6-documented APIs, and include inline comments referencing the manual for non-trivial calls.
- **FR-013**: Core logic MUST be encapsulated inside reusable helpers such as `entryCondition()`, `stopLossLogic()`, `takeProfitLogic()`, risk filters, and alert payload builders so pairs can share identical behavior.
- **FR-014**: Signal evaluation MUST occur only on `barstate.isconfirmed`, and each entry/exit MUST emit an `alertcondition()` payload formatted as `pair|timeframe|signal|reason` for downstream automation.
- **FR-015**: All configurable parameters (timeframe tier, take-profit range, ATR multiples, enable toggles) MUST be exposed via `input.*` controls with descriptive titles, defaults, bounds/steps, and tooltips that remain identical across every supported pair.
- **FR-016**: Indicator calculations (ATR, EMAs, structure lookbacks) MUST be cached per bar to keep execution below 100 ms and avoid redundant `ta.*` calls even when the strategy runs across multiple pairs concurrently.
- **FR-017**: A trend filter MUST be active on every signal using either an EMA crossover (fast vs. slow tiers) or a MACD histogram direction check; only one of these is allowed per deployment and must be documented in inputs.
- **FR-018**: Momentum confirmation MUST rely on either RSI or StochRSI thresholds (user-selectable) and MUST pass before any long entry is emitted.
- **FR-019**: Optional Bollinger Band confirmation MUST be available; when enabled it requires price to be above the middle band on entries and relaxes automatically when disabled without altering other parameters.

#### Timeframe Tier Presets

| Tier Label | Chart Range | Fast EMA | Slow EMA | ATR Multiplier | MACD Hist Threshold | RSI Threshold | StochRSI %K/%D | Notes |
|------------|-------------|----------|----------|----------------|---------------------|---------------|----------------|-------|
| Tier 1 | 15m | 21 | 55 | 1.5× | > 0.08 | > 58 | 0.75 / 0.55 | Tightest volatility buffer for intraday scalps |
| Tier 2 | 30m | 34 | 89 | 1.75× | > 0.05 | > 56 | 0.7 / 0.5 | Balances responsiveness with noise filtering |
| Tier 3 | 1h | 55 | 144 | 2.0× | > 0.03 | > 55 | 0.65 / 0.45 | Default swing tier per spec clarifications |
| Tier 4 | 4h | 89 | 233 | 2.5× | > 0.02 | > 53 | 0.6 / 0.4 | Uses MACD bias more often on higher TF |
| Tier 5 | 8h | 144 | 377 | 3.0× | > 0.01 | > 52 | 0.55 / 0.35 | Widest stops to accommodate overnight moves |

Parameter changes must be applied before the next confirmed candle after a tier switch; helper inputs in the implementation need to read these presets verbatim.

### Key Entities *(include if feature involves data)*

- **Strategy Profile**: Stores pair, timeframe, parameter tier, take-profit setting, activation status, and links to performance metrics; one profile per symbol/timeframe combo.
- **Trade Ticket**: Represents each executed or skipped trade with entry timestamp, entry price, stop-loss distance, take-profit target, allocation, and compliance flags (drawdown, leverage, pyramiding checks).
- **Risk Ledger**: Aggregates realized/unrealized PnL, rolling 14-day profitability, drawdown versus starting capital, and suspension reasons shared by all Strategy Profiles.
- **Market Snapshot**: Captures volatility metrics (ATR), recent swing points, spread/liquidity checks per pair at signal time for reproducibility.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In backtests and live dry-runs, the BTC-EUR strategy produces a positive net PnL over every rolling 14-day window across the last 90 days of data, evidenced by logs stored in `tests/backtests/rolling_pnl_90d.log` or equivalent exports per timeframe tier.
- **SC-002**: No trade execution or simulated trade breaches the 15% drawdown ceiling; monitoring must show 0 violations during verification.
- **SC-003**: 100% of executed trades adhere to the EUR 1,000 allocation with exactly one concurrent position per pair (validated across 500 historical signals).
- **SC-004**: When the user switches timeframes, updated parameter tiers apply before the next candle closes 95% of the time, ensuring signals use the intended configuration.
- **SC-005**: Suspension alerts reach operators within 1 minute of detecting rolling-loss or drawdown breaches in 95% of monitored incidents.
- **SC-006**: TradingView installation must succeed 100% of the time in QA by adding the Pine Script strategy to a chart and running the native backtester with no compilation errors across at least three supported timeframes.
- **SC-007**: Pine Script runtime profiling shows median execution under 100 ms per bar and zero redundant `ta.*` computations in strategy profiler logs across BTC-EUR, ETH-EUR, and one additional pair.
- **SC-008**: Entry/exit alerts fire within the same closing bar 99% of the time with the `pair|timeframe|signal|reason` payload schema validated in QA alert logs.

## Assumptions

- Historical and live market data include reliable OHLCV, spread, and volume information for all supported spot pairs.
- Volatility for stop placement uses a 14-period ATR by default; teams can tune the multiple but not the indicator choice in this spec.
- Profitability calculations include realized and open PnL net of trading fees but ignore funding costs because the product is spot-only.
- Starting capital is defined once per deployment and refreshed manually when capital is topped up.
- Strategy execution venue enforces minimum tick sizes compatible with calculated stop distances; if not, the system rounds to the nearest valid tick.
