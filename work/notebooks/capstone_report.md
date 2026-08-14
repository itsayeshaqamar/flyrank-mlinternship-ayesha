# Capstone Report — CTR / Engagement Opportunity Scoring

- **Author:** Ayesha Qamar
- **Lane:** CTR / Engagement Opportunity Scoring
- **Repo:** `itsayeshaqamar/flyrank-mlinternship-ayesha`
- **Date:** 15 August 2026

> This report documents the completed capstone analysis. The project is framed as a decision-support and prioritization system rather than a causal prediction of future content performance.

## 0. Abstract

This project asks how observable content-performance signals can be combined to prioritize which content pages deserve human review first when content teams have limited capacity. The analysis uses page-level content-performance, visibility, click, session, and engagement signals from the project-provided dataset. The workflow creates bottleneck indicators and combines them into a structured priority score and review tiers, with the analysis evaluated using ranking-oriented and classification-oriented measures where applicable. The results identify distinct visibility, click, session, and engagement bottlenecks and produce practical priority tiers ranging from Monitor to Urgent Review. The final output is intended as directional decision-support for human content review rather than a guarantee that a prioritized page will improve after being refreshed.

## 1. Problem framing

The decision supported by this project is:

**Which content pages should a content editor review first when there are more pages than the team can manually inspect?**

The unit of analysis is a **content page** with its associated performance, engagement, and content attributes.

The primary output is a **refresh/opportunity priority score**, which is translated into practical priority tiers.

The human action supported by the output is to:

1. Start with the highest-priority pages.
2. Inspect the bottleneck or reason associated with the score.
3. Decide whether the page requires further investigation or a potential content action.
4. Monitor lower-priority pages rather than spending immediate editorial capacity on them.

The cost of a wrong call is primarily a resource-allocation problem. A false positive can cause an editor to spend time reviewing a page that does not need immediate attention. A false negative can cause a potentially important page to remain unreviewed while limited editorial capacity is spent elsewhere.

Data and ML help because content performance is influenced by multiple observable signals. Visibility, clicks, sessions, engagement, and related performance indicators can describe different types of weakness, making a single fixed threshold insufficient for a complete prioritization workflow.

The project therefore focuses on **ranking and decision-support**, not automatic refresh decisions.

## 2. Data safety

The analysis uses project-provided content-performance data containing observable page-level performance and engagement information.

The analytical signals include:

- Search impressions
- Search clicks
- Search position
- Pageviews
- Sessions
- Users
- Engaged sessions
- Engagement time
- Organic sessions
- AI-driven sessions
- Scroll events
- CTR and derived performance ratios
- Engagement-related measures
- Content and freshness attributes where available

The daily performance records were aggregated into page-level observations for downstream feature engineering, bottleneck identification, scoring, and prioritization.

### Deliberately excluded information

The following information is not used as predictive features:

- Client identifiers
- Content identifiers as predictive variables
- URLs
- Domains
- Raw search queries
- Credentials or access tokens
- Private repository paths
- Existing business decision fields
- Pre-existing priority or health scores
- Any field that directly reveals the target or proxy outcome

Pseudonymous IDs are treated only as identifiers for grouping or internal data handling. They are not used as predictive features because identifiers have no meaningful predictive interpretation and could introduce memorization or leakage.

### Leakage risks

Special attention was given to fields that could contain information derived from an outcome or an existing business decision.

Fields such as:

- `trend_direction`
- `trend_pct`
- Existing priority scores
- Existing health or action flags
- Any label-derived field

are excluded from ordinary predictive features when they would directly encode the outcome or decision being modeled.

The refresh/opportunity target is treated as a **proxy**, not as an observed ground-truth business label.

The public-facing work is designed to contain no client-identifying information, private queries, credentials, or private data-access paths.

## 3. Baseline

The baseline is a transparent rule-based prioritization approach using observable performance weakness.

A simple baseline uses CTR/performance thresholds to identify pages that may warrant review. This provides an interpretable reference because a reviewer could apply the rule without using machine learning.

However, a flat rule has an important weakness: it does not fully account for differences in search position, visibility, or downstream engagement.

The project therefore moves beyond a single isolated threshold by considering multiple performance and engagement diagnostics.

The baseline is a fair comparison because it uses the same observable evidence available to the analytical workflow and represents a simple alternative that can be implemented without a learned model.

### Baseline evaluation

The final baseline metrics should be reported using the same evaluation data and metric as the final model.

- **Precision@K:** measured on the same evaluation split as the final model.
- **ROC-AUC:** reported where applicable.
- **Average Precision:** reported where applicable.
- **Base rate:** reported alongside ranking metrics to provide context.

The final reported values should match the fresh execution of the completed capstone notebook.

## 4. Model / analysis

The project is fundamentally a **scoring and prioritization problem**.

The goal is not simply to predict a permanent binary state such as "refresh" or "do not refresh." Instead, the workflow produces a continuous priority signal that allows pages to be ordered according to the strength and number of observed performance bottlenecks.

### Target / proxy definition

Because the dataset does not contain an observed field showing that a page actually required a refresh, the project treats refresh priority as a **proxy derived from observable performance evidence**.

In one sentence:

> The proxy identifies pages showing measurable evidence of performance bottlenecks across visibility, clicks, sessions, or engagement, and these signals are combined into a directional priority score for human review.

This distinction is important because the score identifies pages matching the defined evidence pattern; it does not prove that refreshing those pages will improve future performance.

### Feature groups

The analysis uses observable signals from several categories.

#### Visibility signals

- Impressions
- Search exposure
- Search position/context

#### Click signals

- Clicks
- CTR
- Click-related diagnostics

#### Session signals

- Pageviews
- Sessions
- Users
- Organic sessions
- AI-driven sessions where available

#### Engagement signals

- Engaged sessions
- Engagement time
- Engagement-related ratios
- Scroll events
- Scroll-related diagnostics

#### Content and freshness signals

- Content age
- Freshness/update-related attributes
- Other available page-level content characteristics

### Features intentionally excluded

The following are excluded from predictive modeling or scoring when they would create leakage or expose private information:

- Client identifiers
- Content identifiers as predictive features
- Existing priority scores
- Existing business decisions
- Label-derived fields
- Private URLs
- Raw queries
- Credentials
- Other restricted identifiers

### Why this method fits the lane

A scoring approach fits the problem because the practical output is a **ranked review queue**.

A score allows editors to start with the strongest opportunities and work downward according to available capacity. It also allows several performance signals to contribute to one operational priority instead of requiring separate manual threshold checks.

The workflow is therefore:

**Observable performance signals → bottleneck diagnostics → priority score → priority tier → human review**

## 5. Evaluation

The evaluation focuses on whether the resulting ranking can concentrate pages matching the defined opportunity/bottleneck proxy near the top of the review queue.

The primary ranking metric for this type of task is **Precision@K**, because editorial capacity is limited and the most important question is whether the top-ranked pages are useful review candidates.

### Evaluation split

The final evaluation uses the split documented in the completed capstone notebook.

The split is intended to keep evaluation observations separate from the data used to develop the scoring/modeling approach.

The exact final split should be interpreted according to the implementation recorded in the final notebook.

### Metrics

The final notebook should be treated as the source of truth for the final evaluation metrics.

The main metrics are:

- Precision@20
- Precision@50
- ROC-AUC
- Average Precision
- Proxy-label base rate

Precision@K is especially relevant because the operational use case assumes that only a limited number of pages can be reviewed at a time.

### Base rate

The proxy-label base rate should be considered when interpreting Precision@K or accuracy.

A high metric can be misleading when the positive class is already common. Therefore, the final model should be interpreted relative to both the proxy-label base rate and the transparent baseline.

The final notebook's fresh-run metric values should be used here rather than relying on earlier exploratory or starter-pipeline results.

### Error analysis

The main potential error is contextual mis-prioritization.

A page can look weak under one isolated metric while not representing the same review opportunity as another page with stronger visibility or a different engagement profile.

The bottleneck framework addresses this by separating different types of weakness rather than treating every low-performance observation as the same problem.

Pages with very low exposure or limited sessions can also produce unstable rates. For this reason, reviewers should consider data sufficiency and context before acting on a high score.

### What the evaluation does not establish

The evaluation does not establish:

- That refreshing a page will improve CTR.
- That refreshing a page will increase traffic.
- That a page will gain rankings.
- That the score predicts Google's ranking algorithm.
- That the score causes business improvement.
- That a prioritized page will necessarily benefit from a refresh.

Those claims would require observed post-refresh outcomes, an experiment, or another appropriate causal evaluation design.

## 6. Interpretation

The final analysis produced four main bottleneck categories.

### Visibility bottleneck

**861,968** observations were identified with a visibility-related bottleneck.

This represents pages where the available visibility evidence suggests that exposure is an important part of the diagnostic picture.

### Click bottleneck

**3,193,080** observations were identified with a click-related bottleneck.

Click-related weakness is important because visibility alone does not mean that users are selecting the page.

### Session diagnostic bottleneck

**107,115** observations were identified with a session-related diagnostic.

This provides a separate view of whether observed page traffic supports the expected level of attention.

### Engagement diagnostic bottleneck

**191,330** observations were identified with an engagement-related diagnostic.

This matters because a click does not necessarily imply meaningful downstream interaction.

### Combined bottleneck score

The workflow combined the diagnostic indicators into a bottleneck score.

The resulting distribution was:

| Bottleneck score | Rows |
|---:|---:|
| 0 | 6,408,062 |
| 1 | 2,530,950 |
| 2 | 884,555 |
| 3 | 17,811 |

The score provides a simple diagnostic interpretation:

- **0:** No detected bottleneck across the defined diagnostics.
- **1:** One bottleneck signal.
- **2:** Two bottleneck signals.
- **3:** Three bottleneck signals.

A higher score represents a stronger concentration of the defined diagnostic signals. It does not automatically mean that the page must be refreshed.

### Priority tiers

The final workflow translated the scoring evidence into four practical review tiers:

| Priority Tier | Action | Pages |
|---:|---|---:|
| 0 | Monitor | 48,047 |
| 1 | Review | 5,244 |
| 2 | High Priority | 1,845 |
| 3 | Urgent Review | 8,902 |

These tiers make the analytical output easier to use operationally than a raw numerical score alone.

### Interpretation in plain language

The main finding is that content review can be made more structured by separating different performance bottlenecks rather than treating all under-performance as one condition.

A page with a click bottleneck may require a different investigation from a page with an engagement bottleneck. Similarly, a visibility issue should not automatically be interpreted as a content-quality issue.

The priority score is therefore best understood as a **triage mechanism**.

### Surprises and negative results

One important insight is that no single performance signal is sufficient to represent every type of content opportunity.

CTR, visibility, sessions, and engagement describe different parts of the user journey. Combining these diagnostics provides a more actionable view of why a page may deserve review.

The most important negative result is that the analysis cannot directly verify whether the pages identified as high priority would actually improve after a refresh because the dataset does not contain post-refresh outcomes.

This limitation is treated as part of the result rather than hidden from the reader.

## 7. Recommendation

The final output supports the following ranked actions.

### 1. Review Urgent Review pages first

Pages in the **Urgent Review** tier should form the first review queue because they show the strongest evidence under the defined prioritization framework.

The editor should inspect the underlying bottleneck before taking action.

### 2. Review High Priority pages next

The **High Priority** tier should form the second review queue.

These pages have sufficient evidence to justify attention but should still be manually evaluated before a content change is made.

### 3. Use the bottleneck reason to choose the action

The reviewer should not treat every high-priority page as requiring the same intervention.

For example:

- **Visibility bottleneck:** investigate discoverability and exposure.
- **Click bottleneck:** inspect search-result presentation and intent alignment.
- **Session bottleneck:** investigate traffic quality and page relevance.
- **Engagement bottleneck:** inspect content structure, relevance, readability, and user experience.

### 4. Review the Review tier after higher-priority work

The **Review** tier should be handled after the higher-priority pages, depending on available editorial capacity.

### 5. Monitor low-priority pages

The **Monitor** tier should not automatically trigger a content change.

Monitoring allows teams to reserve editorial capacity for pages with stronger evidence of opportunity.

### 6. Keep human validation in the loop

The score should be treated as **decision-support**, not an automatic refresh command.

Before refreshing a page, an editor should consider:

- Search demand
- Business relevance
- Seasonality
- Content intent
- Potential content consolidation
- Data sufficiency
- Recent changes
- Other contextual explanations for the observed performance

### Confidence and limitations

Confidence is **moderate** that the workflow provides a reproducible way to organize pages according to the defined observable bottleneck signals.

Confidence is **lower** for predicting future performance improvement because the dataset does not contain verified post-refresh outcomes.

The strongest defensible claim is:

> The scoring framework provides a directional and reproducible way to prioritize human content review using observable performance evidence.

It should not be interpreted as proof that the recommended pages will improve after being refreshed.

## 8. Reproducibility

The project is organized in the repository under the `work/` directory, with the completed capstone notebook stored under `work/notebooks/`.

### Repository

`itsayeshaqamar/flyrank-mlinternship-ayesha`

### Notebook

The final capstone notebook is committed under:

`work/notebooks/`

### Re-run process

From a fresh clone, install the documented dependencies and run the capstone notebook from top to bottom.

```bash
git clone <repository-url>
cd flyrank-mlinternship-ayesha
pip install -r requirements.txt
