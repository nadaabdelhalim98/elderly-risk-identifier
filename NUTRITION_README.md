# Elderly Nutritional Risk Classifier
### A clinical AI portfolio project · MSc Digital Health · Nada Abdelhalim

---

## Overview

Malnutrition in elderly patients is one of the most underdiagnosed conditions in European healthcare. Studies estimate that 20–60% of hospitalised older adults arrive malnourished, and undernutrition in community-dwelling elderly patients frequently goes undetected until it becomes a clinical emergency. Germany alone has over 18 million people aged 65+, making early nutritional screening a direct public health priority.

This project builds an XGBoost classifier that predicts the nutritional status of elderly patients across three categories — **Undernutrition**, **Adequate**, and **Overnutrition** — using dietary, anthropometric, clinical, and social variables. The model output is designed to triage patients for early dietitian intervention before malnutrition becomes irreversible.

---

## Clinical Use Case

> A geriatric care platform or hospital dietetics workflow runs this model on incoming patient data. Patients classified as Undernourished are automatically flagged for priority dietitian review. The model does not replace clinical judgment — it tells the care team *where to look first*.

**Target setting:** Geriatric wards, care homes, community dietetics services, digital health platforms for elderly care  
**Target users:** Dietitians, geriatric nurses, care coordinators, clinical operations teams  
**Clinical alignment:** Mini Nutritional Assessment (MNA), MUST (Malnutrition Universal Screening Tool), ESPEN guidelines for clinical nutrition in geriatrics

---

## Why XGBoost?

This project uses **XGBoost (Extreme Gradient Boosting)** — a step up from the Random Forest used in the companion adherence project. XGBoost builds decision trees sequentially, each one correcting the errors of the previous, making it particularly effective on structured clinical tabular data. It is one of the most cited algorithms in clinical prediction research and appears consistently in digital health and health AI job descriptions.

Key advantages for this use case:
- Handles class imbalance better than standard Random Forest
- Native feature importance ranking aligned with clinical interpretability
- Compatible with SHAP values for individual patient explanation

---

## Dataset

Synthetic dataset of **600 elderly patients** (age 65–95), generated with logic grounded in validated geriatric nutrition screening tools.

| Feature | Type | Clinical rationale |
|---|---|---|
| Age | Numeric | Nutritional needs and risk profiles shift significantly after 75 |
| Gender | Binary | Women have lower muscle mass baseline — sarcopenia risk differs |
| Lives in care home | Binary | Institutionalisation associated with higher undernutrition prevalence |
| BMI | Numeric | Primary anthropometric indicator; <22 kg/m² in elderly = undernutrition risk |
| Weight loss — 3 months (kg) | Numeric | Unintentional weight loss is a core MNA screening criterion |
| Calf circumference (cm) | Numeric | <31 cm is the ESPEN threshold for sarcopenia screening |
| Grip strength (kg) | Numeric | Functional muscle strength proxy; <16 kg (women) / <27 kg (men) = sarcopenia |
| Daily calories (kcal) | Numeric | Total energy intake against age-adjusted requirements |
| Protein intake (g) | Numeric | ESPEN recommends ≥1.0–1.2 g/kg/day in elderly — often unmet |
| Meals per day | Numeric | Meal frequency as proxy for dietary structure and appetite |
| Appetite score (1–10) | Numeric | Self-reported appetite; low score is a leading indicator of decline |
| Fluid intake (ml) | Numeric | Dehydration compounds malnutrition risk in elderly |
| Dysphagia | Binary | Swallowing difficulty directly limits food intake — most important predictor |
| Dementia | Binary | Cognitive decline affects eating behaviour, self-reporting, and independence |
| Polypharmacy | Numeric | ≥5 medications associated with appetite suppression and nutrient depletion |
| Recent hospitalisation | Binary | Acute illness episode triggers nutritional depletion |
| Mobility score (1–5) | Numeric | Physical activity drives energy expenditure and appetite |
| Albumin (g/dL) | Numeric | Biochemical marker of chronic undernutrition; <3.2 g/dL = significant risk |
| Pressure ulcer | Binary | Indicates chronic protein deficiency and metabolic stress |
| Eats alone | Binary | Social isolation reduces food intake in elderly populations |
| Caregiver feeding assistance | Binary | Dependency signal — indicates severe functional decline |
| MNA proxy score | Numeric | Composite score approximating Mini Nutritional Assessment (0–30 scale) |

**Class distribution:** Undernutrition 57.7% · Adequate 30.0% · Overnutrition 12.3%  
This reflects realistic prevalence in European elderly care settings.

---

## Model Performance

**Algorithm:** XGBoost multi-class classifier  
**Classes:** Undernutrition · Adequate · Overnutrition

### Confusion Matrix
![Confusion Matrix](nutrition_confusion_matrix.png)

| Class | Correctly Classified | Notes |
|---|---|---|
| Undernutrition | 56 / 69 | Strongest performance — highest clinical priority class |
| Adequate | 23 / 36 | 12 misclassified as Overnutrition — BMI overlap likely cause |
| Overnutrition | 6 / 15 | Most difficult class — underrepresented in training data |

**Clinical interpretation of misclassifications:** The model performs best on Undernutrition — the highest-stakes class for patient safety. Overnutrition misclassification is less clinically critical; a patient incorrectly classified as Adequate rather than Overnutrition is unlikely to face immediate harm. In a real deployment, threshold tuning would prioritise sensitivity on the Undernutrition class.

---

## Key Finding: What Predicts Nutritional Status in Elderly Patients?

![Feature Importance](nutrition_feature_importance.png)

Two features crossed the high-impact threshold (importance score > 0.08):

1. **Dysphagia (0.083)** — the single strongest predictor. Swallowing difficulty directly restricts food intake regardless of appetite or motivation. This is both the most clinically significant finding and the most actionable: dysphagia management (texture-modified diets, thickened fluids, speech therapy referral) is a modifiable intervention point.

2. **BMI (0.082)** — the primary anthropometric anchor. In elderly populations, BMI <22 kg/m² signals undernutrition risk even before biochemical markers deteriorate.

Notable mid-tier predictors (0.04–0.08):
- **MNA Proxy Score** — composite tool outperforms individual features, validating the MNA as a screening instrument
- **Daily Calories & Protein Intake** — dietary intake variables confirm that what patients eat matters more than demographic characteristics
- **Albumin** — biochemical marker of chronic nutritional depletion, not just acute illness
- **Mobility Score & Appetite Score** — functional and subjective indicators that capture what anthropometrics miss

**Key clinical insight:** Dysphagia ranking above BMI is clinically meaningful and often overlooked in standard nutritional screening. Most tools weight BMI heavily; this model's output suggests that the *reason* a patient cannot eat (dysphagia, dementia, eating alone) may matter as much as the anthropometric consequence of not eating.

---

## Exploratory Data Analysis

![EDA](nutrition_eda.png)

Key patterns observed:
- **BMI:** Clear left shift for Undernutrition group — most patients cluster below BMI 22; Overnutrition group clusters above 27
- **Appetite score:** Adequate patients show higher median appetite, but distributions overlap substantially — appetite alone is insufficient for classification
- **Daily calories:** Overnutrition group skews toward higher intake (median ~2,100 kcal); Undernutrition patients cluster below 1,500 kcal
- **Albumin:** Undernutrition patients show lower median albumin but wide variance — reflects that albumin is also affected by inflammation, not pure nutrition
- **Grip strength:** Subtle differences across groups — sarcopenia markers are more sensitive in combination than individually
- **Class distribution:** Undernutrition dominates (346/600) — reflecting real-world prevalence in European geriatric populations

---

## Ethical Considerations

| Concern | Notes |
|---|---|
| GDPR compliance | All inputs pseudonymised; data minimisation applied — only features with direct clinical rationale included |
| Algorithmic bias | Elderly women are disproportionately represented in care homes; model requires fairness audit by gender and age band (65–74 vs 75–84 vs 85+) |
| Albumin misinterpretation | Albumin reflects inflammation as well as nutrition — model may overclassify acutely ill patients as undernourished |
| Clinical validation | Synthetic dataset only — prospective validation against real MNA screening data required before any clinical use |
| EU AI Act | Healthcare AI classifying patients into risk categories is likely high-risk under Annex III — requires conformity assessment, transparency documentation, and human oversight mechanism |
| Explainability | XGBoost feature importance provides model-level explanation; SHAP values (implemented in notebook) provide individual patient-level explanation — meeting transparency expectations for clinical AI |

---

## Limitations

- **Synthetic dataset:** Distributions are clinically informed but real-world data will include missing values, temporal patterns, and demographic noise not captured here
- **Class imbalance:** Overnutrition (12.3%) is underrepresented — SMOTE oversampling or class weighting recommended for production use
- **Cross-sectional snapshot:** Nutritional decline is a trajectory — a single record misses the direction of change (stable vs. deteriorating)
- **Albumin caveat:** Not a reliable nutritional marker in acute inflammatory states — prealbumin or CRP-adjusted interpretation preferred in hospitalised patients
- **No temporal features:** Weight trend over 6 months would substantially improve the model

---

## Tech Stack

- **Language:** Python 3.10
- **Environment:** Google Colab
- **Libraries:** pandas · numpy · xgboost · shap · scikit-learn · matplotlib · seaborn
- **Algorithm:** XGBoost (multi:softmax objective, 3-class)
- **Explainability:** SHAP TreeExplainer

---

## Repository Structure

```
elderly-nutrition-risk-classifier/
├── elderly_nutrition_classifier.ipynb    # Full annotated Colab notebook
├── elderly_nutrition_dataset.csv         # Synthetic dataset (600 elderly patients)
├── nutrition_feature_importance.png      # XGBoost feature importance chart
├── nutrition_confusion_matrix.png        # Model evaluation
├── nutrition_eda.png                     # Exploratory data analysis
├── nutrition_shap.png                    # SHAP values (if generated)
└── README.md                             # This file
```

---

## Companion Project

This project is part of a growing digital health AI portfolio:

| Project | Algorithm | Domain |
|---|---|---|
| [Medication Adherence Risk Predictor](https://github.com/nadaabdelhalim98/adherence-risk-predictor) | Random Forest | Digital health behavior |
| **Elderly Nutritional Risk Classifier** | **XGBoost** | **Geriatric nutrition** |

---

## Next Steps

- [ ] Validate against real MNA screening data from a geriatric ward or community dietetics service
- [ ] Apply SMOTE oversampling to address Overnutrition class imbalance
- [ ] Build SHAP waterfall plots for individual patient explanation cards
- [ ] Add longitudinal weight trend as a time-series feature
- [ ] Deploy as a FastAPI endpoint for integration with a dietitian workflow dashboard

---

## About

**Nada Abdelhalim**  
Medical Advisor · MSc Digital Health candidate, TH Deggendorf (Campus Pfarrkirchen)  
BSc Nutrition & Dietetics · BSc Psychology  
6+ years in medico-scientific communication across pharma and digital health

[LinkedIn](https://www.linkedin.com/in/) · [GitHub](https://github.com/nadaabdelhalim98)

---

*This project was developed as part of an MSc Digital Health portfolio. The dataset is fully synthetic and not derived from any real patient data. Clinical references are included for educational framing only.*
