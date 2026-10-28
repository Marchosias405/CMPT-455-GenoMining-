# Hospital Readmission: EDA & Preprocessing

**Team:** Simon Doan · Ertugrul Yurtseven · Kazi Boni Amin

Explore diabetes hospital encounters and prepare inputs for predicting recorded readmission within 30 days at discharge. For example, compare prior inpatient visits across readmission groups. This stage covers **EDA and preprocessing**, with no prediction accuracy claimed.

## Run it

Install **Git and Python 3.12**. In **PowerShell**, paste these commands and enter the repository's HTTPS clone URL when prompted. This README and notebook should be at the repository root.

```powershell
$repoUrl = Read-Host 'Repository HTTPS clone URL'
git clone "$repoUrl" readmission-eda
cd readmission-eda
```

Already copied/downloaded the folder? Open PowerShell inside it and start here:

```powershell
py -3.12 -m venv ..\.eda-venv
..\.eda-venv\Scripts\python.exe -m pip install -r requirements.txt
..\.eda-venv\Scripts\python.exe -m ipykernel install --sys-prefix --name python3
..\.eda-venv\Scripts\python.exe -m nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=900 EDA_Preprocessing.ipynb
```

- Rerun the **last command** to regenerate the notebook outputs, figures and prepared data. The report stays unchanged.
- Open the notebook in VS Code: expect **14 executed code cells, four figures and six PASS checks** in the final cell.
- Allow a few minutes and about 2 GB free RAM. Internet is needed for cloning/package installation; analysis uses bundled data. Tested with Python 3.12.14 on Windows; minor numerical differences across computers are possible.
- **macOS/Linux:** Inside the cloned/copied folder, replace `py -3.12` with `python3.12` and the Windows Python path with `../.eda-venv/bin/python`.

## Dataset

- **[Diabetes 130 US Hospitals, 1999–2008](https://doi.org/10.24432/C5230J)**, UCI: **101,766 encounters, 71,518 patients, 50 columns**.
- Bundled unchanged: `data/diabetic_data.csv` (encounters) and `data/IDS_mapping.csv` (admission/discharge code descriptions). The notebook verifies the encounter-file checksum.
- **Attribution:** John Clore, Krzysztof Cios, Jon DeShazo and Beata Strack; [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). The processing below produces derived data.

## Methods

- **Filter/label:** Exclude death/hospice discharge codes **11, 13, 14, 19, 20, 21**. Target: `<30` → 1; `>30`/`NO` → 0. IDs are excluded from predictors.
- **Split:** Approximately **60/20/20%** development/calibration/test, with no patient overlap; seed **459**. EDA and learned preprocessing use development data only.
- **Clean:** Check duplicates/ranges; `?` is missing, laboratory `None` means “not performed.” Drop >95%-missing or constant inputs; group ICD-9 diagnoses.
- **Transform:** Numeric median imputation and standardization; categorical `Missing` imputation and one-hot encoding, grouping rare levels (<25 records).
- **Select:** Mutual information selects **20 whole inputs before encoding**.
- **Outliers/PCA:** Flag with 1.5×IQR and Isolation Forest (3%); retain all rows, with no synthetic augmentation. Separately apply PCA to all 216 encoded columns to retain ≥95% variance.

## Results

- **99,343 eligible encounters; 11.39% early readmission.** Development/calibration/test: **59,670 / 19,746 / 19,927** encounters.
- Five inputs dropped; **42 retained → 20 selected → 147 encoded columns**. Prepared matrix: **59,670 × 147**.
- **65 PCA components retain 95.11% variance**. Strongest absolute numeric Spearman correlation: **0.463** (stay length vs. medications).
- Development data: **96.81% missing weight; 1,791 Isolation Forest flags**, with no outlier removal. Class imbalance and missingness are key challenges.

Associations are not causal; PCA variance is not prediction accuracy. Later cross-validation must refit preprocessing and selection inside each training fold.

## Files

| File | Purpose |
|---|---|
| `EDA_Preprocessing.ipynb` | All analysis, explanations, figures and checks. |
| `config.json` / `requirements.txt` | Analysis settings / pinned Python packages. |
| `data/` | Two source CSVs described above. |
| `outputs/figures/` | Four PNGs: missingness/classes, distributions, correlations, selection/PCA. |
| `outputs/prepared/development_top20.npz` | Sparse feature matrix; load with `scipy.sparse.load_npz`. |
| `outputs/prepared/development_row_index.csv` | Matching row order, IDs and target. |
| `outputs/prepared/encoded_feature_names.csv` | Matching matrix column order/names. |
| `Report.pdf` | Reference report with Ctrl+F terms connecting explanations to code; review before submitting. |

**Flow:** data + config → notebook → figures + prepared data. Finalize the report, then ZIP the folder as **EDA_Preprocessing.zip**, under **30,000,000 bytes**; exclude environments and caches.
