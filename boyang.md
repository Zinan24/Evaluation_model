# Results

## Current Run — 10 generations, 2000 games per eval

All models evaluated against a fixed panel of `standard`, `conservative`, `patient` opponents.
Inference: stochastic (`deterministic=False`). Seed: `42`.

---

## Chart

![Results Chart](evaluation_results/results_chart.png)

---

## Summary Table

Structure follows [EVALUATION_METRICS.md](EVALUATION_METRICS.md).
**Bold** = best generation by Skill Share. ★ = primary 3-number summary.

### Recommended Metrics (EVALUATION_METRICS.md)

> ★ = 3-number summary: `(skill_share, VTR, norm_rank_mean)`

| Gen | Skill Share ★ | Valid Team Rate (VTR) ★ | Norm Rank mean ★ | Norm Rank std | Budget Efficiency (CBE) |
| --- | --- | --- | --- | --- | --- |
| GEN_1 | 0.2670 | 80.1% | 0.280 | 0.356 | 0.0721 |
| GEN_2 | 0.2770 | 86.5% | 0.179 | 0.301 | 0.0593 |
| GEN_3 | 0.2762 | 89.2% | 0.165 | 0.288 | 0.0557 |
| GEN_4 | 0.2765 | 91.4% | 0.135 | 0.260 | 0.0452 |
| GEN_5 | 0.2826 | 93.6% | 0.093 | 0.210 | 0.0479 |
| GEN_6 | 0.2794 | 90.8% | 0.118 | 0.248 | 0.0259 |
| GEN_7 | 0.2825 | 94.5% | 0.080 | 0.192 | 0.0444 |
| GEN_8 | 0.2829 | 94.5% | 0.092 | 0.214 | 0.0264 |
| GEN_9 | 0.2806 | 94.5% | 0.077 | 0.191 | 0.0329 |
| **GEN_10** | 0.2836 | 93.8% | 0.074 | 0.193 | 0.0306 |

### Supplementary Metrics

| Gen | Win Rate | Invalid Team Rate | Avg Score | ELO (rules) | H2H win% vs prev |
| --- | --- | --- | --- | --- | --- |
| GEN_1 | 53.1% | 19.9% | 63.7 | 1694 | — |
| GEN_2 | 67.0% | 13.5% | 70.9 | 1743 | 49.0% |
| GEN_3 | 68.8% | 10.8% | 72.5 | 1689 | 25.2% |
| GEN_4 | 72.5% | 8.6% | 74.9 | 1806 | 41.0% |
| GEN_5 | 79.4% | 6.4% | 78.1 | 1884 | 43.4% |
| GEN_6 | 76.6% | 9.2% | 75.4 | 1843 | 37.4% |
| GEN_7 | 81.5% | 5.5% | 79.0 | 1860 | 45.0% |
| GEN_8 | 79.8% | 5.5% | 78.6 | 1834 | 37.2% |
| GEN_9 | 82.2% | 5.5% | 78.9 | 1893 | 54.4% |
| **GEN_10** | 83.7% | 6.2% | 78.7 | 1856 | 45.6% |

### Behavioural Metrics

| Gen | Avg Bid Fraction | Pos fill: GK | Pos fill: DEF | Pos fill: ATT | vs Standard | vs Conservative | vs Patient |
| --- | --- | --- | --- | --- | --- | --- | --- |
| GEN_1 | 0.1470 | 95.8% | 97.8% | 91.2% | 76.3% | 70.3% | 68.2% |
| GEN_2 | 0.1466 | 94.4% | 98.9% | 95.2% | 83.9% | 81.8% | 79.3% |
| GEN_3 | 0.1794 | 98.6% | 98.7% | 95.4% | 86.0% | 83.1% | 80.5% |
| GEN_4 | 0.2168 | 98.0% | 97.5% | 97.2% | 90.5% | 84.4% | 83.4% |
| GEN_5 | 0.1952 | 98.5% | 99.1% | 96.7% | 94.1% | 88.6% | 88.9% |
| GEN_6 | 0.2343 | 96.7% | 97.1% | 97.0% | 91.5% | 86.4% | 86.0% |
| GEN_7 | 0.1903 | 98.5% | 98.2% | 97.5% | 94.7% | 90.9% | 89.8% |
| GEN_8 | 0.2308 | 97.9% | 98.3% | 98.6% | 93.4% | 89.4% | 88.8% |
| GEN_9 | 0.2071 | 98.8% | 97.2% | 96.2% | 94.3% | 91.0% | 90.6% |
| **GEN_10** | 0.1974 | 98.2% | 97.8% | 96.6% | 93.8% | 92.5% | 91.0% |

---

**Best generation: GEN_10**
`skill_share = 0.2836` · `VTR = 93.8%` · `norm_rank = 0.074`

> **Metric roles**
> - **Skill Share** — primary; reward-agnostic, comparable across all experiments
> - **VTR / Norm Rank** — secondary; catch validity and competitiveness regressions
> - **CBE** — diagnostic only; do not optimise directly
> - ELO, H2H, Win Rate — supplementary context; not comparable across different training configs
