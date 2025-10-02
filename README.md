# BANA-785-JSL-Rochester-Data-Analysis
**JSL Rochester Data Analysis — README**  
## What this project does (at a glance)
1. Builds a governed analytics mart in DuckDB from JSL’s Scheduled, PRN, and QShift streams (clean joins; no PHI—pseudonymous client_id only).
2. Engineers standardized signals (ADL-Eating ordinal, Meal % eaten, Fluid intake ml + route, PRN/Scheduled event counts) at daily → monthly grains with coverage controls.
3. Model 1 (Workload): predicts next-month distinct documentation workload per client; evaluated with rolling, time-ordered CV and latest-month predictions for ops.
4. Plan 1 (Questions usefulness): computes coverage per question and a model-based contribution ranking (permutation R²-drop), plus a per-client Top-N list for the latest month.
5. Plan 2 (ACCUITY): produces a transparent, weighted composite Acuity Index per client-month (higher = worse), with risk bands and latest/trend exports.
6. All reports/charts are written to the Reports/ folder for easy sharing.

## Repository layout (how to run)
1. Database Creation & Transformation.py: Builds jsl.duckdb, dimensions, facts, standardized features, labels (workload), and base views.
2. Modeling.py: Trains/evaluates the workload model (rolling CV) and creates latest-month next-month predictions + feature importance.
3. Report.py: Generates KPI tables/plots; Plan 1 (coverage, contribution, per-client Top-N); Plan 2 (Acuity index & trend).

## 
