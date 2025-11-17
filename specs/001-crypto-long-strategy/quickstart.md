# Quickstart: Long-Only Crypto Strategy

## 1. Import & Setup
1. Open TradingView → Pine Editor → paste `pine/strategies/long_only_crypto_strategy.pine` and click **Add to chart**.
2. Confirm the header shows `//@version=6` and the script compiles without errors.

## 2. Configure Inputs
| Input | Default | Notes |
|-------|---------|-------|
| Timeframe Tier | Auto-detect (chart) | Applies parameter presets per 15m/30m/1h/4h/8h. |
| Trend Filter | EMA crossover | Toggle to MACD for higher timeframes. |
| Momentum Mode | RSI > 55 | StochRSI option available with %K/%D inputs. |
| ATR Multiple | 2.0 | Multiplies ATR(14); combined with swing-low buffer tick. |
| Take-Profit % | 4.5% | Slider 3–6% in 0.5% increments. |
| Bollinger Confirmation | false | When true, price must close above mid-band. |
| Drawdown Cap | 15% | Blocks new entries when breached. |
| Pair Enable | true | Disable to prevent trading on the current symbol. |

### Backtest Time Range (added)
| Input | Default | Notes |
|-------|---------|-------|
| Use custom backtest date range | false | Enable to pick explicit start and end timestamps for the backtest. If disabled, the script backtests for the last N days below. |
| Backtest duration (days) | 365 | Number of days to include when custom backtest range is disabled. Defaults to 365 (1 year). |
| Custom Backtest Start | 2024-01-01 (example) | When custom range is enabled, select an inclusive start date. |
| Custom Backtest End | 2025-01-01 (example) | When custom range is enabled, select an inclusive end date. |

Notes:
- Setting a custom date range limits new entries to the selected window. Exits will still be processed by the strategy so that positions can close normally outside the range when necessary.  
- If the custom end date is earlier than the start date the script logs a warning and disables entries until fixed.

Tooltips must reference Pine Script v6 manual sections for each indicator.

## 3. Validate Signals
1. Run Strategy Tester on BTC-EUR (1h) for last 30 days → verify net PnL positive and drawdown <15%.
2. Switch chart to ETH-EUR, duplicate the script, and ensure identical inputs/alerts appear (cross-pair parity).
3. Toggle Bollinger confirmation to confirm entries pause when mid-band condition fails.

## 4. Monitor Risk & Alerts
- Watch on-chart table/labels for rolling 14-day PnL and drawdown status.  
- Alerts are sent with payload `pair|timeframe|signal|reason`; sample: `BTC-EUR|1h|LONG_ENTRY|EMA_RSI_CONFIRMED`.  
- When drawdown or rolling loss breach occurs, “SUSPEND_*” alerts fire; re-enable trading only after manual review.

## 5. Performance Checklist
- Open Pine profiler → ensure execution time per bar <100 ms; if exceeded, reduce optional indicators.  
- Confirm alert log timestamps match candle close times (same minute) for latency compliance.  
- Export Strategy Tester report to `tests/backtests/` and commit results for regression tracking.
