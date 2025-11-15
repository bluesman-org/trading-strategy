# Tech Stack: Long-Only Crypto Strategy

- **Language & Runtime**: Pine Script v6 (`//@version=6`) executed inside TradingView charts; only documented v6 APIs permitted per constitution.
- **Execution Platform**: TradingView strategy engine with native backtester, `barstate.isconfirmed` evaluation, and `alertcondition()` delivery to webhooks/notifications.
- **Market Data Inputs**: Spot OHLCV + spread/liquidity metrics for BTC-EUR and other approved crypto pairs; volatility sourced from built-in ATR/EMA functions cached per bar.
- **Core Modules**: Reusable helpers `entryCondition()`, `stopLossLogic()`, `takeProfitLogic()`, risk ledger calculators, and alert payload builder (schema `pair|timeframe|signal|reason`).
- **User-Facing Controls**: `input.*` parameters for timeframe tiers (15m–8h), take-profit band (3–6%), ATR multiples, drawdown cutoffs, and per-pair enable switches with identical defaults/tooltips.
- **Risk & Monitoring Hooks**: Rolling 14-day profitability tracker, drawdown guard (<15%), logging to TradingView strategy performance report, and alert auditing to verify <1 bar latency.
- **Performance Guardrails**: Cached indicator results ensure <100 ms execution per bar; no redundant `ta.*` calls, ensuring scalability across multiple active charts.
