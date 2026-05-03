# BTC Strategy Lab

> A power-law-based investment calculator for Bitcoin, accompanied by a multi-method
> statistical investigation of BTC's long-term price dynamics.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Status: Research](https://img.shields.io/badge/Status-Research-orange.svg)](#)
[![Not financial advice](https://img.shields.io/badge/Not-Financial%20Advice-red.svg)](#disclaimer)

---

## Overview

This repository contains the output of a self-directed research project on Bitcoin
price dynamics over 16 years (July 2010 – April 2026, N = 5,760 daily closes). The
project answers two complementary questions:

1. **Can Bitcoin's long-term price be modelled with a few parameters?**
   We find that yes — a Power Law on log-time achieves R² = 0.961, and a cycle-aware
   extension reaches R² = 0.987.

2. **Can statistical deviations from that trend inform an investment strategy?**
   Empirically, σ-deviations from the Power Law trend predict 1-year forward returns
   with strong asymmetry: zones below −1σ have produced very large median returns,
   while zones above +1σ have produced negative returns.

The repository delivers three artefacts described below.

---

## What's in this repository

| File | Description | Size |
|------|-------------|-----:|
| [`btc_strategy_lab.html`](./btc_strategy_lab.html) | Interactive web calculator (single self-contained file, no internet required) | ~50 KB |
| [`btc_strategy_manual.pdf`](./btc_strategy_manual.pdf) | User manual & technical reference for the calculator | ~650 KB |
| [`btc_research_report.pdf`](./btc_research_report.pdf) | Academic-style research report on BTC price dynamics | ~1.8 MB |

### `btc_strategy_lab.html` — the calculator

A single-page HTML application that runs entirely in your browser. No build step,
no server, no internet. Just open the file and use it.

**Inputs:**
- Current BTC price
- Your BTC holdings + cumulative spent (used to derive average buy price)
- Base monthly DCA budget
- Date

**Outputs:**
- Recommended action (BUY / STOP / SELL) with a specific dollar amount
- Current σ-zone with a visual gradient bar
- Power Law fair price for the selected date
- Position metrics: avg buy, current value, unrealized P/L
- Empirical 1-year median return for your zone (from historical data)
- Strategy curve in σ-coordinates with three coloured zones
- Reference table at 11 σ-points around your current position

**Optional risk controls** (gated behind a checkbox):
- `risk factor` — scale all recommendations by 0.1×–2.0×
- `max monthly` — hard ceiling on dollar amount
- `unconstrained curve overlay` — dashed line showing what the strategy would recommend
  without your constraints

**Privacy:** All calculations are local. Nothing leaves your browser.

### `btc_strategy_manual.pdf` — user manual

A 14-page user guide with screenshots and worked examples. Covers:
- How the strategy function works mathematically
- Field-by-field walkthrough of the application interface
- Risk control mechanics
- Three concrete worked examples (current undervaluation, strong undervaluation, bubble)
- Limitations and honest disclosures

### `btc_research_report.pdf` — research report

A 15-page academic-style report on the underlying price dynamics, structured as a
scientific paper:

1. **Power Law trend (M2):** baseline model, R² = 0.961
2. **Cycle-aware extension (M13):** three damped harmonics with logarithmically drifting
   period, R² = 0.987, MAPE = 31.7%
3. **Spectral analysis:** discrete Fourier transform identifies a dominant 1,440-day
   peak (≈3.94 years, close to the 1,461-day halving interval); Morlet wavelet shows
   cycle compression over time
4. **Random Matrix Theory:** weekly-scale (W = 5) return correlations show moderate
   deviation from Marchenko-Pastur null (ratio ≈ 1.20); monthly+ scales are
   statistically indistinguishable from random matrix noise
5. **Hurst analysis:** H ≈ 0.63 in returns (mild persistence), H ≈ 1.01 in detrended
   residuals (strong cyclical structure)
6. **Forward projections through 2050:** central forecast and bootstrap confidence
   intervals

---

## How the strategy works

### The mathematical core

```
σ = (log₁₀(price) − log₁₀(trend)) / 0.3022

if 1.0 < σ < 1.5:    multiplier = 0                        (STOP zone)
if σ ≤ 1.0:          multiplier = max(e^(−0.69σ) − 0.5, 0)  (BUY zone)
if σ ≥ 1.5:          multiplier = −0.5 · (σ − 1.5)          (SELL zone)

amount = round(multiplier × base_monthly)
```

Where `trend` is the Power Law fair price for today's date:
```
log₁₀(trend) = −16.4774 + 5.6800 · log₁₀(days_since_genesis)
```

### Why this beats naive DCA

Across walk-forward backtests with six different start dates (2018–2023), the adaptive
strategy beat naive monthly DCA in **6 of 6** scenarios. In Monte Carlo simulations
covering 10-year horizons:

| Strategy | Median ROI | 5%–95% range |
|----------|-----------:|-------------:|
| Naive DCA | +403% | $200k–$390k |
| Adaptive (this repo) | **+531%** | **$272k–$504k** |

(based on $500/month over 10 years = $60k total invested)

The improvement of ~128 percentage points comes from concentrating purchases at
statistically cheap moments and pausing during overvaluation, rather than committing
the same amount each month regardless of price.

---

## Quick start

### Use the calculator

1. Download [`btc_strategy_lab.html`](./btc_strategy_lab.html)
2. Open it in any modern browser (Chrome, Firefox, Safari, Edge)
3. Override the auto-populated price with the real BTC market price
4. Enter your BTC holdings and cumulative spent (or leave at 0 for a fresh start)
5. Set your base monthly amount

That's it. The recommendation updates as you type.

### Read the documents

- For practical use → start with `btc_strategy_manual.pdf`
- For technical understanding → start with `btc_research_report.pdf`

---

## Methodology summary

| Method | Purpose | Key finding |
|--------|---------|-------------|
| Nonlinear least squares | Fit Power Law trend | R² = 0.961, σ_log = 0.302 |
| Damped harmonic regression | Capture cyclical residuals | R² = 0.987, cycle ≈ 3.94 yr |
| Discrete Fourier transform | Identify cycle frequencies | Dominant peak at 1,440 days |
| Morlet wavelet transform | Time-frequency localisation | Confirms cycle compression |
| Random Matrix Theory | Detect short-term momentum | Weekly: ratio 1.20× MP edge |
| R/S analysis | Long-range dependence | H ≈ 0.63 (returns), H ≈ 1.01 (residuals) |
| Walk-forward backtest | Out-of-sample validation | 6/6 wins vs naive DCA |
| Monte Carlo (500 sims) | 10-year forecast distribution | Median ROI +531% |

All computations were performed in Python 3 using NumPy, SciPy, and standard
scientific libraries.

---

## Limitations

This is a research project, not a trading system. Several caveats apply:

- **Small effective sample.** Bitcoin has only existed through ~4 complete halving
  cycles. Cycle-specific parameters are estimated from a small effective N.
- **Structural break risk.** The Power Law assumes mean-reversion to a trend that
  has held for 16 years. A regulatory shock, technological disruption, or
  fundamental change in adoption dynamics could invalidate it.
- **Psychological execution risk.** The strategy recommends buying aggressively
  during deep drawdowns, which is psychologically difficult. The risk-controlled
  version with a hard cap is more realistic in practice.
- **No model comparison.** We assume a Power Law functional form rather than testing
  alternatives (logistic, exponential-with-saturation). This would be a useful
  extension.

---

## Disclaimer

**This is not financial advice.** The strategy is mathematically sound and historically
validated, but past performance does not guarantee future results. Cryptocurrency
investment involves substantial risk of total loss.

You alone are responsible for your investment decisions. Only invest amounts you
can afford to lose entirely. The author accepts no liability for losses incurred
from use of this material.

---

## License

MIT. See [LICENSE](./LICENSE) for details.

You are free to use, modify, and distribute this work, with attribution. The PDFs
and the HTML calculator may be shared as-is.

---

## Citation

If you reference this work in academic or professional contexts, please cite as:

```
BTC Strategy Lab (2026). A power-law-based investment calculator for Bitcoin
with a multi-method statistical investigation of long-term price dynamics.
GitHub repository: <repo URL>
```

---

## Acknowledgements

This work builds on power-law modelling of Bitcoin established by independent
researchers in cryptocurrency analysis (notably Santostasi 2024), and on standard
quantitative finance methods including Random Matrix Theory (Laloux et al. 1999;
Plerou et al. 2002), Hurst-exponent estimation (Hurst 1951), and wavelet analysis
(Torrence & Compo 1998).
