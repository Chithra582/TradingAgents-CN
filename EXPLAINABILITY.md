# EXPLAINABILITY.md

This document explains the internal mechanisms, data lineage, operational boundaries, and governance framework of **TradingAgents-CN Market Analyst** (`tradingagents-cn`) in accordance with the **OpenGAP v0.1.0** specification for the **HiDevs GitAgent Passport** clearance pipeline.

> **Agent Name:** TradingAgents-CN Market Analyst (`tradingagents-cn`)  
> **Specification:** OpenGAP v0.1.0  
> **Category / Domain:** Finance / Quantitative Chinese Financial Market Analysis  
> **Compliance Standard:** OpenGAP Checkpoint 2 (Explainability & Decision Governance), OWASP LLM Top 10, MITRE ATLAS  

---

## How the Agent Decides

TradingAgents-CN Market Analyst is an autonomous quantitative financial analysis and market intelligence agent specialized for Chinese capital markets (A-Shares, STAR Market, ChiNext, and Hong Kong Stock Connect). Its primary operational purpose is to ingest high-frequency market tickers, macroeconomic disclosures, corporate financial filings, and real-time sentiment streams to synthesize auditable investment research reports and risk-hedged trading strategies with mathematical transparency.

### 1. Decision Architecture

The market data ingestion, quantitative indicator calculation, multi-factor ranking, and risk audit pipeline operates across a deterministic, five-stage architecture:

```
Market Event / Analytical Directive (Stock Ticker / Disclosure Event / Macro Report / Portfolio Request)
    │
    ▼
[Stage 1: Market Data & Filing Ingestion]
    │  - Ingests real-time A-Share price series, order book depth, and regulatory disclosures
    │  - Normalizes financial statements across Chinese accounting standards (CAS)
    │  - Sanitizes and validates data streams against exchange boundary conditions
    ▼
[Stage 2: Quantitative Factor Computation & Signal Synthesis]
    │  - Computes deterministic technical indicators (MACD, RSI, Bollinger Bands, KDJ)
    │  - Evaluates fundamental valuation factors (P/E, P/B, EV/EBITDA, ROE)
    │  - Ingests NLP financial sentiment indicators from EastMoney, Xueqiu, and regulatory feeds
    ▼
[Stage 3: Multi-Factor Composite Scoring & Alpha Ranking]
    │  - Combines technical momentum, fundamental quality, and sentiment alpha
    │  - Applies factor risk attribution and industry sector neutralization
    │  - Generates ranked asset conviction scores across target investment universes
    ▼
[Stage 4: Risk Boundary Audit & Compliance Verification]
    │  - Evaluates maximum portfolio drawdown constraints and single-stock position caps
    │  - Enforces CSRC regulatory trading boundaries (price limits: +/-10% main board, +/-20% STAR)
    │  - Verifies stop-loss and liquidity thresholds prior to report finalization
    ▼
[Stage 5: Research Report Delivery & Trajectory Archival]
    │  - Synthesizes transparent, auditable investment research briefs with complete mathematical citations
    │  - Formats clear disclaimers that outputs represent quantitative analysis, not financial advice
    │  - Commits structured decision logs to local filesystem for regulatory audit
    ▼
Validated Quantitative Research Brief & Auditable Financial Trajectory Record
```

### 2. Decision Logic & Quantitative Scoring Formulations

TradingAgents-CN evaluates asset conviction, factor alpha, and volatility boundaries using deterministic mathematical models:

1. **Composite Alpha Conviction Score ($S_{\text{alpha}}$)**:
   $$S_{\text{alpha}} = (w_f \cdot F_{\text{fundamental}}) + (w_t \cdot T_{\text{technical}}) + (w_s \cdot S_{\text{sentiment}})$$
   where:
   - $F_{\text{fundamental}} \in [0, 1]$ represents normalized ROE and cash flow stability.
   - $T_{\text{technical}} \in [0, 1]$ represents trend momentum (EMA slope + MACD divergence).
   - $S_{\text{sentiment}} \in [0, 1]$ represents NLP sentiment score from Chinese financial news feeds.
   - Weights: $w_f = 0.45, w_t = 0.35, w_s = 0.20$ ($\sum w_i = 1.0$).

2. **Maximum Risk Exposure Constraint ($R_{\text{exposure}}$)**:
   $$R_{\text{exposure}}(i) = \min\left( C_{\text{stock\_cap}}, \frac{\text{RiskBudget}}{\text{ATR}_{14}(i) \times \text{Price}(i)} \right)$$
   where single-stock exposure is strictly capped at $C_{\text{stock\_cap}} = 10\%$ of total portfolio NAV to guarantee defensive diversification.

### 3. Thresholding & Refusal Decision Criteria

TradingAgents-CN Market Analyst enforces strict operational safety, ethical boundaries, and regulatory compliance:
- **Refusal to Execute Unregistered Real-Money Orders**: Instructions to execute live financial trades on real brokerage accounts are deterministically rejected with code `ERR_LIVE_TRADING_PROHIBITED`. The agent outputs analytical research reports only.
- **Refusal to Generate Market Manipulation Signals**: Queries requesting strategies designed to pump-and-dump, spoof order books, or manipulate illiquid micro-caps are blocked (`ERR_MARKET_MANIPULATION_REFUSED`).
- **Turn Ceiling Enforcement**: Quantitative calculation loops enforce a ceiling of `max_turns: 25` to prevent runaway simulation cycles (`WARN_TURN_BUDGET_REACHED`).
- **Regulatory Boundary Guard**: Tickers undergoing exchange trading halts or regulatory investigations are flagged with immediate suspension warnings (`WARN_REGULATORY_HALT_DETECTED`).

### 4. Fallback Decision Mechanism

Continuous financial analysis is maintained through multi-tier fault recovery:
- **Model Cascade Failover**: When the primary foundation model experiences latency spikes or HTTP 429 rate limits, the orchestrator cascades automatically between `claude-3-5-sonnet`, `gpt-4o`, and `gemini-2.0-flash`.
- **Deterministic Pure-Python Factor Fallback**: If LLM narrative synthesis fails, the system serves structured CSV and Markdown reports containing pure mathematical factor tables and indicator outputs.
- **Graceful Data Feed Degradation**: If real-time tick feeds experience latency, the agent falls back to end-of-day daily bar historical data with explicit timestamp warnings.

### 5. Human-in-the-Loop Governance

Human portfolio managers and analysts retain full investment discretion and final sign-off authority:
- **Mandatory Human Investment Discretion**: All research reports, factor rankings, and simulated portfolio allocations are explicitly labeled as informational analysis requiring professional human validation.
- **Emergency Session Kill Switch**: Operators can halt quantitative simulations or data ingest loops instantly using standard `Ctrl+C` interrupt signals.
- **Transparent Mathematical Factor Citing**: Every score, indicator value, and valuation ratio in generated reports provides the exact formula and raw underlying financial data inputs.

---

## The Data It Uses

TradingAgents-CN operates under strict privacy, data minimization, and local workspace isolation standards.

### 1. Ingested Input Data

The agent processes only operational assets necessary to fulfill financial market analysis:
- **Market Data Feeds**: Historical and daily OHLCV price bars, volume, turnover, and order book snapshots.
- **Corporate Financial Statements**: Income statements, balance sheets, and cash flow reports from Chinese corporate filings.
- **Financial News & Disclosures**: Public regulatory notices, exchange announcements, and macroeconomic indicator releases.

### 2. Configuration & Reference Data

- **Technical Indicator Specifications**: Mathematical formulas and parameter defaults for MACD, RSI, KDJ, and Bollinger Bands.
- **Sector Classification Taxonomies**: Shenwan (SW) industry classification codes and CSI 300 / CSI 500 index constituent weights.
- **Regulatory Rule Matrix**: CSRC trading rules, price change limits (+/-10%, +/-20%), and T+1 settlement schemas.

### 3. Base Model & Inference Lineage

- **Deterministic Algorithmic Engines**: Quantitative factor calculators, TA-Lib indicator processors, and statistical covariance estimators executed natively in Python (100% deterministic with zero LLM variance).
- **Foundation LLMs**: High-capability frontier models (`claude-3-5-sonnet`, `gpt-4o`, `gemini-2.0-flash`) utilized for financial filing comprehension, Chinese sentiment extraction, and qualitative thesis synthesis.
- **Zero Training on Proprietary Portfolios**: User investment strategies, portfolio holdings, and private trade histories are never transmitted to external cloud servers or used for model training.

### 4. Data Privacy, Storage, and Retention

- **OWASP LLM & MITRE ATLAS Hardened**: Defended against prompt injection, malicious data poisoning, and unauthorized agency.
- **Local-Only Financial Database**: All cached historical price bars, factor rankings, and generated reports reside exclusively on the user's filesystem.
- **Credential Scrubbing**: Financial data API keys, database connection strings, and user paths are scrubbed from generation logs.
- **Zero Commercial Monetization**: User portfolio allocations, watchlist queries, and investment notes are never shared, monetized, or sold to third parties.

---

## Limitations

Understanding the operational boundaries and technical constraints of TradingAgents-CN is essential for responsible financial analysis.

### 1. Extreme Tail-Risk Black Swan Events
- **Limitation**: Quantitative factor models calibrate on historical distributions and cannot foresee sudden geopolitical or macroeconomic black swan disruptions.
- **Mitigation**: The agent enforces defensive position sizing caps and requires stress-testing against historical crisis scenarios.

### 2. Small-Cap Liquidity Slippage
- **Limitation**: Simulated factor backtests assume continuous liquidity, whereas real-world trading in illiquid micro-caps incurs substantial slippage.
- **Mitigation**: The system incorporates bid-ask spread penalties and filters out securities with average daily turnover below liquid thresholds.

### 3. Financial Filing Reporting Lag
- **Limitation**: Quarterly corporate disclosures reflect historical performance with a 1 to 4 month reporting delay.
- **Mitigation**: The agent pairs fundamental accounting data with high-frequency technical momentum and real-time news sentiment.

### 4. Sentiment NLP Nuance in Chinese Financial Slang
- **Limitation**: Retail financial forums frequently utilize evolving slang and irony that can distort naive sentiment classifiers.
- **Mitigation**: The sentiment analyzer utilizes domain-specific financial sentiment dictionaries tuned specifically for Chinese market terminology.

### 5. Non-Stationarity of Alpha Factors
- **Limitation**: Quantitative factors that delivered outperformance in previous market regimes experience alpha decay as market efficiency increases.
- **Mitigation**: The agent evaluates factor performance across rolling 12-month windows and dynamically reweights factor contributions.

---

## Summary & Compliance Checklist

| Checkpoint 2 Requirement | Corresponding Section | Status |
| :--- | :--- | :---: |
| **How the agent decides** | [How the Agent Decides](#how-the-agent-decides) | **Covered** |
| - Decision architecture & 5-stage pipeline | Section 1 | Verified |
| - Decision logic & quantitative scoring formulations | Section 2 | Verified |
| - Thresholding & refusal decision criteria | Section 3 | Verified |
| - Fallback decision mechanism | Section 4 | Verified |
| - Human-in-the-loop governance & oversight | Section 5 | Verified |
| **The data it uses** | [The Data It Uses](#the-data-it-uses) | **Covered** |
| - Ingested market feeds, filings & disclosures | Section 1 | Verified |
| - Configuration, indicator specs & regulatory rules | Section 2 | Verified |
| - Base model lineage & deterministic engines | Section 3 | Verified |
| - Data privacy, retention lifecycle & MITRE/OWASP | Section 4 | Verified |
| **Its limitations** | [Limitations](#limitations) | **Covered** |
| - Extreme tail-risk black swan events | Section 1 | Verified |
| - Small-cap liquidity slippage | Section 2 | Verified |
| - Financial filing reporting lag | Section 3 | Verified |
| - Sentiment NLP nuance in Chinese financial slang | Section 4 | Verified |
| - Non-stationarity of alpha factors | Section 5 | Verified |
