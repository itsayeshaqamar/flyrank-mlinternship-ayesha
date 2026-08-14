# Capstone Report — CTR / Engagement Opportunity Scoring

- **Author:** Ayesha Qamar
- **Lane:** CTR / Engagement Opportunity Scoring
- **Repo:** `itsayeshaqamar/flyrank-mlinternship-ayesha`
- **Date:** 15 August 2026

> This report documents the completed capstone analysis. The project is framed as a decision-support and prioritization system rather than a causal prediction of future content performance.

## 0. Abstract

This project asks how observable content-performance signals can be combined to prioritize which content pages deserve human review first when content teams have limited capacity. The analysis uses page-level content-performance data covering visibility, clicks, sessions, engagement, and related content signals. The methodology combines diagnostic bottleneck indicators into a structured priority score and review framework, treating refresh priority as a proxy because the dataset does not contain an observed ground-truth outcome showing which pages actually required a refresh or improved after one. The analysis identified visibility, click, session, and engagement bottlenecks and produced a practical four-level priority framework ranging from Monitor to Urgent Review. The resulting output is intended as directional decision-support that helps human reviewers focus attention on pages with stronger evidence of performance bottlenecks rather than automatically deciding which pages should be refreshed.

## 1. Problem framing

The decision this project supports is:

**Which content pages should be reviewed first when a content team has limited time and a large number of pages to evaluate?**

The unit of analysis is a **content page** with its associated performance and engagement signals.

The primary output is a **refresh/opportunity priority score**, which is translated into four practical priority tiers:

- Monitor
- Review
- High Priority
- Urgent Review

The human action supported by the output is to start with the highest-priority pages, inspect the underlying bottleneck signals, and then decide whether further investigation or a content refresh is appropriate.

The cost of a wrong call is mainly an allocation-of-effort problem. A false positive can cause an editor to spend time reviewing a page that does not need immediate attention. A false negative can leave a potentially important page unreviewed while limited editorial capacity is spent elsewhere.

Data and ML help because content performance cannot be fully represented by one simple threshold. Visibility, clicks, sessions, engagement, and related performance signals describe different parts of the page-performance journey. Combining these signals provides a more structured way to rank pages for human review.

The project therefore produces **decision-support**, not an automatic refresh decision.

## 2. Data safety

The analysis uses the project-provided content-performance data containing page-level performance, traffic, engagement, and content-related signals.

The main analytical signals include:

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
- CTR and derived performance measures
- Engagement-related measures
- Content and freshness attributes where available

The daily performance data was aggregated to the page level before downstream feature engineering, bottleneck identification, scoring, and prioritization.

### Deliberately excluded fields

The following information is deliberately excluded from predictive features or public outputs:

- Client identifiers
- Content identifiers as predictive features
- URLs
- Domains
- Raw search queries
- Credentials and access tokens
- Private repository paths
- Existing priority scores
- Existing health/action flags
- Other fields that directly encode a business decision

Pseudonymous IDs are used only for grouping or internal data handling and are not used as predictive features.

This prevents identifiers from acting as accidental predictive variables and avoids exposing client- or page-identifying information in the public-facing analysis.

### Leakage risks

Special attention was given to fields that may already contain information derived from the outcome or an existing business decision.

In particular, fields such as:

- `trend_direction`
- `trend_pct`
- Existing priority scores
- Existing health/action flags
- Other label-derived fields

are excluded when they would directly encode the target or proxy outcome.

The refresh/opportunity target is treated as a **proxy**, not as an observed ground-truth label.

The public-facing `work/` content should contain no client-identifying details, private queries, credentials, restricted URLs, or private dataset access paths.

## 3. Baseline

The transparent baseline is a simple rule-based prioritization approach using observable performance weakness.

A basic CTR/performance threshold provides an interpretable starting point because it can be implemented without machine learning and is easy for a reviewer to understand.

The limitation of a flat rule is that it does not fully account for the context in which the performance occurs. For example, the same CTR value can have different meanings depending on search visibility and position.

The final analysis therefore uses multiple performance diagnostics rather than relying on one isolated threshold.

The baseline is a fair comparison because it uses observable signals available to the same analytical workflow and represents a realistic simple alternative to a more structured scoring approach.

The final baseline and model must be evaluated on the same data and metric.

### Baseline results

The final notebook should be treated as the source of truth for the final baseline metrics.

- **Precision@K:** `[Insert final notebook value]`
- **ROC-AUC:** `[Insert final notebook value, if evaluated]`
- **Average Precision:** `[Insert final notebook value, if evaluated]`
- **Base rate:** `[Insert final notebook value]`

Only values from the fresh final capstone run should be reported here.

## 4. Model / analysis

The project is a **scoring and prioritization analysis** designed to rank pages according to the strength of observable performance bottlenecks.

The output is a continuous priority signal rather than a simple automatic "refresh" or "do not refresh" decision.

The workflow can be summarized as:

**Performance signals → bottleneck indicators → bottleneck score → priority score → priority tier → human review**

### Feature groups

The analysis uses observable page-level signals from the following groups:

**Visibility**

- Impressions
- Search exposure
- Search position/context

**Clicks**

- Clicks
- CTR
- Click-related diagnostics

**Sessions**

- Pageviews
- Sessions
- Users
- Organic sessions
- AI-driven sessions where available

**Engagement**

- Engaged sessions
- Engagement time
- Engagement-related measures
- Scroll events
- Scroll-related diagnostics

**Content and freshness**

- Content age
- Freshness/update-related attributes
- Other available page-level content characteristics

### Features intentionally left out

The following are intentionally excluded:

- Client identifiers
- Content identifiers as predictive features
- Private URLs and domains
- Raw queries
- Credentials
- Existing business priority scores
- Existing decision flags
- Label-derived fields such as `trend_direction` and `trend_pct` where they would create leakage

### Target / proxy definition

> The refresh/opportunity proxy identifies pages showing measurable evidence of performance bottlenecks across visibility, clicks, sessions, or engagement, and the resulting signals are combined into a directional priority score for human review.

This is a proxy because the dataset does not contain an observed outcome showing that a page was refreshed and subsequently improved.

The score therefore indicates that a page matches the defined evidence pattern; it does not prove that the page will improve after a refresh.

## 5. Evaluation

The evaluation focuses on whether the prioritization workflow can place pages matching the defined opportunity/bottleneck proxy near the top of the review queue.

The primary ranking-oriented metric is **Precision@K**, because the practical use case assumes that editors can only review a limited number of pages at a time.

### Evaluation split

The final evaluation uses the split documented in the completed capstone notebook.

The evaluation data is kept separate from the observations used to develop the final scoring/modeling approach.

The exact final split should be reported exactly as implemented in the final notebook.

**Final split:** `[Insert exact final notebook split]`

### Metrics

The final notebook should be treated as the source of truth for the final evaluation metrics.

| Metric | Baseline | Final model / scoring approach |
|---|---:|---:|
| Precision@20 | `[Value]` | `[Value]` |
| Precision@50 | `[Value]` | `[Value]` |
| ROC-AUC | `[Value]` | `[Value]` |
| Average Precision | `[Value]` | `[Value]` |

Only metrics actually calculated in the final notebook should be included.

### Base rate

The proxy-label base rate should be reported alongside Precision@K or accuracy.

**Proxy-positive rate:** `[Insert final notebook value]`

This is important because a high Precision@K can be less informative when the positive class is already very common.

The final result should therefore be interpreted relative to both the base rate and the transparent baseline.

### Error analysis

The main potential error is contextual mis-prioritization.

A page may appear weak under one isolated signal while not representing the same review opportunity as another page with stronger visibility or a different engagement profile.

The bottleneck framework reduces this problem by separating different types of weakness instead of treating all under-performance as one condition.

Pages with limited traffic or exposure can also produce unstable rates. Therefore, reviewers should consider data sufficiency and context before acting on a high priority score.

### What the evaluation cannot claim

The evaluation does not establish:

- That refreshing a page will improve CTR.
- That refreshing a page will increase traffic.
- That refreshing a page will improve rankings.
- That the model predicts Google's algorithm.
- That the score causes business improvement.
- That every high-priority page will benefit from a refresh.

Those claims would require observed post-refresh outcomes, experimentation, or an appropriate causal evaluation design.

## 6. Interpretation

The analysis identified four major bottleneck categories.

### Visibility bottleneck

**861,968** observations were identified with a visibility-related bottleneck.

This indicates pages where the available visibility evidence suggests that exposure is an important part of the performance diagnosis.

### Click bottleneck

**3,193,080** observations were identified with a click-related bottleneck.

This indicates pages where the available click evidence suggests that visibility is not translating into clicks as expected under the defined diagnostic rules.

### Session diagnostic bottleneck

**107,115** observations were identified with a session-related diagnostic.

This provides an additional view of page traffic beyond search visibility and clicks.

### Engagement diagnostic bottleneck

**191,330** observations were identified with an engagement-related diagnostic.

This captures cases where clicks or sessions do not necessarily translate into strong downstream engagement.

### Bottleneck score

The individual diagnostics were combined into a bottleneck score.

| Bottleneck score | Rows |
|---:|---:|
| 0 | 6,408,062 |
| 1 | 2,530,950 |
| 2 | 884,555 |
| 3 | 17,811 |

The score can be interpreted as the number of defined bottleneck conditions detected for an observation:

- **0:** No detected bottleneck.
- **1:** One detected bottleneck.
- **2:** Two detected bottlenecks.
- **3:** Three detected bottlenecks.

A higher score indicates stronger concentration of the defined diagnostic signals, but it is not itself proof that a page requires a refresh.

### Priority tiers

The final scoring workflow translated the results into four practical priority tiers.

| Priority Tier | Action | Pages |
|---:|---|---:|
| 0 | Monitor | 48,047 |
| 1 | Review | 5,244 |
| 2 | High Priority | 1,845 |
| 3 | Urgent Review | 8,902 |

These tiers make the output easier to operationalize than a raw numerical score.

### What the analysis found

The analysis shows that content-performance issues are not represented by one universal failure mode.

Visibility, clicks, sessions, and engagement capture different parts of the user journey. Separating these signals allows a reviewer to understand the type of bottleneck associated with a prioritized page rather than receiving only a single unexplained score.

The resulting priority system is therefore best interpreted as a **triage mechanism for human review**.

### Surprises and negative results

An important negative result is the absence of an observed post-refresh outcome.

The analysis cannot directly determine whether the highest-priority pages would actually produce the largest improvement after being refreshed.

The measured result is instead that these pages show stronger evidence under the project's defined bottleneck and prioritization framework.

This distinction is important for keeping the interpretation honest.

## 7. Recommendation

The ranked recommendations are:

### 1. Start with Urgent Review pages

Pages in the **Urgent Review** tier should be reviewed first because they have the strongest evidence under the final prioritization framework.

The reviewer should inspect the underlying bottleneck before deciding on an action.

### 2. Review High Priority pages next

Pages in the **High Priority** tier should form the second review queue.

They have sufficient evidence to justify attention but should still be manually evaluated before any content change.

### 3. Use bottleneck reasons to guide the review

Different bottlenecks should lead to different investigative actions.

- **Visibility:** investigate discoverability and exposure.
- **Click:** inspect search-result presentation and intent alignment.
- **Session:** investigate traffic quality and page relevance.
- **Engagement:** inspect content structure, relevance, readability, and user experience.

### 4. Work through the Review tier according to capacity

The **Review** tier should be addressed after the higher-priority pages, depending on available editorial capacity.

### 5. Monitor lower-priority pages

The **Monitor** tier should not automatically trigger a refresh.

Monitoring allows editorial resources to remain focused on pages with stronger evidence of potential opportunity.

### 6. Keep a human in the loop

The score should be used as **decision-support**, not as an automatic refresh command.

Before taking action, an editor should consider:

- Search demand
- Business relevance
- Seasonality
- Content intent
- Data sufficiency
- Recent changes
- Potential content consolidation
- Other contextual explanations for the observed performance

### Confidence

Confidence is **moderate** that the workflow provides a reproducible way to organize pages according to the defined observable bottleneck signals.

Confidence is **lower** for predicting future improvement because the dataset does not contain verified post-refresh outcomes.

The strongest defensible claim is:

> The scoring framework provides a directional and reproducible way to prioritize human content review using observable performance evidence.

The project does not claim that the recommended pages will necessarily improve after being refreshed.

## 8. Reproducibility

The project is organized in the repository under the `work/` directory, with the completed capstone notebook stored under `work/notebooks/`.

### Repository

`itsayeshaqamar/flyrank-mlinternship-ayesha`

### Notebook

The final capstone notebook is committed under:

`work/notebooks/`

### Re-run commands

From a fresh clone:

```bash
git clone <repository-url>
cd flyrank-mlinternship-ayesha
pip install -r requirements.txt
## 9. Acknowledgments & data credit

Built on the **FlyRank ML Internship dataset** provided for this capstone analysis.

Data source: [https://flyrank.ai](https://flyrank.ai)

Crediting the data source is standard research practice and identifies the dataset used for this analysis. The public-facing report does not include client-identifying details, private queries, credentials, or other restricted information.

---

> **Claims checklist before submitting:** observed / measured / directional / decision-support  
>
> **Metrics vs. base rate:** report the task's base rate (majority-class %) next to any Precision@K or accuracy — a high score can simply reflect a high base rate. AUC / lift over baseline are the more informative discrimination measures.  
>
> **Language:** use careful observed / measured / directional / decision-support language throughout.  
>
> **Causal claims:** make no causal claims without an experiment or appropriate causal design.  
>
> **Google:** do not claim to have predicted Google's algorithm.  
>
> **Privacy:** include no client-identifying details, private queries, credentials, or restricted information.  
>
> **Fresh rerun:** all numbers in this report must match a fresh re-run of the completed notebook.
