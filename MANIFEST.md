# Krauss ML Stat-Arb — Curated Set

Source: https://github.com/ariusmak/krauss-ml-statarb (commit `af8d86c`).

This is a trimmed copy keeping only the **latest, most accurate version** of the project:
the **Datastream + H2O paper-parity pipeline** (the README's closest reproduction of
Krauss, Do & Huck 2017), plus the signal-construction and cost-aware-execution extensions.

**Removed:**
- **Regime-aware part** — `src/krauss/backtest/regime.py` (was imported nowhere) and the
  market-regime analysis artifacts. NOTE: `cost_regime` (baseline vs. no-trade-band) is *kept* —
  that's the cost-aware execution work, not the market-regime analysis.
- **CRSP comparison pipeline** — `build_data.py`, `build_features_labels.py`, `run_phase1.py`,
  `run_phase2.py`, `run_robustness.py`, `build_permno_ticker_map.py`, `build_vw_mkt.py`, and the
  pure-Python `dnn_phase1.py` / `rf_phase1.py` / `xgb_phase1.py` models.
- **Streamlit app** — a precomputed demo tightly coupled to the regime parquet artifacts
  (see "Not included" below).

All kept Python files compile, and no kept file references a dropped one.

---

## Read in this order

### 1. Orientation
1. `README.md` — project overview, findings, research design.
2. `pyproject.toml` — package + dependencies.
3. `docs/reproduction_process.md` — how the reproduction was run.
4. `docs/universe_spec.md` — universe construction rules.
5. `docs/reproduction_deviations.md` — where/why it differs from the paper.

### 2. Core engine — `src/krauss/`
**Data layer** (`src/krauss/data/`)
6. `identifiers.py` — security identifier handling.
7. `wrds_extract.py` — Datastream extraction via WRDS.
8. `universe.py` — monthly no-lookahead S&P 500 membership panel.
9. `prices_returns.py` — RI → returns.
10. `features.py` — lagged-return features `R1..R20, R40..R240` (defines `FEATURE_COLS`).
11. `labels.py` — binary + continuous (`U`) targets.
12. `study_periods.py` — 750/250 walk-forward windows.
13. `features_live.py` — live-feature builder (used by `refresh_live`).

**Models** (`src/krauss/models/`)
14. `h2o_dnn_phase1.py`, `h2o_gbt_phase1.py`, `h2o_rf_phase1.py` — H2O paper-parity models.
15. `ensembles_phase1.py` — ENS1/ENS2/ENS3 over P̂.
16. `rf_extension.py`, `xgb_extension.py` — classifier+regressor pairs (P̂ and Û).
17. `dnn_multitask.py` — shared-trunk multitask DNN (P̂ head + Û head).
18. `ensembles_phase2.py` — ensembles over the extension outputs.

**Backtest** (`src/krauss/backtest/`)
19. `ranking.py` — score → top-k / bottom-k selection.
20. `portfolio.py` — daily long-short portfolio construction.
21. `costs.py` — turnover + 5 bps/half-turn transaction costs.
22. `rebalance.py` — rebalancing logic.
23. `no_trade_band.py` — cost-aware no-trade band (swap only if Û edge clears hurdle).
24. `annualization.py` — annualized metrics across cost regimes.
25. `simulator.py` — backtest driver.

**Evaluation** (`src/krauss/evaluation/`)
26. `metrics.py`, `risk.py`, `diagnostics.py` — performance / risk diagnostics.
27. `ff_factors.py` — Fama-French factor regressions.
28. `phase2_ds_backtest_utils.py` — Datastream phase-2 backtest helpers.
29. `reports.py` — result tables.

**Simulator API & utils**
30. `src/krauss/simulator/api.py` — high-level `run_backtest` / `run_simulation`.
31. `src/krauss/utils/` — `config`, `dates`, `io`, `logging`, `seeds`, `validation`.

### 3. Pipeline scripts — `src/.. run via scripts/` (execution order)
32. `scripts/fetch_external_data.py`, `scripts/fetch_sp500_index.py` — external inputs.
33. `scripts/build_data_datastream.py` — build Datastream price/return panel.
34. `scripts/build_features_labels_datastream.py` — features + labels.
35. `scripts/run_phase1_h2o.py` — H2O reproduction (paper parity).
36. `scripts/run_phase2_datastream.py` — signal-construction extension.
37. `scripts/run_phase3.py` — simulator phase.
38. `scripts/build_predictions_unified.py` → `scripts/build_returns_unified.py` — unify outputs.
39. `scripts/validate_backtest.py` — validation checks.
40. `scripts/test_datastream_h2o.py` — end-to-end smoke test.
41. `scripts/extend_datastream_latest.py`, `scripts/refresh_live.py` — extend/refresh to latest data.

### 4. Notebooks (results, read after running)
- `notebooks/reproduction_results_ds_h2o.ipynb` — the accurate (Datastream+H2O) reproduction.
- `notebooks/u_only_full_backtest.ipynb`, `p_only_full_backtest.ipynb` — single-signal baselines.
- `notebooks/composite_zscore_full_backtest.ipynb`, `composite_gate_full_backtest.ipynb` — best composites.
- `notebooks/cost_band_test.ipynb` — no-trade-band / cost-aware results.

### 5. Tests — `tests/`
`test_universe_no_lookahead`, `test_feature_alignment`, `test_label_alignment`,
`test_backtest_timing`, `test_costs`, `test_reproducibility`.

---

## Not included
- **Streamlit app (`app/`)** — precomputed research demo. Excluded because it directly loads the
  regime artifacts (`app/lib/data.py` reads `regime_labels.parquet`; page 4 "Results & Risk
  Diagnostics" has ~33 regime references and renders regime k-sensitivity / leg-decomposition).
  A regime-free app would need those pages edited, not just files dropped. Ask if you want a
  stripped, runnable version of the app.
- CRSP and pure-Python reproduction paths (see "Removed" above).

## Setup
```bash
cd krauss-curated
pip install -e ".[dev]"   # WRDS access required for data; H2O required for reproduction scripts
```
