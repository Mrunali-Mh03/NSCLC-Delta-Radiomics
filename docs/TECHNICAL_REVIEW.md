# Technical review required before public release

This review is based on static inspection of the two supplied notebooks. It is not a
full re-execution or independent validation of the analysis.

## 1. Target-label leakage through ComBat covariates — blocking issue

In both notebooks, `FoldSafeHarmonizer.fit()` and `.transform()` construct ComBat
covariates containing:

- `SITE` (the scanner/batch label); and
- `label_2yr` (the binary outcome being predicted).

The harmonized feature representation for a held-out patient therefore depends on
that patient's true `label_2yr`. That label would not be known when predicting a new
patient. This affects both the cross-validation pipeline and the held-out evaluation.

**Required action:** redesign the predictive harmonization workflow so that
transformation of validation/test/new-patient data uses only covariates available at
prediction time. Do not pass the held-out outcome into `harmonizationApply`. Re-run
all affected analyses after the change. If outcome-preserving harmonization is retained
for a separate retrospective/inferential analysis, clearly separate it from the
prospective prediction pipeline and do not report it as an unbiased prediction result.

## 2. Permutation test requires redesign

The main notebook constructs `X_eval` with the original `label_2yr` column as a
harmonization covariate, then shuffles `y_real` for the permutation loop. The original
outcome remains embedded in the harmonizer covariates while the target is shuffled.
This undermines the permutation null.

**Required action:** rerun the permutation procedure only after the target-label
leakage has been removed from the full pipeline. Ensure each permutation shuffles the
target consistently and that no unshuffled target copy is supplied as a feature or
covariate.

## 3. Held-out evaluation is supplementary, not external validation

The held-out notebook evaluates multiple train/test ratios on the same 48-patient
cohort. These are not independent external cohorts, and the smaller test partitions
produce unstable estimates.

**Required action:** describe these results as internal held-out experiments. Avoid
claims of external validation or clinical readiness. Report uncertainty and class
counts alongside metrics.

## 4. Reproducibility items

Before release:

- Record the exact Python and package versions from a clean, successful run.
- Document the data source, access conditions, data version and citation.
- Keep the patient-level CSV and any identifiable or restricted data out of Git.
- Re-run both notebooks from top to bottom after the harmonization correction.
- Confirm all reported figures and tables are generated from the final corrected run.
- Add a suitable OSI-approved license only after confirming that you have authority
  to license all code in the repository and have addressed collaborator contributions.

## Scope

This review identifies issues visible in the notebook source. It does not establish
that all other methodological choices are correct. The pipeline should receive
supervisor/co-author review before publication or scientific claims are finalized.
