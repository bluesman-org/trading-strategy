# Data Model

## Strategy Profile
- **Keys**: `symbol`, `timeframe`, `profileId`
- **Fields**: trend filter mode (EMA or MACD + parameters), momentum mode (RSI or StochRSI + thresholds), ATR multiplier, swing buffer ticks, take-profit %, Bollinger toggle, enable flag.
- **Relationships**: 1:N with Trade Tickets; 1:1 with Risk Ledger snapshot per chart.
- **State Transitions**: Enabled → Suspended (drawdown/rolling loss) → Re-enabled.

## Trade Ticket
- **Keys**: `ticketId`
- **Fields**: timestamp, entry price, size (EUR 1,000), stop distance, take-profit %, active indicators at entry, alerts fired, exit reason.
- **Relationships**: Belongs to Strategy Profile; logs events to Risk Ledger.
- **States**: Pending → Active → Closed (TP / SL / Suspension) → Logged.

## Risk Ledger
- **Keys**: `profileId`
- **Fields**: starting capital, current equity, rolling 14-day net PnL, max drawdown %, suspension flags, last alert payload.
- **Relationships**: Aggregates over Trade Tickets; fed to monitoring/quickstart instructions.
- **State Transitions**: Healthy (<12% DD) → Warning (12–15%) → Locked (≥15%, blocks entries).

## Market Snapshot
- **Fields**: OHLCV candle, ATR(14), swing high/low, Bollinger mid/upper/lower, spread/liquidity metrics, timestamp (bar close), cached indicator structs.
- **Usage**: Derived each confirmed bar; reused by entry/momentum/stop helpers to stay under 100 ms.

## Alert Event
- **Fields**: payload (`pair|timeframe|signal|reason`), timestamp, bar index, delivery channel, acknowledgement status.
- **Relationships**: Generated from Strategy Profile + Risk Ledger context; logged externally via TradingView alert logs.
