
# Transparent Survival Analysis Using Tree-Based Ensembles

Investigating how training sample size affects predictive performance and feature-importance stability across survival models using the German Breast Cancer Study Group 2 (GBSG2) dataset.

## Project Overview

Survival analysis is used to study how long it takes for an event to occur, while accounting for patients whose event has not been observed during the study period.

Machine learning models can help identify patterns associated with patient outcomes. However, a model's predictive performance does not necessarily mean its explanations are consistent.

This project examines two related questions:

1. How does training sample size affect the predictive performance of survival models?
2. How does training sample size affect the stability of feature-importance rankings when models are trained on different patient samples?

The project also investigates whether using overlapping training samples makes feature-importance rankings appear more stable than they actually are.

## Research Question

**How does training sample size affect predictive performance and the stability of feature-importance rankings across different survival models?**

## Dataset

The analysis uses the German Breast Cancer Study Group 2 (GBSG2) dataset available through `scikit-survival`.

- **Patients:** 686
- **Predictors:** 8
- **Observed events:** 299 (43.6%)
- **Censored observations:** 387
- **Missing values:** 0

The event indicator represents recurrence or death. The follow-up time records the time to the event or the end of observation.

The eight predictors are:

| Variable | Description |
|---|---|
| `age` | Patient age |
| `estrec` | Estrogen receptor measurement |
| `horTh` | Hormonal therapy |
| `menostat` | Menopausal status |
| `pnodes` | Number of positive nodes |
| `progrec` | Progesterone receptor measurement |
| `tgrade` | Tumor grade |
| `tsize` | Tumor size |

The data were divided into a stratified 75% training set (514 patients) and 25% test set (172 patients), using a fixed random seed.

## Models

Three survival models were compared:

- **Cox Proportional Hazards (Cox PH):** A statistical survival model used as a baseline.
- **Random Survival Forest (RSF):** An ensemble of survival trees that can capture nonlinear relationships and interactions.
- **Gradient-Boosted Survival (GBS):** A boosting-based survival model that builds an ensemble sequentially.

The Cox model's functional form was selected using cross-validation. Random Survival Forest and Gradient-Boosted Survival hyperparameters were tuned using five-fold cross-validation on the training set.

## Methodology

The analysis follows these main steps:

1. **Exploratory data analysis:** Examine predictor distributions, missing values, event rates, and overall survival patterns.
2. **Preprocessing:** One-hot encode categorical variables, fitting the encoder on the training data only.
3. **Model development:** Fit and tune the three survival models.
4. **Predictive evaluation:** Evaluate model performance on the held-out test set.
5. **Risk stratification:** Divide test patients into low-, medium-, and high-risk groups using predicted risk, then compare their survival patterns.
6. **Feature importance:** Use grouped permutation importance to measure how much model performance changes when each original predictor is shuffled.
7. **Noise analysis:** Estimate how much feature-importance rankings change due to permutation randomness alone.
8. **Sample-size experiment:** Train models using samples of 100, 150, 200, and 256 patients and compare rankings across independently drawn samples.
9. **Overlap comparison:** Compare disjoint training samples with overlapping samples to assess how shared patients affect apparent ranking stability.

The sample-size experiment uses 25 pairs per sample size. For the disjoint design, the two models in each pair are trained on non-overlapping patients. The overlapping design provides a comparison. A size-matched baseline estimates agreement when the same fitted model is evaluated with different permutation shuffles.

## Evaluation Metrics

### Predictive performance

- **Harrell's C-index:** Measures how well a model ranks patients by risk. Higher values indicate better discrimination.
- **Uno's C-index:** An alternative concordance measure that accounts for censoring.
- **Time-dependent AUC:** Measures discrimination at specified follow-up times.
- **Integrated Brier Score (IBS):** Measures prediction error over time. Lower values indicate better performance.

Bootstrap confidence intervals and paired model comparisons were calculated for Harrell's C-index.

### Feature-importance stability

- **Spearman correlation:** Measures agreement in the overall ranking of features.
- **Weighted Kendall's tau:** Measures ranking agreement while giving greater weight to important positions.
- **Tie-aware top-3 Jaccard index:** Measures overlap between the most important feature sets while retaining ties at the third position.

## Key Findings

### 1. Predictive performance

On the held-out test set, the Random Survival Forest had the highest Harrell's C-index (0.707), followed by Cox PH (0.698) and Gradient-Boosted Survival (0.689).

| Model | Harrell's C | Uno's C | Mean TD AUC | IBS |
|---|---:|---:|---:|---:|
| Cox PH | 0.698 | 0.679 | 0.755 | 0.159 |
| Random Survival Forest | 0.707 | 0.687 | 0.758 | 0.160 |
| Gradient-Boosted Survival | 0.689 | 0.670 | 0.731 | 0.163 |

The paired bootstrap confidence intervals for differences in Harrell's C included zero. Therefore, the results do not establish a statistically clear performance difference between the three models.

### 2. Risk stratification

All three models separated the test patients into low-, medium-, and high-risk groups with statistically significant differences in survival patterns:

| Model | Log-rank p-value |
|---|---:|
| Cox PH | 8.94 × 10⁻⁸ |
| Random Survival Forest | 2.50 × 10⁻⁷ |
| Gradient-Boosted Survival | 1.98 × 10⁻⁷ |

These results suggest that each model's predicted risk scores captured meaningful differences in patient outcomes within this test set.

### 3. Training sample size and ranking stability

Agreement between models trained on disjoint patient samples generally improved as sample size increased for Cox PH.

For the tree-based models, improvements were less consistent across sample sizes.

**Spearman agreement between disjoint samples:**

| Model | 100 patients | 150 patients | 200 patients | 256 patients |
|---|---:|---:|---:|---:|
| Cox PH | 0.402 | 0.574 | 0.644 | 0.686 |
| Random Survival Forest | 0.658 | 0.661 | 0.690 | 0.666 |
| Gradient-Boosted Survival | 0.583 | 0.621 | 0.658 | 0.645 |

The results suggest that larger training samples can improve ranking agreement, but the relationship is not uniform across models.

### 4. Overlapping samples and apparent stability

The overlapping design generally produced higher ranking agreement than the disjoint design.

For example, at 200 patients, the average Spearman agreement was:

- Cox PH: 0.784 with overlapping samples versus 0.644 with disjoint samples.
- Random Survival Forest: 0.730 versus 0.690.
- Gradient-Boosted Survival: 0.742 versus 0.658.

This illustrates why shared observations between training samples matter when evaluating explanation stability.

### 5. Important predictors

`pnodes` and `progrec` were consistently among the most important predictors across the three models and sample sizes.

The frequency analysis also showed differences in how the models ranked other variables. For example, `age` appeared more frequently among the top three features for Gradient-Boosted Survival and Random Survival Forest at some sample sizes, while `tgrade` appeared more often in the Cox model's top three.

These rankings describe model reliance on predictors; they should not be interpreted as causal effects.

## Limitations

- **Single dataset:** The analysis uses only the GBSG2 dataset. Results require external validation before broader conclusions can be drawn.
- **Fixed test set:** Predictive performance is evaluated on one 172-patient holdout set.
- **Repeated subsampling:** Patients may appear in different pairs across repetitions. The 25 pairs are not 25 independent external cohorts.
- **Restricted sample sizes:** The disjoint-pair design is limited to a maximum of 256 patients because two non-overlapping samples must be drawn from the 514-patient training pool.
- **Hyperparameter tuning:** Settings selected using the full training set are reused for the smaller sample-size experiments.
- **Correlated predictors:** Correlation between variables can affect how permutation importance is distributed across features.
- **Model reliance, not causation:** Permutation importance measures how much a fitted model relies on a predictor. It does not establish causal relationships or clinical usefulness.

## Reproducibility

The notebook was developed in Google Colab and uses a fixed global random seed. Long-running experiment results are cached to reduce unnecessary reruns.

Main packages:

- Python
- NumPy
- pandas
- scikit-learn
- scikit-survival
- lifelines
- SciPy
- Matplotlib

The notebook records the package versions used in the executed environment.

## Repository Structure

```text
survival-analysis-explainability/
├── README.md
├── requirements.txt
├── notebooks/
│   └── survival_analysis_gbsg2.ipynb
├── data/
│   └── README.md
└── results/
    ├── figures/
    └── tables/
```

## Future Work

Potential extensions include:

- External validation using another survival dataset.
- Evaluating whether feature-importance stability changes with different model configurations.
- Investigating the effect of correlated predictors on importance rankings.
- Exploring additional explainability methods for survival models.

## Acknowledgements

The GBSG2 dataset is accessed through the `scikit-survival` package.
