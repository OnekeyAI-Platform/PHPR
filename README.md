# PHPR: Postoperative risk stratification in pancreatic ductal adenocarcinoma using CT habitat and peritumoral radiomics   

This project investigates the prognostic value of intratumoral habitat and peritumoral CT radiomics in patients undergoing resection for pancreatic ductal adenocarcinoma. The retrospective, single-centre study included 315 patients, divided into training (n = 221) and internal test (n = 94) cohorts. Five imaging representations were evaluated: whole-tumour radiomics, 1-, 3-, and 5-mm peritumoral bands, and a three-habitat intratumoral representation derived from voxel-wise CT features. Cox-based models were developed to assess overall survival, with an integrated model combining habitat and 3-mm peritumoral scores with clinicopathological variables. The habitat model showed the strongest discrimination among the imaging-only models. The integrated model provided no clear improvement over habitat radiomics alone. These findings support further investigation of CT-based postoperative risk stratification, while independent external validation remains necessary. This repository will provide analysis code, configuration files, and supporting documentation upon acceptance of the associated manuscript.

## Workflow

Run the notebooks from the repository root in the order below, executing cells from top to bottom.

| Notebook | Purpose |
| --- | --- |
| `Step0. Prepare_training_data.ipynb` | Match images, masks, and survival records; prepare cohort splits. |
| `Step1.1 Radiomics.ipynb` | Extract whole-tumor features, select predictors, and fit a Cox model. |
| `Step1.1.1 ICC.ipynb` | Assess feature reproducibility using repeated segmentations (ICC threshold: 0.7). |
| `Step1.2.1 Habitat.ipynb` | Extract and combine features from three habitats; fit a Cox model. |
| `Step1.3.1 Peri1mm.ipynb` | Analyze the 1-mm peritumoral region. |
| `Step1.3.2 Peri3mm.ipynb` | Analyze the 3-mm peritumoral region. |
| `Step1.3.3 Peri5mm.ipynb` | Analyze the 5-mm peritumoral region. |
| `Step1.4 Habitat_cluster_visualization.ipynb` | Visualize habitat clusters and Calinski-Harabasz scores. |
| `Step2.1 Clinical_baseline_statistics.ipynb` | Summarize baseline clinical characteristics. |
| `Step3.2. Clinical_model.ipynb` | Fit the clinical Cox model and export selected clinical variables. |
| `Step4.0 Combined_model.ipynb` | Combine clinical and imaging predictors; generate survival plots and a nomogram. |
| `Step4.1 Cindex.ipynb` | Compare C-indices and merge model predictions. |
| `Step4.2 Time_dependent_AUC.ipynb` | Evaluate 1-, 2-, and 3-year ROC curves, calibration, decision curves, and DeLong comparisons. |

ICC and habitat visualization are supplementary analyses. Prepared habitat maps and peritumoral masks are required; their complete generation pipeline is not included.

## Setup

Use a Python environment with Jupyter Notebook and the **Onekey (`onekey_algo`) toolkit**, including its custom radiology, habitat, survival, statistics, and nomogram components. Onekey is an external dependency and is not bundled here. Other dependencies include NumPy, pandas, SciPy, scikit-learn, scikit-survival, lifelines, Matplotlib, seaborn, nibabel, SimpleITK, and PyRadiomics. Exact package versions are not recorded; use a compatible Onekey environment and record versions for reproducibility.

1. Update `radio_dir`, `clinic_file`, `label_file`, and `survival_file` in `config.yaml`. The supplied absolute Windows paths refer to the original local environment.
2. Review `event_col` (default `OS`), `duration_col` (default `OSTime`), `task_column` (default `label`), `subsets`, and `summary_models`.
3. Create working directories and launch Jupyter:

   ```bash
   mkdir data features results img split_info
   jupyter notebook
   ```

`Launch_Jupyter.bat` is an optional Windows launcher tied to the original Conda installation; adapt its paths before use. CT extraction settings are in `custom_settings/exampleCT.yaml`; `exampleMR.yaml` is an additional example configuration.

## Input data

Patient images and clinical records are not included. Supply NIfTI images and corresponding masks with identical filenames, such as `001.nii.gz`. Notebook-specific directories under `radio_dir` are:

- `images/`, `images_std/`, and `images_re/`: images used by habitat/ICC, whole-tumor, and peritumoral analyses, respectively.
- `masks/` and `ICC/`: primary and repeated tumor segmentations.
- `masks_p1/`, `masks_p3/`, and `masks_p5/`: peritumoral masks.
- `habitat/cluster_3/`: habitat segmentation maps with labels 1, 2, and 3.
- `Survival.csv`: survival records read by Step0.

Step0 also refers to `masks_PDAC_nii/*/tumor.nii.gz` for source segmentations. Habitat visualization additionally expects `habitat/cluster_3/clusters.npy`, `habitat/ch.csv`, and an example image grid under `doc/`; adapt its example path.

The configured label file (default `group.csv`) should contain `ID`, `group` (`train` or `test`), `label`, `OS`, and `OSTime`. Use consistent identifiers across all files. Encode events as 1 and censoring as 0; the evaluation notebook assumes survival durations in months.

The baseline notebook reads `clinic_file`; the clinical model separately reads `data/clinical.csv`, which requires numerical clinical predictors, `ID`, and `group`. Step0 exports files to `split_info/`; explicitly select the intended split in `config.yaml` or prepare `group.csv` before modeling.

## Outputs and reproducibility

Features and selected predictors are saved in `features/`, predictions and evaluation tables in `results/`, and figures in `img/`. The clinical model exports `data/Clinical_clinical_sur.csv` for the combined model.

These are study analysis notebooks and require local data and a compatible external toolkit. Review notebook-specific paths and parameters alongside `config.yaml`. Optional tutorial calls retain the toolkit's original lookup keys as Unicode escapes for compatibility.

The existing analyses include test-cohort-based penalty searches and cohort-specific log-rank cutoff searches. Review these choices when designing independent validation. English translation preserves the modeling procedures and does not establish agreement with manuscript results.
