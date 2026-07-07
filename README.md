# MIM: Dual-Pathway Analysis of Mobile Instant Messaging and Psychological Workload

Reproducible Python package for the study *"Digital Connectivity and Occupational Stress:
A Job Demands–Resources Analysis of Dual Pathways to Psychological Workload."*

The package estimates how workplace **mobile instant messaging (MIM)** use relates to
**psychological workload** through two competing pathways — a resource pathway
(via **interactivity**) and a demand pathway (via **task interruption**) — and tests
**communication quality** as a moderator. Everything runs from a single command and
writes tables and figures to `outputs/`.

> **Repository:** https://github.com/caceps/MIM

---

## Theoretical model

```
                     +----  Interactivity (resource) ----+
                     |                                    v
   MIM Use  ---------+                             Psychological Workload
                     |                                    ^
                     +----  Task Interruption (demand) ---+
                                    ^        ^
                                    |        |
                        Communication Quality (moderator)
```

The framework treats digital technologies not as fixed job demands or resources, but as
**conditional work characteristics** whose demand-versus-resource classification is set by
contextual resource conservation (here, communication quality).

---

## Installation

```bash
git clone https://github.com/caceps/MIM.git
cd MIM
python -m venv .venv && source .venv/bin/activate      # optional
pip install -r requirements.txt
```

Python 3.9+ is required. Dependencies: `numpy`, `pandas`, `scipy`, `matplotlib`, `semopy`.

---

## Quick start

Reproduce the entire analysis (tables + figures) in one command:

```bash
python scripts/run_all.py
```

Results are written to `outputs/`:

| File | Contents |
|------|----------|
| `results_report.txt` | Full human-readable report of every analysis |
| `reliability.csv` | Loading range, Cronbach's alpha, composite reliability (CR), AVE per construct |
| `descriptives.csv`, `correlations.csv` | Means/SDs and the correlation matrix (sqrt-AVE on diagonal) |
| `vif.csv` | Full-collinearity VIFs (Kock, 2015) |
| `mediation_indirect.csv` | Bootstrap indirect effects |
| `moderated_interactions.csv`, `moderated_conditional.csv` | Moderated-mediation results |
| `fig_*.png` | Descriptives, interaction plots, indirect-effect forest plot |

### Use as a library

```python
import mim

df = mim.load_data()                       # validated survey data
scores = mim.composite_scores(df)          # five construct composites

mim.reliability_table(df)                  # alpha / CR / AVE
mim.correlation_matrix(scores, df)         # Fornell-Larcker matrix
mim.full_collinearity_vif(scores)          # CMV via VIF
mim.parallel_mediation(scores)             # PROCESS Model 4 logic
mim.moderated_mediation(scores)            # PROCESS Model 14 logic
mim.sensitivity_power(n=len(df), k=6)      # sensitivity power analysis
```

---

## What the package computes

1. **Reliability & convergent validity** — Cronbach's alpha, plus composite reliability (CR)
   and average variance extracted (AVE) from the standardized loadings of the joint
   five-factor confirmatory measurement model.
2. **Descriptives & discriminant validity** — construct means/SDs and a correlation matrix
   with the square root of AVE on the diagonal (Fornell-Larcker criterion).
3. **Common method variance & multicollinearity** — Harman's single-factor test and the
   full-collinearity VIF approach (all VIF < 3.3 indicates no CMB; Kock, 2015).
4. **Parallel mediation** — standardized a-, b-, and c'-paths with bias-corrected
   bootstrap confidence intervals (5,000 resamples) for both indirect effects.
5. **Moderated mediation** — communication quality on the b-paths, the index of moderated
   mediation, and conditional indirect effects at ±1 SD of the moderator.
6. **Sensitivity power analysis** — minimum detectable interaction effect size (Cohen's
   *f²*) at 80% and 95% power for the given N.

All figures are generated **from the fitted results**, not from hard-coded numbers.

---

## Results on the dataset (N = 235)

Running `scripts/run_all.py` on `data/MIM_Workload_Data.csv` yields:

**Reliability (CR and AVE from the five-factor CFA; all α > 0.88)**

| Construct | α | CR | AVE |
|-----------|---|----|-----|
| MIM Use | 0.899 | 0.901 | 0.603 |
| Interactivity | 0.882 | 0.884 | 0.656 |
| Task Interruption | 0.906 | 0.907 | 0.765 |
| Communication Quality | 0.884 | 0.888 | 0.666 |
| Psychological Workload | 0.910 | 0.909 | 0.770 |

**Mediation (standardized, 5,000 bootstraps)**

| Path | Coefficient |
|------|-------------|
| MIM Use → Interactivity (a1) | **0.570** |
| MIM Use → Task Interruption (a2) | **−0.399** |
| Interactivity → Workload (b1) | **−0.242** |
| Task Interruption → Workload (b2) | **0.581** |
| Indirect via Interactivity | **−0.138** [−0.223, −0.069] ✓ |
| Indirect via Task Interruption | **−0.232** [−0.326, −0.153] ✓ |
| Total indirect | **−0.369** [−0.480, −0.272] ✓ |

**Moderated mediation (communication quality on the b-paths)**

| Interaction | Coef | Index of moderated mediation (95% CI) | Significant |
|-------------|------|----------------------------------------|-------------|
| Interactivity × CQ → Workload | −0.140 | −0.080 [−0.133, −0.022] | ✓ |
| Task Interruption × CQ → Workload | −0.142 | 0.056 [0.012, 0.097] | ✓ |

Communication quality **strengthens** the workload-reducing resource pathway and **weakens**
the workload-increasing demand pathway. Both moderated-mediation indices are significant.

**Diagnostics** — Harman single factor = 46.6% (passes < 50%); max VIF = 2.81 (passes Kock < 3.3);
Fornell-Larcker discriminant validity satisfied.

**Power** — with N = 235, the design detects an interaction as small as *f²* = 0.034 at 80% power.

These results correspond to the findings reported in the manuscript.

---

## Data provenance

`data/MIM_Workload_Data.csv` is derived from the authoritative survey workbook
`MIM_WorkStress_Dataset_2022-2024.xlsx` (data collected 2022–2024), which includes a
codebook, full item-level responses, and demographic variables. Item responses show
normal within-respondent variation and reliabilities in the expected 0.88–0.91 range.

Several other files circulated during drafting. Always run the pipeline against the authoritative
workbook above.

---

## Repository structure

```
MIM/
├── README.md
├── CHANGELOG.md
├── LICENSE                     # MIT
├── requirements.txt
├── setup.py
├── data/
│   └── MIM_Workload_Data.csv   # survey data (N = 235)
├── mim/                        # analysis package
│   ├── __init__.py
│   ├── config.py               # item groupings & constants (edit to re-point)
│   ├── data_loading.py         # load + validate
│   ├── reliability.py          # alpha, CR, AVE
│   ├── descriptives.py         # descriptives + Fornell-Larcker
│   ├── cmv.py                  # Harman + full-collinearity VIF
│   ├── mediation.py            # parallel mediation + bootstrap
│   ├── moderation.py           # moderated mediation
│   ├── power.py                # sensitivity power analysis
│   └── plots.py                # figures
├── scripts/
│   └── run_all.py              # one-command reproduction
├── tests/
│   └── test_smoke.py
└── outputs/                    # generated tables & figures
```

To adapt the pipeline to a renamed dataset, edit only `mim/config.py`.

---

## Data dictionary

| Column(s) | Meaning |
|-----------|---------|
| `Respondent_ID` | Participant identifier |
| `Gender`, `Age`, `Education`, `Work_Experience_Years`, `Industry` | Demographics |
| `MIM_Tools_Count`, `Years_MIM_Use`, `MIM_Contacts`, `Primary_MIM_Tool` | Usage context |
| `TX1–TX6` | MIM Use items |
| `JH1–JH4` | Interactivity items |
| `GR1–GR3` | Task Interruption items |
| `ZL1–ZL4` | Communication Quality items |
| `FH1–FH3` | Psychological Workload items |
| `MIM_Use … Psychological_Workload` | Pre-computed composite (mean) scores |

All items are 7-point Likert (1 = strongly disagree, 7 = strongly agree).

---

## Testing

```bash
pip install pytest
pytest tests/
```

---

## Citation

If you use this code, please cite the associated manuscript and this repository:

```
CACEPS (2026). MIM: Dual-pathway JD-R analysis of mobile instant messaging and
psychological workload. https://github.com/caceps/MIM
```

## License

MIT — see [LICENSE](LICENSE).
