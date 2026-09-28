# Causal Inference for HPC Reliability

**Moving from correlation to causation in supercomputer job failures.** This project applies causal discovery and interventional analysis to real scheduler logs from two HPC systems, MIT Supercloud and ALCF Theta, to ask whether resource choices actually *cause* jobs to fail.

Final project for **CS 520: Causal Inference and Learning**, University of Illinois Chicago (Fall 2025).
📄 [Read the full report](report/CS520_Final_Report.pdf)

---

## The problem

Earlier studies of HPC logs found that bigger jobs (more nodes, cores, memory) fail more often, but those findings are correlational. We wanted to answer counterfactual questions such as: *would this job's failure probability change if it had requested fewer resources?*

This is hard because scheduler logs mix discrete and continuous variables, are heavily skewed, and have non-linear relationships, which break the assumptions of many standard causal discovery methods.

## Approach

```
EDA ──► Dual preprocessing ──► Causal discovery ──► Graph falsification ──► Effect estimation
        (discrete vs.          (PC, FCI, GES,       (permutation-based     (DoWhy GCM,
         continuous)            NOTEARS)             test, DoWhy)           do-interventions)
```

1. **Exploratory analysis.** Spearman vs. Pearson heatmaps and Q-Q plots showed monotonic but non-linear, heavy-tailed, non-Gaussian data.
2. **Dual preprocessing.** Every experiment ran on a *discrete* version (log transform, then binning) and a *continuous* version (standardized), to measure how representation affects the learned graph.
3. **Causal discovery.**
   * MIT Supercloud: PC, FCI and GES (constraint-based and score-based).
   * ALCF Theta: NOTEARS (optimization-based) plus FCI on a 10k-job slice to check for hidden confounders.
   * PNL (post-nonlinear model) for pairwise edge direction.
4. **Validation.** A permutation-based graph falsification test (Eulig et al., AAAI 2025) checked whether the learned DAG was consistent with the data.
5. **Effect estimation.** Fitted a graphical causal model with DoWhy and estimated average causal effects of `do(resource = level)` on job status.

## Key results

### MIT Supercloud: final causal graph (GES, discretized)

<img src="assets/mit_final_ges_dag.png" width="480" alt="Final GES DAG for MIT Supercloud">

* Baseline runs showed **no clear causal parents of job status**. Switching from G² / Fisher-Z tests to the **Pillai trace CI test with BIC-CG scoring**, forcing `status` as a sink node, and using **aggressive binning (at most 4 bins)** produced a stable graph with meaningful edges into `status`.
* The falsification test flagged `partition` as problematic. After removing it, the revised graph passed: **0 of 20 permutations fell in its Markov equivalence class, and it beat 100% of permuted DAGs.**

<img src="assets/falsification_test.png" width="620" alt="Graph falsification test">

**Average causal effects on job failure** (`status = 1` means failure):

| Intervention on | Large vs. Small | Medium vs. Small |
|---|---|---|
| GPU allocation | +0.0042 | +0.0003 |
| Runtime | -0.0026 | -0.0008 |
| CPU allocation | +0.0023 | +0.0042 |
| Memory allocation | -0.0035 | +0.0001 |

The effects are small, but the direction is consistent: more GPUs and CPUs slightly *raise* failure risk, while more memory (fewer out-of-memory crashes) and longer runtimes (fewer short debugging runs) slightly *lower* it.

### ALCF Theta (2017 to 2023)

<img src="assets/theta_notears_dag.png" width="620" alt="NOTEARS DAG for ALCF Theta">

* NOTEARS recovered an intuitive timing structure (requested/used resources → job duration → runtime and walltime) but found no strong direct causes of failure.
* FCI found a clear link between **user-specified walltime and job failure**, with the direction unresolved, pointing to possible hidden confounding.

## What we learned

* **Representation matters as much as the algorithm.** The same data gave very different graphs depending on binning and the choice of CI test.
* **Validate graphs without ground truth.** Falsification testing caught a bad variable that would otherwise have distorted effect estimates.
* **Causal graphs don't transfer across systems.** MIT and Theta log different features (Theta has no GPU fields), so each system needs its own graph.

## Repository structure

```
├── mit_supercloud/
│   ├── eda.ipynb              # EDA: distributions, linearity, Q-Q plots
│   ├── discover.ipynb         # PC / FCI / GES experiments
│   ├── discover_script.py     # CLI version of the discovery runs
│   ├── test_dags.ipynb        # Graph falsification + DoWhy ACE estimation
│   ├── pnl_run.py             # Pairwise direction tests with PNL
│   ├── *.gml                  # Learned graphs used downstream
│   └── figures/               # All learned DAGs
├── theta_analysis/            # Theta EDA, NOTEARS and FCI scripts + results
├── assets/                    # Figures used in this README
├── data/                      # Where to put the datasets (not included)
├── report/                    # Final report (PDF)
└── requirements.txt
```

## Running it

```bash
pip install -r requirements.txt
```

1. Download the datasets as described in [`data/README.md`](data/README.md).
2. MIT Supercloud: run the notebooks in `mit_supercloud/` in order (`eda` → `discover` → `test_dags`), or use the script:
   ```bash
   cd mit_supercloud
   python discover_script.py dis GES 0.01   # discrete data, GES
   python discover_script.py mixed PC 0.01  # mixed data, PC
   ```
3. Theta: see [`theta_analysis/README.md`](theta_analysis/README.md).

## Tech stack

Python · causal-learn · DoWhy (GCM) · CausalNex (NOTEARS) · pgmpy · pandas · scikit-learn · NetworkX · Graphviz

## Team

Group project by **Armaan Ashfaque, Smita Darmora and Sarah Zahir Syeda**.

**My contributions:** _[Fill in, e.g. "Led the ALCF Theta analysis: data cleaning, NOTEARS and FCI experiments, and EDA."]_

## References

* Samsi et al., *The MIT Supercloud Dataset*, IEEE HPEC 2021.
* Eulig et al., *Toward Falsifying Causal Graphs Using a Permutation-Based Test*, AAAI 2025.
* Zheng et al., *DAGs with NO TEARS*, NeurIPS 2018.
* Sharma and Kiciman, *DoWhy: An End-to-End Library for Causal Inference*, 2020.

Full references are in the [report](report/CS520_Final_Report.pdf).
