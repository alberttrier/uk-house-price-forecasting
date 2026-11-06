# UK House Price Dynamics

**Question:** Does UK house price growth have momentum, and can simple time-series models forecast it better than a naive guess?

**Status:** In progress. Steps 1–2 (data, momentum and noise tests) are done. Next: a SQL-based anomaly detector, then an out-of-sample forecast comparison.

How I developed the ideas and decisions behind this project is documented in [RESEARCH_LOG.md](RESEARCH_LOG.md).

---

## Data

- **Source:** UK House Price Index (UK HPI), full file, gov.uk, August 2026 release
- **Measure:** `AveragePriceSA`, the seasonally adjusted average price
- **Regions:** London, Scotland, United Kingdom
- **Sample:** January 2005 – August 2026, monthly (about 260 months)

## Key choices

- **Growth instead of price levels.** Prices drift upwards over time; models work better on monthly growth, which moves around a stable average. I use log growth: 100 × (log price this month − log price last month).
- **Seasonally adjusted prices.** Removes the spring/summer selling pattern so the analysis focuses on real changes.
- **Start in 2005.** The UK HPI covers England and Wales from 1995, Scotland from 2004 and Northern Ireland from 2005, so the UK series only starts in 2005. All regions use the same period.
- **Older data removed.** The file goes back to 1968, but the early data shows flat months followed by sudden jumps, which is not real monthly movement.

## Findings so far

### 1. Momentum lasts about six months

| Region | Lag-1 autocorrelation | Lag-3 autocorrelation |
|---|---|---|
| London | 0.22 | 0.41 |
| Scotland | −0.18 | 0.17 |
| United Kingdom | 0.01 | 0.56 |

- In all three regions, growth is significantly linked to growth in the past ~6 months, but not further back.
- **Lag 1 alone is misleading:** the UK's lag-1 value is almost zero, but its lag-3 value is the strongest in the data.
- **Scotland zig-zags** month to month (negative lag 1), likely because fewer recorded sales make the monthly average noisy, but it shows positive links at lags 2–8.
- **Why lag 3 is strongest is unclear.** It may reflect the market or the way the index is built; see Next steps.

### 2. It is not noise

| Test | London | Scotland | UK |
|---|---|---|---|
| Ljung–Box, 6 lags (p-value) | 2.7e-28 | 3.5e-06 | 9.3e-24 |
| Plain shuffle, lag 3 (p-value) | < 0.0001 | 0.0052 | < 0.0001 |
| Volatility-aware shuffle, 2 / 3 / 4 groups | < 0.0001 (all) | 0.015 / 0.024 / 0.032 | < 0.0001 (all) |

- **Shuffle tests** destroy the time order 10,000 times and check how often chance matches the real value.
- The **volatility-aware shuffle** (my design) only swaps months with similar volatility, so calm and crisis periods are not mixed. London and the UK stay significant in every version. Scotland weakens as the test gets stricter but still passes at the 5% level.
- With about 260 months, noise autocorrelations spread by roughly 0.06, so the UK's 0.56 lies about 9 spreads from zero.

### 3. How long the pattern lasts differs by region

The Ljung–Box p-value is smallest at 6 lags for London and the UK, then rises at 12 lags. Scotland's evidence keeps growing up to 12 lags, likely because month-to-month noise hides its short-term momentum.

## Limitations

- The most recent months are provisional and are likely to be revised as more sales are registered.
- The index only counts completed sales, so it reacts to market changes with a delay.
- Short sample: about 260 months per region.
- I tested lag 3 because it was the strongest bar, which makes it more likely to look significant by chance.
- Crises (2008, 2020–21) break the equal-volatility assumption of the Ljung–Box test, so exact p-values should not be taken literally.
- "Not noise" does not prove a market effect; the pattern may partly come from how the index is constructed.

## Next steps

- **Anomaly detector in SQL:** flag months whose growth is unusual relative to the previous 12 months.
- **Forecast comparison:** naive, historical average, AR and AR + Bank Rate models, tested out of sample with a rolling window and a Diebold–Mariano test.
- **Robustness:** rank-based versions of the shuffle tests; repeat the lag-3 check on non-adjusted prices.
- Further research ideas are in the [research log](RESEARCH_LOG.md).

## Tools

Python (pandas, NumPy, statsmodels, matplotlib) in Google Colab. I used Claude (AI) to learn methods and debug code; the research questions, test designs and interpretation are my own.
