# Calibration of Elo-Based Win Probabilities in College Football

This project examines how well Elo rating differences translate into calibrated win probabilities for NCAA football games. The focus is on **probability calibration**—whether events predicted at probability $p$ actually occur approximately $p$ of the time—rather than simply maximizing prediction accuracy.

## Data

Game-level data are pulled from the [CollegeFootballData API](https://collegefootballdata.com/) for the 2023 regular season. Each observation represents a completed game with non-missing pregame Elo values.

**Key fields:**
- `homePregameElo`, `awayPregameElo`
- `homePoints`, `awayPoints`
- `neutralSite` (used to define home-field indicator)

## Problem Setup

**Definitions:**
- **Elo difference:** $\Delta E = E_{\text{home}} - E_{\text{away}}$
- **Outcome (home win indicator):** $Y = \mathbb{1}\{\text{homePoints} > \text{awayPoints}\} \in \{0,1\}$
- **Home-field indicator:** $H = \mathbb{1}\{\text{neutralSite} = \text{False}\}$

**Goal:** Estimate the mapping $(\Delta E, H) \mapsto \mathbb{P}(Y=1 \mid \Delta E, H)$ and evaluate calibration.

## Model

We fit a logistic regression:

$$\mathbb{P}(Y=1 \mid \Delta E, H) = \sigma(\beta_0 + \beta_1 \Delta E + \beta_2 H)$$

where $\sigma(z) = \frac{1}{1 + e^{-z}}$ is the logistic function, and $\hat{p}_i$ denotes the fitted probability.

## Calibration Diagnostic

We define the **residual (probability error):**

$$r_i = Y_i - \hat{p}_i$$

Perfect calibration (conditional on features) implies $\mathbb{E}[r \mid \Delta E, H] = 0$.

**Calibration assessment:** We bin $\Delta E$ into intervals and plot the mean residual within each bin:

$$\bar{r}(b) = \frac{1}{n_b} \sum_{i:\Delta E_i \in b} (Y_i - \hat{p}_i)$$

Bins are filtered to a minimum sample size to avoid noise. A well-calibrated model should have $\bar{r}(b) \approx 0$ across all bins.

## Assumptions

1. **Conditional independence:** Given $(\Delta E, H)$, games are treated as conditionally exchangeable—no explicit team, coach, or matchup interaction effects are modeled.

2. **Correct functional form:** The log-odds of a home win is assumed linear in $\Delta E$ and $H$.

3. **Stationarity:** The Elo-to-probability mapping is assumed stable throughout the season (no early/late season regime changes).

4. **Elo as sufficient statistic:** Pregame Elo captures relevant team strength; unmodeled factors (injuries, weather, rivalry effects) are treated as noise.

5. **No data leakage:** Only pregame information is used for prediction.

## Repository Contents

- **`elo_calibration.ipynb`**: End-to-end pipeline (data retrieval → cleaning → modeling → calibration analysis)
- **`calibration_curve.png`**: Diagnostic plot showing mean residual vs. Elo bin

## Why This Matters

Probability calibration is essential for principled decision-making under uncertainty. This repository demonstrates a complete workflow for:
- Specifying a probabilistic model
- Explicitly stating modeling assumptions
- Validating calibration empirically rather than relying on accuracy metrics alone

This approach is applicable beyond sports prediction to any domain requiring trustworthy probability estimates.
