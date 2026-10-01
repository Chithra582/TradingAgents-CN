# EXPLAINABILITY — TradingAgents-CN Market Analyst Agent

> **Admissibility & Transparency Report for OpenGAP / Agent Passport**  
> *Agent Name:* TradingAgents-CN Market Analyst Agent (`tradingagents-cn-market-analyst`)  
> *Specification:* OpenGAP v0.1.0  
> *Domain:* Finance / Quantitative Market Analysis  

---

## 1. Overview & Operational Purpose

TradingAgents-CN Market Analyst Agent is an AI-powered multi-agent equity research assistant engineered to analyze companies and macroeconomic trends in the Chinese A-share market. Its primary operational purpose is to provide researchers, data analysts, and software engineers with a transparent, debate-driven framework for evaluating stock fundamentals, financial statements, and news sentiment without relying on single-model black-box recommendations.

By assigning distinct analytical personas—such as conservative valuation auditors, growth-focused bull analysts, and technical indicator specialists—the system generates balanced multi-scenario perspectives. All generated outputs are explicitly designed for academic research, education, and paper trading simulations, maintaining strict compliance with financial research guidelines.

---

## 2. How the Agent Decides (Decision-Making Logic)

TradingAgents-CN Market Analyst Agent operates across a deterministic, multi-stage decision pipeline:

```
[Stage 1: Ingestion & Ticker Parse] ──> [Stage 2: Financial & Quote Fetch] ──> [Stage 3: Multi-Persona Role Assignment]
                                                                                               │
                                                                                               ▼
[Stage 6: Report Synthesis & Audit] <── [Stage 5: Consensus & Risk Gate]   <── [Stage 4: Adversarial Debate Exchange]
```

### 2.1 Ingestion & Ticker Disambiguation
- **Decision:** The agent parses the user inquiry to extract ticker codes (e.g., 600519.SH), industry sectors, or screening criteria.
- **Rules:** Validate ticker format against official exchange patterns (Shanghai .SH / Shenzhen .SZ / Beijing .BJ). Discard ambiguous company names lacking distinct ticker mappings.

### 2.2 Financial & Market Data Acquisition
- **Decision:** Retrieve standardized financial statements, valuation metrics (PE, PB, PEG), and historical price series through verified runtime APIs.
- **Rules:** Reject outdated cache files exceeding 24-hour staleness during active market trading sessions. Ensure all financial metrics reflect reported fiscal period dates.

### 2.3 Adversarial Debate & Multi-Scenario Synthesis
- **Decision:** Orchestrate multi-turn debate rounds between the Optimistic (Bull) and Conservative (Bear) researcher agents across five depth tiers.
- **Rules:** Require each debater to ground claims in empirical financial statement entries or verified news announcements. Never synthesize a one-sided outlook without counter-arguments.

### 2.4 Consensus Synthesis & Risk Gate Verification
- **Decision:** Aggregate multi-agent debate outputs into structured reports with explicit scenario probabilities and risk disclaimers.
- **Rules:** Enforce mandatory non-investment-advice disclaimers on all generated summaries. Reject any agent response containing direct purchase or sale recommendations.

---

## 3. Data Flow & Boundary Privacy

The market research agent strictly enforces data boundaries, protecting proprietary user research criteria and local cache files.

| Component / Boundary | Data Received | Processing & Retention | Destination / External Transmission |
|---|---|---|---|
| User Input Interface | Stock screening prompts, ticker queries | In-memory query parsing; session-scoped retention | Local analyst runtime |
| Financial Data Cache | Historical quotes, balance sheets, announcements | Local MongoDB/Redis caching; zero external telemetry | Local database instances |
| Multi-Agent Debate Bus | Turn-by-turn arguments, rebuttal notes | In-memory message passing between analyst roles | Internal debate coordinator |
| Audit Logger | Screened tickers, debate transcripts, query timestamps | Tamper-evident structured JSON logging on disk | Local disk audit trail |

TradingAgents-CN Market Analyst Agent complies with operational security and privacy standards:
- **No Cloud Data Exfiltration:** Proprietary user research criteria, paper trading balances, and notes are never transmitted to unauthorized external servers.
- **Epistemic Isolation:** Memory contexts are partitioned per research session to prevent cross-stock premise bleed.
- **Sanitized Model Payloads:** Prompts sent to model providers contain public market data and abstract analytical templates, scrubbing any user identifiers.
- **Data Minimization:** Only financial fields relevant to the requested valuation model (e.g. cash flow lines for DCF) are retrieved.

---

## 4. Known Limitations & Failure Modes

Reviewers, auditors, and users should note the following operational constraints:

1. Data Latency in Fast-Moving Markets
   - *Limitation:* Free or community data feeds may exhibit a 15-minute quote delay during high-volatility trading sessions.
   - *Mitigation:* The agent explicitly displays quote timestamps and alerts users that outputs reflect historical snapshots.

2. Divergent Viewpoints Without Consensus
   - *Limitation:* In highly contentious or volatile turnaround stocks, bull and bear debaters may fail to agree on a valuation range.
   - *Mitigation:* Present both optimistic and pessimistic scenario tables side-by-side rather than forcing an artificial consensus.

3. Accounting Policy Variations Across Sectors
   - *Limitation:* Comparing metrics like inventory turnover or gross margin across divergent sectors (e.g. banking vs software) can yield misleading comparisons.
   - *Mitigation:* The agent enforces sector-specific normalization and compares tickers exclusively against their peer industry cohort.

4. Model Hallucination on Unreported Metrics
   - *Limitation:* When asked about unreleased quarterly figures, LLM agents might attempt to extrapolate numbers speculatively.
   - *Mitigation:* Ground all quantitative calculations in structured database records; flag any forward-looking estimates as speculative projections.

---

## 5. Verification, Safety & Human Oversight

TradingAgents-CN Market Analyst Agent incorporates robust verification, safety gates, and human oversight controls across every layer of execution:

- **Real-Time Human Approval Gate:** All paper trading simulations, parameter revisions, and report exports require explicit human user initiation.
- **Emergency Session Interrupt:** Users can terminate running multi-agent debate loops or web search routines immediately with an instant interrupt command.
- **Step Quota Guardrails:** Strict session ceilings (maximum 25 turns) and debate depth caps prevent runaway reasoning loops and API cost inflation.
- **Structured Audit Logging:** Every data query, agent rebuttal, valuation model output, and user prompt is recorded in structured JSON logs for audit review.
