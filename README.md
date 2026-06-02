# What Makes a Runner Fast?

A multi-modal data visualization study of the physiological, environmental, and demographic factors that determine long-distance running performance.

**University of Milan — Scientific Visualization 2025–2026**

---

## Research Questions

1. Does age determine peak running performance? At what age are runners fastest?
2. Do men and women pace themselves differently across a marathon?
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
| 2 | Strava training data — 36,412 athletes worldwide | [Kaggle: mexwell/long-distance-running-dataset](https://kaggle.com/datasets/mexwell/long-distance-running-dataset) | CSV | weekly km, nationality, target marathon |
| 3 | Race-day weather (temperature, precipitation, wind) | [Open-Meteo API](https://open-meteo.com) (free, no key) | JSON / REST API | temp_mean_c, precip_mm, wind_kmh |

**Integration:** Dataset 2 is joined to Dataset 1 at group level on `(gender × age group × city × year)` using the athlete's `major` field. Dataset 3 is joined on `(city, race_date)`.

---

## Visualizations

| Chart | File | What it shows |
|-------|------|---------------|
| 1 | `01_age_vs_time.png` | Age vs finish time — peak performance at ~30–33 y (LOESS + scatter) |
| 2 | `02_pacing_gender.png` | First vs second half pace by gender — women run more evenly |
| 3 | `03_temp_vs_time.png` | Temperature vs finish time — regression with 95% CI |
| 4 | `04_choropleth_country.png` | World map of median finish time by runner nationality |
| 5 | `05_year_trend.png` | Median finish times 2000–2019 with participation count |
| 6 | `06_training_bubble.png` | Training volume vs finish time — bubble chart (Strava data) |
| 7 | `07_finish_distribution.png` | Finish time distribution by gender — histogram + KDE |
| 8 | `08_predictor_correlations.png` | Pearson r for each factor vs finish time |

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
│   ├── 02_pacing_gender.png
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
pip install pandas matplotlib seaborn requests geopandas plotly scipy numpy jupyter
```

### 2. Download the data

**Dataset 1 — Marathon results (Zenodo):**
```bash
# Download manually from https://zenodo.org/record/6959864
# Place the CSV file(s) in data/raw/marathon/
```

**Dataset 2 — Strava training data (Kaggle):**
```bash
# Requires Kaggle CLI: pip install kaggle
# Place kaggle.json in ~/.kaggle/
kaggle datasets download -d mexwell/long-distance-running-dataset -p data/raw/athletes/ --unzip
```

**Dataset 3 — Weather:** Fetched automatically in `03_weather_fetch.ipynb` via the Open-Meteo API.

### 3. Run notebooks in order

```bash
jupyter notebook notebooks/
```

Run `01` → `02` → `03` → `04` → `05`.

---

## Key Findings

*(Populated after running all notebooks)*

---

## Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| pandas | ≥2.0 | Data loading, cleaning, merging |
| matplotlib | ≥3.7 | Core plotting |
| seaborn | ≥0.12 | Statistical charts |
| requests | ≥2.28 | Open-Meteo API calls |
| geopandas | ≥0.13 | Choropleth world map |
| plotly | ≥5.14 | Interactive choropleth |
| scipy | ≥1.10 | Regression, KDE, statistical tests |
| numpy | ≥1.24 | Numerical operations |
