# Capstone Report — CTR / Engagement Opportunity Scoring

- **Author:** Ayesha Qamar
- **Lane:** CTR / Engagement Opportunity Scoring
- **Repo:** https://github.com/itsayeshaqamar/flyrank-mlinternship-ayesha
- **Date:** 15 August 2026

> This report documents the completed capstone analysis. The project is framed as an observed, directional decision-support system for prioritizing content pages for human review rather than a causal prediction of future content performance.

## 0. Abstract

This project asks whether observable search-performance and engagement signals can be combined into a reliable scoring framework for prioritizing content pages for review when review capacity is limited. The analysis uses the FlyRank pseudonymized warehouse release `flyrank_pseudonymized_warehouse_release_v20260703`, with daily facts running through June 30, 2026, and aggregates the available performance data to 427,292 page-level observations. The methodology creates data-driven bottleneck indicators for visibility, clicks, sessions, and engagement, defines a proxy priority target as pages with at least two detected bottlenecks, and compares a Random Forest against a Logistic Regression baseline using a client-grouped 80/20 held-out split. On the held-out test set, the Random Forest achieved ROC-AUC 0.8473, Average Precision 0.8423, and Precision@10% 0.9878, compared with 0.7583, 0.6583, and 0.6461 for the baseline. The resulting ranking and priority tiers are intended to help human reviewers focus limited content-review effort on pages showing stronger evidence of observable performance bottlenecks.

## 1. Problem framing

The decision this project supports is:

> **Which content pages should a content team review first when review capacity is limited?**

The unit of analysis is one **content page**, represented internally by an anonymized content identifier and associated with aggregated search-performance and engagement signals.

The output is a **priority probability and ranked review queue**, which is translated into four practical priority tiers:

- **Monitor**
- **Review**
- **High Priority**
- **Urgent Review**

The human action is to use the ranking to decide which pages deserve earlier review, inspect the underlying performance bottlenecks, and then determine whether a content, technical, or editorial intervention is appropriate.

The cost of a wrong call is primarily an allocation-of-effort problem. A false positive can cause an editor to spend limited time reviewing a page that does not require immediate attention. A false negative can cause a page with stronger observable evidence of performance bottlenecks to receive less attention while editorial resources are spent elsewhere.

Data and ML help because content performance is not represented by one universal metric. Visibility, clicks, sessions, and engagement capture different parts of the page-performance journey. A structured scoring system can combine these observable signals and produce a consistent review order instead of requiring a reviewer to manually inspect every page.

The project therefore supports **content-review prioritization**, not automatic refresh decisions and not prediction of Google's ranking algorithm.

## 2. Data safety

The analysis uses the **FlyRank pseudonymized warehouse release**:

`flyrank_pseudonymized_warehouse_release_v20260703`

The notebook documents that the release was exported on July 3, 2026, and that the freshest daily facts in the release run through June 30, 2026.

The analysis uses the daily content-performance table and aggregates the available performance records to one row per client and content page.

The resulting aggregated page-level dataset contains:

- **427,292 page-level observations**
- **13 aggregated fields before derived metrics**

The aggregated performance fields include:

- `impressions`
- `clicks`
- `sum_position`
- `pageviews`
- `sessions`
- `users`
- `engaged_sessions`
- `engagement_sec`
- `organic_sessions`
- `ai_sessions`
- `scroll_events`

The notebook derives:

- `ctr`
- `engagement_rate`
- `avg_position`
- `organic_session_rate`
- `ai_session_rate`

The final bottleneck framework uses four data-driven thresholds based on the positive-only 25th percentile of the corresponding observed distributions:

| Signal | Threshold |
|---|---:|
| Visibility / impressions | 20.000000 |
| CTR | 0.001554 |
| Sessions | 2.000000 |
| Engagement rate | 0.030140 |

The notebook explicitly handles unavailable metrics by filling unavailable GA4 metrics with zero and unavailable average position with zero after determining that no impressions were available.

### Exclusions

The following information is excluded from modeling or public reporting:

- Client names
- Domains
- URLs
- Private search queries
- Credentials
- Raw identifying content information
- Existing business decision flags
- Fields that directly define the proxy priority target

Pseudonymous identifiers such as `client_hash_id` and `content_hash_id` are used for grouping, joining, ranking, and internal data handling but are not used as model features.

### Leakage risks

The priority target is constructed from the following variables:

- `impressions`
- `clicks`
- `ctr`
- `sessions`
- `engagement_rate`
- `bottleneck_score`

These target-defining variables are deliberately excluded from the model feature list.

The final model uses:

- `avg_position`
- `pageviews`
- `users`
- `organic_sessions`
- `ai_sessions`
- `ai_session_rate`
- `scroll_events`

The notebook performs an explicit leakage check between the model features and target-defining variables.

The result was:

`Leakage overlap: set()`

The notebook then asserts that the overlap is empty and reports:

`Leakage check passed.`

Label-derived fields such as `trend_direction` and `trend_pct`, where present in the underlying data, are not used as model features.

The public-facing work is intended to contain no client-identifying information, private queries, credentials, or restricted business information.

## 3. Baseline

The transparent baseline is a **Logistic Regression** classifier.

The baseline uses:

- StandardScaler
- Logistic Regression
- `max_iter=1000`
- `class_weight="balanced"`
- `random_state=42`

The baseline provides a transparent linear comparison against the main Random Forest model. It is a fair comparison because both models use the same seven non-target-defining features and are evaluated on exactly the same client-held-out test set.

The baseline is not intended to represent a production system. It provides a simple reference for measuring whether the more flexible Random Forest captures additional ranking signal.

### Baseline results

On the held-out client test set, the Logistic Regression baseline achieved:

| Metric | Logistic Regression |
|---|---:|
| ROC-AUC | 0.7583 |
| Average Precision | 0.6583 |
| Precision@10% | 0.6461 |

The test-set positive proxy rate was **48.44%**, meaning the majority class represented **51.56%** of the test observations.

This base-rate context is important when interpreting Precision@10%. The baseline Precision@10% of 0.6461 is above the 48.44% positive rate, while the Random Forest reaches 0.9878.

## 4. Model / analysis

The main model is a **Random Forest Classifier** used to rank pages according to their estimated probability of belonging to the defined priority class.

The Random Forest configuration is:

- `n_estimators=200`
- `max_depth=12`
- `min_samples_leaf=20`
- `class_weight="balanced"`
- `random_state=42`
- `n_jobs=-1`

The seven model features are:

1. `avg_position`
2. `pageviews`
3. `users`
4. `organic_sessions`
5. `ai_sessions`
6. `ai_session_rate`
7. `scroll_events`

The following variables are intentionally left out because they directly define the proxy target:

- `impressions`
- `clicks`
- `ctr`
- `sessions`
- `engagement_rate`
- `bottleneck_score`

Client and content identifiers are also excluded from the feature matrix.

### Bottleneck construction

Four interpretable bottleneck indicators are created.

A **visibility bottleneck** is identified when a page has positive impressions but impressions are below the data-driven visibility threshold.

A **click bottleneck** is identified when a page has impressions and CTR is below the data-driven CTR threshold.

A **session diagnostic** is identified when a page has sessions but sessions are below the data-driven session threshold.

An **engagement diagnostic** is identified when a page has sessions but engagement rate is below the data-driven engagement threshold.

The four binary indicators are summed into a `bottleneck_score`.

The observed bottleneck-score distribution is:

| Bottleneck score | Rows | Percentage |
|---:|---:|---:|
| 0 | 150,340 | 35.18% |
| 1 | 125,467 | 29.36% |
| 2 | 110,882 | 25.95% |
| 3 | 28,770 | 6.73% |
| 4 | 11,833 | 2.77% |

### Target / proxy definition

The project defines the priority proxy as:

> **A page is a priority case when its observed bottleneck score is greater than or equal to 2.**

This produces:

| Priority target | Rows |
|---|---:|
| 0 | 275,807 |
| 1 | 151,485 |

The overall positive proxy rate is **35.45%**.

This is a constructed target rather than an observed `refresh_needed` label or post-refresh outcome.

The model therefore learns to discriminate pages matching the defined diagnostic priority pattern; it does not predict whether a page will actually improve after a refresh.

## 5. Evaluation

The evaluation uses a **client-grouped train/test split**.

`GroupShuffleSplit` is used with:

- `n_splits=1`
- `test_size=0.20`
- `random_state=42`

The grouping variable is `client_hash_id`.

The resulting split is:

| Split | Shape | Priority rate |
|---|---:|---:|
| Training | 363,254 × 7 | 33.16% |
| Test | 64,038 × 7 | 48.44% |

The notebook confirms:

**Clients shared between train/test: 0**

This grouping prevents pages belonging to the same client from appearing in both training and testing data, reducing the risk that client-specific patterns are learned during training and then evaluated on the same client.

The test set has a higher positive proxy rate than the training set:

- Training: **33.16%**
- Test: **48.44%**

This distribution shift is an important limitation when interpreting the evaluation.

### Metrics

Both models were evaluated on the same held-out client test set.

| Model | ROC-AUC | Average Precision | Precision@10% |
|---|---:|---:|---:|
| Logistic Regression | 0.7583 | 0.6583 | 0.6461 |
| Random Forest | 0.8473 | 0.8423 | 0.9878 |

The Random Forest improved over the Logistic Regression baseline by:

- **ROC-AUC:** 0.7583 → 0.8473
- **Average Precision:** 0.6583 → 0.8423
- **Precision@10%:** 0.6461 → 0.9878

The largest practical result is Precision@10%. Among the highest-ranked 10% of the held-out test pages, **98.78% were positive priority cases under the constructed proxy target**.

The relevant test-set positive base rate is **48.44%**, while the majority-class rate is **51.56%**.

Therefore, the 98.78% Precision@10% result should be understood as strong concentration of the constructed priority cases at the top of the model's ranking, rather than as evidence of future refresh success.

### Error analysis

The primary error type is contextual mis-prioritization.

Because the target is constructed from observed bottleneck conditions, a page can be classified as a priority case because it satisfies the project's diagnostic definition even when an editor might determine that no content intervention is necessary after considering business, technical, seasonal, or editorial context.

The test-set distribution shift is another important source of uncertainty. The priority rate is 48.44% in the test set compared with 33.16% in training, showing that the held-out clients differ meaningfully from the training clients.

The model also produces tied probabilities for multiple pages. Consequently, small differences between adjacent ranks should not be interpreted as meaningful differences in expected impact.

### What the evaluation does not establish

The evaluation does not establish that:

- Refreshing a prioritized page will improve CTR.
- Refreshing a prioritized page will increase traffic.
- Refreshing a prioritized page will improve rankings.
- The model predicts Google's ranking algorithm.
- The priority score causes business improvement.
- Every high-priority page will benefit from a content refresh.

These claims would require observed post-refresh outcomes, an experiment, or another appropriate causal evaluation design.

## 6. Interpretation

The analysis identified four observed performance bottleneck categories.

| Bottleneck | Pages |
|---|---:|
| Visibility bottleneck | 76,675 |
| Click bottleneck | 197,516 |
| Session diagnostic | 39,192 |
| Engagement diagnostic | 167,490 |

The click bottleneck is the most frequently observed bottleneck under the defined threshold, followed by engagement, visibility, and session diagnostics.

These counts are not mutually exclusive because a page can satisfy multiple bottleneck conditions.

### Bottleneck score interpretation

The bottleneck score counts how many of the four diagnostic conditions are simultaneously present.

The final distribution was:

| Bottleneck score | Rows |
|---:|---:|
| 0 | 150,340 |
| 1 | 125,467 |
| 2 | 110,882 |
| 3 | 28,770 |
| 4 | 11,833 |

The highest score, 4, represents observations meeting all four defined bottleneck conditions.

The priority target considers scores of **2 or higher** as positive.

This resulted in:

- **151,485 priority observations**
- **275,807 non-priority observations**
- **35.45% overall positive proxy rate**

### Model interpretation

The Random Forest outperformed the Logistic Regression baseline across all three reported evaluation metrics.

The observed results were:

- ROC-AUC: **0.8473**
- Average Precision: **0.8423**
- Precision@10%: **0.9878**

Compared with the baseline:

- ROC-AUC increased by **0.0890**
- Average Precision increased by **0.1840**
- Precision@10% increased by **0.3417**

This indicates that the Random Forest provided stronger discrimination and ranking concentration for the constructed priority proxy on the held-out client test set.

The result supports using the Random Forest output as a prioritization mechanism, subject to the limitations of the constructed target and held-out distribution.

### Priority tiers

The Random Forest probabilities were converted into four practical priority tiers using the following probability boundaries:

| Probability range | Priority tier |
|---|---|
| ≤ 0.50 | Monitor |
| > 0.50 to 0.75 | Review |
| > 0.75 to 0.90 | High Priority |
| > 0.90 | Urgent Review |

The resulting test-set distribution is:

| Priority Tier | Pages |
|---|---:|
| Monitor | 48,047 |
| Review | 5,244 |
| High Priority | 1,845 |
| Urgent Review | 8,902 |

These tiers are intended to turn the model's ranking output into an operational review queue.

A higher tier means stronger model-estimated probability of belonging to the constructed priority class. It does not mean that a page is guaranteed to require a refresh.

### Recommendation distribution

The bottleneck score is also translated into an action recommendation:

| Observed condition | Recommended action |
|---|---|
| 3–4 bottlenecks | Comprehensive review |
| 2 bottlenecks | Priority review |
| 1 bottleneck | Targeted review |
| 0 bottlenecks | Monitor |

Within the held-out ranked recommendation set, the resulting action counts are:

| Recommended Action | Pages |
|---|---:|
| Priority review | 25,527 |
| Monitor | 23,388 |
| Targeted review | 9,627 |
| Comprehensive review | 5,496 |

These recommendations are based on observed bottleneck severity and are intended to support human review rather than automatically prescribe an intervention.

### Surprises and negative results

An important finding is that the available data does not provide a directly observed refresh-needed label or a post-refresh outcome.

Therefore, the analysis cannot validate whether the pages identified as high priority would actually produce the largest improvement after a refresh.

The strongest supported result is that the Random Forest can rank pages matching the project's defined bottleneck-based priority proxy effectively on a held-out client test set.

This is a useful decision-support result, but it is not evidence of causal content-performance improvement.

## 7. Recommendation

The ranked actions supported by the final output are:

### 1. Start with Urgent Review pages

Pages in the **Urgent Review** tier should receive the earliest human attention because their model-estimated probability of belonging to the priority class is above 0.90.

The reviewer should inspect the underlying bottlenecks before deciding whether a content change is appropriate.

### 2. Review High Priority pages next

Pages in the **High Priority** tier have model-estimated probabilities above 0.75 and up to 0.90.

These pages should form the next review queue after Urgent Review pages.

### 3. Use the bottleneck type to guide investigation

The observed bottleneck should guide the review:

- **Visibility bottleneck:** review search visibility, discoverability, indexing, and content targeting.
- **Click bottleneck:** review the relationship between search visibility and clicks, including title and metadata presentation.
- **Session diagnostic:** review traffic acquisition and landing-page performance.
- **Engagement diagnostic:** review content usefulness, page experience, and opportunities to improve engagement.
- **Multiple bottlenecks:** prioritize for a broader content and performance review.

### 4. Use the Review tier according to capacity

Pages in the **Review** tier should be handled after higher-priority pages when additional editorial capacity is available.

### 5. Monitor lower-priority pages

Pages in the **Monitor** tier should not automatically trigger a content change.

They can remain in the monitoring queue while review capacity is focused on pages with stronger observed evidence of bottlenecks.

### 6. Keep a human in the loop

The ranking should be treated as **decision-support**.

A FlyRank editor should consider additional context before taking action, including:

- Search demand
- Business relevance
- Seasonality
- Content intent
- Data sufficiency
- Recent changes
- Technical context
- Potential content consolidation
- Editorial judgment

### Confidence and limits

Confidence is **moderate** that the workflow provides a reproducible way to prioritize pages according to the project's defined observable performance bottlenecks.

Confidence is **lower** for predicting future performance improvement because there is no observed post-refresh outcome in the available data.

The strongest defensible conclusion is:

> The Random Forest provides a strong observed ranking signal for identifying pages that match the project's constructed bottleneck-based priority definition on the held-out client test set.

It should not be interpreted as proof that refreshing those pages will improve future performance.

## 8. Reproducibility

The completed capstone notebook is stored under:

`work/notebooks/capstone.ipynb`

The repository is:

`https://github.com/itsayeshaqamar/flyrank-mlinternship-ayesha`

### Environment

The notebook uses Python with:

- NumPy
- pandas
- Matplotlib
- scikit-learn
- datasets
- huggingface_hub
- DuckDB

The notebook installs the dataset-access dependencies directly with:

```bash
pip install datasets huggingface_hub duckdb

## 9. Acknowledgments & Data Credit

Built on the **FlyRank ML Internship dataset** provided for this capstone analysis.

**Data source:** [https://flyrank.ai](https://flyrank.ai)

Crediting the data source is standard research practice and identifies the dataset used for this analysis. The public-facing report does not include client-identifying details, private queries, credentials, or other restricted information.

---

> **Claims checklist before submitting:** observed / measured / directional / decision-support
>
> **Metrics vs. base rate:** The test-set priority-positive rate is **48.44%**, while the majority-class rate is **51.56%**. Precision@10% should therefore be interpreted relative to this base rate rather than as a standalone number. The Random Forest achieved **0.8473 ROC-AUC** and **0.8423 Average Precision**, compared with **0.7583** and **0.6583** for the Logistic Regression baseline.
>
> **Language:** Claims throughout this report use observed, measured, directional, and decision-support framing where appropriate.
>
> **Causal claims:** No causal claims are made because the dataset does not contain a causal experiment or verified post-refresh outcome.
>
> **Google:** This project does not claim to predict Google's algorithm.
>
> **Privacy:** No client-identifying details, private queries, credentials, or restricted information are included in the public-facing report.
>
> **Numbers:** All reported metrics, split sizes, bottleneck counts, target rates, priority tiers, and recommendation counts above are taken from the completed capstone notebook.
