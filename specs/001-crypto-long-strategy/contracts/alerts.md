# Alert Contract

## Payload Schema
```
pair|timeframe|signal|reason
```
- **pair**: TradingView symbol (e.g., `BTCEUR`)  
- **timeframe**: Chart resolution (e.g., `15`, `60`, `240`, `480`).  
- **signal**: `LONG_ENTRY`, `EXIT_TP`, `EXIT_SL`, `EXIT_DRAWDOWN`, `EXIT_MANUAL`, `SUSPEND_ROLLING_LOSS`, `RESUME_TRADING`.  
- **reason**: Coded justification (examples below).

## Reason Codes
| Code | Description |
|------|-------------|
| `EMA_RSI_CONFIRMED` | EMA crossover + RSI threshold satisfied. |
| `MACD_STOCH_CONFIRMED` | MACD direction + StochRSI gate satisfied. |
| `BOLLINGER_FILTER_FAIL` | Optional mid-band rule blocked entry. |
| `ATR_STOP_HIT` | ATR+swing stop triggered. |
| `TP_FIXED` | Fixed 3–6% take-profit target hit. |
| `MAX_DRAWDOWN` | 15% capital drawdown lock engaged. |
| `ROLLING_LOSS` | Rolling 14-day PnL ≤ 0 triggered suspension. |
| `MANUAL_DISABLE` | User toggled pair enable off. |

## Alertcondition Definitions
| Name | Expression | Message |
|------|------------|---------|
| `LONG_ENTRY` | `entryCondition()` true | `${pair}|${resolution}|LONG_ENTRY|${reason}` |
| `EXIT_TP` | `takeProfitHit` | `${pair}|${resolution}|EXIT_TP|TP_FIXED` |
| `EXIT_SL` | `stopLossHit` | `${pair}|${resolution}|EXIT_SL|ATR_STOP_HIT` |
| `EXIT_DRAWDOWN` | `riskGuards().drawdownLock` | `${pair}|${resolution}|EXIT_DRAWDOWN|MAX_DRAWDOWN` |
| `SUSPEND_ROLLING_LOSS` | `riskGuards().rollingLossLock` | `${pair}|${resolution}|SUSPEND_ROLLING_LOSS|ROLLING_LOSS` |
| `RESUME_TRADING` | `riskGuards().resume` | `${pair}|${resolution}|RESUME_TRADING|ROLLING_LOSS` |

All alert conditions must be wrapped in `if barstate.isconfirmed` blocks per constitution principle III.
