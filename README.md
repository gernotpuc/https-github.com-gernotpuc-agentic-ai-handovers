# AI-supported physician handovers: randomized and as-treated analyses

Python analysis code supporting *Prospective Clinical Evaluation of a Multi-Agent AI System for Physician-to-Physician Handovers* (submission draft).


## Analyses

1. **Primary ward-level ITT:** equal-weighted mean handover duration in each randomized ward, with exact randomization test and 95% confidence set by test inversion under a constant additive effect.
2. **Exploratory as-treated duration:** `log(duration) ~ actual AI use + study day + ward fixed effects`, with ward-clustered CR2 standard errors and coefficient-specific Satterthwaite degrees of freedom.
3. **Exploratory count outcomes:** Poisson `interruptions ~ actual AI use + study day + ward fixed effects` and the analogous model for clarification questions, using CR2/Satterthwaite inference. A Poisson working-variance linearization is used to derive the CR2 covariance. Outputs include the rate ratio, its 95% confidence interval, degrees of freedom and P value.
4. **Binary outcomes:** descriptive EHR-consultation risk ratios for observed-only and all-session denominators, displayed as a missing-data sensitivity check.

CR2 changes **standard errors, confidence intervals, P values, and degrees of freedom**, not the Poisson point estimate. With only seven clusters, the approximation can still be unstable. Compare the implementation against independent software such as R `clubSandwich` before using final publication estimates.

## Analysis population

 Only the **135 validated handover sessions** contribute to the models.

This repository **does not include the clinical observation data**, dates-by-ward audit output or any patient-level information.

## Setup and execution

Python 3.10 or later:

```bash
python -m pip install -r requirements.txt
python -m unittest discover -s tests -v
```

The source workbook must be inside a private `Handover Manuscript.zip` archive, with sheet `Dokumentation`.

```bash
python ward_handover_analysis.py \
  --zip-file "/secure/path/Handover Manuscript.zip" \
  --output-dir results
```

For the primary randomized comparison, add the **four confirmed randomized AI-assigned ward IDs** (never infer these from actual use or worksheet group labels):

```bash
python ward_handover_analysis.py \
  --zip-file "/secure/path/Handover Manuscript.zip" \
  --ai-wards WARD_1 WARD_2 WARD_3 WARD_4 \
  --randomization-scheme unrestricted \
  --output-dir results
```


## Results and quality assurance

The `results/` directory (gitignored) contains:

- `summary.json`: quality-control counts, ITT results if verified, as-treated duration and count analyses.
- `as_treated_duration.csv`: duration CR2/Satterthwaite analysis.
- `as_treated_interruptions_poisson_CR2.csv` and `as_treated_questions_poisson_CR2.csv`: Poisson CR2 results.
- `ITT_primary.csv` and `ITT_ward_level_summary.csv`: only if randomized ward assignments are provided.
- `as_treated_descriptives.csv`, `ward_adherence_summary.csv`: aggregate supporting summaries.

The command `--export-session-audit` generates session-level information. **Do not commit that output** to GitHub. The test suite uses synthetic records, not trial outcomes.


## Citation and licensing

- The citation metadata is in [`CITATION.cff`](CITATION.cff). GitHub can surface this through **Cite this repository**. Author credits in that file currently name the corresponding researcher; verify software-contributor attribution before publication. Add a permanent repository URL and Zenodo DOI when available.
- A proposed [MIT license](LICENSE) is included for straightforward reuse. **Confirm institutional intellectual-property ownership and approval before making the repository public.** Publishing a repository with this `LICENSE` file grants those permissions; if approval is unavailable, remove or replace the proposed license before release.
- The software version is tracked in [`CHANGELOG.md`](CHANGELOG.md). `0.1.0` is a release candidate, not a claim of a validated publication release.
- Do not publish the original clinical ZIP archive or restricted session-level audit.

### Reproducible dependency setup

`requirements.txt` contains supported dependency ranges for development and CI. `requirements-pinned.txt` contains the **exact direct-dependency versions** used for local verification on Python 3.13.5; it is not a complete lockfile of transitive dependencies. To recreate the locally tested direct-dependency environment:

```bash
python -m pip install -r requirements-pinned.txt
python -m unittest discover -s tests -v
```

For archival reproducibility of a validated final run, record the Python version and export a complete environment specification or dependency lock alongside the resulting software release.
