# SemProject_MLEngine: MindTrack ML Engine

The machine-learning engine of **MindTrack**, a mobile app that helps university students notice how their phone habits and mood are connected.

**Final ML approach: Random forest.** The complete, final engine is in **[`RandomForest_Final.ipynb`](RandomForest_Final.ipynb)**.

> ⚠️ The model estimates a *questionnaire score group*. It is a screening aid, **not a diagnosis**.

---

## What the engine does
- **Predicts** whether a student's **Beck Depression Inventory** score is likely to be **11 or more** (the first band above "normal ups and downs", i.e. mild low mood or stronger).
- **Inputs:** 20 answers a student can give in a phone app:
  - their phone-dependence score (TMD, 22 items summed);
  - 13 anxiety items (STAI);
  - age and semester;
  - living arrangement;
  - family expression and family activities;
  - how long they have had a phone.
- **Output:** a risk score and a level, **Low** or **Elevated**. The Elevated cut-off is set for about 75% sensitivity, because this is a screening tool.

## Notebook structure (CRISP-DM)

| CRISP-DM stage | Notebook sections |
|---|---|
| Business understanding | §1 Problem definition |
| Data understanding | §2 Load and align; §4 EDA, reliability (Cronbach's alpha), hypothesis test |
| Data preparation | §3 Cleaning; §5 Train/test split; §6 Feature groups; §7 PCA |
| Modelling | §8 Model comparison and tuning; §9 Compact question set; §11 Regression |
| Evaluation | §10 Threshold and one-time test evaluation; §10.1 Base models |
| Deployment | §12 Export to JSON for the Android app, with a parity check |

A **plain-language glossary** of all ML terms is at the top of the notebook.

## Results

| Model | Test ROC-AUC | Notes |
|---|---|---|
| **Base model: baseline** (always predicts the training rate) | 0.50 | The score to beat |
| **Base model: first random forest** (background + phone-dependence total) | ≈ 0.65 | Previous feature set |
| **Final random forest** (20 inputs) | **≈ 0.76** | PR-AUC ≈ 0.38 (no-skill 0.18); sensitivity 0.80 at the screening threshold |

Eleven models were compared with repeated 5-fold cross-validation:
- logistic regression;
- a decision tree;
- gradient boosting;
- extra trees;
- several random forests.

The random forests did best. PCA was tested and made predictions worse, so it isn't used.

## Repository contents

| Path | Contents |
|---|---|
| `RandomForest_Final.ipynb` | **The final ML engine**: the whole pipeline, run top to bottom |
| `Digital Sedentary Behavior, Mobile Device Dependen/` | The raw dataset: 5 linked questionnaires (1,220 students) plus the data dictionary |
| `data/processed/`, `data/splits/` | Cleaned table and the 80/20 train/test split (created by the notebook) |
| `models/mindtrack_rf.joblib` | **Final model** (scikit-learn) + feature list + threshold |
| `models/base_model_baseline.joblib`, `models/base_model_previous_feature_set.joblib` | **Base models** |
| `results/`, `results/figures/` | Tables and 12 figures (created by the notebook) |
| `app/mindtrack_model.json` | The final forest exported as JSON; it runs on the phone in the Android app |
| `app/test_vectors.json`, `app/app_input_fields.csv` | Parity-test data and the 20 inputs the app collects |
| `previous_dataset/` | Our earlier work on a different dataset (see below) |

## How to run
Python 3.10 or newer (we used 3.12):

```bash
python3 -m pip install -r requirements.txt
```

Then open `RandomForest_Final.ipynb` in VS Code or Jupyter and choose **Restart → Run All** (about 3–6 minutes). Everything is regenerated: cleaned data, splits, results, figures, models and the app export. The random seed is fixed (42), so the results are reproducible.

## Dataset
**Digital Sedentary Behavior, Mobile Device Dependence** dataset:
- 1,220 university students;
- five linked questionnaires: background and phone use, Test of Mobile Dependence (TMD), Nordic musculoskeletal questionnaire, Beck Depression Inventory, and the State-Trait Anxiety Inventory (STAI).

## Why we changed datasets
We started with `mental_health_and_technology_usage_2024.csv` (see `previous_dataset/`). We switched to the Digital Sedentary Behavior dataset because it has **validated clinical questionnaires**: a real depression score (Beck), a standard phone-dependence scale (TMD) and an anxiety scale (STAI). That gives a meaningful target and inputs a student can actually answer in an app.

## Related repositories
- **Mobile app** (Android, Kotlin/Jetpack Compose), which uses this model on the device and adds explainable AI ("Why this result?" and "What if…?"): `MindTrack_mobile_app`

## Limitations
- The data is cross-sectional and from one institution.
- The test set has only 44 positive cases.
- The model shows **association, not causation**, and is **not a diagnostic tool**.
