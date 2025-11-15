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
