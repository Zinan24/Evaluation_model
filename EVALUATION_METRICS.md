# Evaluation Metrics

## Motivation

Before running experiments with the V2 improvements, we need a stable set of evaluation metrics that are **agnostic to the variables being experimented with**: number of opponents, auction setup, and reward function. The metrics below satisfy this requirement and allow direct comparison across all experimental runs.

---

## Why existing metrics are insufficient

| Metric | Problem |
|--------|---------|
| ELO | Relative to a pool of trained models — changes as new models are trained, not comparable across runs |
| H2H win rate vs previous generation | Depends on a specific opponent generation; meaningless when comparing different training configurations |
| Episodic reward | Directly contaminated by reward function changes (Changes 7/8) |

---

## Recommended Metric Suite

### 1. Skill Share (primary)

Run the trained model against a **frozen, fixed panel of rule-based opponents** — the same panel for every experiment, never updated.

```
skill_share = agent_team_skill / sum(all_team_skills_in_game)
```

Average over ~2,000 games.

**Properties:**
- In a 4-player game the random baseline is exactly **0.25** — interpretable and stable
- Works for any number of opponents: divide by total skill, not by 4
- Independent of the player pool drawn (skill is relative within each game)
- Completely independent of reward function
- Captures both winning players *and* not overbidding (wasted budget hurts relative skill share)

A well-trained model should consistently sit above 0.28–0.30. Comparing across experiments tells you directly whether a change improved bidding quality.

---

### 2. Valid Team Rate — VTR (secondary)

```
VTR = games with valid team / total games
```

Simple, interpretable, reward-agnostic. Report alongside skill share — an agent could inflate skill share by being aggressive and lucky while sacrificing team validity. Target ≥ 97%.

---

### 3. Normalised Rank (secondary)

```
normalised_rank = (finishing_position - 1) / (n_players - 1)
```

0 = always first, 1 = always last. Summarise as **mean ± std** over many games.

**Properties:**
- Works for any `n_players` — normalising by `(n_players - 1)` makes it comparable across different opponent counts
- Captures whether the agent is consistently competitive
- Complements skill share, which can be distorted by one very high-skill player dominating a game

---

### 4. Conditional Budget Efficiency — CBE (diagnostic only)

```
CBE = mean(remaining_budget / starting_budget | valid_team)
```

**Not a primary performance metric** — an agent that bids £0 on everything has perfect CBE but wins nothing. Use only to diagnose overbidding behaviour, particularly when comparing reward function changes (Changes 7/8). Do not optimise for it directly.

---

## Summary

| Metric | Role | Random baseline | Target |
|--------|------|-----------------|--------|
| Skill share | Primary | 0.25 (4-player) / 1/n | > 0.28 |
| Valid team rate | Secondary | ~85% (untrained) | ≥ 97% |
| Normalised rank | Secondary | 0.5 | < 0.4 |
| Budget efficiency | Diagnostic | — | Monitor only |

The **3-number summary** per experiment — `(skill_share, VTR, mean_normalised_rank)` — is sufficient to rank experiments unambiguously and catch regressions.

---

## Evaluation Protocol

- **Panel**: 3 fixed rule-based opponents (one of each available strategy), never updated between experiments
- **Games**: 2,000 per evaluation run
- **Inference**: stochastic (`deterministic=False`) — tests the policy distribution, not a single point estimate
- **Reproducibility**: fix the random seed for panel evaluation games so results are comparable across runs
