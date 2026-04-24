# DSE Quant Diagnosis Engine
### Institutional-Grade Portfolio Analytics for the Dar es Salaam Stock Exchange

---

![Python](https://img.shields.io/badge/Python-3.8%2B-3776AB?style=flat-square&logo=python&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=flat-square&logo=jupyter&logoColor=white)
![Status](https://img.shields.io/badge/Status-Active%20Research-00D4AA?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-A29BFE?style=flat-square)
![Exchange](https://img.shields.io/badge/Exchange-DSE%20Tanzania-FFB300?style=flat-square)

---

## The Problem

Most investors in the Dar es Salaam Stock Exchange manage portfolios using intuition, spreadsheets, or raw return figures alone. None of that tells you whether the return you earned was worth the risk you took, which stocks are quietly dragging the portfolio down, how concentrated your risk actually is, or what the next 12 months could realistically look like across thousands of market scenarios.

This project fills that gap. Built on the same analytical frameworks used by quantitative hedge funds — Markowitz mean-variance optimization, Monte Carlo simulation, Expected Shortfall, and risk contribution decomposition — it turns a list of stock weights into a complete investment diagnosis: what is working, what is not, and exactly how to reallocate for better risk-adjusted outcomes.

---

## What It Does

### 🔬 Phase 1 — Diagnosis
Computes 13 performance and risk metrics for the total portfolio and for every individual stock, including CAGR, annualized volatility, Sharpe ratio, Sortino ratio, maximum drawdown, Calmar ratio, VaR, and CVaR. This gives a full picture of both return generation and risk exposure at every level.

### 🏷️ Phase 2 — Classification
Automatically labels each stock using a rules-based classification engine:

| Label | Meaning |
|---|---|
| 🚀 **Alpha Generator** | High Sharpe + High CAGR — increase allocation |
| ⚠️ **Risky Performer** | High return but high volatility — monitor closely |
| 🛡️ **Defensive Anchor** | Low vol + low drawdown — portfolio stabilizer |
| 📊 **Stable Hold** | Moderate across all metrics — maintain |
| 🔴 **Value Drag** | Below-median return + above-median risk — reduce or exit |

### ⚙️ Phase 3 — Optimization
Applies Markowitz Mean-Variance Optimization using a Sequential Least Squares Programming (SLSQP) solver to find the portfolio weights that maximize the Sharpe ratio. Weight bounds are constrained between 2% and 40% per stock to prevent degenerate corner solutions. The output is a side-by-side comparison of current versus recommended weights with INCREASE / REDUCE / HOLD signals.

### 📡 Phase 4 — Forward Projection
Runs 10,000 Monte Carlo simulations over a 252-trading-day horizon using the portfolio's historical drift and volatility. Outputs a full distribution of possible future NAV values — P5 (bear case), P25, P50 (median), P75, and P95 (bull case) — along with the probability of profit and probability of a loss exceeding 20%.

### 📊 Phase 5 — Visualization & Report
Generates a five-figure professional dashboard and an auto-written investment report that narrates the findings in precise, institutional language — not just numbers, but conclusions.

---

## Tech Stack

| Library | Purpose |
|---|---|
| `pandas` | Time series construction, return computation, data wrangling |
| `numpy` | Vectorized matrix operations, simulation engine |
| `scipy.optimize` | SLSQP solver for mean-variance portfolio optimization |
| `matplotlib` | All charting — NAV curve, drawdown, rolling vol, scatter |
| `seaborn` | Correlation heatmap with diverging colormap |
| `scipy.stats` | Distribution fitting, statistical tests |

---

## Project Structure

```
dse-quant-engine/
│
├── citadel_portfolio_diagnosis.ipynb   ← Main analytics engine (10 sections)
│
├── data/
│   └── dse_prices.csv                  ← Drop your price data here (optional)
│
├── outputs/
│   ├── DSE_Portfolio_Report_[date].txt ← Auto-generated investment report
│   └── citadel_dashboard.png           ← Exported master dashboard
│
└── README.md
```

---

## How to Run

### Option 1 — Instant Start (Synthetic DSE Data)
Works out of the box. No data file needed.

```bash
git clone https://github.com/yourusername/dse-quant-engine.git
cd dse-quant-engine
pip install pandas numpy matplotlib seaborn scipy
jupyter notebook citadel_portfolio_diagnosis.ipynb
```

Run all cells. The notebook generates realistic synthetic DSE-like price series automatically.

---

### Option 2 — Your Own CSV File
Your CSV should have dates as the index and stock tickers as columns:

```
Date,CRDB,NMB,TPCC,TBL
2020-01-02,280,3400,1800,2200
2020-01-03,282,3390,1812,2215
...
```

Then in **Section 1B**, replace the data generation call with:

```python
prices = pd.read_csv('data/dse_prices.csv', index_col='Date', parse_dates=True)
```

---

### Option 3 — Live Data via Yahoo Finance

```python
import yfinance as yf
prices = yf.download(TICKERS, start=START_DATE, end=END_DATE)['Adj Close']
```

*Note: DSE-listed stocks may not be available on Yahoo Finance. This option works best for cross-listed securities or for adapting the engine to other exchanges.*

---

## Configuration

All key parameters are defined at the top of Section 1. Edit these to match your actual portfolio:

```python
# Your portfolio — stocks and weights (must sum to 1.0)
PORTFOLIO_WEIGHTS = {
    'CRDB':      0.20,
    'NMB':       0.18,
    'TPCC':      0.15,
    'SWISSPORT': 0.12,
    'TOL':       0.10,
    'TBL':       0.10,
    'DCB':       0.08,
    'MAENDELEO': 0.07,
}

START_DATE       = '2020-01-01'    # Analysis start
END_DATE         = '2024-12-31'    # Analysis end
INITIAL_CAPITAL  = 100_000_000     # Starting capital in TZS
RISK_FREE_RATE   = 0.08            # Tanzania T-Bill rate (8%)
CONFIDENCE_LEVEL = 0.95            # VaR / CVaR confidence
MC_SIMULATIONS   = 10_000          # Monte Carlo paths
```

---

## Methodology

This section explains not just *what* the metrics are, but *why* they were chosen.

### Why Sharpe Ratio Over Raw Return
A stock that returns 30% with 40% volatility is inferior to one returning 18% with 8% volatility — but raw return figures will mislead you into preferring the first. The Sharpe ratio, formalized by William Sharpe (1966), divides excess return (above the risk-free rate) by volatility. It forces every return to be expressed per unit of risk taken. This project uses it as the primary ranking criterion and as the optimization objective.

### Why CVaR Over VaR
Value-at-Risk (VaR) at 95% confidence tells you the loss you will *not* exceed on 95% of days — but says nothing about how bad the remaining 5% of days can get. Conditional Value-at-Risk (CVaR), also called Expected Shortfall, is the mean loss in that tail. During market dislocations, it is CVaR that destroys portfolios, not VaR. This project reports both, at 90%, 95%, and 99% confidence levels.

### Why Mean-Variance Optimization
Harry Markowitz (1952) demonstrated that for any set of assets, there exists a portfolio weighting that maximizes return for a given level of risk — the efficient frontier. This engine finds the point on that frontier that maximizes the Sharpe ratio using a constrained numerical solver (SLSQP). Position bounds of [2%, 40%] are enforced to prevent the solver from producing impractical all-or-nothing allocations.

### Why Monte Carlo Over Simple Projection
A single projected return path is a fiction. Markets do not follow straight lines. Monte Carlo simulation generates 10,000 independent return paths drawn from the portfolio's historical distribution, producing a *range* of possible futures rather than a false point estimate. The resulting percentile fan chart is far more honest — and more useful — than any single forecast.

### Why Risk Contribution Decomposition
A stock with a 10% portfolio weight does not necessarily contribute 10% of total portfolio risk. Due to correlations, it may contribute 25% or 3%. Marginal risk contribution analysis — derived from the covariance matrix — reveals the true source of portfolio volatility. This is where most retail portfolio analyses fail completely.

---

## Sample Output

```
══════════════════════════════════════════════════════════════════════════
  PORTFOLIO — MASTER METRICS SUMMARY
══════════════════════════════════════════════════════════════════════════
  CAGR                    +9.43%
  Annualized Volatility   12.87%
  Sharpe Ratio             0.812
  Sortino Ratio            1.143
  Max Drawdown           -18.34%
  Calmar Ratio             0.514
  VaR (95%)               -1.203%
  CVaR (95%)              -1.891%

  Final NAV:   TZS 157,200,000
  Total P&L:   TZS +57,200,000
══════════════════════════════════════════════════════════════════════════

  STOCK RANKING — RISK-ADJUSTED COMPOSITE SCORE
  ──────────────────────────────────────────────
  🥇  TBL        Sharpe 1.241   CAGR +14.2%   → 🚀 ALPHA GENERATOR
  🥈  CRDB       Sharpe 0.934   CAGR +11.7%   → 🚀 ALPHA GENERATOR
  🥉  TOL        Sharpe 0.871   CAGR  +8.1%   → 🛡️  DEFENSIVE ANCHOR
      NMB        Sharpe 0.712   CAGR  +7.4%   → 📊 STABLE HOLD
      SWISSPORT  Sharpe 0.601   CAGR +13.2%   → ⚠️  RISKY PERFORMER
      TPCC       Sharpe 0.318   CAGR  +2.1%   → 🔴 VALUE DRAG
      DCB        Sharpe 0.201   CAGR  -1.8%   → 🔴 VALUE DRAG
```

---

## Limitations & Future Work

**Current limitations worth noting:**

- Synthetic data does not model real DSE market microstructure — including illiquidity, wide bid-ask spreads, and thin order books that affect execution
- Transaction costs, stamp duty, and brokerage fees are not included in return calculations
- The Monte Carlo simulation assumes normally distributed returns. Real markets exhibit fat tails and regime shifts not captured by Gaussian assumptions
- Mean-variance optimization is sensitive to input estimates. Small changes in expected return assumptions can produce meaningfully different weight recommendations

**Planned extensions:**

- [ ] Live DSE data feed integration via web scraping or official API
- [ ] Factor model decomposition (market beta, size, value factors)
- [ ] Hidden Markov Model for bull/bear regime detection
- [ ] Transaction cost-aware rebalancing with turnover constraints
- [ ] Stress testing against specific historical crisis periods (2008, COVID-2020)
- [ ] Streamlit dashboard for non-technical stakeholders

---

## Author

**[Your Name]**  
Quantitative Finance Researcher | Dar es Salaam, Tanzania

This project was built to address a genuine gap: investors on the Dar es Salaam Stock Exchange have access to the same asset class complexity as any other emerging market, but almost none of the institutional analytical infrastructure that investors in developed markets take for granted. This engine is an attempt to change that — bringing rigorous, hedge-fund-grade portfolio diagnostics to the DSE context.

📎 [LinkedIn](https://linkedin.com/in/yourprofile) · [GitHub](https://github.com/yourusername)

---

## License

MIT License — free to use, adapt, and build on with attribution.

---

*Built with Python · Inspired by quantitative research practices at leading global hedge funds · Applied to Tanzania's capital markets*
