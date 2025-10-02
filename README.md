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

## What’s implemented vs the plan  
1. Plan 1 — “Which questions are useful?”  
    - Step 1 (Identify useful questions): Done.
        - monthly_question_activity, vw_question_coverage, vw_question_coverage_monthly quantify coverage.
        - Permutation importance on coverage Top-K questions ranks predictive contribution to the workload model; results saved as Reports/question_contribution_permutation*.csv.
    - Step 2 (Per-client formatting): Done.
        - vw_client_top_questions_latest and Reports/client_topN_questions_latest.csv list each client’s Top-N recorded, high-contribution questions for the latest month.
2. Plan 2 — “Create an aggregated ACCUITY score”
Acuity Index: Done.
Z-score components (oriented so higher = worse): ADL-Eating (worse↑), Meal % eaten (better↓), Fluid ml (better↓), PRN events (worse↑), Scheduled events (worse↑), Oral-days rate (better↓).
Weighted sum (default weights: ADL .35, Fluid .20, Meal .15, PRN .15, Scheduled .10, Oral .05) → acuity_monthly.
Risk bands by month (vw_acuity_scored), with exports:
Reports/acuity_latest_top25.csv (latest month)
Reports/acuity_trend_last4.csv (last 4 months per client)
Model 1 — Workload (baseline intervention prediction)
Done & evaluated.
Features: monthly ADL, Meal %, Fluid ml, PRN/Scheduled distinct docs, Oral-days; optional lag feature supported.
Rolling, time-aware CV reports R²/MAE; latest-month next-month predictions saved to Reports/top20_predicted_workload_next_month.csv, with feature importance in Reports/figs/workload_feature_importance.png.
