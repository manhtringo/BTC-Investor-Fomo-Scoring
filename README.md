# BTC-investor-fomo-scoring
Label-free FOMO score (0–100) for BTC investors per market cycle: 15 expert-based behavioural features + a walk-forward HMM, with leakage, sensitivity and synthetic-investor validation.

# Cycle-based FOMO score for BTC investors

Scores FOMO (0–100) at the **investor × time-window** level from simulated BTC trade logs (3,000 investors, 337,732 trades, 2021-01 to 2025-12). There are no labels, so the score is built from expert-described behaviours and validated without ground truth.

**Final score** = 0.5 × feature score + 0.5 × HMM score

- **Feature score:** 15 features, one or more per expert statement, in 6 groups. Each is converted to a walk-forward 0–1 mid-rank, pruned for high Spearman correlation (|ρ| ≥ 0.6), and weighted by group and by (1 − η²) so that features driven by the overall market count less.
- **HMM score:** a 3-state diagonal-Gaussian HMM (missing values allowed) retrained at every cycle, forward-filtered only. The score is the expected FOMO level of the states.
- **Windows:** each window is a market cycle that starts on a rally-onset day (16 cycles, 2021–2025).
- **Levels:** Low / Medium / High from the 50th / 90th percentiles of the score. The data dictionary's suggested 40/70 cut-offs leave High almost empty on this scale (see the appendix, section 12).
- A logistic regression is used only to explain the score, never to compute it.

## Repository layout

| Path | Content |
|---|---|
| `FOMO.ipynb` | Main notebook, executed. Sections 1–8 define the pipeline functions; section 9 runs it (9.1 holds all parameters, one per line) |
| `FOMO_appendix.ipynb` | HMM diagnostics, label-free validation results, limitations, validation code |
| `validate/fomo_pipeline.py`, `validate/params_default.py` | Pipeline functions and default parameters, extracted from the notebook |
| `validate/validate.py`, `validate/synthetic.py` | Label-free validation: parameter and weight sensitivity, synthetic investors, plausibility, edge cases, leakage (about 25 minutes) |
| `data/` | Empty placeholder. Input data is not versioned (see below) |
| `outputs/submit/SAMPLE_SUBMISSION.csv` | Submission file: 3,000 investors, whole period 2021-01-01 to 2025-12-31, same columns as the template |
| `outputs/submit/submission_cycles.csv` | Same schema, one row per investor × cycle |
| `outputs/result/` | Cycle boundaries, whole-period scores, all features and scores per investor × cycle |
| `outputs/model/` | Feature groups, tags, η², weights, correlations, explanatory regression coefficients |
| `outputs/validate/` | Validation report and synthetic-investor results |
| `docs/METHODOLOGY.md` | Assumptions, feature definitions, scoring steps, validation logic |

## Running it

```bash
pip install -r requirements.txt
# put btc_market_ohlcv.csv and btc_trade_log.csv in data/ (input data is not versioned; see data/README.md)
jupyter nbconvert --to notebook --execute --inplace FOMO.ipynb   # about 3 minutes, writes to outputs/
python validate/validate.py                                      # optional, about 25 minutes
```

Change parameters in cell 9.1 of the notebook (or in `validate/params_default.py` for `validate/validate.py`). The pipeline is deterministic: re-running reproduces the files in `outputs/submit/` exactly.

## Principles

- No hidden labels; no future information in any feature.
- Percentiles, correlations, η², weights, the HMM and the level thresholds use only data up to the cycle being scored (walk-forward).
- Leakage test: truncating the data at 2023-06-30 and re-running gives identical scores, confidence, feature scores and HMM scores for the cycles already closed (maximum difference 0.0).
