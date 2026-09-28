# Data

The datasets are not included in this repository because of their size and licensing. Download them and place them as shown below.

```
data/
├── mit_supercloud/
│   ├── slurm-log.csv          # raw Slurm scheduler log
│   └── slurm_log_buff.csv     # processed log with derived features (made in eda.ipynb)
└── theta/
    ├── cleaned_joblog_2017..2023 1231.csv
    └── cleaned_joblog_20181231_with_exit.csv
```

* **MIT Supercloud Dataset** (scheduler logs from the TX-Gaia cluster): https://dcc.mit.edu/data
* **ALCF Theta job logs** (`DIM_JOB_COMPOSITE`): https://reports.alcf.anl.gov/data/theta.html

For Theta, exit code 0 is treated as success and any non-zero exit code as failure.
