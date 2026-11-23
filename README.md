# Predicting Sulfur Adsorption Capacity of Zeolite and Activated Carbon Sorbents

## Abstract

Supervised regression was applied to 36 sulfur adsorption capacity measurements (mg of sulfur per gram of sorbent) compiled from published adsorptive desulfurization studies. Twenty measurements were on activated carbons (ACs) and 16 on rare-earth (RE) and Cu-RE exchanged Y zeolites. Ridge regression, lasso and XGBoost predicted capacity from measured material properties and were evaluated on materials held out of training.

For the ACs, lasso reached a mean absolute error (MAE) of 0.90 mg S/g, compared with 1.50 mg S/g for a baseline that predicts the mean capacity for the same sulfur compound. The Raman I<sub>D</sub>/I<sub>G</sub> ratio was the most informative input, but the margin over the baseline narrowed when one AC was removed. For the zeolites, only ridge regression outperformed the baseline (MAE 0.49 compared with 0.56 mg S/g), and only when inputs derived from the RE 4f electron count were included. Because the number of independent materials was small, the conclusions are exploratory.

## Introduction

The goal of this project was to build and critically evaluate regression models that predict the sulfur adsorption capacity of zeolite and activated carbon sorbents from their measured properties, using data compiled from published adsorptive desulfurization studies. The specific goals were:

1. To compile the published capacity data and material properties into one dataset with the source of every value recorded.
2. To train ridge regression, lasso and XGBoost models and compare them with simple baselines, using an evaluation that holds out whole materials, because the data contain few materials and repeated measurements of the same material.
3. To identify which material properties the models relied on, and to test how stable these results were when inputs or individual materials were removed.
4. To state the limits that the sample size places on the conclusions.

## 1. Data

### 1.1 Sources and test conditions

Data were extracted from Lee and Valla (2017), Lee, Tsilomelekis and Valla (2018), Lee et al. (2019), Lee et al. (2021) and Sokol et al. (2025).

Capacity was measured in fixed-bed adsorption experiments. The feed was a model fuel of n-octane containing a single sulfur compound (100 ppmw as sulfur) and a competing aromatic compound. The compounds used for each group are listed in Table 1. Capacity is the mass of sulfur adsorbed per gram of sorbent at saturation, in mg S/g.

### 1.2 Sorbents

**Table 1.** Sorbent groups. Each zeolite group has four materials, one for each RE (La, Ce, Nd, Sm).

| Group | Sorbent | Sulfur compound | Competing aromatic | Measurements |
|---|---|---|---|---|
| RE-Y | Y zeolite exchanged with one RE | BT | 20 wt% benzene | 4 |
| Cu-RE-Y | Y zeolite exchanged with Cu and one RE | BT | 20 wt% benzene | 4 |
| RE-SAY | SAY zeolite (mesoporous Y) exchanged with one RE | DBT | 1 wt% naphthalene | 4 |
| Cu-RE-SAY | SAY zeolite (mesoporous Y) exchanged with Cu and one RE | DBT | 1 wt% naphthalene | 4 |
| ACs | Ten activated carbons, listed below | DBT and DMDBT | 1 wt% naphthalene | 20 |

- **Sulfur compounds:** BT is benzothiophene, DBT is dibenzothiophene and DMDBT is 4,6-dimethyldibenzothiophene.
- **ACs:** FWAC15 to FWAC65 are six ACs made from food waste, where the number is the activation yield in wt%. MisAC, WAC and CoAC are made from miscanthus, walnut shell and coconut shell. CAC is a commercial AC.

### 1.3 Model inputs

**Table 2.** Inputs of the two models.

| Sorbent | Inputs | Number |
|---|---|---|
| ACs | Micropore and mesopore surface area; micropore and mesopore volume; elemental composition (C, H, N, O, P, Ca, K, Na); Raman I<sub>D</sub>/I<sub>G</sub> ratio; sulfur compound indicator (DMDBT) | 14 |
| Zeolites | RE loading and Cu loading; micropore and mesopore volume; sulfur compound indicators (BT, DBT); 4f electron count n<sub>f</sub>; distance \|n<sub>f</sub> − 1\|; product of this distance and the Cu loading | 9 |

Before the adsorption experiments, the zeolites were reduced in situ in the fixed-bed column under H<sub>2</sub> flow at 450 °C for 2 h (Lee et al., 2021). In situ XANES under a similar reduction (5% H<sub>2</sub> in He at 450 °C) showed that Ce, present as a mixture of 3+ and 4+ before reduction, was reduced to 3+ (Sokol et al., 2022). La, Nd and Sm are trivalent, so all four REs were taken as RE<sup>3+</sup> and encoded by the number of 4f electrons n<sub>f</sub> of the RE<sup>3+</sup> ion: 0 for La, 1 for Ce, 3 for Nd and 5 for Sm. A DFT 4f occupation of about 1.2 for Ce (Sokol et al., 2024) is consistent with n<sub>f</sub> = 1. Ce gave the highest capacity of the four REs in every group, a pattern that a count increasing from La to Sm cannot represent. The distance |n<sub>f</sub> − 1| from Ce (n<sub>f</sub> = 1) and its product with the Cu loading were therefore added. These inputs were defined after the pattern was observed in the same data, so results involving them are exploratory.

## 2. Methods

**Models.** Ridge regression and lasso were fit on standardized inputs. XGBoost used 100 trees of depth 2, a learning rate of 0.1, L2 regularization of 5.0 and a row subsample of 0.8. Its hyperparameters were not tuned.

**Evaluation.** Each material was predicted by a model trained on all other materials (leave-one-material-out cross-validation). All rows of a material were held out together, because an AC tested with both DBT and DMDBT would otherwise leak its own measurement into training. For the zeolites, which have one measurement per material, this is leave-one-out cross-validation. Predictions for all held-out materials were pooled and scored by MAE, root-mean-square error (RMSE) and R<sup>2</sup>.

**Regularization.** The regularization strength of ridge and lasso was chosen from 40 values between 0.01 and 1000 by an inner cross-validation within each training set (nested cross-validation), so that the held-out material did not influence it.

**Baselines.** The mean baseline predicts the average capacity of the training materials. The group baseline predicts the average capacity of the training materials in the same group: the sulfur compound for the ACs, and the four groups of Table 1 for the zeolites.

**Additional analyses.** The models were refit without I<sub>D</sub>/I<sub>G</sub> for the ACs and without the two 4f-distance inputs for the zeolites. The AC analysis was repeated without CAC, the AC with the lowest capacity.

## 3. Results

### 3.1 ACs

I<sub>D</sub>/I<sub>G</sub> had the strongest correlation with capacity (r = 0.60 across the 20 measurements and 0.74 across the 10 materials, using mean capacity per material), followed by hydrogen content (r = −0.56), micropore volume (0.52), micropore surface area (0.51) and potassium content (0.50). Within a single sulfur compound, the correlation with I<sub>D</sub>/I<sub>G</sub> was 0.84 for DBT and 0.67 for DMDBT.

**Table 3.** Cross-validated predictions for the ACs, using all 14 inputs.

| Model | MAE (mg S/g) | RMSE (mg S/g) | R<sup>2</sup> |
|---|---|---|---|
| Mean baseline | 1.84 | 2.25 | −0.15 |
| Group baseline (sulfur compound) | 1.50 | 2.03 | 0.06 |
| Ridge | 0.93 | 1.29 | 0.62 |
| Lasso | 0.90 | 1.15 | 0.70 |
| XGBoost | 1.12 | 1.49 | 0.50 |

All three models outperformed both baselines, and lasso was the best. The largest errors were for CAC with DMDBT (measured 2.0 mg S/g, predicted 4.1 to 5.9 by the three models) and MisAC with DMDBT (measured 9.1, predicted 5.8 to 6.7).

The dependence on I<sub>D</sub>/I<sub>G</sub> and on a single AC was tested by refitting the models with different input sets, with and without CAC. CAC had the lowest capacity (2.15 mg S/g on average) and the lowest I<sub>D</sub>/I<sub>G</sub> (1.1).

**Table 4.** MAE (mg S/g) of the AC models for different input sets. The group baseline MAE is 1.50 with all ten ACs and 1.24 without CAC.

| Input set | ACs | Ridge | Lasso | XGBoost |
|---|---|---|---|---|
| All 14 inputs | 10 | 0.93 | 0.90 | 1.12 |
| Without I<sub>D</sub>/I<sub>G</sub> | 10 | 1.90 | 1.83 | 1.18 |
| I<sub>D</sub>/I<sub>G</sub> and sulfur compound indicator only | 10 | 1.31 | 1.30 | 1.65 |
| All 14 inputs | 9, without CAC | 1.47 | 1.01 | 1.05 |
| Without I<sub>D</sub>/I<sub>G</sub> | 9, without CAC | 2.07 | 2.23 | 1.15 |
| I<sub>D</sub>/I<sub>G</sub> and sulfur compound indicator only | 9, without CAC | 1.37 | 1.45 | 1.15 |

- Removing I<sub>D</sub>/I<sub>G</sub> increased the error of every model, but only slightly for XGBoost (1.12 to 1.18).
- I<sub>D</sub>/I<sub>G</sub> and the sulfur compound indicator alone gave an MAE of 1.31 with ridge and 1.30 with lasso, better than the group baseline (1.50) but not as good as the models with all inputs (0.93 and 0.90).
- Without CAC, ridge with all inputs no longer outperformed the group baseline (1.47 compared with 1.24), whereas lasso (1.01) and XGBoost (1.05) did. The correlation between I<sub>D</sub>/I<sub>G</sub> and capacity fell to 0.36 across measurements and 0.50 across materials.

### 3.2 Zeolites

**Table 5.** Cross-validated predictions for the zeolites, using all 9 inputs.

| Model | MAE (mg S/g) | RMSE (mg S/g) | R<sup>2</sup> |
|---|---|---|---|
| Mean baseline | 0.75 | 1.05 | −0.14 |
| Group baseline (four groups of Table 1) | 0.56 | 0.87 | 0.22 |
| Ridge | 0.49 | 0.71 | 0.48 |
| Lasso | 0.61 | 0.78 | 0.37 |
| XGBoost | 0.61 | 0.96 | 0.06 |

Only ridge outperformed the group baseline, by 0.07 mg S/g, a margin too small to interpret at this sample size. Without the two 4f-distance inputs, the MAE was 0.64 for ridge, 0.62 for lasso and 0.72 for XGBoost, and no model outperformed the group baseline.

The largest errors were for CuCeSAY (measured 4.20 mg S/g, predicted 1.7 to 2.2 by the three models) and CuLaSAY (measured 3.30, predicted 1.3 to 2.1). All three models under-predicted both.

## 4. Discussion

**ACs.** Removing I<sub>D</sub>/I<sub>G</sub> raised the MAE of ridge from 0.93 to 1.90 and of lasso from 0.90 to 1.83, whereas XGBoost changed little (1.12 to 1.18). A model with only I<sub>D</sub>/I<sub>G</sub> and the sulfur compound indicator reached 1.31 with ridge and 1.30 with lasso. I<sub>D</sub>/I<sub>G</sub> was only weakly correlated with the phosphorus, calcium, potassium and sodium contents (r = 0.10 to 0.40), so it did not act as a proxy for them. CAC, however, differed from the other ACs in capacity, I<sub>D</sub>/I<sub>G</sub> and potassium content (it was the only AC without potassium), and without CAC ridge no longer outperformed the group baseline, while the margins of lasso and XGBoost narrowed. The I<sub>D</sub>/I<sub>G</sub> result is therefore suggestive and needs more ACs to be confirmed.

**Zeolites.** The group baseline (MAE 0.56) already captured most of the signal, and only ridge improved on it (0.49). That improvement depended on the two 4f-distance inputs: without them no model outperformed the group baseline. These inputs were defined after the Ce maximum was observed in the same data and, with four REs, are recodings of RE identity, so the improvement is weak, exploratory evidence and not a confirmation.

## 5. Limitations

- **Small, grouped sample.** The analysis used 10 ACs and 16 zeolites. The zeolites form four groups of four, so any comparison within a group rests on four materials.
- **Confounding in the zeolite data.** The Y and SAY zeolites differ in sulfur compound, zeolite type and competing aromatic at the same time, so they cannot be compared directly. The Cu-RE zeolites carry about half the RE loading of the RE zeolites and the Cu loading varies only from 1.9 to 2.8 wt%, so the effects of Cu and RE loading cannot be separated.
- **Approximate values.** Fifteen of the 36 capacities (13 zeolite and 2 AC) were read from figures and are less precise than the others, which adds noise to the targets.
- **Oxidation state assumption.** The 4f counts assume RE<sup>3+</sup> for all four REs. For Ce this rests on XANES of CeY and CuCeY reduced under similar conditions (Sokol et al., 2022), not on measurements of the zeolites that were tested.
- **No uncertainty estimates.** Each metric comes from a single pass of leave-one-material-out predictions, and comparisons between input sets are further selection steps on the same data.
- **Library version dependence.** XGBoost results depended on the library version: the same code and seed with older libraries (pandas 1.3.5, scikit-learn 1.0.2, XGBoost 1.6.2) left ridge and lasso unchanged but moved XGBoost MAEs by up to 0.19 mg S/g.

## 6. Conclusions

1. For the ACs, lasso with 14 inputs predicted held-out materials better than the group baseline (MAE 0.90 compared with 1.50 mg S/g). I<sub>D</sub>/I<sub>G</sub> was the most informative input, and the margin over the baseline narrowed without CAC.
2. For the zeolites, only ridge regression outperformed the group baseline (MAE 0.49 compared with 0.56 mg S/g), and only with the exploratory 4f-based inputs.
3. The number of independent materials was too small for firm conclusions in either family. More materials per group would be the next step.

## 7. Repository and reproducibility

```
data/        the compiled dataset and the model-ready tables
Notebooks/   01_eda.ipynb, 02_feature_engineering.ipynb, 03_modeling.ipynb
reports/     figures produced by the notebooks
```

| Notebook | Content |
|---|---|
| `01_eda.ipynb` | Distributions, correlations and group comparisons |
| `02_feature_engineering.ipynb` | Construction of the model-ready tables |
| `03_modeling.ipynb` | Models, cross-validation, baselines, bias-variance and learning curves, and error analysis (about 8 minutes) |

The notebooks are run in this order. `requirements.txt` lists exact package versions and needs Python 3.12 or newer. To set up an environment and open the notebooks:

```
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter lab
```

The outputs saved in the notebooks were generated in an environment built from `requirements.txt` with Python 3.12 (pandas 3.0.5, numpy 2.5.3, matplotlib 3.11.2, scikit-learn 1.9.1 and XGBoost 3.4.1).

### Data dictionary

The compiled dataset, `adsorptive_desulfurization_dataset.csv`, has 56 rows.

| Column | Meaning |
|---|---|
| `material`, `sorbent_class`, `source_paper`, `source_page`, `notes` | Identity and provenance of each row. `sorbent_class` is `zeolite_Y`, `zeolite_SAY` or `activated_carbon`. |
| `metal_1`, `metal_2`, `metal_1_wt_pct`, `metal_2_wt_pct` | Exchanged metals and their ICP-measured loadings, as listed in the source (zeolites) |
| `RE_type`, `RE_wt_pct`, `Cu_wt_pct` | The same information, arranged so that each column has the same meaning in every row |
| `BET_total_m2g`, `BET_micro_m2g`, `BET_meso_m2g`, `Vtotal_cm3g`, `Vmicro_cm3g`, `Vmeso_cm3g` | Surface areas and pore volumes from N<sub>2</sub> physisorption |
| `inorganic_P_wt_pct`, `inorganic_Ca_wt_pct`, `inorganic_K_wt_pct`, `inorganic_Na_wt_pct` | Inorganic content of the ACs |
| `carbon_C_wt_pct`, `carbon_H_wt_pct`, `carbon_N_wt_pct`, `oxygen_O_wt_pct` | Elemental composition of the ACs (oxygen by difference) |
| `ID_IG_ratio` | Raman I<sub>D</sub>/I<sub>G</sub> ratio of the ACs |
| `sulfur_compound`, `feed_conc_ppmw`, `solvent`, `co_adsorbate` | Test conditions |
| `capacity_mgS_g` | Saturation capacity, the modeling target |
| `breakthrough_mL_g` | Effluent volume per gram of sorbent before sulfur was detected (not modeled) |
| `target_available` | Which of the two targets a row reports |
| `value_type` | `exact` for a value stated in the source, `approx` for a value read from a figure or a bound stated in the source |

## References

1. Lee, K. X.; Valla, J. A. Investigation of metal-exchanged mesoporous Y zeolites for the adsorptive desulfurization of liquid fuels. *Appl. Catal., B* **2017**, 201, 359-369.
2. Lee, K. X.; Tsilomelekis, G.; Valla, J. A. Removal of benzothiophene and dibenzothiophene from hydrocarbon fuels using CuCe mesoporous Y zeolites in the presence of aromatics. *Appl. Catal., B* **2018**, 234, 130-142.
3. Lee, K. X.; Wang, H.; Karakalos, S.; Tsilomelekis, G.; Valla, J. A. Adsorptive desulfurization of 4,6-dimethyldibenzothiophene on bimetallic mesoporous Y zeolites: effects of Cu and Ce composition and configuration. *Ind. Eng. Chem. Res.* **2019**, 58, 18301-18312.
4. Lee, K. X.; Crowl, T. B.; Sokol, H. J.; Morales-Acosta, M. D.; Valla, J. A. Understanding the role of rare earths in zeolite Y on the removal of sulfur from hydrocarbon fuels. *J. Phys. Chem. C* **2021**, 125, 9107-9118.
5. Sokol, H. J.; Ebrahim, A. M.; Caratzoulas, S.; Frenkel, A. I.; Valla, J. A. In situ XAFS, XRD, and DFT characterization of the sulfur adsorption sites on Cu and Ce exchanged Y zeolites. *J. Phys. Chem. C* **2022**, 126, 1496-1512.
6. Sokol, H. J.; Lee, K. X.; Valla, J. A. The mechanism of adsorptive desulfurization on Cu and Ce exchanged Y zeolite using density functional theory. *Comput. Mater. Sci.* **2024**, 235, 112813.
7. Sokol, H. J.; Pedram, S.; Jankovic, J.; Valla, J. A. Renewable activated carbon sorbent for the desulfurization of liquid fuels. *ACS Omega* **2025**, 10, 42725-42734.
