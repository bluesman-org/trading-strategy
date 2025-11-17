# Research Log

## Decision: Trend Filter Option Set (EMA vs MACD)
- **Rationale**: EMA fast/slow crossover reacts fastest on 15m–1h tiers, while MACD histogram direction provides smoother confirmation on 4h–8h charts. Offering both via input lets traders select per chart without changing risk math. Both indicators are fully documented in the Pine Script v6 reference (`ta.ema`, `ta.macd`).
- **Alternatives Considered**: Single fixed EMA pair (too rigid across timeframes); Hull MA (undocumented in Pine v6, higher computation cost); supertrend (requires ATR-based trailing logic that overlaps with stop system).

## Decision: Momentum Check (RSI vs StochRSI)
- **Rationale**: RSI threshold (>55 default) is lightweight, broadly understood, and matches constitution’s determinism rule. StochRSI adds faster oscillation checks for traders needing earlier confirmation. Both exist in `ta.rsi`/`ta.stochrsi` and are inexpensive if cached per bar.
- **Alternatives Considered**: CCI or Momentum indicator (less intuitive for ops, additional params); no momentum gate (risk of entering into exhaustion moves, violates stack requirement).

## Decision: Alert Payload Schema `pair|timeframe|signal|reason`
- **Rationale**: Single delimited string is easy to parse by webhooks, keeps payload under TradingView limits, and satisfies constitution requirement for pair/timeframe/reason transparency. Using reason codes (`SIGNAL_LONG`, `EXIT_TP`, `SUSPEND_DRAWDOWN`) enables automation and auditing.
- **Alternatives Considered**: JSON-like strings (more expressive but brittle in TradingView alert UI); multiple alerts per event (risks desync, slower to maintain).

## Decision: Backtest Window Interface (Default to 1 year)

- **Rationale**: TradingView free-tier users cannot restrict the historical range using native chart controls. To emulate a user-settable backtest period, we provide two complementary inputs: a numeric `Backtest duration (days)` which defaults to 365 (1 year) and a boolean `Use custom backtest date range` that enables explicit `Custom Backtest Start` and `Custom Backtest End` timestamps using `input.time`. This hybrid approach preserves dynamic defaults (relative window) while enabling precise custom ranges.
- **Alternatives Considered**: Only `input.time` for start/end — rejected because Pine's `input.time` requires compile-time default values making a user-friendly default (one year ago) difficult. Only `Backtest length (days)` with a smaller UI — rejected because users asked to be able to precisely set start/end windows.
- **Implementation Note**: When `Use custom backtest date range` is false the script computes the start range as `time - backtestDurationDays * 24*60*60*1000`. When enabled it uses `input.time` values and validates end >= start. New entries are blocked outside the window while exits still execute so positions can close normally.
