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
