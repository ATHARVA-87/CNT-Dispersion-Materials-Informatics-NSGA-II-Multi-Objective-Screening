# CNT Dispersion Materials Informatics — NSGA-II Multi-Objective Screening

A materials-informatics workflow for **carbon nanotube (CNT) dispersion formulation screening** under limited experimental data. The project combines molecular descriptors, formulation/process variables, regression modelling, chemical generalization tests, applicability-domain (AD) screening, and constrained **NSGA-II** optimization.

> **Scope:** This is a screening and hypothesis-generation workflow. Pareto candidates are model predictions and require experimental validation; RME and sonication are treated as data-derived process descriptors rather than directly executable machine settings.

## Objectives

- **Dscore:** microscopy-derived dispersion quality, higher is better.
- **IG/ID:** Raman-based structural/crystallinity index, higher is better.
- Multi-objective search identifies trade-offs between dispersion quality and structural preservation.

## Dataset

The study uses the supplied CNT dispersion dataset with:

- **669 raw training rows**; 666 usable Dscore measurements.
- **496 usable IG/ID measurements**.
- **36 dispersants**, **22 solvents**, **2 CNT types**.
- **289 observed dispersant–solvent pairs** and **347 observed chemistry–process designs**.
- An external validation file containing **56 extrapolative** and **9 interpolation** cases.

## Workflow

```text
Experimental data
      ↓
Cleaning + data audit
      ↓
Molecular / compatibility / formulation / process features
      ↓
Random + grouped cross-validation
      ↓
RF + HGB surrogate models
      ↓
External interpolation / extrapolation validation
      ↓
Applicability-domain + RF tree-spread screening
      ↓
Constrained NSGA-II
      ↓
Pareto hypotheses for experimental testing
```

### Model and optimization specification

- Final surrogate predictions use an average of **Random Forest** and **HistGradientBoosting** models.
- Categorical variables are one-hot encoded; numerical features use fold-safe imputation and low-variance filtering.
- **3-seed grouped CV** evaluates random, pair, solvent, and dispersant hold-outs.
- AD uses standardized **5-nearest-neighbour distance**.
- RF tree prediction spread is used as a **model-disagreement / uncertainty proxy**.
- NSGA-II uses **6 decision variables**: observed chemistry-process design, CNT type, wt%, dispersant/CNT ratio, RME, and sonication.
- Optimization uses **100 individuals × 80 generations** with three active constraints: kNN AD distance and RF spread caps for both targets.

## Key Results

### Predictive performance

| Evaluation | Dscore R² | IG/ID R² |
|---|---:|---:|
| Random CV (RF) | **0.533 ± 0.005** | **0.674 ± 0.006** |
| Pair hold-out (RF) | 0.351 ± 0.013 | 0.515 ± 0.052 |
| Solvent hold-out (RF) | 0.111 ± 0.011 | 0.291 ± 0.049 |
| Dispersant hold-out (RF) | 0.056 ± 0.031 | 0.489 ± 0.021 |
| Interpolation validation | **0.602** | — |
| 4-disper­sant extrapolation | **0.253** | — |

The project shows that random CV can overestimate performance for **unseen chemical environments**, making grouped validation important for materials-informatics screening.

The notebook also reproduces the source paper's reported `Predicted*` metrics: **Dscore R² = 0.566, MAE = 0.078** and **IG/ID R² = 0.728, MAE = 9.837**.

### Optimization output

- **36,640** finite-grid candidates evaluated.
- **9,687** candidates satisfy all screening constraints.
- **35** constrained finite-grid Pareto designs.
- **92** feasible unique NSGA-II Pareto formulations after deduplication.
- Predicted feasible Pareto ranges: **Dscore 0.717–0.905** and **IG/ID 33.5–126.0**.
- Hypervolume comparison with the finite enumeration reference: **1.059×**; this is a reference comparison, not an optimizer accuracy score.

Selected hypotheses are exported for experimental testing with predicted objectives, AD distance, RF spread, process descriptors, and feasibility information.

## Important Limitations

- Pareto candidates are **surrogate predictions, not experimentally validated formulations**.
- CNT type is optimized independently, so some CNT–chemistry combinations may represent **controlled extrapolation**.
- kNN AD and RF tree spread are screening diagnostics, not calibrated prediction probabilities.
- IG/ID has fewer usable observations because of measurement interference/missing spectra.
- Dscore represents a short-term optical-uniformity measurement rather than long-term dispersion stability.
- Process descriptors do not expose all machine-level operating parameters; RPM is fixed at the observed sequence maximum in optimization.
- Feature importance is predictive interpretation, not proof of experimental causality.

## Reproduction

Place the supplied training and validation CSV files in the project data location and run:

-Please change the data directory and download the data from the attachments of reference research paper

```bash
pip install -r requirements.txt
jupyter notebook
```

Main notebook:

`cnt_dispersion_audit_optimizer_corrected_final(3).ipynb`

Typical outputs include:

```text
figures/
pareto_candidates_corrected.csv
nsga2_hypotheses_to_test.csv
```

## References

1. H. Jintoku, **“Machine-Learning-Based Dispersion Optimizer for Carbon Nanotubes across Dispersant−Solvent−Process Space,”** *ACS Applied Materials & Interfaces* **2026**, 18, 22287–22299. DOI: https://doi.org/10.1021/acsami.6c01563
2. 2. J. Blank and K. Deb, "Pymoo: Multi-Objective Optimization in Python," *IEEE Access*, vol. 8, pp. 89497-89509, 2020. DOI: [10.1109/ACCESS.2020.2990567](https://doi.org/10.1109/ACCESS.2020.2990567)
3. F. Pedregosa et al., **Scikit-learn: Machine Learning in Python**, *JMLR* **2011**, 12, 2825–2830.
4. S. M. Lundberg, S.-I. Lee, **A Unified Approach to Interpreting Model Predictions**, *NeurIPS* **2017**.
5. RDKit: Open-source cheminformatics software, https://www.rdkit.org/

## Tools

**Python · NumPy · pandas · scikit-learn · RDKit · Matplotlib · Seaborn · SHAP · pymoo · XGBoost/LightGBM (benchmark support)**
