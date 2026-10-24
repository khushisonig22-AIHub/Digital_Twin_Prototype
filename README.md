# Digital Twin Proof-of-Concept – Happiest Health Challenge 2026

## Team Details
- **Team name:** [TEAM_Vitals]
- **Team leader:** [Khushi Soni], [khushisonig22@gmail.com],
- **Members:** [Individual
- **College / Incubator:** [IIT Patna]

## Project Title
**Digital Twin: Fusing Static EHR and Real-Time Wearable Streams to Predict Adverse Glucose Events**

## Problem Statement and Healthcare Use Case
Clinical records are static and infrequent, while wearables produce continuous data about heart rate, sleep,
activity and glucose. Doctors rarely see these two sources together. This prototype builds a *virtual patient*
(a digital twin) that combines both streams and predicts whether a patient's glucose will exceed 180 mg/dL
within the next 2 hours.

**Use case:** A doctor reviews a virtual patient's risk before a consultation and sees which factors
(diabetes status, recent glucose trend, HRV, sleep) drive the risk.

## Data Streams (Sandbox Rules – no real patient data)
| Stream | Type | Source in this prototype |
|---|---|---|
| Static / historical | Simulated EHR: age, sex, BMI, HbA1c, diabetes, hypertension, genetic-risk flag, LDL | `src/generate_data.py` (synthetic) |
| Dynamic / real-time | Simulated hourly wearable/CGM: glucose, heart rate, HRV, steps, sleep quality (14 days) | `src/generate_data.py` (synthetic) |

Synthetic data is generated in code so no privacy-restricted (DPDP / HIPAA) data is used. For a larger study the
same pipeline can take Synthea (EHR) and MIMIC-IV (clinical) data, or open-source wearable datasets.

## Technical Stack and AI/ML Model
- **Language:** Python 3.10+
- **Libraries:** NumPy, pandas, scikit-learn
- **Model:** `HistGradientBoostingClassifier` on fused features:
  - Static: age, BMI, HbA1c, diabetic, hypertension, genetic risk, LDL
  - Dynamic: last glucose, 6h glucose mean/slope/max, 24h heart rate, 7-day HRV mean and trend, 24h steps, 7-day sleep quality
- **Dashboard (concept):** HTML + vanilla JavaScript (`dashboard/index.html`)

## Results (held-out 25% of 30,000 windows)
Run `python src/digital_twin.py` to reproduce.

| Model | AUC | Accuracy |
|---|---|---|
| Static EHR only | 0.981 | 0.984 |
| Wearable stream only | 0.989 | 0.986 |
| Fused twin (EHR + stream) | 0.988 | 0.984 |

**Honest note:** Only about 1.5% of windows are positive (spike events), so accuracy is close to the
majority-class baseline and is not a meaningful measure. AUC is the main metric. On this synthetic data the
fused model does not beat the wearable-only model. The result is likely because the synthetic generator
ties spikes to the same glucose signal the wearable features already capture. Testing on real or more realistic
open data is the next step.

## How to Run
```bash
pip install -r requirements.txt
python src/generate_data.py        # optional: writes CSVs to data/
python src/digital_twin.py         # trains models, writes dashboard/*.json
# open dashboard/index.html in a browser (serve the folder, e.g. python -m http.server)
```

## Architecture
```
 EHR (static) ─────────────┐
                           ├─► Feature fusion ─► Gradient-boosted model ─► Risk score ─► Doctor dashboard
 Wearable stream (hourly) ─┘     (7d window)       (2h glucose spike)
```
(Architecture diagram in PDF/PPT: [ADD FILE LINK])

## Demo Video (min. 20 minutes)
[ADD UNLISTED YOUTUBE LINK OR POWERPOINT RECORDING LINK]

## Presentation (PDF/PPT)
[ADD FILE LINK]

## Limitations and Next Steps
- Data is synthetic; clinical validity is not established.
- Positive class is rare; use precision/recall and calibration on real data.
- Dashboard is a concept mock-up, not connected to live devices.

## License
MIT License – see [LICENSE](LICENSE).
