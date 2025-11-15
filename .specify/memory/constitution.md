<!--
Sync Impact Report
Version: 0.0.0 -> 1.0.0
Modified Principles: All placeholders replaced with Pine Script-focused directives (I–V)
Added sections: Execution Standards; Governance for Technical Decisions
Removed sections: None
Templates requiring updates: ✅ .specify/templates/plan-template.md, ✅ .specify/templates/spec-template.md, ✅ .specify/templates/tasks-template.md
Follow-up TODOs: None
-->

# Trading Strategy Pine Script Constitution

## Core Principles

### I. Pine Script v6 Compliance (NON-NEGOTIABLE)
All TradingView artifacts MUST be authored in Pine Script v6, start with `//@version=6`, and rely exclusively on functions documented in the official reference (https://www.tradingview.com/pine-script-docs/en/v6/). Every new construct requires an inline comment referencing the relevant manual section so reviewers can verify compliance quickly.

### II. Modular Strategy Functions
Strategy logic MUST be decomposed into reusable helpers such as `entryCondition()`, `stopLossLogic()`, `takeProfitLogic()`, risk filters, and alert payload builders. Each function must have a single responsibility so it can be independently unit-tested, reused across pairs, and reasoned about during reviews.

### III. Deterministic Signals & Alerts
Signals may only trigger on confirmed bars (`barstate.isconfirmed`) to eliminate repainting. Every entry and exit requires a paired `alertcondition()` with payloads that include pair, timeframe, and reason codes so downstream automation can consume alerts deterministically.

### IV. User-Centric Inputs & Cross-Pair Parity
All tunable parameters MUST be exposed through descriptive `input.*` controls with defaults, ranges, and tooltips. The same parameter set, naming, and defaults must remain identical across every supported crypto pair so that copying a profile never alters risk unintentionally.

### V. Performance & Reliability Guardrails
Indicator calculations must be optimized (cache `ta.*` results, avoid redundant requests) so scripts execute well under 100 ms per bar on TradingView infrastructure. Alert delivery must occur within the closing bar, and any reliability fallback logic (e.g., re-arm checks) must be documented and tested.

## Execution Standards

### Code Quality
- Reference the Pine Script v6 manual when introducing any new function or pattern; undocumented behavior is prohibited.
- Keep files ASCII-only and declare constants/variables with descriptive names that explain their trading purpose (e.g., `riskPct`, `atrStopMultiplier`).
- Encapsulate reusable math or validation inside helper functions/modules instead of duplicating expressions across the script.

### User Experience Consistency
- Every `input.*` call must specify title, default, bounds/step, and tooltip text describing the control’s intent and relevant manual section.
- Provide clear sections in the properties panel for timeframe tier selection, take-profit targets (3–6%), and volatility stop behavior so users can install and run without documentation.
- Define matching entry/exit alerts whose payload schema is `pair|timeframe|signal|reason`; reuse this schema across all markets for consistency.
- Confirm signals on bar close and document this determinism in the script header so users understand latency assumptions.

### Performance Requirements
- Cache expensive calculations (e.g., ATR, EMA) in variables per bar; never call `ta.*` redundantly within the same bar evaluation.
- Audit scripts against TradingView’s execution quotas to ensure multi-symbol watchlists remain responsive; document expected runtime per bar (<100 ms) in the plan.
- Ensure alerts fire within one bar close; add watchdog logic and logging comments if TradingView limits are approached.

## Governance for Technical Decisions

- **Documentation Alignment**: Before adopting any Pine Script feature, confirm it exists in the v6 reference and cite the URL in the code review description. No experimental or undocumented calls are allowed.
- **Architecture Decision Records (ADRs)**: Capture why specific technical choices (e.g., `ta.atr` for stop sizing, `request.security()` frequency) were made, including the measured impact on execution time and drawdown control.
- **Review Process**: Every change requires peer review that explicitly checks Pine Script v6 compliance, modularity, deterministic alerts, and performance budgeting. Reviews must link to the relevant constitution principles they verified.
- **Change Control**: Inputs, alert payloads, and risk logic modifications must be versioned (update change log/comment header) and accompanied by rationale plus validation evidence (e.g., benchmark screenshots).

## Governance

- This constitution supersedes prior process docs for TradingView strategy work. Any conflicts are resolved in favor of the stricter requirement.
- Semantic versioning applies: MAJOR for breaking governance or principle rewrites, MINOR for new principles/sections, PATCH for clarifications.
- Amendments require (a) documented rationale, (b) updated Sync Impact Report, (c) confirmation that plan/spec/tasks templates reflect the change, and (d) communication to all contributors before the next planning cycle.
- Compliance reviews occur at `/speckit.plan` (Constitution Check gate), `/speckit.specify` validation, and during code review; violations block merges until resolved or formally waived with an ADR.

**Version**: 1.0.0 | **Ratified**: 2025-11-15 | **Last Amended**: 2025-11-15
