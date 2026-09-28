# H World Group (HTHT) — DCF and Comparable Company Analysis

## Overview

Valuation analysis of H World Group Limited (NASDAQ: HTHT), a Chinese multi-brand hotel operator (formerly Huazhu Group) trading as an ADR on NASDAQ. Selected as a small-to-mid-cap company with cross-border characteristics: China-based operations, a NASDAQ listing, and an acquisition of a German hotel group (Deutsche Hospitality), aligning with a cross-border investment/financing (跨境投融资) analytical angle.

This project builds on the methodology used in a prior Nike (NKE) DCF project, extended to handle complications specific to a Chinese ADR: RMB-denominated financials, a 1:10 ADS-to-ordinary-share ratio, and several data-quality issues in third-party financial data that required manual correction rather than blind use.

## Data Source

All financial data sourced via the `yfinance` Python library (free, no API key required). Raw data cached locally as CSV files on first fetch to avoid repeated API calls.

## Repository Structure
dcf-hworld/
├── analysis.ipynb Main analysis notebook (ratios, DCF, Comps)
├── hworld_income_statement_raw.csv Cached income statement data
├── hworld_balance_sheet_raw.csv Cached balance sheet data
├── hworld_cash_flow_raw.csv Cached cash flow statement data
├── hworld_peer_comps_raw.csv Cached peer comps data
└── README.md


## Methodology

### 1. Financial Ratio Analysis

- Margin analysis: gross, operating, and net margin trends, with diagnosis of below-the-line drivers (e.g., a 2024 net margin dip traced to a swing in Other Income/Expense, not interest or tax as initially assumed).
- DuPont ROE decomposition (Net Margin x Asset Turnover x Equity Multiplier), including a check on whether a spike in Equity Multiplier was driven by rising debt or by equity shrinkage from prior losses (confirmed: equity shrinkage).
- Leverage analysis: identified that 77-85% of reported "Total Debt" is capital lease obligations rather than conventional bank/bond debt, a structural feature of the asset-light hotel model that a plain Debt-to-Equity ratio would not reveal.
- Working capital analysis: separated "Operational Working Capital" (Receivables + Inventory − Payables) from "Reported Working Capital" (which also includes financing items like current lease/debt maturities), showing that negative reported working capital is not automatically a distress signal for a cash-upfront hospitality business.

### 2. DCF Valuation

**Currency-matching principle:** all cash flows, WACC inputs, and intermediate valuation figures are kept in RMB throughout, matching the currency HTHT's financials are actually reported in. Conversion to USD-per-ADS happens only at the final step, applying the 1 ADS = 10 ordinary shares ratio and the prevailing USD/CNY exchange rate.

**WACC — three corrections made to raw data before the number could be trusted:**

1. Beta: yfinance's reported beta (0.127) was rejected. It is computed against a US benchmark (S&P 500), which does not reflect a business whose fundamentals are driven by the Chinese domestic market — the benchmark mismatch produces an artificially low, unusable figure. Replaced with an assumed global hotel industry beta of 0.95, plus an explicit 300bps ADR structural risk premium added to cost of equity, reflecting VIE-structure, delisting, and capital-repatriation risks specific to holding a Chinese ADR that a generic industry beta does not capture.
2. Cost of debt: the interest-expense-derived pretax cost of debt (0.93%) was rejected as implausible. Since roughly 80% of "Total Debt" is lease obligations, and lease interest does not fully flow through the reported Interest Expense line under current lease accounting standards, this figure understated true borrowing cost. Replaced with a market-based assumption (China risk-free rate plus a ~230bps corporate credit spread, approximately 4.0% pretax).
3. Final WACC: 7.60%, built from a market-value-weighted capital structure (70.6% equity / 29.4% debt).

**FCF projections — Bear / Base / Bull / Stress scenarios:**

The raw yfinance-reported FCF margin (29.80% for FY2025) was also rejected, after a sanity check against HTHT's actual trading history (all-time high closing price: 56.74 USD, February 2021) showed the resulting DCF implied a Base-case value roughly 38% above the highest price this stock has ever traded at, even during the most bullish sentiment period it has experienced. Diagnosis: under current lease accounting (ASC 842 / IFRS 16), the principal portion of lease payments is classified as a financing cash outflow rather than an operating one, even though paying rent on leased hotel properties is a real, recurring operating cost. This overstates Operating Cash Flow (and therefore FCF) for a lease-heavy operator like HTHT. Correction: estimated the lease-principal portion of financing-activity debt repayments (using the same ~84% lease share found on the balance sheet) and subtracted it from reported FCF, producing a lease-adjusted FCF margin of 19.04%, used as the anchor for all four scenarios.

| Scenario | Revenue Growth | FCF Margin | Terminal Growth | Implied Price per ADS (USD) |
|---|---|---|---|---|
| Stress | -2% | 10% | 1.5% | 5.01 |
| Bear | 2% | 15% | 3.0% | 26.91 |
| Base | 6% | 19% | 3.0% | 47.05 |
| Bull | 10% | 22% | 3.0% | 69.35 |

### 3. Comparable Company Analysis

Peer group: global asset-light hotel operators (Marriott, Hilton, IHG) and China-listed/ADR hotel operators (Atour Lifestyle, Jinjiang International). GreenTree Hospitality was initially included but excluded from the final peer set.

**Data quality corrections made before the Comps table could be trusted:**

1. IHG's yfinance-reported `enterpriseValue` field was corrupted, implying a net debt position of roughly 122.7 billion USD — far beyond anything realistic for a company of IHG's size. Enterprise Value was rebuilt manually from Market Cap + Total Debt − Cash using balance sheet data, correcting EV/EBITDA from an implausible 113.36x to 20.70x, in line with global peers.
2. Atour and GreenTree report financial statement figures (revenue, EBITDA) in CNY while trading and market cap data is in USD (ADR trading currency) — a currency mismatch that produced nonsensical negative EV/EBITDA multiples. Corrected by converting financial-statement figures to USD before computing any ratio.
3. GreenTree Hospitality was excluded from the peer average despite the currency fix: a market cap of only ~102 million USD and a resulting EV/EBITDA of 2.10x are consistent with a distressed or going-private situation, not a valid trading comparable.

**Finding:** a clear valuation gap exists between global and China-listed hotel peers — global peers trade at 20.70x-26.83x EV/EBITDA, while China peers (Atour, Jinjiang) trade at 9.69x-11.38x. HTHT's own current EV/EBITDA (10.89x) sits in line with the China peer average (10.54x), not the global average (23.36x), confirming that the China peer group — not the global one — is the appropriate benchmark for this company.

## Key Findings

| Method | Implied Price per ADS (USD) |
|---|---|
| DCF — Base case | 47.05 |
| Comps — China peer benchmark | 40.45 |
| Current market price | 42.00 |

Two independent valuation methods — one based on projected future cash flows, one based on what the market currently pays for comparable China-listed hotel operators — converge closely around the current market price. This suggests HTHT is fairly valued to modestly undervalued, not dramatically undervalued as an uncorrected version of either method would have suggested. The corrections made along the way (beta, cost of debt, FCF margin, EV field integrity, currency consistency) were the substantive analytical work of this project; the final valuation range is a secondary output of getting those corrections right.

## Limitations

- Beta (0.95) and the ADR structural risk premium (300bps) are analyst judgment calls, not directly sourced market figures.
- The assumed cost of debt (4.0% pretax) is a market-based estimate, not derived from HTHT's actual bond yields or credit rating.
- The lease-principal adjustment to FCF applies the balance-sheet lease share (83.9%) to aggregate debt repayment cash flow as an approximation; it does not use a line-item lease amortization schedule.
- Revenue growth and FCF margin assumptions for the 5-year explicit forecast period remain subjective, anchored to management guidance where available (2026 guidance: 4-8% revenue growth) but not independently verified beyond that.
- The Comps peer set (5-6 companies) is small; statistical robustness is limited compared to a larger peer universe.

## Running Instructions

1. Create and activate a virtual environment:
python -m venv .venv
.venv\Scripts\activate

2. Install dependencies:
pip install pandas numpy yfinance jupyter

3. Launch Jupyter and open `analysis.ipynb`:
jupyter notebook

4. Run all cells in order from top to bottom. Raw data is cached to CSV on first run.

## Disclaimer

This analysis is for educational and portfolio purposes only. It does not constitute investment advice. All assumptions, corrections, and judgment calls are documented above and should be independently verified before any real-world use.