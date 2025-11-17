# Backtest Date Range Validation

This file documents manual steps to validate that the long-only crypto strategy respects the "Backtest duration (days)" and custom date range inputs.

Steps:
1. Open TradingView → Pine Editor → paste `pine/strategies/long_only_crypto_strategy.pine` and click **Add to chart**.
2. In the script settings (gear icon) under Inputs:
   - Leave `Use custom backtest date range` disabled (default)
   - Set `Backtest duration (days)` to 365
3. Use the Strategy Tester to run the backtest on a symbol (BTC-EUR) across 1H timeframe. Verify only trades from the last ~365 days are present.
4. Enable `Use custom backtest date range` and set `Custom Backtest Start` to a date 90 days ago; set `Custom Backtest End` to today. Run the Strategy Tester - only trades between the selected dates should be present.
5. Try swapping start/end (end earlier than start) — script should log a warning and disable new entries until corrected.

Expected Results:
- When using duration-based range, new entries only occur after `time >= now - backtestDurationDays`.
- When using custom date range, new entries only occur between the selected start/end timestamps. Exits still execute normally.
- Invalid custom ranges disable new entries and log a message explaining the issue.

Notes:
- The script uses current bar `time` (ms epoch). Strategy Tester "from-to" range in TradingView app will also limit data; this script enforces the logical window independent of the user's TradingView plan (useful for free plan users who cannot set historical ranges in the UI).
