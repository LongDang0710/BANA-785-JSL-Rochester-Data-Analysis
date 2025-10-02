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
    - Acuity Index: Done.
        - Z-score components (oriented so higher = worse): ADL-Eating (worse↑), Meal % eaten (better↓), Fluid ml (better↓), PRN events (worse↑), Scheduled events (worse↑), Oral-days rate (better↓).
        - Weighted sum (default weights: ADL .35, Fluid .20, Meal .15, PRN .15, Scheduled .10, Oral .05) → acuity_monthly.
        - Risk bands by month (vw_acuity_scored), with exports:
            - Reports/acuity_latest_top25.csv (latest month)
            - Reports/acuity_trend_last4.csv (last 4 months per client)
3. Model 1 — Workload (baseline intervention prediction)
    - Done & evaluated.
        - Features: monthly ADL, Meal %, Fluid ml, PRN/Scheduled distinct docs, Oral-days; optional lag feature supported.
        - Rolling, time-aware CV reports R²/MAE; latest-month next-month predictions saved to Reports/top20_predicted_workload_next_month.csv, with feature importance in Reports/figs/workload_feature_importance.png.

## Why ADL/Fluid change models are not done/finished. 
- I attempted multiple label definitions and robust parsing; at monthly granularity (Nov 2024 → May 2025) I still see:
    - Fluid decline labels: 0 positives under reasonable thresholds—even “any month-to-month drop” yields near-zero positives after coverage gating. Interpretation: fluid totals are often flat/zero across adjacent months for many clients in this slice; the short window + missingness means no learnable change signal monthly.
    - ADL-Eating: monthly adl_eating_m is largely NULL despite numeric/text fallbacks. Interpretation: ADL self-performance entries were either sparse in this window or logged in ways that don’t aggregate cleanly at month level without more coverage.
- Decision: Park ADL/Fluid change models at monthly grain to avoid over-fitting/false conclusions.
- Path forward: try weekly aggregation or extend the time window—both increase time slices and observed movement.

## Data & modeling guardrails (what makes this defensible)  
1. Distinct-doc counting: workload uses COUNT(DISTINCT doc_id) to avoid response-level inflation.
2. Coverage controls: monthly features enforce day coverage; fluids COALESCE to avoid NULL deltas.
3. Parsing hardening: numeric regex for ml/cc; ADL string mapping + numeric fallback; route synonyms (PO/by mouth/PEG/NG/IV).
4. Time-aware eval: rolling CV; no leakage (month t features → month t+1 labels).
5. No PHI: only pseudonymous client_id; outputs are aggregate/risk lists.

## Clarifications requested (professor/teammates) for future-proofing
1. Label definitions (clinical):
    - Fluid decline: confirm threshold (any drop vs ≥10%/≥20%) and minimum oral-days per month; confirm whether non-oral intake should adjust labels.
    - ADL-Eating: confirm the authoritative field(s) and acceptable coverage threshold for aggregating to month/week.
2. Aggregation level:
    - Approve weekly rollups for ADL/Fluid change models if monthly remains too sparse.
3. Question dictionary stability:
    - Confirm that std_question_id mapping is stable across drops or provide a canonical dictionary version for long-term use.
4. Acuity weighting:
    - Keep current weights or switch to data-driven (normalize SHAP means from the workload model).
    - Optionally provide a Top-3 signal variant if a simpler index is desired.
5. Chronic conditions:
    - Approve inclusion (e.g., dementia/diabetes/CHF flags) and source table/fields for a richer feature set.
6. Fairness & slices:
    - Specify any compliance slices (e.g., by age band, gender) for workload performance reporting.
7. Operational deliverables:
    - Confirm the Top-N size for per-client “useful questions” and whether a coverage-rate floor (e.g., ≥20%) should be applied.
    - Approve the risk list cadence (monthly) and artifacts the care team wants (CSV vs dashboard).
8. Access/governance:
    - Confirm where Reports/ artifacts should be shared and any naming/versioning requirements.
9. Future work (optional):
    - LangChain/SQL agent wired to governed views.
    - Deployment target (schedule, storage location, handoff format).

## Note: 
**_I want to acknowledge any mistakes I may have made along the way and sincerely apologize for the delay in submitting this work. I care a lot about the quality and reproducibility of our deliverables, and I’ve tried to make the pipeline transparent, well-documented, and easy to run. If anything is unclear, missing, or could be structured better, I’d truly appreciate your guidance. Please feel free to share any advice or suggestions—on methodology, data handling, evaluation, or how to present the results more effectively to the care team. I’m happy to revise quickly, iterate on the models, add diagnostics, or tailor the outputs to your preferences. Thank you for your time, patience, and direction; I’m committed to improving this work and making it as useful as possible to the project._**
