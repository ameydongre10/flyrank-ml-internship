# FL-01 Prompt Engineering Iteration Log: Foundations & Signal Audit

**Track:** Applied Search Intelligence & Machine Learning (FlyRank ML Internship)  
**Phase:** Foundations (FL-01 Target Task Audit)  
**Estimated Hours:** 6  
**Author:** Amey Dongre (`ameydongre10`)  
**Repository Location:** [`work/FL01_PROMPT_ITERATION_LOG.md`](file:///C:/Users/ameyd/.gemini/antigravity/scratch/flyrank-ml-internship/work/FL01_PROMPT_ITERATION_LOG.md)  
**Public GitHub Link:** `https://github.com/ameydongre10/flyrank-ml-internship/blob/main/work/FL01_PROMPT_ITERATION_LOG.md`

---

## 1. Target Task Overview (FL-01 Audit)

### Task Definition
In enterprise search intelligence (Lane 2: Content Refresh & Decay Opportunity Scoring), raw search telemetry (impressions, clicks, average SERP position, CTR, update staleness) is prone to extreme statistical distortion:
1. **Severe Heavy-Tail Distributions:** Organic search traffic follows a power-law Pareto curve where a tiny fraction of URLs capture millions of impressions while the long tail has single digits. Standard linear estimators (Pearson correlation, unscaled regressors) are broken by high-leverage outliers.
2. **Non-Linear Position Cliffs:** Moving from SERP position 2.0 to 4.0 cuts organic clicks by $>60\%$, whereas moving from position 42.0 to 44.0 has negligible impact.
3. **Catastrophic Feature Leakage:** Upstream data pipelines frequently blend post-decay engagement metrics with pre-decay observation windows, giving models artificially inflated offline benchmark scores that collapse in production.

**The Target Task:** Engineer a production-grade diagnostic prompt that audits a candidate feature set, identifies distribution anomalies and temporal leakage, and outputs standardized signal verdicts (`CONFIRMED`, `OPPOSITE`, `MIXED`, or `REJECTED/LEAKAGE`) with explicit feature transformation formulas.

---

## 2. Standard Evaluation Dataset (Controlled Reference)

To ensure every prompt iteration was evaluated under strict experimental controls, all prompt versions were tested against the identical candidate feature summary table extracted from the FlyRank search telemetry dataset:

| Feature Name | Type | Mean | Median | Min | Max | Skew / Kurtosis Profile | Raw Pearson ($r$) with Decay Label | Spearman Rank ($\rho$) with Decay Label | Feature Definition / Observation Window |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `impressions_90d` | Float | 14,280 | 1,820 | 12 | 1,420,000 | Extreme Right-Tail (Kurtosis > 18.4) | -0.04 | +0.28 | Organic search impressions in 90-day pre-period |
| `clicks_90d` | Float | 142 | 14 | 0 | 18,900 | Extreme Right-Tail (Kurtosis > 22.1) | -0.02 | +0.19 | Organic clicks in 90-day pre-period |
| `avg_position` | Float | 14.8 | 9.4 | 1.1 | 88.5 | Moderate Right-Skew | +0.12 | +0.34 | Mean organic ranking position in pre-period |
| `ctr` | Float (%) | 0.82% | 0.44% | 0.00% | 14.2% | Zero-Inflated / Right-Skew | -0.08 | -0.22 | Click-through rate in pre-period |
| `days_since_last_update` | Integer | 118 | 84 | 4 | 720 | Bimodal Skew | +0.03 | +0.18 | Days elapsed since content was last modified |
| `sessions_post_decay` | Float | 48 | 8 | 0 | 4,200 | Extreme Right-Tail | -0.42 | -0.61 | On-site sessions recorded in the 30 days *after* decay event |

---

## 3. Prompt Iteration Ladder (6 Versions)

---

### Run 0: Naive Version (The Baseline)

#### The Prompt:
```text
Audit these search signals and tell me if they are good for predicting content decay.

[Input Candidate Feature Table]
```

#### Output (Representative Excerpt):
> "Based on the table, here is an evaluation of the search signals:
> - `sessions_post_decay` is clearly the best and most predictive feature because it has the strongest correlation with the decay label (-0.42 Pearson, -0.61 Spearman). You should definitely include this in your model.
> - `avg_position` is also a useful feature with a positive correlation of +0.12.
> - `impressions_90d` and `clicks_90d` have virtually zero correlation (-0.04 and -0.02) with content decay, so they are not very informative and you can probably drop them.
> - `days_since_last_update` has a very low correlation (+0.03), meaning content freshness doesn't matter much.
> 
> General Advice: You should split your data 80/20 into training and testing sets, normalize your features using standard scaling, and train a Random Forest or Logistic Regression model."

#### Notes on Run 0:
1. **What changed in the prompt:** Baseline zero-shot naive prompt with no assigned role, no domain context, and no constraints.
2. **Observed output difference:** The model completely failed the foundational audit. It enthusiastically recommended `sessions_post_decay` (a fatal temporal leakage that would ruin the model in production), blindly dismissed `impressions_90d` based on raw Pearson correlation while ignoring heavy-tail skew, and closed with textbook machine learning platitudes.

---

### Run 1: Version 1 (+ Role Assignment)

#### Named Technique:
**Role Assignment**

#### The Prompt:
```text
You are a Principal Machine Learning Scientist and Search Intelligence Auditor at FlyRank, specializing in feature store engineering, non-parametric statistical evaluation, and data leakage prevention in ranking systems.

Audit these search signals and tell me if they are good for predicting content decay.

[Input Candidate Feature Table]
```

#### Output (Representative Excerpt):
> "As a Search ML Auditor, here is my technical assessment of the feature store signals:
> 
> 1. Target Leakage Warning on `sessions_post_decay`:
> Although `sessions_post_decay` shows the highest nominal correlation (-0.42 Pearson / -0.61 Spearman), its definition specifies it is measured *after* the decay event. This is a severe temporal feature leakage. Training on this feature will cause artificial offline over-optimism and complete operational failure. It must be disqualified immediately.
> 
> 2. Metric Evaluation:
> - `avg_position` shows legitimate monotonic signal (Spearman +0.34).
> - `impressions_90d` and `clicks_90d` exhibit near-zero Pearson values (-0.04 / -0.02) but positive Spearman rank correlations (+0.28 / +0.19).
> - `days_since_last_update` shows weak linear correlation (+0.03).
> 
> Recommendation: Remove leaked variables, review feature distributions, and apply appropriate transformations before model training."

#### Notes on Run 1:
1. **What changed in the prompt:** Added an explicit **Role Assignment** establishing a Principal ML Scientist persona focused on leakage prevention and non-parametric statistics.
2. **Observed output difference:** The persona immediately triggered a critical safety behavior: the model caught the temporal leakage in `sessions_post_decay` and warned against using it. The authoritative tone eliminated junior marketing fluff, although the analysis of the remaining features remained unstructured and lacked concrete transformation steps.

---

### Run 2: Version 2 (+ Context and Motivation)

#### Named Technique:
**Context and Motivation**

#### The Prompt:
```text
You are a Principal Machine Learning Scientist and Search Intelligence Auditor at FlyRank, specializing in feature store engineering, non-parametric statistical evaluation, and data leakage prevention in ranking systems.

Context & Business Motivation:
We are developing an automated priority scoring model for enterprise content refreshes (Lane 2). Editorial capacity is strictly limited: recommending a healthy page or a dead page with zero demand wastes significant human writing effort. 
Search traffic metrics are heavily Pareto-distributed (extreme right tails with massive kurtosis), meaning standard linear Pearson correlation is dominated by outliers and routinely misleads engineers. Furthermore, any temporal leakage between pre-decay observations and post-decay outcomes will completely invalidate our offline-to-online precision.

Audit these search signals and determine their genuine predictive viability for content decay scoring.

[Input Candidate Feature Table]
```

#### Output (Representative Excerpt):
> "Technical Audit under Enterprise Search Operational Constraints:
> 
> **Heavy-Tail Discrepancy & Pearson vs. Spearman Divergence:**
> In enterprise search, `impressions_90d` (Mean: 14,280 vs Median: 1,820, Max: 1.42M, Kurtosis > 18) and `clicks_90d` are extreme power-law distributions. A Pearson $r$ of -0.04 does NOT mean zero signal; rather, a handful of multi-million impression outliers are dominating the covariance calculation. The Spearman rank correlation of **+0.28** reveals a strong, monotonic relationship: higher-demand URLs experience more measurable decay trajectories.
> 
> **Operational Decision Risk:**
> - `sessions_post_decay`: Disqualified. Temporal boundary violation.
> - `impressions_90d`: Confirmed viable once log-transformed (`log1p`). It provides the necessary volume floor to ensure editorial capacity is not wasted on low-volume zombie pages.
> - `avg_position`: Confirmed viable. Position degradation reflects loss of SERP visibility."

#### Notes on Run 2:
1. **What changed in the prompt:** Added **Context and Motivation** explaining the business cost of false recommendations, the mathematical reality of Pareto power laws in search data, and the distinction between linear vs rank correlation.
2. **Observed output difference:** The model stopped treating low Pearson values as evidence of absence; it correctly diagnosed the mathematical mechanism (outlier leverage on Pearson covariance) and explained why high-demand pages need logarithmic transformation to preserve editorial prioritization.

---

### Run 3: Version 3 (+ Few-Shot Examples)

#### Named Technique:
**Few-Shot Examples**

#### The Prompt:
```text
You are a Principal Machine Learning Scientist and Search Intelligence Auditor at FlyRank, specializing in feature store engineering, non-parametric statistical evaluation, and data leakage prevention in ranking systems.

Context & Business Motivation:
[Context as in Version 2]

Here are examples of how signal audit decisions must be evaluated and classified:

[EXAMPLE 1]
Feature: search_volume_30d
Observed Stats: Mean 50k, Median 2k, Kurtosis 24.2. Pearson r = -0.01, Spearman rho = +0.31. Measured in pre-period.
Verdict: CONFIRMED (NON-LINEAR)
Reasoning: Raw Pearson is suppressed to near-zero by high-volume tail outliers. Spearman rho (+0.31) demonstrates monotonic predictive power. 
Required Action: Apply log1p transformation before feeding to tree or linear models.

[EXAMPLE 2]
Feature: bounce_rate_during_churn
Observed Stats: Pearson r = +0.58. Measured during or after the user departure event.
Verdict: REJECTED (TEMPORAL LEAKAGE)
Reasoning: Target leakage. Variable is observed concurrently with or after the target label condition occurs.
Required Action: Drop completely from feature store; cannot be available at inference time.

Now, audit the following candidate signals using this exact analytical standard:

[Input Candidate Feature Table]
```

#### Output (Representative Excerpt):
> "Signal Audit Evaluations:
> 
> **1. Feature: `impressions_90d`**
> - Verdict: **CONFIRMED (NON-LINEAR)**
> - Reasoning: Severe right-tail skew (Mean 14.2k vs Median 1.8k; Kurtosis > 18.4) suppresses Pearson $r$ to -0.04. Spearman $\rho$ of +0.28 confirms monotonic predictive signal. Pre-period observation is leakage-free.
> - Required Action: Apply `log1p(impressions_90d)` transformation.
> 
> **2. Feature: `sessions_post_decay`**
> - Verdict: **REJECTED (TEMPORAL LEAKAGE)**
> - Reasoning: High correlation (-0.42 / -0.61) is entirely artifactual. The feature captures on-site sessions after the decay event has already transpired.
> - Required Action: Drop immediately from training dataset.
> 
> **3. Feature: `avg_position`**
> - Verdict: **CONFIRMED (MONOTONIC)**
> - Reasoning: Moderate skew; Spearman $\rho$ of +0.34 indicates higher numerical rank (poorer SERP placement) aligns with decay.
> - Required Action: Model with non-linear binning or decision splits (positions 1-3, 4-10, 11-20).
> 
> **4. Feature: `ctr`**
> - Verdict: **CONFIRMED (INVERSE)**
> - Reasoning: Zero-inflated; Spearman $\rho$ of -0.22 indicates declining CTR precedes formal traffic drop.
> - Required Action: Apply logit or quantile scaling."

#### Notes on Run 3:
1. **What changed in the prompt:** Injected high-standard **Few-Shot Examples** illustrating how to classify signal verdicts and articulate explicit engineering actions.
2. **Observed output difference:** The model mirrored the exact structured reasoning of the examples. Every single candidate feature received an explicit classification verdict (`CONFIRMED`, `REJECTED`), a mathematically grounded explanation, and a production engineering recommendation.

---

### Run 4: Version 4 (+ Output Structure)

#### Named Technique:
**Output Structure**

#### The Prompt:
```text
You are a Principal Machine Learning Scientist and Search Intelligence Auditor at FlyRank, specializing in feature store engineering, non-parametric statistical evaluation, and data leakage prevention in ranking systems.

Context & Business Motivation:
[Context as in Version 2]

Examples:
[Few-Shot Examples as in Version 3]

Output Structure Requirements:
You must structure your response strictly inside the following XML tags:
<executive_summary>
One paragraph summarizing total features audited, count passed, count rejected, and key risk findings.
</executive_summary>

<feature_audit_table>
A Markdown table with exactly these 6 columns:
| Feature Name | Distribution Risk | Observed Signal Verdict | Pre/Post Validity | Production Transformation | Action / Decision |
</feature_audit_table>

<remediation_protocol>
Bulleted list of exact code/pipeline fixes for the data engineering team.
</remediation_protocol>

Do not include any conversational filler outside these tags.

[Input Candidate Feature Table]
```

#### Output:
```xml
<executive_summary>
Audited 6 candidate search performance features for the Lane 2 Content Refresh model. Four features (impressions_90d, avg_position, ctr, days_since_last_update) are CONFIRMED as statistically valid predictors when coupled with appropriate non-linear transforms. One feature (clicks_90d) is classified as REDUNDANT/COLLINEAR with impressions. One feature (sessions_post_decay) is REJECTED due to fatal temporal feature leakage.
</executive_summary>

<feature_audit_table>
| Feature Name | Distribution Risk | Observed Signal Verdict | Pre/Post Validity | Production Transformation | Action / Decision |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `impressions_90d` | Extreme Right-Tail (Kurtosis > 18) | CONFIRMED (Non-Linear) | Valid (Pre-period) | `log1p(x)` | Retain as primary demand baseline |
| `clicks_90d` | Extreme Right-Tail (Kurtosis > 22) | MIXED (Collinear Risk) | Valid (Pre-period) | `log1p(x)` or Interaction ratio | Use primarily to compute empirical CTR |
| `avg_position` | Moderate Skew | CONFIRMED (Monotonic) | Valid (Pre-period) | Position-tier bucketing (1-3, 4-10, 11-20) | Retain as primary rank visibility metric |
| `ctr` | Zero-Inflated / Skewed | CONFIRMED (Inverse) | Valid (Pre-period) | Quantile transformation / Box-Cox | Retain as early-warning CTR decay signal |
| `days_since_last_update` | Bimodal Distribution | CONFIRMED (Lagging Risk) | Valid (Pre-period) | Min-Max clip at 365d | Retain as content freshness risk score |
| `sessions_post_decay` | Extreme Right-Tail | REJECTED (Target Leakage) | INVALID (Post-period) | None | DROP IMMEDIATELY; purge from feature store |
</feature_audit_table>

<remediation_protocol>
- Pipeline Purge: Drop `sessions_post_decay` from the feature extraction DAG before model training.
- Log Transforms: Apply numpy `log1p` to `impressions_90d` and `clicks_90d` in the feature transformation layer.
- Non-Linear SERP Encoding: Bucket `avg_position` into categorical tiers to reflect non-linear click-curve dropoffs.
- Data Contract Assertion: Add Great Expectations / Pydantic schema check ensuring timestamp(feature_observation) < timestamp(decay_label).
</remediation_protocol>
```

#### Notes on Run 4:
1. **What changed in the prompt:** Imposed rigid **Output Structure** using explicit XML container tags (`<executive_summary>`, `<feature_audit_table>`, `<remediation_protocol>`) with fixed column headers.
2. **Observed output difference:** The response became completely clean and machine-parseable. Zero conversational preamble remained; the output transitioned from prose advice to an actionable engineering contract ready to plug into an automated review pipeline.

---

### Run 5: Version 5 (+ Step Decomposition)

#### Named Technique:
**Step Decomposition**

#### The Prompt:
```text
You are a Principal Machine Learning Scientist and Search Intelligence Auditor at FlyRank, specializing in feature store engineering, non-parametric statistical evaluation, and data leakage prevention in ranking systems.

Context & Business Motivation:
[Context as in Version 2]

Examples:
[Few-Shot Examples as in Version 3]

Output Structure Requirements:
[XML Tags as in Version 4]

Step-by-Step Execution Protocol:
Execute your evaluation sequentially through these 4 steps before populating the output:
Step 1: Distribution & Heavy-Tail Diagnostic — Calculate the Mean/Median ratio and evaluate whether kurtosis invalidates linear Pearson covariance.
Step 2: Temporal Boundary & Leakage Check — Verify whether the observation window precedes the label event. If temporal violation is detected, hard-fail the feature.
Step 3: Non-Parametric Rank Correlation Audit — Compare Pearson r with Spearman rho; diagnose whether divergence indicates monotonic signal hidden by tail outliers.
Step 4: Synthesis & Remediation Specification — Map each confirmed feature to its mathematically appropriate transformation function (log1p, quantile binning, tier-encoding).

[Input Candidate Feature Table]
```

#### Output:
```xml
<executive_summary>
Rigorous 4-step feature audit conducted across 6 candidate search signals. The audit successfully intercepted a severe target leakage flaw in `sessions_post_decay` (hard-failed in Step 2). Step 1 and Step 3 diagnostics confirmed that extreme right-tail kurtosis (Mean/Median ratio of 7.8x on impressions, 10.1x on clicks) severely suppresses Pearson correlation (-0.04), while non-parametric Spearman rank analysis (+0.28) proves significant monotonic predictive power. Four features are passed for production feature engineering with mandatory non-linear transformations.
</executive_summary>

<feature_audit_table>
| Feature Name | Distribution Risk | Observed Signal Verdict | Pre/Post Validity | Production Transformation | Action / Decision |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `impressions_90d` | Extreme Right-Tail (Mean/Median = 7.8x, Kurtosis > 18) | CONFIRMED (Non-Linear) | PASS (Pre-period) | `f(x) = np.log1p(x)` | Retain as primary volume baseline |
| `clicks_90d` | Extreme Right-Tail (Mean/Median = 10.1x, Kurtosis > 22) | MIXED (Collinear Risk) | PASS (Pre-period) | Ratio transform: `clicks / impressions` | Transform into observed CTR feature |
| `avg_position` | Moderate Skew (Mean 14.8, Median 9.4) | CONFIRMED (Monotonic) | PASS (Pre-period) | Spline / Piecewise Linear: [1-3, 4-10, 11-20, >20] | Retain as non-linear rank cliff predictor |
| `ctr` | Zero-Inflated / Right-Skew | CONFIRMED (Inverse) | PASS (Pre-period) | `f(x) = np.log((ctr + 1e-4) / (1 - ctr + 1e-4))` (Logit) | Retain as primary CTR decay signal |
| `days_since_last_update` | Bimodal Skew (Median 84d, Max 720d) | CONFIRMED (Freshness Risk) | PASS (Pre-period) | Robust Cap at 365d: `min(days, 365)` | Retain as staleness feature |
| `sessions_post_decay` | Extreme Right-Tail | REJECTED (Hard Temporal Leakage) | FAIL (Post-period) | Disqualified | DROP IMMEDIATELY; add CI schema gate |
</feature_audit_table>

<remediation_protocol>
- Pipeline Purge: Drop `sessions_post_decay` from the feature extraction DAG before model training.
- Mathematical Transforms: Apply `np.log1p` to impressions and logit transform to CTR to normalize heavy tails.
- SERP Cliff Encoding: Discretize `avg_position` to preserve non-linear rank dropoffs between Page 1 and Page 2.
- Data Contract Gate: Enforce automated assertion `check_temporal_precedence(feature_timestamp, label_timestamp)`.
</remediation_protocol>
```

#### Notes on Run 5:
1. **What changed in the prompt:** Implemented **Step Decomposition** enforcing a 4-step chain-of-thought analysis (Distribution diagnostic $\rightarrow$ Temporal check $\rightarrow$ Non-parametric divergence $\rightarrow$ Transformation synthesis).
2. **Observed output difference:** The model's technical depth peaked: it computed specific Mean-to-Median skew ratios (7.8x and 10.1x), diagnosed the exact mathematical failure of Pearson covariance under high kurtosis, provided exact mathematical functions (`np.log1p`, logit transform), and specified automated schema gate assertions.

---

## 4. Cross-Model Comparison: Claude 3.5 Sonnet vs. ChatGPT (GPT-4o)

The identical final prompt (Version 5) was executed on both **Claude 3.5 Sonnet** and **ChatGPT (GPT-4o)** using the same reference dataset.

| Evaluation Axis | Claude 3.5 Sonnet | ChatGPT (GPT-4o) | Specific Divergence & Honest Findings |
| :--- | :--- | :--- | :--- |
| **Tone & Stance** | Austere, highly technical, uncompromising auditor mindset. Zero conversational softening. | Highly competent and polite; adopted a consulting/advisory tone with explanatory framing. | Claude felt like a senior ML engineer writing an internal architecture memo. GPT-4o felt like an expert consultant teaching the user. |
| **Statistical Accuracy** | Correctly identified that Pearson $r$ is mathematically suppressed by high kurtosis and recommended monotonic log1p/spline transformations. | Correctly identified the Pearson/Spearman divergence, but initially suggested standard z-score normalization alongside log transforms. | **Claude won on mathematical precision:** Z-score normalization fails on heavy tails because the mean and variance are themselves destabilized by outliers. Claude recognized that only non-linear monotonic transforms (`log1p` / rank binning) are mathematically valid here. |
| **Leakage Detection** | Hard-failed `sessions_post_decay` at Step 2 with immediate disqualification and CI schema gate recommendation. | Caught the leakage clearly and recommended removal, but still listed descriptive stats for it in the table. | Both models successfully caught the temporal leakage that the naive baseline failed, but Claude completely refused to assign it any valid feature engineering transformation. |
| **Structural Compliance** | Flawless XML adherence. Emitted strictly `<executive_summary>`, `<feature_audit_table>`, and `<remediation_protocol>` with zero characters outside the tags. | Followed XML tags, but prepended a conversational markdown intro (*"Here is the feature audit according to your 4-step protocol:"*) and appended closing remarks. | Claude strictly obeyed negative constraints against extra-tag tokens, making it superior for direct CI/CD pipeline integration. |
| **Failure Points / Quirks** | Was overly rigid: classified `clicks_90d` as strictly redundant without exploring potential interaction features. | Tended to be overly verbose in table cells, causing horizontal scrolling overflow. Suggested standard scaling which is mathematically inappropriate for Pareto tails. | Claude's failure was over-conservatism; GPT-4o's failure was mild mathematical over-simplification (z-scoring heavy tails). |

---

## 5. Final Reusable Prompt Template (Production Ready)

This template can be adopted by any ML engineer, data scientist, or search analyst across any tabular feature audit task without needing FlyRank-specific context.

```markdown
# SYSTEM ROLE
You are a Principal Machine Learning Scientist and Feature Store Auditor specializing in data quality verification, non-parametric statistical evaluation, and data leakage prevention.

# OBJECTIVE
Perform a comprehensive diagnostic signal audit on the provided candidate feature set for: {{TARGET_PREDICTION_TASK}}. 
Identify distribution anomalies, detect heavy-tail distortions, hard-fail any temporal data leakage, and specify mathematically sound transformations.

# DOMAIN CONTEXT & EVALUATION PRINCIPLES
1. Heavy-Tail Reality: In tabular datasets (especially engagement, traffic, or financial metrics), variables often follow heavy-tailed power-law distributions. A low linear Pearson correlation does not imply lack of predictive power; extreme outliers suppress linear covariance. Rank-based Spearman correlation must be used to detect monotonic signals.
2. Temporal Leakage Hard-Gate: Any feature whose observation window overlaps with or occurs after the target event must be immediately REJECTED. No exceptions.
3. Waste Minimization: Disqualify uninformative, zero-variance, or collinear signals to avoid model complexity.

# FEW-SHOT BENCHMARK
Feature: metric_alpha | Stats: Mean 50k, Med 2k, Kurtosis 20+. Pearson r = -0.02, Spearman rho = +0.35. Pre-period.
Verdict: CONFIRMED (Non-Linear) | Action: Apply np.log1p(x) transformation to resolve tail suppression.

Feature: metric_beta | Stats: Pearson r = +0.72. Observed concurrently with or post-target event.
Verdict: REJECTED (Temporal Leakage) | Action: Hard-fail; drop from DAG.

# STEP-BY-STEP EXECUTION PROTOCOL
Execute these 4 steps sequentially:
1. Distribution Diagnostic: Compare Mean vs. Median and evaluate Kurtosis. Determine if outliers distort linear metrics.
2. Temporal Boundary Check: Verify observation window relative to target event. Hard-fail any leakage.
3. Non-Parametric Correlation Analysis: Compare Pearson r against Spearman rho to detect non-linear monotonic signals.
4. Remediation & Transformation Specification: Prescribe exact mathematical transformations (e.g. log1p, logit, spline binning).

# OUTPUT SPECIFICATION
Emit your analysis strictly inside these XML tags with zero exterior text:
<executive_summary>
[High-level summary of total features audited, pass/reject counts, and critical findings]
</executive_summary>

<feature_audit_table>
| Feature Name | Distribution Risk | Observed Signal Verdict | Pre/Post Validity | Production Transformation | Action / Decision |
</feature_audit_table>

<remediation_protocol>
[Bulleted actionable engineering fixes and automated data contract checks]
</remediation_protocol>

---

# INPUT TASK PARAMETERS
- Target Task: {{TARGET_PREDICTION_TASK}}
- Target Label Definition: {{TARGET_LABEL_DEFINITION}}

# INPUT FEATURE CANDIDATES
{{CANDIDATE_FEATURE_METADATA_TABLE}}
```

---

## 6. Verification and Teammate Usage Guide

1. **How to run this prompt:** Copy the template from Section 5.
2. **Fill the 3 Parameters:**
   - `{{TARGET_PREDICTION_TASK}}`: e.g., "Organic Content Decay Classification"
   - `{{TARGET_LABEL_DEFINITION}}`: e.g., "Page traffic dropped >30% over trailing 30 days"
   - `{{CANDIDATE_FEATURE_METADATA_TABLE}}`: Paste your feature summary matrix.
3. **Execute:** Run in Claude 3.5 Sonnet or GPT-4o to receive an immediate, machine-parseable feature audit.
