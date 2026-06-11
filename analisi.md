# Analisi — What Makes a Runner Fast?

**Course:** Scientific Visualization — University of Milan 2025–2026
**Project:** Multi-modal data visualization study of marathon performance factors

---

## 1. Data Sources and Integration

### 1.1 Datasets Used

Three datasets of fundamentally different modalities were combined into a single analytical foundation:

| Modality | Dataset | Format | Integration Key |
|----------|---------|--------|----------------|
| Structured tabular | Marathon Results 2000–2019 (Zenodo) | CSV | city + race_date |
| Structured tabular | Strava training data — 36,412 athletes | CSV | gender × age_group × city × year |
| Semi-structured API | Race-day weather (Open-Meteo) | JSON / REST | city + race_date |

### 1.2 Preprocessing Steps

Before joining, each dataset was cleaned independently:

**Marathon data:** Finish time strings (e.g. `"3:42:15"`) were parsed into decimal minutes. City names were standardised. Rows with physiologically impossible times (<60 min or >720 min) were removed. An `age_group` column was derived from raw age to enable the Strava join.

**Strava data:** The `major` field (e.g. `"BOSTON 2019, LONDON 2018"`) was parsed to extract each athlete's target race and year. Weekly training volume was derived from monthly distance ÷ 4.33 weeks/month.

**Weather data:** Fetched once per unique `(city, race_date)` pair via the Open-Meteo historical API and cached locally in `weather_cache.json` to avoid redundant API calls on subsequent notebook runs.

### 1.3 Integration Steps

**Step 1 — Race results + weather:**
For each unique `(city, race_date)` pair in the marathon dataset, the Open-Meteo API returned daily mean temperature, max temperature, precipitation, and wind speed. Merged on `city + race_date` → adds 4 weather columns to every runner's row.

**Step 2 — Race results + Strava training volume:**
This is the multi-modal integration step. The Strava dataset has no individual runner IDs that link to the marathon dataset, so the join operates at **group level**. For every unique combination of `(gender, age_group, city, year)`, the median weekly training km of matching Strava athletes is computed and assigned to every marathon finisher in that demographic bucket.

In concrete terms: every male runner aged 35–54 who finished Boston 2019 receives the same `strava_weekly_km` value — the median of all Strava male athletes aged 35–54 who listed "BOSTON 2019" in their `major` field. This is dataset enrichment, not individual matching. The assigned value represents the typical preparation of a demographic cohort, not any specific runner's training.

**Result:** `data/processed/master.csv` — 2.85 million rows, 25 columns covering runner demographics, finish time, weather conditions, and group-level training volume.

---

## 2. Design Choices and Course Concept Justifications

### Visualization 1 — Age vs. Finish Time (Scatter + LOESS)

**Design choices:**
- Scatter plot with alpha=0.07 to handle overplotting across 50,000 sampled points (Tufte: show the data without distortion)
- Binned median + `uniform_filter1d` smoothing to approximate a LOESS curve — reveals the non-linear age–performance relationship (Stephen Few: don't force a linear model where the pattern is curved)
- Two colors (one per gender) using preattentive color hue — viewers distinguish M/F series in <250ms without reading a legend (Lezione 3a)
- Direct annotation of peak age eliminates manual reading of the minimum — reduces working memory load

**Course concepts:** Preattentive attributes (color hue), data-ink ratio (alpha scatter, no spines), graphical integrity (untruncated Y-axis), iconic/working memory (Lezione 3a).

### Visualization 3 — Temperature Effect on Finish Times (Scatter + Regression)

**Design choices:**
- 95% confidence band around the regression line shows uncertainty honestly — a regression line without a CI implies false precision (Tufte: graphical integrity)
- Single color scatter (no categorical gender split) — the question is about temperature alone; splitting by gender would invite the wrong comparison
- Title states the finding, not the variable names — moves the viewer directly to the cognition stage of the Sensation → Perception → Cognition pipeline

**Course concepts:** Graphical integrity, Sensation → Perception → Cognition pipeline (Lezione 4), data-ink ratio.

### Visualization 4 — Running Culture by Country (Choropleth)

**Design choices:**
- Data source is `run_ww_2019_w.csv` (Strava), not master.csv — nationality is only available in the Strava dataset; the marathon dataset covers US-only venues and cannot provide performance by nationality
- Median weekly training km used as proxy for running culture intensity, not race performance — the title and subtitle make this limitation explicit (honest framing)
- `nunique('athlete')` for country filtering, not row count — each athlete has ~52 weekly rows, so row-counting inflates small countries
- Sequential colormap (Blues) for the continuous variable — never rainbow (Lezione 3b)
- Countries with <20 unique athletes shown in neutral grey — honest signalling of low statistical confidence (Tufte)

**Course concepts:** Colormaps (sequential for continuous data, Lezione 3b), graphical integrity, long-term memory schemas (choropleth leverages geographic knowledge, Lezione 3a).

### Visualization 5 — Performance Trend Over Time (2000–2019)

**Design choices:**
- IQR shaded band behind the median line — shows distributional spread, not just central tendency (Tufte: show many numbers in small space)
- Dual-panel layout: time trend above, participation count below, sharing an X-axis — applies the Overview+Detail interaction principle (Lezione 2)
- Bar chart for participation count (not a line) — counts are discrete and non-negative; a line would imply a continuous process (Stephen Few: match chart type to data semantics)
- First and last values annotated directly on the chart — avoids asking the viewer to read the Y-axis twice and subtract

**Course concepts:** Data-ink ratio, Overview+Detail (Lezione 2), Gestalt continuity (linked axes), Stephen Few.

### Visualization 6 — Training Volume vs. Finish Time (Bubble Chart)

**Design choices:**
- Data binned into a 10-km × 10-min grid; bubble area encodes runner count in each bin — reduces 700k points to a readable density map (Tufte: show many numbers in small space)
- Bubble area ∝ count, not radius — using radius would inflate large groups by π (graphical integrity)
- Sequential colormap (Blues_r) on finish time bins — consistent with Viz 4's convention that darker = faster, reducing working memory load across charts

**Course concepts:** Tufte multi-level detail, graphical integrity (area encoding), consistent palette (Lezione 3a working memory).

### Visualization 7 — Distribution of Finish Times (Histogram + KDE)

**Design choices:**
- KDE overlay on histogram: the histogram bars show raw bin counts; the KDE reveals the underlying continuous shape more honestly (Stephen Few)
- Freedman–Diaconis bin width (IQR-based) — data-driven, avoids the arbitrary spikes and gaps that appear with default bin counts
- Overlapping fills at alpha=0.4 — Gestalt figure/ground allows either series to be read without hiding the other
- Density (not count) on the Y-axis — makes M/F groups comparable despite different sample sizes (1.7M men vs 1.15M women)
- Dashed median lines with direct text labels — removes the need to read axis values mentally

**Course concepts:** Stephen Few chart design, Gestalt figure/ground (Lezione 4), iconic memory (distribution shape is retained instantly), data-ink ratio.

---

## 3. Findings

### Q1 — Does age determine peak running performance?

Yes, strongly but non-linearly. Both men and women peak between ages 28–34. After 35 the decline is gradual at first, then accelerates sharply past 50. By age 70, median finish time is more than an hour slower than at peak. A linear regression would miss this shape entirely — LOESS smoothing was essential to reveal it.

### Q3 — How much does race-day temperature affect finish times?

Race-day mean temperature correlates with finish time at r = 0.23 (p ≈ 0). Each additional °C adds roughly 3 minutes to the median finish time. The effect is real but moderate — individual fitness dominates within any temperature band. Max temperature is an even stronger predictor (r = 0.24), suggesting heat accumulated during the race matters more than the morning starting temperature.

### Q4 — Which countries produce the fastest recreational runners?

The marathon dataset covers only US race venues so nationality-linked finish times cannot be extracted. Instead, using Strava training data: runners from Peru, Canada, Czech Republic, and Poland log the highest median weekly training km (~30–34 km/week) among athletes targeting a major marathon. This is interpreted as a proxy for running culture intensity rather than race performance directly.

### Q5 — Have finish times changed over 20 years despite surging participation?

Remarkably stable. Men's median finish time has been flat at 4h19m from 2000 to 2019. Women improved slightly from 4h50m to 4h46m. Over the same period, annual finisher counts nearly doubled from ~120k to ~160k. The influx of new, slower recreational runners did not pull the median upward — suggesting the distribution widened at both ends simultaneously.

### Q6 — Does training volume (km/week) correlate with performance?

Surprisingly weakly: r = 0.06. There is enormous variance in finish time at every training volume level. Runners logging 60+ km/week span times from sub-3h to over 6h. Raw training volume measured at group level appears to be a poor predictor of individual race performance — consistency, intensity, genetics, and experience likely matter more.

### Q7 — What is the overall distribution of finish times by gender?

Men: median 4h22m, n = 1.7 million. Women: median 4h49m, n = 1.15 million. Both distributions are right-skewed — there is a hard physiological floor around 2h00m but no ceiling, so the tail extends far rightward. The women's distribution is wider and flatter than the men's, indicating greater spread across ability levels. Both show a secondary bump around 5h00m, likely reflecting the large cohort of first-time recreational runners targeting the 5-hour barrier.

---

## 4. Limitations

- **Group-level Strava join:** The training volume join operates at the demographic group level (gender × age group × race × year), not per individual runner. The `strava_weekly_km` column in master.csv represents the median training load of a cohort, not any specific person's preparation. This introduces noise in Visualization 6.
- **Weather is daily average:** Open-Meteo provides daily mean values, not hour-of-race conditions. Race-start temperature (typically early morning) may differ significantly from the daily mean, which includes afternoon highs.
- **Country training data is self-selected:** Strava athletes are urban, tech-savvy, and motivated enough to track every run. They do not represent the average runner in any country. Countries like the US are over-represented simply because Strava's user base is concentrated there.
- **Marathon dataset is US-only:** All 2.85 million finishers raced at one of 10 US venues. Findings about age, temperature, and trends reflect the US recreational marathon population and may not generalise globally.
