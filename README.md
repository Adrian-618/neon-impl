# Neon-HTLC: evaluation artifact

This repository reproduces the evaluation (Section 6 and Appendix C) of

> Sebastian Faust, Noemi Glaeser, Marius Hagemann, and Ziyan Qu. *Neon-HTLC: Collateral-Free HTLCs with Logarithmic Timeout.* ACM CCS 2026. https://doi.org/10.1145/3830454.3846741

The notebook `bribery-strategy.ipynb` computes the required safe timeouts of He-HTLC, He⁺-HTLC and Neon-HTLC for deposits from \$10 to \$100,000 and draws the figures of the paper. All computations are deterministic.

## Contents

| Path | Content |
|---|---|
| `bribery-strategy.ipynb` | Configuration, protocol models, experiments and figures |
| `real_distribution_69.txt` | Simulated Bitcoin mining power distribution used in the paper: 19 pools + 50 small miners (Section 6.1, Table 4) |
| `real_distribution_519.txt` | Same, with a longer tail of 500 small miners (Appendix C.4) |
| `real_distribution_decentralized.txt` | Decentralized Zipf distribution with 500 miners (Appendix C.5) |
| `data/` | Precomputed CSV results for `real_distribution_69`, as used for the figures of the paper |
| `results/<distribution>/` | Created by the notebook: `data/` (CSV results) and `figures/` (PDF figures) |

## Requirements

1. **Python 3.10 or newer** with the packages in `requirements.txt` (NumPy, Matplotlib, SciencePlots, Jupyter):

   ```bash
   python3 -m venv .venv && source .venv/bin/activate
   pip install -r requirements.txt
   ```

2. **LaTeX.** The figures use the SciencePlots styles `science` and `ieee`, which render all text with LaTeX (`text.usetex = True`). Matplotlib then needs a LaTeX installation including the `cm-super` fonts, and `dvipng` to show the figures inside Jupyter. Without them, the plotting cells fail.

   - Ubuntu/Debian: `sudo apt install texlive-latex-extra texlive-fonts-recommended cm-super dvipng`
   - macOS: [MacTeX](https://www.tug.org/mactex/), which includes all of the above.

   `latex --version` and `dvipng --version` should both work in the shell that starts Jupyter.

Tested with Python 3.13, NumPy 2.5.4, Matplotlib 3.11.2, SciencePlots 2.2.2 and TeX Live 2023 on Ubuntu 24.04.

## Usage

1. Open the notebook: `jupyter lab bribery-strategy.ipynb`
2. In the **Configuration** cell (second cell), choose the mining power distribution:

   | `DISTRIBUTION` | Used for |
   |---|---|
   | `"real_distribution_69"` (default) | Main results: Section 6, Figures 5–9, Table 4 |
   | `"real_distribution_519"` | Longer tail: Appendix C.4, Figures 10–13 |
   | `"real_distribution_decentralized"` | Decentralized distribution: Appendix C.5, Figures 14–17 |

3. Run all cells (*Kernel → Restart Kernel and Run All Cells*).

The results are written to `results/<DISTRIBUTION>/data/` and the figures to `results/<DISTRIBUTION>/figures/`, so runs with different distributions never overwrite each other. Running the notebook does not modify `data/` or the distribution files.

A run with `real_distribution_519` takes longer than the others (hours).

**Without the Jupyter interface:**

```bash
jupyter nbconvert --to notebook --execute --ExecutePreprocessor.timeout=-1 \
    bribery-strategy.ipynb --output-dir results --output executed_real_distribution_69.ipynb
```

To run all three distributions in one go, use [papermill](https://papermill.readthedocs.io/) (`pip install papermill`), which overrides the configuration cell:

```bash
mkdir -p results
for d in real_distribution_69 real_distribution_519 real_distribution_decentralized; do
    papermill bribery-strategy.ipynb "results/executed_$d.ipynb" -p DISTRIBUTION "$d"
done
```

### Further options in the configuration cell

| Option | Default | Meaning |
|---|---|---|
| `FEE_A` | `1.0` | Alice's transaction fee *f*<sub>A</sub> in USD |
| `T1` | `12` | Payment-phase timeout *T*<sub>1</sub> in blocks |
| `DEP_VALUES` | 500 log-spaced values from \$10 to \$100,000 | Deposits that are evaluated |
| `REGENERATE_DISTRIBUTION_FILES` | `False` | If `True`, the generator cells rewrite `real_distribution_<N>.txt` and `real_distribution_decentralized.txt`. Not needed to reproduce the paper. The experiments always use the file selected by `DISTRIBUTION`. |
| `NUM_UNKNOWN_MINERS` | `50` | Number of small miners for the realistic generator: 50 gives the 69-miner distribution, 500 the 519-miner one |

## Reproducing the figures and numbers of the paper

Set `DISTRIBUTION`, run all cells, and find the outputs in `results/<DISTRIBUTION>/`:

| Paper | `DISTRIBUTION` | Output in `results/<DISTRIBUTION>/` | Produced by |
|---|---|---|---|
| Figure 5 | `real_distribution_69` | `figures/HTLC_Legacy_Dep_Time.pdf` | `plot_legacy_failures_linear` |
| Figure 6 | `real_distribution_69` | `figures/HTLC_Percentage_Collaterals.pdf` | `plot_percentage_collaterals_inset` |
| Figure 7 | `real_distribution_69` | `figures/Neon_Escalation.pdf` | `plot_escalation_comparison` |
| Figure 8 | `real_distribution_69` | `figures/he-htlc.pdf` | `plot_collateral_vs_timeout` |
| Figure 9 | `real_distribution_69` | `figures/kappa-linear.pdf` | `plot_kappa_trend_linear` |
| Table 4 | any | printed by the first generator cell (with `NUM_UNKNOWN_MINERS = 50`) | `generate_realistic_bitcoin_distribution` |
| Figures 10, 11, 12, 13 | `real_distribution_519` | `figures/kappa-linear.pdf`, `HTLC_Legacy_Dep_Time.pdf`, `Neon_Escalation.pdf`, `HTLC_Percentage_Collaterals.pdf` | same functions |
| Figures 14, 15, 16, 17 | `real_distribution_decentralized` | the same four files | same functions |

Figures 5–7 and their appendix counterparts read the CSV files that the experiment cells write before them, so always run all cells. For some appendix figures, the paper uses different axis ranges than the defaults in the plotting cells; the curves are the same. The dotted payment-cap lines in Figure 5 (vertical asymptotes, Eq. (7)) are computed for `real_distribution_69` and are therefore only drawn for that distribution.

The numbers for a \$100,000 payment in Sections 1.1 and 6 are the rows with `Deposit_USD = 100000.0` in `results/real_distribution_69/data/`:

| Protocol and collateral | CSV file | Total timeout (blocks) |
|---|---|---|
| He-HTLC, 5% (\$5k) | `He_HTLC_col5pct.csv` | 496 |
| He-HTLC, 10% (\$10k) | `He_HTLC_col10pct.csv` | 54 |
| He⁺-HTLC, 5% (\$5k) | `He_plus_col5pct.csv` | 38 |
| Neon-HTLC, tight bound, no collateral | `Neon_Tight_col0.csv` | 26 |
| Neon-HTLC, tight bound, 5% (\$5k) | `Neon_Tight_col5pct.csv` | 17 |
| Neon-HTLC, distribution-agnostic bound, no collateral | `Neon_Agnostic_col0.csv` | 30 |

**CSV format.** Each file has the columns `Deposit_USD` and `Safe_T`, where `Safe_T` is the total timeout in blocks (*T*<sub>1</sub> + *T*<sub>2</sub> for He-HTLC, *T*<sub>1</sub> + 1 + *T*<sub>2</sub> for He⁺-HTLC and Neon-HTLC) and `inf` means that no timeout is safe. File names encode protocol and collateral: `He_HTLC` = He-HTLC, `He_plus` = He⁺-HTLC, `Neon_Tight` / `Neon_Agnostic` = Neon-HTLC with the tight bound (Appendix C.1) / distribution-agnostic bound (Eq. (9)); `col0` = no collateral, `col20` / `col20_flat` and `col50` / `col50_flat` = flat \$20 / \$50 collateral, `col5pct`, `col10pct`, `col20pct` = 5%, 10%, 20% of the deposit.

**Checking a run.** With `real_distribution_69`, the last cell of the notebook compares the new CSV files with the precomputed ones in `data/`; every file should be reported as `identical`.
