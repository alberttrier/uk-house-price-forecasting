# UK-house-price-forecasting
**Question:** Can simple time-series models forecast UK house price growth better than a naive guess?

## Data
- Source: UK House Price Index, full file, gov.uk (August 2026 release)
- Column: AveragePriceSA (seasonally adjusted)
- Regions: London, Scotland, United Kingdom
- Sample: January 2005 to August 2026, monthly

## Why these choices
- **Growth instead of price levels.** House prices rise over time, so price levels drift upwards. Models work better on monthly growth, which moves around a stable average. I use log growth: 100 × (log price this month − log price last month).
- **Seasonally adjusted prices.** There exists a stable volatility-pattern in real-estate sales, more houses statistically sell in Summer and Autumn. The adjusted series removes this pattern, so the models focus on real changes. Other influencing patterns also assumed but not yet identified nor adjusted.
- **Start in 2005.** The UK HPI covers England and Wales from 1995, Scotland from 2004 and Northern Ireland from 2005, so the UK series only starts in 2005. I start all regions in 2005 so they cover the same period.
- **Older data removed.** The file goes back to 1968, but the early data shows flat months followed by sudden jumps, which is not real monthly movement. Distorts results and measure of quality of forecasting mechanism, therefore removed.

## First observations
- **Scotland is noisier** than London and the UK. A likely reason: fewer sales are recorded each month, so the average moves more from month to month.
- **A large spike in 2020–21.** This lines up with the temporary stamp duty cuts during the pandemic, when many buyers rushed to complete before the tax break ended.
- **2008–09** shows the sharpest falls in all three regions (global financial crisis), fire-sales including real estate driving down prices.

## Models
_To come._

## How I test the models
_To come._

## Results
_To come._

## Limitations
- The most recent months are provisional and are likely to be revised as more sales are registered.
- The index only counts completed sales, so it reacts to market changes with a delay.
- Short sample: about 250 months per region.

## Next steps
_To come._
