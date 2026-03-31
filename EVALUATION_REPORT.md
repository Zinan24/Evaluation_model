# Evaluation Report — GEN_8

## Overview

This document records the evaluation metrics for **GEN_8**, evaluated using the updated `evaluate_model.py` (see Changes below). Metrics are designed to be **reward-agnostic** and comparable across experimental runs.

---

## Changes to `evaluate_model.py`

The following metrics and features were added to the existing evaluation script:

| Change | Description |
|--------|-------------|
| **Default games** | Reduced from 10,000 → 5,000 |
| **Bid shading ratio** | `agent_bid / estimated_fair_value` — measures how aggressively the agent discounts bids relative to value |
| **Bid accuracy** | Fraction of bids within a reasonable range of fair value |
| **Overpayment rate** | Fraction of won auctions where agent paid above estimated fair value |
| **Budget efficiency** | `remaining_budget / initial_budget` at end of game |
| **Rolling win rate** | Win rate computed in 100-game windows — tracks stability across the evaluation run |
| **Δ Win Rate** | Full observation win rate minus masked observation win rate — measures how much the agent relies on market information |
| **Panel evaluation** (`--panel`) | Fixed opponent panel (Standard, Sniper, Patient), stochastic inference, seed=42 — reward-agnostic, comparable across all runs |
| **H2H evaluation** (`--h2h`) | Head-to-head tournament between two trained generations |

---

## Why existing metrics are insufficient for cross-experiment comparison

| Metric | Problem |
|--------|---------|
| ELO | Relative to a trained opponent pool — changes as new models are added, not comparable across runs |
| H2H win rate vs previous generation | Tied to a specific opponent generation; meaningless when comparing different training configurations |
| Episodic reward | Directly contaminated by reward function changes |

---

## Recommended Metric Suite

### 1. Skill Share (primary)

Run against a **frozen, fixed panel of rule-based opponents** — the same panel for every experiment, never updated.

```
skill_share = agent_team_skill / sum(all_team_skills_in_game)
```

Averaged over 2,000 games.

**Properties:**
- Random baseline is exactly **0.25** in a 4-player game — interpretable and stable
- Independent of reward function
- Captures both winning players *and* not overbidding (wasted budget hurts relative skill share)

A well-trained model should consistently sit above **0.28–0.30**.

---

### 2. Valid Team Rate — VTR (secondary)

```
VTR = games with valid team / total games
```

Simple, reward-agnostic. An agent could inflate skill share by being aggressive while sacrificing team validity. Target ≥ 97%.

---

### 3. Normalised Rank (secondary)

```
normalised_rank = (finishing_position - 1) / (n_players - 1)
```

0 = always first, 1 = always last. Reported as **mean ± std**.

Works for any `n_players` — normalising by `(n_players - 1)` makes it comparable across different opponent counts.

---

### 4. Conditional Budget Efficiency — CBE (diagnostic only)

```
CBE = mean(remaining_budget / starting_budget | valid_team)
```

**Not a primary performance metric.** Use only to diagnose overbidding. Do not optimise for it directly.

---

## Full Run Results — GEN_1 to GEN_10 (2,000 games per eval, panel evaluation)

All models evaluated against a fixed panel of `standard`, `conservative`, `patient` opponents. Inference: stochastic (`deterministic=False`). Seed: 42.

### Recommended Metrics (primary 3-number summary ★)

★ = 3-number summary: `(skill_share, VTR, norm_rank_mean)`

| Gen | Skill Share ★ | Valid Team Rate (VTR) ★ | Norm Rank mean ★ | Norm Rank std | Budget Efficiency (CBE) |
|-----|--------------|------------------------|-----------------|---------------|------------------------|
| GEN_1 | 0.2670 | 80.1% | 0.280 | 0.356 | 0.0721 |
| GEN_2 | 0.2770 | 86.5% | 0.179 | 0.301 | 0.0593 |
| GEN_3 | 0.2762 | 89.2% | 0.165 | 0.288 | 0.0557 |
| GEN_4 | 0.2765 | 91.4% | 0.135 | 0.260 | 0.0452 |
| GEN_5 | 0.2826 | 93.6% | 0.093 | 0.210 | 0.0479 |
| GEN_6 | 0.2794 | 90.8% | 0.118 | 0.248 | 0.0259 |
| GEN_7 | 0.2825 | 94.5% | 0.080 | 0.192 | 0.0444 |
| GEN_8 | 0.2829 | 94.5% | 0.092 | 0.214 | 0.0264 |
| GEN_9 | 0.2806 | 94.5% | 0.077 | 0.191 | 0.0329 |
| **GEN_10** | **0.2836** | **93.8%** | **0.074** | **0.193** | **0.0306** |

**Best generation: GEN_10** — skill_share = 0.2836 · VTR = 93.8% · norm_rank = 0.074

---

### Supplementary Metrics

| Gen | Win Rate | Invalid Team Rate | Avg Score | ELO (rules) | H2H win% vs prev |
|-----|----------|-------------------|-----------|-------------|------------------|
| GEN_1 | 53.1% | 19.9% | 63.7 | 1694 | — |
| GEN_2 | 67.0% | 13.5% | 70.9 | 1743 | 49.0% |
| GEN_3 | 68.8% | 10.8% | 72.5 | 1689 | 25.2% |
| GEN_4 | 72.5% | 8.6% | 74.9 | 1806 | 41.0% |
| GEN_5 | 79.4% | 6.4% | 78.1 | 1884 | 43.4% |
| GEN_6 | 76.6% | 9.2% | 75.4 | 1843 | 37.4% |
| GEN_7 | 81.5% | 5.5% | 79.0 | 1860 | 45.0% |
| GEN_8 | 79.8% | 5.5% | 78.6 | 1834 | 37.2% |
| GEN_9 | 82.2% | 5.5% | 78.9 | 1893 | 54.4% |
| **GEN_10** | **83.7%** | 6.2% | 78.7 | 1856 | 45.6% |

---

### Behavioural Metrics

| Gen | Avg Bid Fraction | Pos fill: GK | Pos fill: DEF | Pos fill: ATT | vs Standard | vs Conservative | vs Patient |
|-----|-----------------|--------------|---------------|---------------|-------------|-----------------|------------|
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

## GEN_8 Results (5,000 games)

### Core Tournament Metrics

| Metric | Value | Notes |
|--------|-------|-------|
| Win rate | **86.6%** | |
| Loss rate | 13.0% | |
| Draw rate | 0.4% | |
| Invalid team rate | 5.1% | Target: ≤ 5% |
| Average team score | 79.4 | |
| Final ELO | **1977** | Starting ELO: 1500 |

### Bidding Behaviour Metrics

| Metric | Value | Interpretation |
|--------|-------|----------------|
| Bid shading ratio | **0.949** | Agent bids ~95% of estimated value — close to 1.0, slight underbid as expected in first-price auctions |
| Bid accuracy | 64.8% | |
| Overpayment rate | 66.5% | High — agent frequently wins at above fair value |
| Budget efficiency | 92.5% | Retains ~7.5% of budget on average |

### Win Rate by Opponent Strategy

| Opponent | Win rate |
|----------|----------|
| Standard | 94.5% |
| Aggressive | 98.2% |
| Sniper | 94.8% |
| Patient | 90.8% |
| Conservative | 90.5% |

> Hardest opponents: Patient and Conservative — both preserve budget well and bid efficiently.

---

## Panel Evaluation Summary

| Metric | Role | Random baseline | Target | Status |
|--------|------|-----------------|--------|--------|
| Skill share | Primary | 0.25 (4-player) | > 0.28 | *Run `--panel` to populate* |
| Valid team rate | Secondary | ~85% (untrained) | ≥ 97% | *Run `--panel` to populate* |
| Normalised rank | Secondary | 0.5 | < 0.4 | *Run `--panel` to populate* |
| Budget efficiency | Diagnostic | — | Monitor only | *Run `--panel` to populate* |

To generate panel results:
```bash
python evaluate_model.py --model GEN_8 --panel
```

---

## Head-to-Head: GEN_8 vs GEN_1 (1,000 games)

| | Win rate | Avg team score |
|-|----------|----------------|
| **GEN_8** | 50.2% | **77.5** |
| GEN_1 | 49.8% | 15.4 |

> Win rates are near-equal — both agents compete against 3 shared rule-based opponents so win counts are similar. However, GEN_8's average team score is **5× higher**, reflecting substantially better player selection quality over 10 generations of self-play.

---

## Evaluation Protocol

- **Panel**: Standard, Sniper, Patient (fixed, never updated)
- **Games**: 5,000 (tournament) / 2,000 (panel) / 1,000 (H2H)
- **Inference**: stochastic (`deterministic=False`) for panel; deterministic for tournament
- **Reproducibility**: panel evaluation uses fixed seed = 42
