# Theta Analysis

Causal discovery on ALCF Theta Cobalt job logs (2017 to 2023), with job failure (`EXIT_FAIL`) as the outcome.

## Scripts

| Script | What it does | Output |
|---|---|---|
| `eda_plots.py` | EDA across all years: correlation heatmaps, requested vs. used resources, runtime/walltime distributions, failures over time | `results/eda_plots/` |
| `run_notears.py` | Linear NOTEARS structure learning on the 2018 logs (30k-job sample) including `EXIT_FAIL` | edge list CSV in `results/` |
| `run_fci.py` | FCI (G² test, alpha = 0.01) on a 10k-job slice to check for hidden confounders | `results/fci_logs_2018_q4alpha001_edges.txt` |
| `plot_fci_pag.py` | Reruns FCI and draws the labeled PAG with `EXIT_FAIL` as the sink | PAG images + legend in `results/` |

## Final results

* `results/notears_2018_with_exit_full_sink_dot.png`: NOTEARS DAG with `EXIT_FAIL` forced as a sink (Figure 8 in the report), with edges in `results/notears_edges_2018_exit_thr0.1_sink.csv`. NOTEARS found no strong direct causes of failure, but recovered the resource → job duration → runtime/walltime structure.
* `results/fci_logs_pag_sink_labeled_pruned.png`: FCI PAG showing the association between specified walltime and failure (direction unresolved). Edge marks are explained in `results/fci_logs_pag_legend.txt`.
* `results/eda_plots/`: EDA figures, including the nonlinearity heatmap and the cores requested → used plot used in the report.

## How to run

Put the cleaned Theta CSVs in `data/theta/` (see `../data/README.md`), then run from this folder:

```bash
cd theta_analysis
python eda_plots.py
python run_notears.py --w_threshold 0.1
python run_fci.py
python plot_fci_pag.py
```
