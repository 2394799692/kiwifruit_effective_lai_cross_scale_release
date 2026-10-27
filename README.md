# UAV-based Effective LAI inversion and cross-scale mapping in mature trellised kiwifruit orchards

Code, processed measurements, model results, and supplementary analysis files accompanying the study:

> **UAV-based Effective LAI Inversion and Cross-scale Mapping in Mature Trellised Kiwifruit Orchards Using Canopy Geometry and Texture**  
> Wenjie Li, Jianping Qian, Linghuan Ouyang, Qian Chen, Zhen Wang, Zhenzhong Lei, Jiali Li, Zheming Qiu, Hongen Liu, Yongqiang Zheng  
> *Computers and Electronics in Agriculture* (manuscript under revision)

## Study overview

The workflow combines UAV-derived crown radius and canopy texture to estimate optically derived Effective LAI for 82 mature, fruit-bagged, T-bar trellised kiwifruit trees. Crown radius is the primary predictor, and GLCM texture correlation (`Tex_Cor`) adds complementary information. The tree-level Linear SVR model is transferred to a 0.5 m orchard grid, followed by Mean-P99 correction of the grid-prediction distribution.

- Tree-scale model: Linear SVR with `Radius + Tex_Cor` (`C=0.1`, `epsilon=0.05`)
- LOOCV: R² = 0.759; RMSE = 0.272; RPD = 2.051
- Radius-only LOOCV R² = 0.738; Tex_Cor provides a modest incremental improvement
- NIR- and blue-derived texture combinations: repeated 5-fold mean R² approximately 0.750 for both
- Calibration-domain distribution means: ground 2.305; raw grid 3.536; Mean-P99-corrected grid 2.315

The label `LAI` in the supplied CSVs and scripts refers to the optically derived Effective LAI defined in the manuscript.

## Repository contents

```text
.
├── README.md                 Main guide
├── requirements.txt          Python dependencies
├── CITATION.cff              Citation metadata
├── data/                     Processed 82-tree feature tables
├── scripts/                  Main workflow (01–06), ablation (07),
│                             NIR/blue comparison (08), Fig. 12 (09)
├── results/                  Manuscript result tables and figure statistics
├── figures/                  Final Fig. 12 in PNG and PDF formats
├── examples/                 Example input schemas for mapping
├── docs/                     Workflow description and paper-to-code map
├── data_access/              UAV data-access information
└── patent/                   Supporting application documents
```

### Data

- [`data/LAI_Final_17cols.csv`](data/LAI_Final_17cols.csv): 82 samples × 17 columns, covering Effective LAI and 16 candidate predictors.
- [`data/LAI_RGB_Texture_Comparison_82.csv`](data/LAI_RGB_Texture_Comparison_82.csv): the same 82 samples with NIR, grayscale, red-, green- and blue-channel texture features.
- [`data/README.md`](data/README.md): column definitions.

### Main scripts

| Script | Task |
| --- | --- |
| `01_Data_Analysis_Feature_Selection.py` | Feature-response analysis |
| `02_Model_Training_Comparison.py` | PLSR, Linear SVR, RF and BPNN |
| `03_SVR_Formula_Extraction.py` | Final Linear SVR model parameters and equation |
| `04_Mapping_and_Correction.py` | Grid features, model prediction and Mean-P99 correction |
| `05_MAUP_Correction_Analysis.py` | Distributional comparison |
| `06_MeanP99_Spatial_Holdout_and_Mapping.py` | Five-block within-orchard correction evaluation |
| `07_Radius_TexCor_Fixed_Parameter_Ablation.py` | Fixed-parameter ablation, Wilcoxon test, repeated 5-/10-fold CV |
| `08_NIR_Blue_Texture_Comparison.py` | Controlled NIR vs blue texture comparison |
| `09_Fig12_Distribution_Plot.py` | Matched-grid distribution plot for Fig. 12 |

### Results and figures

[`results/README.md`](results/README.md) describes the released tables for feature subsets (Table 2), NIR/RGB texture comparison (Table 3), model settings and stability (Tables 4–5), fixed-parameter ablation, spatial holdout (Table 6) and Fig. 12 distribution statistics.

The final Fig. 12 is available as [`PNG`](figures/Fig12_R3_final.png) and [`PDF`](figures/Fig12_R3_final.pdf).

## Installation

Python 3.10 is recommended.

```bash
python -m pip install -r requirements.txt
```

## Tree-scale analysis

Run from the repository root:

```bash
python scripts/01_Data_Analysis_Feature_Selection.py --data data/LAI_Final_17cols.csv --output-dir outputs/01_feature_selection
python scripts/02_Model_Training_Comparison.py --data data/LAI_Final_17cols.csv --features "crown radius" Tex_Cor --output-dir outputs/02_model_comparison
python scripts/03_SVR_Formula_Extraction.py --data data/LAI_Final_17cols.csv --features "crown radius" Tex_Cor --output-dir outputs/03_svr_formula
python scripts/07_Radius_TexCor_Fixed_Parameter_Ablation.py
python scripts/08_NIR_Blue_Texture_Comparison.py
```

The final published-scale Linear SVR equation is:

```text
Effective LAI = 0.7511 × crown radius + 14.3497 × Tex_Cor − 12.9206
```

## Orchard mapping and cross-scale analysis

The grid workflow uses spatially aligned NDVI/NIR rasters, a crown-radius proxy calibration table, grid-level prediction values and sample-tree coordinates. Example file layouts are included in [`examples/`](examples/).

```bash
python scripts/04_Mapping_and_Correction.py --train-data data/LAI_Final_17cols.csv --radius-calibration-data path/to/radius_calibration.csv --ndvi path/to/NDVI.tif --nir path/to/NIR.tif --output-dir outputs/04_mapping
python scripts/05_MAUP_Correction_Analysis.py --ground-data data/LAI_Final_17cols.csv --grid-data path/to/Map_Grid_Data_Corrected.csv --output-dir outputs/05_distribution
python scripts/06_MeanP99_Spatial_Holdout_and_Mapping.py --ground-data data/LAI_Final_17cols.csv --spatial-data path/to/LAI_Spatial_Coordinates.csv --grid-data path/to/Map_Grid_Data_Corrected.csv --output-dir outputs/06_holdout
python scripts/09_Fig12_Distribution_Plot.py --grid path/to/Map_Grid_Data_Corrected.csv
```

Mean-P99 correction uses the mean and P99 of the ground-observed and raw grid distributions as two linear calibration anchors. The five-block evaluation re-estimates correction parameters using four spatial blocks and applies them to the held-out block.

For Fig. 12, raw and corrected distributions use the same 158,604 calibration cells (`LAI_Pred > 2.5`), with corrected values clipped to 0–6.

## Additional information

- [`docs/PAPER_TO_CODE_MAP.md`](docs/PAPER_TO_CODE_MAP.md): correspondence between manuscript components, scripts and numerical tables.
- [`docs/METHOD_WORKFLOW.md`](docs/METHOD_WORKFLOW.md): analysis workflow and spatial correction.
- [`USAGE_NOTICE.md`](USAGE_NOTICE.md): software and intellectual-property notice.
- [`patent/README.md`](patent/README.md): related Chinese invention patent application.
- [`CITATION.cff`](CITATION.cff): citation metadata.

**Repository:** https://github.com/2394799692/kiwifruit-lai-workbench-github
