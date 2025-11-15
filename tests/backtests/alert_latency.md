# Alert Latency Evidence

Record one row per alert replay to satisfy SC-005 (<= 1 minute delivery) and Principle V.

| Pair | Timeframe | Candle Close Timestamp | Alert Timestamp | Delta (seconds) | Pass/Fail | Notes |
|------|-----------|------------------------|-----------------|-----------------|-----------|-------|
|      |           |                        |                 |                 |           |       |

**Procedure**
1. Export alert logs from TradingView or webhook listener for BTC-EUR and ETH-EUR.
2. Align each alert to the originating candle close time.
3. Compute `delta = alert_time - candle_close`. If delta > 60s, log remediation action.
4. Commit the populated table alongside raw evidence (screenshots or CSV) when available.
