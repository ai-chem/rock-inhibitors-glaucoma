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

All structures are stored as SMILES unless stated otherwise. Files are UTF-8
CSV with a header row.

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

### Label conventions

Read these before using the files or the trained models.

**Corneal permeability.** `Class = 1` if `log_papp` is at or above the median
of the pooled dataset (higher permeability), `0` otherwise. No consensus
permeability threshold exists in the literature, so the median of the pooled
data is used; classifier performance was checked under thresholds shifted by
±0.2 and ±0.4 log units and was not materially affected. Class balance is
81 / 81 and follows from the median split rather than from curation.

**Melanin binding.** `Class = 1` if the fraction unbound is at or above 1%
(weak or no binding), `0` if it is below 1% (strong binding), using the
threshold reported in the source publication. The raw set contains 173 strong
and 607 weak binders; the majority class is downsampled to 200 with
`random_state=42`, giving 373 compounds. Note that strong binding is the
pharmacologically desirable property here, so it is the class labelled `0`.

**Eye irritation.** `Class = 1` for irritant, `0` for non-irritant, using the
threshold of the source dataset. The set is left unbalanced at 3874 / 1346.

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

| File | Description |
|---|---|
| `6ed6_clean.pdb` | Protein with co-crystallised ligand, solvent and ions removed. |
| `6ed6_clean_pocket_12.pdb` | Residues within 12 Å of the co-crystallised ligand. |
| `6ed6_clean_pocket_15.pdb` | Residues within 15 Å of the co-crystallised ligand. |
| `6ed6_ligand.sdf` | Co-crystallised ligand, used as the reference geometry for docking box placement and pose validation. |

---

## reference/

Compound sets retrieved from [ChEMBL](https://www.ebi.ac.uk/chembl/) and used
for comparison and benchmarking, not for training.

| File | Rows | Description |
|---|---|---|
| `known_rock_inhibitors.csv` | 93 | Known ROCK inhibitors, used for the physicochemical and Rule of Four comparison against generated molecules. |
| `approved_ophthalmic_drugs.csv` | 8 | Approved ophthalmic drugs, used as the upper reference in the same comparison. |
| `chembl_rock2_benchmark.csv` | 3031 | Human ROCK2 inhibitors with exact quantitative activity measurements, used to benchmark Boltz-2 against Uni-Dock. Filtered for assay-to-target confidence and measurement validity; structures standardised, duplicate measurements removed, compounds with conflicting activity classifications excluded, and repeated measurements aggregated in logarithmic concentration space. |

Concentration-derived energies in `chembl_rock2_benchmark.csv` are potency
proxies and not thermodynamic binding free energies.

---

## generated/

`generated_molecules.csv` holds every molecule produced during the study, in
long format: one row per molecule per run. Each generative configuration was
run with three random seeds and 256 molecules per run.

| Column | Description |
|---|---|
| `method` | Generative architecture (`RXNFlow`, `TacoGFN`, `EvoSBDD`, `FREED++`, `TargetDiff`, `DecompDiff`, `AliDiff`, `DrugFlow`, `BRICS`). |
| `reward_scheme` | `none` for undirected generation, `D` for docking-only, `D,I,C,M` for docking plus the three ocular objectives. |
| `seed` | Random seed of the run. |
| `smiles` | Generated structure. |
| `qvina` | QVina docking score, kcal/mol. |
| `boltz2_ba` | Boltz-2 predicted binding affinity, kcal/mol. |
| `boltz2_conf` | Boltz-2 confidence score. |
| `cp`, `mb`, `ei` | Classifier outputs for corneal permeability, melanin binding and eye irritation. |
| `s_ocular` | Composite ocular reward, `cp * mb * (1 - ei)`. |
| `logp`, `tpsa`, `qed`, `sa` | Physicochemical descriptors computed with RDKit. |
| `ro4_pass` | Whether the molecule satisfies all four ophthalmic Rule of Four criteria. |

Classifier outputs are reported for every molecule, but many generated
compounds fall outside the applicability domain of the underlying models. They
should be read as a proxy signal used to steer generation, not as validated
predictions of the ocular properties of any individual candidate.

---

## Not included

Input and output files for the free energy perturbation calculations are not
part of this repository owing to their size. They are available from the
corresponding author on reasonable request.
