# What Makes a Runner Fast?

A multi-modal data visualization study of the physiological, environmental, and demographic factors that determine long-distance running performance.

**University of Milan — Scientific Visualization 2025–2026**

---

## Research Questions

1. Does age determine peak running performance? At what age are runners fastest?
2. How much does race-day temperature affect finish times?
3. Which countries have the strongest recreational running culture (weekly training volume)?
4. Have finish times changed over the past 20 years, and how has participation evolved?
5. Does training volume (km/week) correlate with performance?
6. What is the overall distribution of finish times by gender?

---

## Datasets

| # | Dataset | Source | Format | Key variables |
|---|---------|--------|--------|---------------|
| 1 | Marathon Results 2000–2019 (10 major US races, 2.85M finishers) | [Zenodo 6959864](https://zenodo.org/record/6959864) | CSV (~353 MB) | finish time, age, gender, city, year |
| 2 | Strava training data — 36,412 athletes worldwide (weekly file `run_ww_2019_w.csv` only; the dataset's other files are unused) | [Kaggle: mexwell/long-distance-running-dataset](https://kaggle.com/datasets/mexwell/long-distance-running-dataset) | CSV (~149 MB) | weekly km, nationality, target marathon |
| 3 | Race-day weather (temperature, precipitation, wind) | [Open-Meteo API](https://open-meteo.com) (free, no key) | JSON / REST API | temp_mean_c, precip_mm, wind_kmh |

**Integration:** Dataset 2 is joined to Dataset 1 at group level on `(gender × age group × city × year)` using the athlete's `major` field. Dataset 3 is joined on `(city, race_date)`.

---

## Visualizations

| Chart | File | Research question |
|-------|------|-------------------|
| 1 | `01_age_vs_time.png` | Q1 — Age vs finish time: peak performance at ~30–33 y (LOESS + scatter) |
| 2 | `02_temp_vs_time.png` | Q2 — Temperature vs finish time: regression with 95% CI |
| 3 | `03_choropleth_country.png` | Q3 — Median weekly training volume by athlete nationality (world map) |
| 4 | `04_year_trend.png` | Q4 — Median finish times 2000–2019 with participation count |
| 5 | `05_training_bubble.png` | Q5 — Training volume vs finish time: bubble chart (Strava data) |
| 6 | `06_finish_distribution.png` | Q6 — Finish time distribution by gender: histogram + KDE |
| 7 | `07_predictor_correlations.png` | Summary — Pearson r for each factor vs finish time |

---