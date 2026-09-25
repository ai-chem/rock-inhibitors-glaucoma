# Data

Datasets supporting *Generative AI under domain-specific constraints in
ophthalmology for ROCK inhibitor discovery*.

```
data/
├── ocular/        Training data for the three ocular property classifiers
├── receptor/      ROCK-2 structure and binding-site files (PDB ID: 6ED6)
├── reference/     Reference compound sets retrieved from ChEMBL
└── generated/     Molecules produced by the generative models, with scores
```

---

## ocular/

Training data for the corneal permeability (CP), melanin binding (MB) and eye
irritation (EI) classifiers. Each property is a binary classification task.
For CP and MB both the raw measurements and the modelling-ready file are
provided, so that the binarisation and balancing steps can be reproduced.

| File | Rows | Columns | Description |
|---|---|---|---|
| `corneal_permeability_raw.csv` | 163 | `smiles`, `title`, `log_papp` | Apparent permeability coefficients pooled from five ex vivo rabbit corneal perfusion studies. `title` gives the source study. |
| `corneal_permeability.csv` | 162 | `smiles`, `Class` | Modelling-ready set: one duplicate structure removed, then binarised at the median of `log_papp`. |
| `melanin_binding_raw.csv` | 780 | `smiles`, `fraction_unbound` | Fraction unbound in an in vitro melanin binding assay. |
| `melanin_binding.csv` | 373 | `smiles`, `Class` | Modelling-ready set after downsampling of the majority class. |
| `irritation.csv` | 5220 | `smiles`, `Class` | Eye irritation labels taken unchanged from a previously published in silico dataset. |


### Sources

Corneal permeability data were compiled manually from five ex vivo rabbit
corneal perfusion studies; cell-based permeability assays were excluded, as
they capture only part of the corneal transport process. Melanin binding data
come from a published in vitro study. Eye irritation labels come from a
published in silico dataset. Full citations are given in the Methods section
of the article.

Training and evaluation scripts for all three classifiers are in
`scripts/train_classifiers/`; the fitted models are in `models/`.

---

## receptor/

ROCK-2 structure used for docking, Boltz-2 re-ranking and the free energy
calculations, retrieved from the Protein Data Bank under accession code
[6ED6](https://www.rcsb.org/structure/6ED6).

---

## reference/

Compound sets retrieved from [ChEMBL](https://www.ebi.ac.uk/chembl/) and used
for comparison and benchmarking, not for training.

| File | Rows | Description |
|---|---|---|
| `known_rock_inhibitors.csv` | 93 | Known ROCK inhibitors, used for the physicochemical and Rule of Four comparison against generated molecules. |
| `approved_ophthalmic_drugs.csv` | 8 | Approved ophthalmic drugs, used as the upper reference in the same comparison. |
| `chembl_rock2_benchmark.csv` | 3031 | Human ROCK2 inhibitors with exact quantitative activity measurements, used to benchmark Boltz-2 against Uni-Dock. Filtered for assay-to-target confidence and measurement validity; structures standardised, duplicate measurements removed, compounds with conflicting activity classifications excluded, and repeated measurements aggregated in logarithmic concentration space. |

---

## generated/

`generated_molecules.csv` holds every molecule produced during the study, in
long format: one row per molecule per run. Each generative configuration was
run with three random seeds and 256 molecules per run.


