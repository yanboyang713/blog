---
title: "sector aware stock screening"
date: 2026-10-22
draft: false
---

****Version:**** 1.0
****Configuration date:**** 2026-10-22
****Source:**** Finviz screen configurations established in this conversation
****Status:**** Research screening rules; not backtested or live trading recommendations.

\## 1. Screening architecture

1.  ****Quality Screen:**** sector-specific fundamental and descriptive hard filters.
2.  ****Trend Screen:**** Quality pass AND price above 200-day simple moving average.
3.  ****Timing Signals:**** RSI, MACD, moving averages, ATR, relative volume and regime analysis; monitored rather than required by Finviz presets.
4.  ****Opportunity Ranking:**** sector-relative valuation, financial quality, growth, technical timing and risk.

\## 2. Global screening rules

| Field                 | Setting                                                           |
|-----------------------|-------------------------------------------------------------------|
| Country               | USA                                                               |
| Market capitalization | &gt; $2,000,000,000                                               |
| Average daily volume  | &gt; 500,000 shares (Python convention: trailing 30 trading days) |
| Sector                | Strategy-specific                                                 |

****Implementation notes:**** Finviz country classifications and price/volume definitions may not equal those of your data vendor. Preserve the original definitions and document any substitution. All inequalities below are represented as strict comparisons in the JSON config; the UI may differ at exact thresholds.

\## 3. Complete strategy rules

\### 01_IT_Quality_Growth

****Descriptive filters****

| Field                       | Rule               |
|-----------------------------|--------------------|
| Country                     | USA                |
| Market Cap (USD)            | &gt; 2,000,000,000 |
| Average Volume (shares/day) | &gt; 500,000       |
| Sector                      | Technology         |

****Fundamental filters****

| Field                                 | Rule    |
|---------------------------------------|---------|
| EPS Growth Next Year (%)              | &gt; 15 |
| Sales Growth Quarter over Quarter (%) | &gt; 10 |
| Sales Growth TTM (%)                  | &gt; 10 |
| Return on Equity (%)                  | &gt; 10 |
| Return on Invested Capital (%)        | &gt; 10 |

****Buy Setup:**** \`Close &gt; SMA200\`; remaining technical indicators are not hard filters.

\### 02_FIN_Quality_Value

****Descriptive filters****

| Field                       | Rule               |
|-----------------------------|--------------------|
| Country                     | USA                |
| Market Cap (USD)            | &gt; 2,000,000,000 |
| Average Volume (shares/day) | &gt; 500,000       |
| Sector                      | Financial          |

****Fundamental filters****

| Field                          | Rule    |
|--------------------------------|---------|
| pe_ttm                         | &lt; 20 |
| price_to_book                  | &lt; 3  |
| EPS Growth Next Year (%)       | &gt; 5  |
| Return on Invested Capital (%) | &gt; 10 |

****Buy Setup:**** \`Close &gt; SMA200\`; remaining technical indicators are not hard filters.

\### 03_HC_Quality_Growth

****Descriptive filters****

| Field                       | Rule               |
|-----------------------------|--------------------|
| Country                     | USA                |
| Market Cap (USD)            | &gt; 2,000,000,000 |
| Average Volume (shares/day) | &gt; 500,000       |
| Sector                      | Healthcare         |

****Fundamental filters****

| Field                    | Rule    |
|--------------------------|---------|
| EPS Growth Next Year (%) | &gt; 5  |
| Sales Growth TTM (%)     | &gt; 5  |
| Return on Equity (%)     | &gt; 10 |
| Operating Margin (%)     | &gt; 10 |

****Buy Setup:**** \`Close &gt; SMA200\`; remaining technical indicators are not hard filters.

\### 04_CD_Quality_Growth

****Descriptive filters****

| Field                       | Rule               |
|-----------------------------|--------------------|
| Country                     | USA                |
| Market Cap (USD)            | &gt; 2,000,000,000 |
| Average Volume (shares/day) | &gt; 500,000       |
| Sector                      | Consumer Cyclical  |

****Fundamental filters****

| Field                          | Rule    |
|--------------------------------|---------|
| EPS Growth Next Year (%)       | &gt; 10 |
| Sales Growth TTM (%)           | &gt; 5  |
| Return on Invested Capital (%) | &gt; 10 |
| Operating Margin (%)           | &gt; 0  |

****Buy Setup:**** \`Close &gt; SMA200\`; remaining technical indicators are not hard filters.

\### 05_CS_Quality_Growth

****Descriptive filters****

| Field                       | Rule               |
|-----------------------------|--------------------|
| Country                     | USA                |
| Market Cap (USD)            | &gt; 2,000,000,000 |
| Average Volume (shares/day) | &gt; 500,000       |
| Sector                      | Consumer Defensive |

****Fundamental filters****

| Field                          | Rule    |
|--------------------------------|---------|
| EPS Growth Next Year (%)       | &gt; 5  |
| Sales Growth TTM (%)           | &gt; 5  |
| Return on Invested Capital (%) | &gt; 10 |
| Operating Margin (%)           | &gt; 0  |

****Buy Setup:**** \`Close &gt; SMA200\`; remaining technical indicators are not hard filters.

\### 06_IND_Quality_Growth

****Descriptive filters****

| Field                       | Rule               |
|-----------------------------|--------------------|
| Country                     | USA                |
| Market Cap (USD)            | &gt; 2,000,000,000 |
| Average Volume (shares/day) | &gt; 500,000       |
| Sector                      | Industrials        |

****Fundamental filters****

| Field                          | Rule    |
|--------------------------------|---------|
| EPS Growth Next Year (%)       | &gt; 10 |
| Sales Growth TTM (%)           | &gt; 5  |
| Return on Invested Capital (%) | &gt; 10 |

****Buy Setup:**** \`Close &gt; SMA200\`; remaining technical indicators are not hard filters.

\### 07_EN_Quality_Value

****Descriptive filters****

| Field                       | Rule               |
|-----------------------------|--------------------|
| Country                     | USA                |
| Market Cap (USD)            | &gt; 2,000,000,000 |
| Average Volume (shares/day) | &gt; 500,000       |
| Sector                      | Energy             |

****Fundamental filters****

| Field                   | Rule    |
|-------------------------|---------|
| pe_ttm                  | &lt; 25 |
| Debt/Equity             | &lt; 1  |
| Operating Margin (%)    | &gt; 0  |
| price_to_free_cash_flow | &gt; 5  |

****Buy Setup:**** \`Close &gt; SMA200\`; remaining technical indicators are not hard filters.

\### 08_UT_Quality_Value

****Descriptive filters****

| Field                       | Rule               |
|-----------------------------|--------------------|
| Country                     | USA                |
| Market Cap (USD)            | &gt; 2,000,000,000 |
| Average Volume (shares/day) | &gt; 500,000       |
| Sector                      | Utilities          |

****Fundamental filters****

| Field                    | Rule    |
|--------------------------|---------|
| Forward P/E              | &lt; 25 |
| EPS Growth Next Year (%) | &gt; 0  |

****Buy Setup:**** \`Close &gt; SMA200\`; remaining technical indicators are not hard filters.

\### 09_BM_Quality_Value

****Descriptive filters****

| Field                       | Rule               |
|-----------------------------|--------------------|
| Country                     | USA                |
| Market Cap (USD)            | &gt; 2,000,000,000 |
| Average Volume (shares/day) | &gt; 500,000       |
| Sector                      | Basic Materials    |

****Fundamental filters****

| Field                          | Rule    |
|--------------------------------|---------|
| Forward P/E                    | &lt; 25 |
| Return on Invested Capital (%) | &gt; 5  |
| Debt/Equity                    | &lt; 1  |
| Operating Margin (%)           | &gt; 0  |

****Buy Setup:**** \`Close &gt; SMA200\`; remaining technical indicators are not hard filters.

\### 10_COM_Quality_Growth

****Descriptive filters****

| Field                       | Rule                   |
|-----------------------------|------------------------|
| Country                     | USA                    |
| Market Cap (USD)            | &gt; 2,000,000,000     |
| Average Volume (shares/day) | &gt; 500,000           |
| Sector                      | Communication Services |

****Fundamental filters****

| Field                          | Rule    |
|--------------------------------|---------|
| Sales Growth TTM (%)           | &gt; 5  |
| EPS Growth Next Year (%)       | &gt; 10 |
| Return on Invested Capital (%) | &gt; 5  |
| Operating Margin (%)           | &gt; 10 |

****Buy Setup:**** \`Close &gt; SMA200\`; remaining technical indicators are not hard filters.

\### 11_RE_Quality_Value

****Descriptive filters****

| Field                       | Rule               |
|-----------------------------|--------------------|
| Country                     | USA                |
| Market Cap (USD)            | &gt; 2,000,000,000 |
| Average Volume (shares/day) | &gt; 500,000       |
| Sector                      | Real Estate        |

****Fundamental filters****

| Field                | Rule   |
|----------------------|--------|
| Sales Growth TTM (%) | &gt; 0 |
| Operating Margin (%) | &gt; 0 |

****Buy Setup:**** \`Close &gt; SMA200\`; remaining technical indicators are not hard filters.

\### 10_COM_Platform_Growth

****Descriptive filters****

| Field                       | Rule                                                      |
|-----------------------------|-----------------------------------------------------------|
| Country                     | USA                                                       |
| Market Cap (USD)            | &gt; 2,000,000,000                                        |
| Average Volume (shares/day) | &gt; 500,000                                              |
| Sector                      | Communication Services                                    |
| Industry                    | Any (recommend excluding Telecom Services in local model) |

****Fundamental filters****

| Field                          | Rule    |
|--------------------------------|---------|
| Sales Growth TTM (%)           | &gt; 5  |
| EPS Growth Next Year (%)       | &gt; 10 |
| Return on Invested Capital (%) | &gt; 5  |
| Operating Margin (%)           | &gt; 0  |

****Buy Setup:**** \`Close &gt; SMA200\`; remaining technical indicators are not hard filters.

\### 10_COM_Telecom_Value

****Descriptive filters****

| Field                       | Rule                   |
|-----------------------------|------------------------|
| Country                     | USA                    |
| Market Cap (USD)            | &gt; 2,000,000,000     |
| Average Volume (shares/day) | &gt; 500,000           |
| Sector                      | Communication Services |
| Industry                    | Telecom Services       |

****Fundamental filters****

| Field                   | Rule    |
|-------------------------|---------|
| Forward P/E             | &lt; 25 |
| Operating Margin (%)    | &gt; 0  |
| price_to_free_cash_flow | &gt; 5  |

****Buy Setup:**** \`Close &gt; SMA200\`; remaining technical indicators are not hard filters.

\## 4. Additional watchlist rule

\`10_COM_Telecom_Recovery_Watch\`: use the same descriptive and fundamental criteria as \`10_COM_Telecom_Value\`, with ****no SMA200 requirement****. This is a recovery watchlist, not a confirmed buy signal.

\## 5. Technical indicators to calculate

| Indicator                | Specification / role                                                    |
|--------------------------|-------------------------------------------------------------------------|
| SMA20, SMA50, SMA200     | 20-, 50-, 200-session simple moving averages                            |
| RSI14                    | 14-session momentum measure                                             |
| MACD                     | EMA(12) − EMA(26), signal EMA(9)                                        |
| ATR14                    | 14-session average true range; stop-distance and position-risk research |
| Relative volume          | Volume compared with historical daily baseline                          |
| Relative strength        | Stock return relative to sector / market benchmarks                     |
| Volatility regime        | Realized volatility, change points / volatility transition estimates    |
| Sector-specific features | See section 7                                                           |

\## 6. Timing and risk decision states

| State                 | Rule / intent                                                                  |
|-----------------------|--------------------------------------------------------------------------------|
| WATCH                 | Pass Quality filters                                                           |
| TREND_CANDIDATE       | WATCH and Close &gt; SMA200                                                    |
| PULLBACK_CANDIDATE    | Check SMA20/SMA50 proximity, RSI normalization, and price stabilization        |
| BUY_CONFIRMATION      | Demand price action confirmation, valuation margin, and acceptable reward/risk |
| REDUCE_OR_EXIT_REVIEW | Structural deterioration, valuation excess, or breached risk thresholds        |

****Risk parameters to test, not fixed recommendations:**** initial per-position risk 0.5%–1% of NAV; protective stop distances of 1.5–3×ATR; preliminary prospective reward/risk hurdle 2:1; incorporate earnings/event gaps and portfolio concentration.

\## 7. Sector-specific second-stage research

-   ****Financial:**** net_interest_margin, cet1_ratio, nonperforming_loans, deposit_cost, tangible_book_value
-   ****Energy:**** normalized_fcf_yield, wti, henry_hub, capex, reserve_life
-   ****Utilities:**** rate_base_growth, allowed_roe, net_debt_to_ebitda, dividend_coverage, ppa
-   ****Basic Materials:**** normalized_ebitda, commodity_price, unit_cost, capex, inventory_cycle
-   ****Communication Services:**** fcf_yield, arpu, churn, net_debt_to_ebitda, ad_revenue_growth
-   ****Real Estate:**** p_affo, affo_per_share_growth, affo_payout_ratio, occupancy, same_store_noi, net_debt_to_ebitda, debt_maturities

Additional factors by sector not hard-filtered by the screen:

| Sector             | Further metrics                                                      |
|--------------------|----------------------------------------------------------------------|
| Technology         | FCF yield, organic revenue growth, ROIC, PEG, AI capital expenditure |
| Healthcare         | R&amp;D productivity, pipeline risk, patent expirations, FCF         |
| Consumer Cyclical  | Same-store sales, inventory, margins, consumer cycle                 |
| Consumer Defensive | Pricing power, margins, payout coverage                              |
| Industrials        | Backlog, ROIC, capital expenditure, operating leverage               |

\## 8. Suggested software configuration and data contracts

\`\`\`python
quality_pass = evaluate_strategy(sector_rules, point_in_time_data)
trend_pass = close_at_signal_time &gt; sma200_at_signal_time
buy_setup_pass = quality_pass and trend_pass

candidate_score = rank_within_sector(
    valuation, financial_quality, growth, technical_timing, risk
)
\`\`\`

-   Use ****point-in-time**** financial filings and analyst estimates, with their original publication / availability timestamps.
-   Preserve historical universes (including subsequently delisted firms) to avoid survivorship bias.
-   Do not replace missing values with zero; log the missing filter and optionally route the security to manual review.
-   Calibrate thresholds and weights with walk-forward out-of-sample testing, transaction costs, and corporate-action-adjusted prices.
-   Rank within relevant sectors or subsectors before performing portfolio-level cross-sector allocation.
-   Separate independent power producers, banks, telecom operators and REIT subsectors whenever a common metric is not economically comparable.

\## 9. Machine-readable source

Corresponding JSON: \`sector_screening_rules_2026.json\`. The JSON is authoritative for exact programmatic keys and operator directions.
