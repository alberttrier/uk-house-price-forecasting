# Research Log

How this project developed: the questions I asked, the decisions I made and why, and the ideas I want to follow up. The clean results are in the [README](README.md).

---

## 5 November 2026: Data and first patterns

**Choosing the data.** The UK HPI full file goes back to 1968. My first chart showed flat months followed by sudden jumps before the mid-1990s, which is not real monthly movement, so I cut the old data. When I checked where each region's seasonally adjusted series starts, London began in 1995, Scotland in 2004 and the UK in 2005. The gov.uk methodology page confirmed why: Scotland's data joins in 2004 and Northern Ireland's in 2005, so a UK-wide series cannot start earlier. I started all regions in 2005 so they are compared over the same period.

**Does last month predict this month?** I measured autocorrelation. My first question was whether the sign matters or only the size. It matters: London's +0.22 means momentum, while Scotland's −0.18 means the series zig-zags. Similar size, opposite meaning.

**Lag 1 is not the whole story.** The UK's lag-1 value was almost zero, but its chart showed the strongest link in the whole data at lag 3 (about 0.56). Looking at one number would have hidden this. In all regions, the significant links sat in the first ~6 months, which suggests momentum lasts about half a year. This directly shapes the forecasting models: they should use about 3–6 past months.

## 5 November 2026: Can I measure how likely a pattern is noise?

**Three approaches.** The blue confidence band in the charts already answers this visually. The Ljung–Box test answers it for many lags at once. And I built my own shuffle test: if the order of months does not matter, shuffling them should not change anything. I shuffled the UK months 10,000 times; the real lag-3 value was far outside everything chance produced.

**My idea of combining lags by squaring and adding** turned out to be the same principle that the Ljung–Box test uses: it adds up squared autocorrelations across lags.

**Reading the Ljung–Box results by number of lags.** For London and the UK, the evidence is strongest at 6 lags and gets slightly weaker at 12, which confirms the six-month pattern from a second angle. Scotland's evidence keeps growing up to 12 lags. To check what lags 7–12 add, I used the rule that pure noise adds about 1 to the test statistic per extra lag: the UK added about the noise amount (+5.5 for 6 extra lags), London about twice that (+12.9), Scotland well above it (+15.4).

## 5–6 November 2026: Designing my own test

**The problem with plain shuffling.** It mixes calm months with crisis months (2008, 2020–21), which may make the real pattern look stronger than it is.

**My design: a volatility-aware shuffle.** Measure each month's volatility, split months into groups from calm to turbulent, and shuffle only inside each group.

Decisions and reasons:
- **12-month volatility window.** Six months gives an unstable estimate and overlaps with the ~6-month momentum. Twelve months is more stable, longer than the momentum, and contains whole numbers of possible 3-month cycles and of years, so no cycle is cut in half.
- **Crises kept in.** Removing crashes before measuring volatility would defeat the purpose of the grouping.
- **2, 3 and 4 groups, fixed in advance, all reported.** Too many groups leave too few months to shuffle; too few cannot separate calm from crisis. Choosing the grouping after seeing results would make the test meaningless.
- **One change at a time.** I planned the robustness checks as a 2 × 2 design: normal vs rank-based statistic (rank-based is my median idea: it stops a few extreme months dominating), and plain vs volatility-aware shuffle. Each test differs from its neighbour in exactly one thing, so any difference has a clear cause.

**My prediction before running it:** if crisis months inflated the evidence, the p-values should rise under the stricter test.

**Result.** London and the UK stayed at p < 0.0001 in every version: the momentum is robust. Scotland's p-value rose from 0.005 to 0.015–0.032 as the groups got finer, exactly the pattern I predicted for noisier data. Part of Scotland's evidence came from mixing calm and crisis periods, but it still passes at the 5% level.

## Ideas for later

**Do busier markets process information faster?** If effects play out per transaction rather than per month, busier markets should show shorter momentum. London vs the UK does not show this: momentum lasts about six months in both, and London is more persistent month to month. But two series cannot test it, and London is part of the UK. A proper test would compare many regions, measuring momentum length against monthly sales volume, i.e. measuring markets in transactions instead of calendar time.

**Assume the pattern is everywhere, measure how hidden it is.** Instead of asking whether momentum exists in each region, assume a shared momentum plus region-specific noise, and estimate the signal-to-noise ratio (a state-space model). A testable prediction: the noise in an average price should shrink with the number of sales, roughly in proportion to 1 / √(sales). If many regions fit that one rule, the assumption becomes much more credible. The real check would be whether it forecasts better than models without the assumption.

**The market as overlapping patterns.** Seasonality is one pattern; there may be others that together look like noise. Spectral analysis could find cycles of other lengths (for example the lag-3 effect), and factor models could find shared drivers such as interest rates or tax changes. With about 260 months only a few strong components can be found reliably, so named candidates should be tested rather than searched for blindly.

**Big cities vs their surroundings.** How do large cities move relative to their surrounding regions, and where does the link fade? This connects to the UK "ripple effect" and to research on "superstar cities". Starting point: UK cities vs their regions within the same dataset, before any international comparison, where differences in index methods would distort results.

**Normal times vs extreme times.** Markets may behave differently in calm and crisis periods (regime switching). Separating "normal" forecasts from "extreme event" signals could show when to shift towards protection against crises.
