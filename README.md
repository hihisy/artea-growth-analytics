# Artea | E-commerce Growth & Promotion Analytics

SQL and Python analysis of a coupon experiment with 5,000 customers, supported by a local interactive dashboard.

**Decision:** Do not recommend a broad coupon rollout as a proven revenue-growth strategy. Higher purchase activity did not establish higher revenue per customer, and no segment passed the adjusted revenue tests.

**Methods:** SQL aggregation, metric decomposition, A/B test inference, multiple-comparison correction, profitability scenarios, and prospective power analysis.

**Tools:** SQLite, Python, pandas, NumPy, SciPy, and HTML/JavaScript. BigQuery and Tableau are not part of the current implementation.

- [Analysis and recommendations](ANALYSIS_REPORT.md)
- [SQL queries](sql/)
- [Reproducible analysis](scripts/)

## Business decision

Does offering a coupon increase revenue per eligible customer, and which customer segments warrant a follow-up targeting experiment?

This is a reanalysis of an educational case, not an experiment conducted by the analyst or a claim about a live retailer.

## Data and scope

- `Assignment/A3/Artea data.xlsx`, `AB_test`: 5,000 customer records with treatment and outcomes.
- Same workbook, `Next_Campaign`: 6,000 prospective campaign customers; no realized outcomes.
- `week12/Artea C.pdf`, Table 1: acquisition codes and behavioral variable definitions for the related case supplement. Codes: 1 Google, 2 Facebook, 3 Instagram, 4 Referral, 5 Other. Keep numeric codes in exports until the original A/B-case dictionary is checked.
- `ab_test.ipynb`: prior coursework reference, preserved unchanged. Non-significant balance tests do not prove randomization; previous analysis should not be treated as independent confirmation.

No event timestamps, product records, campaign spend, or complete purchase history are present. Do not construct event funnels, retention cohorts, ROAS, or realized profit from these tables. Historical purchase status is not the same as new versus returning website visitors.

## KPI contract

The denominator is all customers assigned to an arm, including non-buyers (intention-to-treat). Keep the outcome window consistent across arms; its exact duration requires the original case definition.

| Metric | Definition | Role |
|---|---|---|
| Recorded revenue per customer | SUM(revenue_after) / COUNT(customers) | Primary decision metric for this reanalysis |
| Purchase rate | COUNT(trans_after > 0) / COUNT(customers) | Secondary |
| Transactions per customer | SUM(trans_after) / COUNT(customers) | Secondary |
| Transactions per purchaser | SUM(trans_after) / COUNT(trans_after > 0) | Descriptive driver |
| Revenue per transaction | SUM(revenue_after) / SUM(trans_after) | Descriptive driver |

Revenue per customer = purchase rate × transactions per purchaser × revenue per transaction.

Conditional metrics among purchasers describe composition as well as behavior. They are not standalone causal effects among the same customers. Coupon discount accounting, exact revenue currency, costs, treatment assignment protocol, and the original outcome-window definition remain to be verified. Do not label recorded revenue as profit or subtract the coupon twice.

## Reproduce

With Python and `requirements.txt` dependencies available, run from this directory:

```sh
python scripts/run_analysis.py
```

For another local source location, pass `--source /path/to/Artea.xlsx`. The runner reads the source, uses an in-memory SQL database, and exports local CSV tables under `outputs/`. It checks missing values, unique IDs, valid flags, SQL quality results, and independent KPI reconciliation.

## Analysis sequence

1. Data quality and KPI definitions: implemented and executable.
2. Overall treatment-minus-control differences and uncertainty: implemented. Revenue uses Welch's test; purchase rate uses a two-proportion test. All tests are two-sided, with 95% confidence intervals. Secondary results are unadjusted exploratory comparisons.
3. Channel, purchase-history, cart, and recency segments: descriptive SQL implemented. Selection after inspecting results is exploratory and needs independent validation.
4. Targeting policy: cart abandonment is an exploratory candidate for a new randomized experiment, not a validated policy. Revenue subgroup tests and omnibus heterogeneity tests are implemented with Holm correction. Previously explored coursework data is not a pristine holdout.
5. Dashboard: local interactive HTML implemented with segment, cost, margin and experiment-population controls. It is not a Tableau deployment. Three insights and three recommendations are included.

## Extended analysis and dashboard

Read `ANALYSIS_REPORT.md` for the decision report, assumptions, source definitions, sample-size plan and limitations. Run:

```sh
python scripts/extend_analysis.py
python scripts/build_dashboard.py
```

Open `outputs/dashboard.html` for the local dashboard. Outputs 06–12 contain balance diagnostics, Shapley revenue decomposition, segment inference, effect-heterogeneity tests, population comparison, profit scenarios and prospective sample-size calculations. SQL file 05 supplies customer-level sufficient statistics for inference. Revenue bootstrap uses 4,000 replicates and seed 20261001.

Runtime used here: local Python environment with pandas, openpyxl, numpy and scipy. No cloud queries or Tableau publication have been performed.

Verification: SQL segment means and confidence intervals were reconciled with raw-row calculations; Shapley contributions reconcile to the overall difference. Dashboard JSON, control references and JavaScript syntax passed static checks. Visual rendering and browser interactions were not verified because browser security policy blocked the local-file preview; no workaround was attempted.

Do not promise optimal targeting or campaign revenue from the current descriptive segment differences. A follow-up randomized experiment is needed to validate the selected policy.

## Initial results

Executed against the local workbook; source files were not modified. All implemented data-quality and reconciliation checks passed.

| Outcome | Control | Coupon | Difference | 95% CI for difference | Two-sided p |
|---|---:|---:|---:|---:|---:|
| Recorded revenue/customer | 7.7802 | 7.5387 | -0.2415 | [-1.5518, 1.0689] | 0.7179 |
| Purchase rate | 10.6886% | 12.3901% | +1.7015 pp | [-0.0693, 3.4724] pp | 0.0597 |
| Transactions/customer | 0.1257 | 0.1519 | +0.0262 | [0.0029, 0.0494] | 0.0273 |

The primary revenue result does not establish either improvement or harm. Purchase rate is higher in the sample, but its two-sided test does not meet a 5% threshold. Transactions per customer has an unadjusted secondary p-value below .05; this is not evidence that the campaign improves revenue. Revenue per transaction is descriptively lower (61.89 versus 49.64), which can reflect discounting and/or buyer composition; the current analysis cannot separate those explanations.

Initial decision: do not recommend blanket rollout as a proven revenue-growth strategy. Investigate exploratory segments, verify revenue accounting, and design a new test of a fixed targeting policy with revenue/customer as its primary endpoint.

## Publication

The workbook contains an explicit reproduction restriction. Original case materials and generated data exports are not bundled for public distribution. `.gitignore` excludes local outputs and data; this is a convenience, not a license grant. No files have been published.
