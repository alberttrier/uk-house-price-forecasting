Research Log

How this project developed: the questions I asked, the decisions I made and why, my predictions, and the ideas I want to follow up. Clean results are in the README.

Data and first patterns

Choosing the data. The UK HPI full file goes back to 1968. First chart showed flat months followed by sudden jumps before the mid-1990s, not real monthly movement, therefore incomplete data and cut. Checking where each region’s seasonally adjusted series starts: London 1995, Scotland 2004, UK 2005. The gov.uk methodology page confirms why: Scotland’s data joins in 2004, Northern Ireland’s in 2005, so no UK-wide series before 2005. All regions start in 2005 to be comparable.
Does last month predict this month? Measured autocorrelation. First question: does the sign matter or only the size? The sign matters. London’s +0.22 means momentum, Scotland’s −0.18 means the series zig-zags. Similar size, opposite meaning.
Lag 1 is not the whole story. The UK’s lag-1 value is almost zero, its chart shows the strongest link of the whole data at lag 3 (about 0.56). One number would have hidden this.
About six months. In all regions the significant links sit in the first ~6 months, momentum seems to last about half a year. Directly shapes the forecasting models: about 3–6 past months as input.
Can I measure how likely a pattern is noise?

Three approaches. The blue confidence band in the charts answers it visually, the Ljung–Box test for many lags at once, and my own shuffle test: if the order of months does not matter, shuffling them should not change anything. Shuffled the UK months 10,000 times, the real lag-3 value was far outside everything chance produced, thus the pattern is not solely noise.
Squaring and adding lags. My idea of combining several lags by squaring and adding turned out to be the same principle the Ljung–Box test uses.
Distance. What counts is the distance relative to the spread of the shuffled values: about 0.06 with 260 months, so 0.56 is about 9 spreads away.
Reading Ljung–Box by number of lags. London and the UK: strongest at 6 lags, slightly weaker at 12, the six-month pattern confirmed from a second angle. Scotland: keeps growing up to 12 lags. Rule of thumb: pure noise adds about 1 to the test statistic per extra lag. From 6 to 12 lags the UK added about the noise amount (+5.5), London about twice (+12.9), Scotland well above it (+15.4).
Designing my own test

Problem with plain shuffling. Mixes calm months with crisis months (2008, 2020–21), may make the pattern look stronger than it is.
My design: volatility-aware shuffle. Measure each month’s volatility, split months into groups from calm to turbulent, shuffle only inside each group. The number of groups is variable: with too few groups we cannot separate calm from crisis, with too many the data pool per group thins out, too few months to shuffle and no trustworthy average.
Decisions and reasons:
12-month volatility window. Six months: unstable estimate, overlaps with the ~6-month momentum. Twelve months: more stable, longer than the momentum, contains whole 3-month cycles and a full year, so no cycle cut in half.
Crises kept in. Removing crashes before measuring volatility would defeat the purpose of the grouping.
2, 3 and 4 groups, fixed in advance, all reported. Choosing after seeing results would make the test meaningless.
Same lag for all regions. Lag 3 everywhere for comparability, even though it is not Scotland’s strongest lag.
One change at a time. Robustness checks planned as a 2 × 2 design: normal vs rank-based statistic (my median idea, stops a few extreme months from dominating) and plain vs volatility-aware shuffle. Each test differs from its neighbour in exactly one thing, so any difference has a clear cause.
Prediction before running: if crisis months inflated the evidence, p-values should rise under the stricter test.
Result. London and the UK unchanged at p < 0.0001 in every version, the momentum is robust. Scotland rose from 0.005 to 0.015–0.032 as groups got finer. Part of its evidence came from mixing calm and crisis periods, still passes at 5%.
Planning an anomaly detector

Question. Which months were unusual compared with the recent past? Turns the project towards risk analysis: flag unusual months, then check each flag, real event, data problem or false alarm.
Approaches compared. Fixed growth threshold (ignores context), rolling z-score (month vs average and spread of the previous 12 months), robust z-score (median instead of average, extreme months cannot distort what counts as normal), model-based (flags months where a forecast misses badly), cross-region comparison.
Decisions:
Main detector rolling z-score, robustness check robust (median-based) z-score. A month flagged by both is a strong signal.
Only past months: the window is the 12 months before each month, not including it, like a real-time system, and an extreme month cannot partly hide itself.
Threshold fixed in advance: 3 spreads.
Built in SQL (DuckDB) with window functions.
Planned addition: model-based detector once the forecasting models exist.
First SQL observations

2008 and 2009. 2008 negative almost everywhere, 2009 already positive. Prices recovered within 2009, yearly averages hide what happens inside the year.
2023 negative for London and the UK. Fits the sharp interest rate rises of 2022–23, argument for adding the Bank Rate to the forecasting models.
Brexit. No clear impact in yearly averages, in comparison to other sectors not surprising. One could also have expected more investment, UK real estate not exposed to EU trade barriers. Effects ambiguous though: uncertainty may scare buyers, the weaker pound made UK property cheaper for foreigners. Also confounded by the April 2016 surcharge on second homes. Worth checking London in 2017–2019 separately.
Worst months. Mostly 2020–21 and policy months, Scotland appears often. Not clustered, except the expected 2008 cluster. Interestingly, the 2008 months are not the most extreme ones: prices fell less instantly, more over a longer run of bad months.
Timing shocks vs fundamental shocks

Observation. Almost all non-2008 extreme months come right after a large move in the opposite direction. In 2008 bad months followed bad months.
Hypothesis. Policy changes make buying more or less opportune at a certain moment. A firm buying a lot of inventory one year buys below average the next, without suffering financially. Same with deadlines: they change when people buy, not how much, so spike followed by reversal (timing shock). 2008 was different, the whole market sank out of control, demand really fell (fundamental shock).
Test (SQL, LEAD). For the 8 largest moves per region: this month plus next month. Reversal gives a sum near 0, persistent fall a clearly negative sum.
Prediction: policy months near 0, 2008 months clearly negative.
Results.
Policy deadlines reverse: UK June 2021 +4.62 then −5.99 (sum −1.36), September 2021 +3.14 then −2.24 (+0.90), both steps of the end of England’s stamp duty holiday. Scotland March 2015 +4.41 then −5.12 (−0.71), its new property tax from April 2015. London March 2020 +2.21 then −2.13 (+0.08), first lockdown.
2008 persists: Scotland June 2008 −3.99 then −1.55 (−5.54), UK November 2008 −2.35 then −1.47 (−3.83).
Refinements.
The test has to be read on the spike month. On the dip month the next month is usually normal, so the sum stays close to the dip.
A long policy window behaves differently from a deadline: Scotland September 2020 (+4.19, then +2.29) came during a tax holiday lasting months, so demand rose over months instead of one spike.
Part of a pre-deadline spike may come from which homes sold (e.g. more homes under the tax-free limit), not only from what homes are worth.
Unexplained: London, May 2011 (−3.91, then +0.77).
Consequence for the detector. Timing shocks are sudden, should be flagged easily. Slow crashes like 2008 maybe not: the rolling window moves down with the crash. Drawdown (distance below the previous peak) could catch them.
Anomaly detector results

Setup. Rolling z-score in SQL, previous 12 months, flag if |z| > 3, all fixed before running.
My predictions before running:
Scotland measured only against itself shows fewer extremes, but because it is more volatile and follows long patterns less, it will have higher z-values on average.
The 2008 months score slightly higher than others, they still stand out in 260 months, but only marginally, since the usual spikes and reversals appear often enough in the data.
Results. 31 flags (London 7, Scotland 11, UK 13). Average |z|: London 1.0, Scotland 0.9, UK 1.1. Largest |z|: London 6.74, Scotland 6.18, UK 9.17 (April 2025).
Prediction 1 wrong, for a useful reason. Scotland has the lowest z-scores. Its constant noise makes its 12-month spread large, so even big moves look small relative to it. The UK average is smooth, so sudden shocks stand out enormously. Noisy series hide shocks, smooth series expose them.
Prediction 2 wrong too. The largest 2008–09 z-score (London 3.33, UK 4.42, Scotland 2.89) is well below each region’s maximum. Deep crash months mostly not flagged at all. Flagged is the start of the crash (UK, December 2007 and January 2008) and the turn to recovery (London, May and June 2009). Confirms my earlier point that 2008 was a long run of bad months: the window moved down with the crash. The detector catches changes of direction, not slow declines.
Further findings.
Policy deadlines dominate the flags: Scotland 2015, April 2016 surcharge, 2020 lockdown, 2020–21 tax holidays, end of the temporary stamp duty cuts in April 2025.
Same shock looks bigger in calm times: April 2025 (−3.1%) scores −9.17, July 2021 (−6.0%) only −4.61, the months before July 2021 were already volatile.
31 flags is about three times what chance would give (around 9), all regions have fat tails.
Open: London May 2011 (z = −6.74) still unexplained. UK June 2026 is provisional data and may disappear after revisions.
Conclusion. A risk system needs at least two kinds of signal: a fast one for sudden shocks (z-score) and a slow one for gradual declines (e.g. drawdown).
Combined-lag test

Problem. Lag 3 was chosen because it was the UK’s strongest bar, a choice made after seeing the data. Picking one lag from many is like picking one clock from a pile, the one that looks best is partly the luckiest.
Idea. Instead of picking one, combine all lags 1–6. Equal weights, fixed in advance, so no weight can be tuned to the results. Squared, so Scotland’s negative lag 1 and positive later lags cannot cancel out.
Prediction: Scotland’s pattern is spread across several lags, so combining them makes it less likely to be chance, p below its lag-3 result of 0.0052.
Result. London 0.545, Scotland 0.138, UK 0.463, all p < 0.0001. Prediction confirmed: Scotland went from 0.0052 to below 0.0001. Pure noise would give about 0.024, Scotland sits about six times above.
Limit. Combining helps against picking a lucky lag, not against problems all lags share. If the pattern comes from how the index is built, every lag carries it, only different data (e.g. non-adjusted prices) can test that.
Ideas for later

Do busier markets process information faster? If effects play out per transaction rather than per month, busier markets should show shorter momentum. London vs the UK does not show this: momentum about six months in both, London more persistent month to month. Two series cannot test it though, and London is part of the UK. Proper test: many regions, momentum length against monthly sales volume, measuring markets in transactions instead of calendar time.
Assume the pattern everywhere, measure how hidden it is. Instead of asking whether momentum exists in each region, assume a shared momentum plus region-specific noise and measure the signal-to-noise ratio (state-space model). Testable: the noise in an average price should shrink with the number of sales, roughly with 1 / √(sales). If many regions fit one such general formula, the assumption becomes more credible. Real check: does it forecast better than models without the assumption.
The market as overlapping patterns. Seasonality is one pattern, others may exist that together look like unreadable noise. Spectral analysis could find cycles of other lengths (e.g. the lag-3 effect), factor models shared drivers like interest rates or tax changes, and their weights. With about 260 months only a few strong components can be found reliably, so named candidates should be tested, not searched for blindly.
Big cities vs their surroundings. How do very large cities move relative to their surrounding regions, with which factors (population, prices, income, cost of living, development), and where does the link fade? Connects to the UK “ripple effect” and research on “superstar cities”. Starting point: UK cities vs their regions within the same dataset, before going international, where different index methods would distort results.
Normal times vs extreme times. Markets may behave differently in calm and crisis periods (regime switching). Splitting normal forecasts from extreme-event signals could show when to shift towards protection against crises. Even with many false alarms this can pay off, as long as each hedge is cheap compared with the crisis payoff.
Does the curve move or shift? A movement along the curve swings back (sums near 0), a change in the market’s structure should leave a lasting new level. One event can be both: Scotland’s 2015 tax caused a timing spike but may also have permanently changed prices of expensive homes, their tax rose for good. Test: event study, price level 12 months before and after each event, back to the old trend or a new one?
The market as a many-dimensional machine. What I measure, like a price index, is a result variable: the point where many effect variables meet and intertwine, like a point on a graph with far more than three axes. The real goal is to dissect these layers and find the rules each effect variable follows, not only observe where they combine. A simple movement may be the same machine with most variables fixed in the short run, or with the affected variables moving in harmony. In economics this is the identification problem: isolating one cause needs variation that moves that cause alone. Possible tools: natural experiments (England and Scotland changed taxes at different times, one can serve as comparison for the other, difference-in-differences), structural vector autoregressions (separating shocks and tracing how each spreads through the system), factor models (hidden common drivers behind many series).
