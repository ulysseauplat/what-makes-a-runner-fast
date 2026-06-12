# What Makes a Runner Fast?

A multi-modal data visualization study of the physiological, environmental, and demographic factors that determine long-distance running performance.

**University of Milan — Scientific Visualization 2025–2026**

---

## Research Questions

1. Does age determine peak running performance? At what age are runners fastest?
3. How much does race-day temperature affect finish times?
4. Which countries produce the fastest recreational runners?
5. Have finish times changed over the past 20 years despite surging participation?
6. Does training volume (km/week) correlate with performance?
7. What is the overall distribution of finish times by gender?

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
| 3 | `03_temp_vs_time.png` | Q3 — Temperature vs finish time: regression with 95% CI |
| 4 | `04_choropleth_country.png` | Q4 — Weekly training volume by runner nationality (world map) |
| 5 | `05_year_trend.png` | Q5 — Median finish times 2000–2019 with participation count |
| 6 | `06_training_bubble.png` | Q6 — Training volume vs finish time: bubble chart (Strava data) |
| 7 | `07_finish_distribution.png` | Q7 — Finish time distribution by gender: histogram + KDE |
| — | `08_predictor_correlations.png` | Summary — Pearson r for each factor vs finish time |

---

## Project Structure

```
what-makes-a-runner-fast/
├── README.md
├── SOURCES.md              # Dataset citations
├── data/
│   ├── raw/                # Original downloaded files
│   └── processed/
│       └── master.csv      # Joined dataset (output of 04_integration.ipynb)
├── notebooks/
│   ├── 01_data_loading.ipynb
│   ├── 02_preprocessing.ipynb
│   ├── 03_weather_fetch.ipynb
│   ├── 04_integration.ipynb
│   └── 05_visualizations.ipynb
├── charts/                 # Exported PNG charts (150 dpi)
│   ├── 01_age_vs_time.png
│   ├── 03_temp_vs_time.png
│   ├── 04_choropleth_country.png
│   ├── 05_year_trend.png
│   ├── 06_training_bubble.png
│   ├── 07_finish_distribution.png
│   └── 08_predictor_correlations.png
└── presentation/
    └── DataViz_RunnerSpeed.pptx
```

---

## How to Run

### 1. Install dependencies

```bash
pip install pandas matplotlib requests geopandas scipy numpy jupyter kaggle plotly
```

### 2. Set the Kaggle token (one-time)

Create an API token at kaggle.com → Settings → API, then:

```bash
export KAGGLE_API_TOKEN=<your-token>   # add to ~/.bashrc to persist
```

### 3. Run notebooks in order

```bash
jupyter notebook notebooks/
```

Run `01` → `02` → `03` → `04` → `05`.

Notebook `01` downloads both raw datasets automatically if missing (marathon results from Zenodo, `run_ww_2019_w.csv` from Kaggle). Weather is fetched in `03` from the Open-Meteo API (no key needed).

---

## Key Findings

- **Age** — performance peaks at 28–34; non-linear decline accelerates sharply past 50
- **Weather** — race-day temperature is the strongest external predictor (Pearson r = 0.23–0.24); each extra °C adds ~3 min to the median finish time
- **Training volume** — surprisingly weak predictor (r = 0.06); raw km/week explains little of the individual variance
- **Trends** — median finish times barely changed 2000–2019 despite participation nearly doubling
- **Gender** — men 4h22m vs women 4h49m at the median; both distributions are strongly right-skewed
- **Running culture** — Strava data shows Peru, Canada, and Central Europe log the most weekly km among marathon-targeting athletes

---

## Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| pandas | ≥2.0 | Data loading, cleaning, merging |
| matplotlib | ≥3.7 | Core plotting |
| requests | ≥2.28 | Open-Meteo API calls |
| geopandas | ≥0.13 | Choropleth world map |
| plotly | ≥5.0 | Interactive choropleth (notebook 05) |
| scipy | ≥1.10 | Regression, KDE, statistical tests |
| numpy | ≥1.24 | Numerical operations |
| kaggle | ≥2.0 | Automated dataset download (notebook 01) |
