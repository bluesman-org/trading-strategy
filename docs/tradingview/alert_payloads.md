# Alert Payload Reference

This document mirrors `specs/001-crypto-long-strategy/contracts/alerts.md` and will track the TradingView payload schema required for automation once the strategy is implemented.

## Payload Schema
```
pair|timeframe|signal|reason
```
- **pair**: TradingView symbol (e.g., `BTCEUR`).
- **timeframe**: Chart resolution string (e.g., `15`, `60`, `240`, `480`).
- **signal**: Event identifier such as `LONG_ENTRY`, `EXIT_TP`, `EXIT_SL`, `EXIT_DRAWDOWN`, `SUSPEND_ROLLING_LOSS`, `RESUME_TRADING`.
- **reason**: Deterministic code that explains why the alert fired (EMA+RSI confirmation, drawdown lock, etc.).

## Reason Codes (Initial Set)
| Code | When it is used |
|------|-----------------|
| `EMA_RSI_CONFIRMED` | EMA crossover + RSI check validated an entry setup. |
| `MACD_STOCH_CONFIRMED` | MACD histogram direction plus StochRSI threshold validated entry. |
| `BOLLINGER_FILTER_FAIL` | Optional Bollinger confirmation blocked the trade. |
| `ATR_STOP_HIT` | ATR + swing-low stop logic exited the position. |
| `TP_FIXED` | Fixed 3–6% take-profit target reached. |
| `MAX_DRAWDOWN` | 15% drawdown guard forced an exit/suspension. |
| `ROLLING_LOSS` | Rolling 14-day PnL monitor suspended trading. |
| `MANUAL_DISABLE` | User disabled the current pair input. |

> Constants `REASON_*` and `SIGNAL_*` declared in `pine/strategies/long_only_crypto_strategy.pine` mirror these codes so `alertPayload()` can emit deterministic messages.

## Next Steps
- Emit each runtime payload using `alert()` in `pine/strategies/long_only_crypto_strategy.pine`; `alertcondition()` is intended for indicators and not supported in Strategies for runtime payload delivery.
- Add payload examples after validation (see `tests/backtests/alert_latency.md` for latency evidence).
