# ICU Vital Sign Trends: First 24-Hours Survivors vs. Non-Survivors 

## Description

This project examines how vital sign trends in the first 24 hours of ICU admission differ between patients who survive their hospital stay and those who do not. Using the MIMIC-III critical care database, I am tracking heart rate, blood pressure, respiratory rate, temperature, and oxygen saturation over time for each group and compare the trajectories.

**Question:** How do vital sign trends in the first 24h differ between patients who survive vs. don't?

## Why MIMIC-III

MIMIC-III is a real de-identified ICU database. I chose to work with this specific dataset since its multidimensional with several linked tables and many inconsistencies in data collection. It's (hopefully) a realistic stand-in for the kind of healthcare data wrangling I want to be doing professionally.

## Tables Used

| Table | Purpose |
|---|---|
| `PATIENTS` | Demographics, date of death (used to determine survival outcome) |
| `ADMISSIONS` | Admission/discharge times, hospital expire flag |
| `ICUSTAYS` | ICU admission time (`INTIME`) — anchor point for the "first 24h" window |
| `CHARTEVENTS` | Vital sign measurements (heart rate, blood pressure, respiratory rate, temperature, SpO2) |

Join keys: `SUBJECT_ID` (patient) and `HADM_ID` (hospital admission), with `ICUSTAY_ID` linking chart events to a specific ICU stay.

## Approach

1. Identify ICU stays with at least 24h of chart data, and label each patient as survivor / non-survivor based on hospital discharge status.
2. Pull the relevant `ITEMID`s from `CHARTEVENTS` for the five target vitals, filtered to the first 24h from `ICUSTAYS.INTIME`.
3. Standardize units, resolve duplicate/conflicting readings at the same timestamp, and bin measurements into consistent hourly intervals per patient.
4. Summarize trends (mean, trajectory) per hour for each vital, split by survival outcome.
5. Plot trend lines for each vital sign, survivors vs. non-survivors, over the 24-hour window.

## Repository Structure

```
├── README.md
├── NOTES.md              # Running log of decisions and rationale
├── notebooks/
│   ├── 01_exploration.ipynb   # Table scouting, cohort definition, messy first passes
│   └── 02_analysis.ipynb      # Clean final analysis and visualizations
├── src/                   # Reusable cleaning/extraction functions (if applicable)
└── .gitignore
```

## Data Access

This project uses MIMIC-III, which requires credentialed access via [PhysioNet](https://physionet.org/) (completed CITI training + signed data use agreement). Raw patient data is **not** included in this repository. To reproduce this analysis, request your own MIMIC-III access and update the data-loading paths accordingly.

## Findings
Based on hourly-aggregated vital signs across the first 24 hours of ICU admission (cohort: n = 136 [90 survivors, 46 non-survivors]):
- heart rate and respiratory rate were consistently elevated in non-survivors relative to survivors across the full 24 hour window, rather than diverging only near the end of the period. This pattern is consistent with tachycardia and physiological stress from early in the ICU stay
- blood pressure was lower in non-survivors, usually with a pronounced dip around hour 5, suggesting early hemodynamic instability in this group
- SpO2 was lower and considerably more volatile in non-survivors, including a sharp desaturation event around hour 3. Instead, survivors maintained a stable, narrow range throughout
- temperature showed that survivors exhibited a steady rise from a low starting point to a more stable plateau, while non-survivors began elevated and declined notably after hour 18

It is important to note that this is a descriptive analysis on the MIMIC-III demo subset. With 46 non surviving patents, group-level trends are less prone to single-patient distortion than a vert small sample would be. This is still a modest cohort that shows association, not causation, and cannot be read as predictive of an outcome.

## Decisions Log

See `NOTES.md` for the running log of table selection, cleaning choices, and trade-offs made throughout this project.
