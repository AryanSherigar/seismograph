# Aadhaar as a Societal Seismograph  
## Detecting Inclusion & Mobility Signals from Identity System Data

This notebook explores how aggregated Aadhaar enrolment and demographic update data can be modeled as passive societal signals to identify patterns, transitions, and stress zones that can support decision-making.

The analysis is aggregate/anonymized in spirit and is explicitly **non-causal** (intended for signal detection, not attribution).

---

## What this notebook does

- Loads large CSV batches for:
  - Aadhaar enrolments (new registrations).
  - Aadhaar demographic updates (post-enrolment updates).
- Cleans and standardizes geographic text fields (state/district variants) and removes invalid geo values (e.g., `100000`).
- Parses dates (DD-MM-YYYY) and constructs a monthly period index for time-series consistency.
- Aggregates data to **monthly district level** and builds two interpretable signals:
  - **Inclusion Momentum Signal (IMS):** total new enrolments across age groups.
  - **Mobility Life-Change Signal (MLS):** total demographic updates across age groups.
- Merges signals into a unified district-month panel dataset and computes normalized intensity measures used for alerting.
- Generates a simple district-level alert framework (NORMAL/MODERATE/HIGH) based on deviations from a district’s historical baseline.
- Performs short-horizon national MLS forecasting using Prophet as an “early-warning extension”.

---

## Data

The notebook expects two folders of CSV files (shown as Google Drive paths in Colab):

- `/content/drive/MyDrive/api_data_aadhar_enrolment/*.csv`
- `/content/drive/MyDrive/api_data_aadhar_demographic/*.csv`

### Enrolment dataset (example columns)

- `date`, `state`, `district`, `pincode`
- `age_0_5`, `age_5_17`, `age_18_greater`

### Demographic update dataset (example columns)

- `date`, `state`, `district`, `pincode`
- `demo_age_5_17`, `demo_age_17_`

---

## Method overview

- **Temporal standardization:** `date` fields are parsed with `dayfirst=True`, then converted to a monthly period (`yearmonth`).
- **Geography cleaning:** state/district variants are mapped to canonical names via explicit dictionaries, then invalid placeholders are filtered.
- **Aggregation:** grouped by (`yearmonth`, `state`, `district`) for stable analysis.
- **Signals:**
  - `IMS = age05 + age517 + age18greater` (as implemented after aggregation/renaming in the notebook).
  - `MLS = demoage517 + demoage17`.
- **Alerting:** districts are flagged when normalized MLS intensity is above mean by +1σ (MODERATE) or +2σ (HIGH); otherwise NORMAL.
- **Forecasting:** Prophet is fit on national monthly MLS intensity and used for a short future horizon forecast.

---

## Outputs you get

Depending on which cells are run, the notebook produces:

- District-month aggregated datasets for enrolments and updates.
- A merged `signalsdist` panel with IMS and MLS per district-month.
- An alert table (including “current month” alerts) with levels like `NORMAL`, `MODERATE Monitor closely`, `HIGH Prepare additional capacity`.
- A national MLS forecast plot generated via Prophet.

---

## How to run

### Option A: Google Colab (recommended)

- Upload `note_book.ipynb` to Colab.
- Run the Drive mount cell (`drive.mount('/content/drive')`).
- Ensure your CSV folders match the expected paths (or edit the glob paths in the notebook).
- Run cells top-to-bottom.

### Option B: Local Jupyter

You can adapt the notebook by removing the Colab Drive mount and pointing file paths to local directories.  
(Prophet installation may require extra setup locally.)

---

## Dependencies

The notebook imports and uses: `pandas`, `numpy`, `matplotlib`, `seaborn`, `glob`, and `prophet` (for forecasting).

---

## Notes & limitations

- Results are intended as **signals** (monitoring/early warning), not causal claims about why changes happen.
- Data quality is sensitive to geography standardization, duplicates, and month alignment; the notebook includes explicit cleaning steps for these.
- Forecasting is short-horizon and illustrative; it should be treated as an operational indicator rather than a long-term prediction system.
