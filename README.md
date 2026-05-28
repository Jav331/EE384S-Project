# EE384S-Project
Queueing-Theoretic Performance Model for Continuous-Batching LLM Inference

## Baseline SimPy simulator

Run the baseline continuous-batching simulator from the repository root:

```powershell
.\.venv\Scripts\python.exe -m simulator.run_baseline --duration 200 --arrival-rate 2 --max-batch-size 8 --kv-budget 8192
```

The command writes per-request results to `results/baseline.csv` and prints TTFT percentiles, mean TTFT, goodput, and blocking probability.

TTFT percentiles are computed over accepted requests that complete before the simulation ends. Rejected requests and accepted requests still in service at the horizon are excluded from TTFT percentile calculations.

## Baseline sweep

Run the default sweep over arrival rates, batch sizes, and KV budgets:

```powershell
.\.venv\Scripts\python.exe -m experiments.run_sweep
```

The default sweep uses duration `1000`, arrival rates `0.5,1,2,3,4,5,6,8`, max batch sizes `1,2,4,8,16`, KV budgets `2048,4096,8192,16384`, and seeds `0,1,2,3,4`. The command writes seed-averaged summary metrics to `results/sweep_summary.csv`.

Generate report figures from the sweep summary:

```powershell
.\.venv\Scripts\python.exe -m experiments.plot_sweep
```

The plotting command writes PNG figures under `figures/`.

The KV-budget plots hold max batch size fixed at `B=8`. As KV budget increases, the simulator may admit more requests; this can reduce blocking while increasing effective load and tail latency, so p99 TTFT can rise even though fewer requests are rejected.

## Analytical approximation

Run the first closed-form analytical baseline:

```powershell
.\.venv\Scripts\python.exe -m analytical.run_analytical
```

The command writes `results/analytical_summary.csv`. This is a simple M/M/C-style approximation using average request sizes, an effective concurrency cap from `min(B, floor(K / avg_kv_per_request))`, and an exponential-tail TTFT estimate. It is intended as a first comparison baseline, not the final Markov-chain solver.

Compare the analytical approximation against the simulator sweep:

```powershell
.\.venv\Scripts\python.exe -m experiments.compare_simulation_analytical
```

The comparison writes `results/comparison_summary.csv` and prints average relative errors for TTFT, goodput, and blocking probability.

Generate simulation-vs-analytical comparison plots:

```powershell
.\.venv\Scripts\python.exe -m experiments.plot_comparison
```

The plotting command writes parity plots and relative-error bars under `figures/`.
