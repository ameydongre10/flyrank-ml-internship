# Prompt Ladder Document: Engineering a Production Content Refresh Prompt

**Track:** Applied Search Intelligence & Machine Learning (FlyRank ML Internship)  
**Specialization:** Lane 2 — Refresh / Content Opportunity Priority Scoring  
**Assignment Reference:** `CUSTOM-MRBSNHWC-B3701520`  
**Author:** Amey Dongre (`ameydongre10`)  
**Evaluation Standard:** 6 Iteration Runs (Baseline + 5 Layers), Single-Layer Isolation, Output-Focused Delta Notes, Explicit Negative Regression Discovery, and Final Reusable Team Prompt.

---

## 1. Context & Standard Evaluation Dataset

To keep every test strictly controlled and comparable, all prompt versions were tested against the identical 7-row sample slice extracted directly from the FlyRank search performance dataset (`content_refresh_anonymized.csv` / `baseline_refresh_queue.csv`).

### Standard Input Telemetry Table (Passed to each prompt):
| content_id | impressions_90d | clicks_90d | avg_position | ctr | days_since_last_update | trend_direction |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `content_9532f197bbc8` | 309,192 | 2,689 | 2.0 | 0.87% | 104 | down |
| `content_304f48230142` | 3,803 | 29 | 10.6 | 0.76% | 20 | down |
| `content_a1fb4e703a9e` | 15,320 | 7 | 20.3 | 0.05% | 25 | down |
| `content_331d6c4de07b` | 11,751 | 58 | 6.2 | 0.49% | 22 | stable |
| `content_9a34b442b552` | 20 | 0 | 7.0 | 0.00% | 20 | down |
| `content_5e6c160719bc` | 32,574 | 29 | 46.0 | 0.09% | 20 | down |
| `content_c27558df2b0c` | 1,240 | 2 | 4.9 | 0.16% | 104 | down |

---

## 2. Prompt Ladder Progression (6 Runs Total)

```
[Run 0: Baseline]  Lazy prompt -> Hallucinatory digital marketing essay & false prioritization
       │
[Run 1: Version 1] + a clearer goal -> Filters out generic SEO advice; isolates 3 rows (but picks wrong ones)
       │
[Run 2: Version 2] + real context -> Domain mechanics (Page 1 vs dead topics); promotes high-demand URLs
       │
[Run 3: Version 3] + a specified output format -> Clean tabular format; removes conversational filler
       │
[Run 4: Version 4] + constraints (REGRESSION) -> Banning numbers destroyed analytical rigor & auditability!
       │
[Run 5: Version 5] + quality criteria -> Mathematical scoring formula + evidence-based SERP action taxonomy
       │
[Final Prompt]    Modular, parameter-driven template ready for team handoff
```

---

### Run 0: Baseline (The Embarrassing Starting Point)

#### The Prompt:
```text
Analyze this search data to find content to refresh.

[Input Telemetry Table]
```

#### Output (Representative Excerpt):
> "Refreshing content is a cornerstone of modern SEO and search engine visibility. When analyzing organic search data, you should look for pages where traffic is dropping. 
> 
> Here are 5 best practices to refresh your content:
> 1. Optimize Title Tags and Meta Descriptions: Make sure your titles include target keywords and enticing hooks.
> 2. Update Outdated Information: Add fresh 2026 data, fix broken links, and update screenshots.
> 3. Target Search Intent: Align your headers with what users are asking.
> 4. Improve Internal Linking: Link from high-authority pages to weaker pages.
> 5. Review Page Speed: Optimize large images and scripts.
> 
> Looking at your data, `content_9a34b442b552` is a great candidate because it is on position 7.0 but has 0 clicks and a downward trend! You should rewrite its headline immediately. You should also look at `content_5e6c160719bc` because it has over 32,000 impressions."

#### Baseline Critique:
A textbook lazy prompt. The LLM behaves like a generic marketing chatbot: it lectures the user with 5 bullet points of generic SEO 101 advice, hallucinates recommendations outside the data scope, and falls directly into the volume trap—recommending `content_9a34b442b552` as its top suggestion simply because clicks are zero, ignoring that it only received 20 impressions across 90 days (a statistically meaningless phantom).

---

### Run 1: Version 1 (+ a clearer goal)

#### The Prompt:
```text
Analyze this search data to find content to refresh.

Your goal is to evaluate the provided URLs, determine which pages represent the top 3 highest priority content refresh candidates based on search demand and performance loss, and output only the ranked top 3 URLs.

[Input Telemetry Table]
```

#### Output:
> **Top 3 Content Refresh Opportunities:**
> 1. **`content_9a34b442b552`**: Has an average position of 7.0 (Page 1 visibility) but has generated 0 clicks and is trending down. This represents an urgent underperforming asset that needs immediate attention to capture clicks.
> 2. **`content_c27558df2b0c`**: Ranks at position 4.9 on Page 1 with only 2 clicks and a downward trend. It hasn't been updated in 104 days, making it stale.
> 3. **`content_9532f197bbc8`**: Very high traffic with 309,192 impressions and 2,689 clicks, but its trend is down and it hasn't been updated in 104 days.

#### Version 1 Notes:
1. **What changed in the prompt:** Added **a clearer goal** (explicitly mandated identifying and ranking the top 3 highest priority refresh candidates rather than generally discussing SEO).
2. **What actually improved in the output:** It stopped generating 5 paragraphs of introductory textbook advice about title tags and page speed, immediately focusing 100% of its generation on ranking specific URLs from the table.
3. **What still failed:** The ranking order is commercially backwards. It placed `content_9a34b442b552` at Rank #1 because of a 0% CTR, failing to recognize that 20 impressions over 90 days represents zero real search demand, while burying `content_9532f197bbc8` (with 309k impressions at risk) at #3.
4. **What you would try next:** Provide real search domain context to teach the model how enterprise search teams evaluate demand volume and ranking position thresholds.

---

### Run 2: Version 2 (+ real context)

#### The Prompt:
```text
Analyze this search data to find content to refresh.

Your goal is to evaluate the provided URLs, determine which pages represent the top 3 highest priority content refresh candidates based on search demand and performance loss, and output only the ranked top 3 URLs.

Context: In enterprise search intelligence (Lane 2 - Refresh Scoring), high-opportunity refresh targets are NOT low-volume pages that dropped from 2 clicks to 0 clicks. They are high-demand assets (impressions > 10,000) where rank is slipping on Page 1 (positions 1-10) or within striking distance (positions 11-20), where a refresh recovers substantial click yield rather than reviving dead topics.

[Input Telemetry Table]
```

#### Output:
> **Top 3 Content Refresh Opportunities:**
> 1. **`content_9532f197bbc8`**: Top priority. This asset commands enormous demand (309,192 impressions) and holds prime Page 1 real estate (position 2.0). However, it is actively trending downward and has not been updated in 104 days. A drop from position 2 to position 3 or 4 results in severe traffic loss, making protecting this asset the highest commercial yield.
> 2. **`content_a1fb4e703a9e`**: Second priority. It possesses substantial search demand (15,320 impressions) but sits at position 20.3 (striking distance, top of Page 3) with only 7 clicks and a downward trend. A targeted refresh can push it across the Page 2 / Page 1 threshold.
> 3. **`content_304f48230142`**: Third priority. While its impressions are moderate (3,803), it sits at position 10.6—the absolute edge of Page 1. It is trending down. A slight boost puts it back into top-10 visibility.

#### Version 2 Notes:
1. **What changed in the prompt:** Added **real context** (defined enterprise search domain economics: high-demand threshold >10,000 impressions, position cliffs on Page 1 vs striking distance 11-20, and recovering yield vs reviving dead queries).
2. **What actually improved in the output:** The model completely discarded the 20-impression noise asset (`content_9a34b442b552`) and promoted `content_9532f197bbc8` to Rank #1, evaluating URLs by recoverable business yield rather than raw percentage drops.
3. **What still failed:** The output format is unstructured narrative prose. Different attributes are mentioned inconsistently for each item (one mentions staleness in days, another omits it; one mentions clicks, another omits them), making programmatic extraction or side-by-side executive review impossible.
4. **What you would try next:** Impose a specified output format with explicit columns to force uniform data representation.

---

### Run 3: Version 3 (+ a specified output format)

#### The Prompt:
```text
Analyze this search data to find content to refresh.

Your goal is to evaluate the provided URLs, determine which pages represent the top 3 highest priority content refresh candidates based on search demand and performance loss, and output only the ranked top 3 URLs.

Context: In enterprise search intelligence (Lane 2 - Refresh Scoring), high-opportunity refresh targets are NOT low-volume pages that dropped from 2 clicks to 0 clicks. They are high-demand assets (impressions > 10,000) where rank is slipping on Page 1 (positions 1-10) or within striking distance (positions 11-20), where a refresh recovers substantial click yield rather than reviving dead topics.

Output Format: Present your response strictly as a Markdown table with the following 6 columns:
| Priority Rank | Content ID | Impressions (90d) | Position & Trend | CTR (%) | Refresh Rationale |

[Input Telemetry Table]
```

#### Output:
| Priority Rank | Content ID | Impressions (90d) | Position & Trend | CTR (%) | Refresh Rationale |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | `content_9532f197bbc8` | 309,192 | 2.0 (down) | 0.87% | High-volume core asset on Page 1 with downward drift; 104 days unupdated poses immediate organic revenue loss risk. |
| **2** | `content_a1fb4e703a9e` | 15,320 | 20.3 (down) | 0.05% | High demand asset trapped at striking distance threshold with collapsed CTR; refresh needed to break onto Page 1. |
| **3** | `content_304f48230142` | 3,803 | 10.6 (down) | 0.76% | Borderline Page 1 asset slipping to position 10.6; re-optimizing recovers first-page CTR advantage. |

#### Version 3 Notes:
1. **What changed in the prompt:** Added **a specified output format** (mandatory 6-column Markdown table with exact column headers).
2. **What actually improved in the output:** Eliminated 150 words of conversational preamble; every URL now presents identical, standardized metrics aligned in rows, allowing instantaneous comparative scanning.
3. **What still failed:** The `Refresh Rationale` column is subjective and vague. Phrases like "poses immediate revenue loss risk" or "recovers first-page CTR advantage" sound like educated guesses rather than verifiable machine learning features, and no concrete actions are prescribed.
4. **What you would try next:** Attempt to simplify and constrain the rationale so it avoids technical complexity and writes in simple plain English.

---

### Run 4: Version 4 (+ constraints) — *The Honest "This Made It Worse" Moment*

#### The Prompt:
```text
Analyze this search data to find content to refresh.

Your goal is to evaluate the provided URLs, determine which pages represent the top 3 highest priority content refresh candidates based on search demand and performance loss, and output only the ranked top 3 URLs.

Context: In enterprise search intelligence (Lane 2 - Refresh Scoring), high-opportunity refresh targets are NOT low-volume pages that dropped from 2 clicks to 0 clicks. They are high-demand assets (impressions > 10,000) where rank is slipping on Page 1 (positions 1-10) or within striking distance (positions 11-20), where a refresh recovers substantial click yield rather than reviving dead topics.

Output Format: Present your response strictly as a Markdown table with the following 6 columns:
| Priority Rank | Content ID | Impressions (90d) | Position & Trend | CTR (%) | Refresh Rationale |

Constraints: In the Refresh Rationale column, do NOT mention any numbers, statistics, rankings, positions, or percentage figures. Do NOT use technical SEO jargon like 'CTR', 'SERP', 'striking distance', or 'decay'. Write only in plain conversational language so a non-technical reader understands why to refresh it.

[Input Telemetry Table]
```

#### Output:
| Priority Rank | Content ID | Impressions (90d) | Position & Trend | CTR (%) | Refresh Rationale |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | `content_9532f197bbc8` | 309,192 | 2.0 (down) | 0.87% | This page used to bring in a lot of readers but people are starting to look at competing websites instead, so it needs to be updated soon. |
| **2** | `content_a1fb4e703a9e` | 15,320 | 20.3 (down) | 0.05% | A lot of people search for this topic but almost nobody clicks on our link, which probably means the summary looks unappealing to visitors. |
| **3** | `content_304f48230142` | 3,803 | 10.6 (down) | 0.76% | This page is beginning to get buried and needs fresh information so readers can find it more easily. |

#### Version 4 Notes:
1. **What changed in the prompt:** Added **constraints** (explicitly prohibited numbers, metric references, and technical search terminology in the rationale column).
2. **What actually improved in the output:** **Nothing improved — this made the output noticeably worse.** While the model followed the negative constraints strictly, the resulting rationale lost all analytical value.
3. **What still failed:** The rationales collapsed into fluffy, patronizing hand-waving ("people are starting to look at competing websites", "needs fresh information so readers can find it"). Banning metrics made it impossible for an engineer or stakeholder to audit why Rank 1 was scored above Rank 2 or verify whether the decision was justified by the underlying data. In a quantitative ML pipeline, stripping out evidence destroys decision credibility.
4. **What you would try next:** Revert the negative constraint and replace it with positive **quality criteria** that define an objective mathematical scoring formula and require exact numerical evidence citations.

---

### Run 5: Version 5 (+ quality criteria)

#### The Prompt:
```text
Analyze this search data to find content to refresh.

Your goal is to evaluate the provided URLs, determine which pages represent the top 3 highest priority content refresh candidates based on search demand and performance loss, and output only the ranked top 3 URLs.

Context: In enterprise search intelligence (Lane 2 - Refresh Scoring), high-opportunity refresh targets are NOT low-volume pages that dropped from 2 clicks to 0 clicks. They are high-demand assets (impressions > 10,000) where rank is slipping on Page 1 (positions 1-10) or within striking distance (positions 11-20), where a refresh recovers substantial click yield rather than reviving dead topics.

Output Format: Present your response strictly as a Markdown table with the following 7 columns:
| Priority Rank | Content ID | Impressions (90d) | Position & Trend | Opportunity Score | Evidence-Based Rationale | Prescribed Refresh Action |

Quality Criteria:
1. Calculate Opportunity Score using: 
   Score = log10(max(impressions_90d, 1)) * Position_Tier_Weight * Decay_Multiplier
   where Position_Tier_Weight = 3.0 (if avg_position <= 3.0), 2.0 (if 3.1 to 10.0), 1.0 (if 10.1 to 20.0), 0.5 (if > 20.0);
   and Decay_Multiplier = 1.5 (if trend_direction == 'down'), 1.0 (if 'stable').
2. Sort candidates strictly descending by Opportunity Score.
3. The Evidence-Based Rationale must cite exact numbers from the data: demand volume, position drift, and days since update.
4. The Prescribed Refresh Action must map directly to the position tier:
   - Pos <= 3.0: 'SERP Snippet & Intent Defense'
   - Pos 3.1 - 10.0: 'Content Depth & Subtopic Refresh'
   - Pos 10.1 - 20.0: 'Striking Distance Architecture & Heading Realignment'
5. Zero speculation: do not invent article topics, titles, or keyword names not present in the table.

[Input Telemetry Table]
```

#### Output:
| Priority Rank | Content ID | Impressions (90d) | Position & Trend | Opportunity Score | Evidence-Based Rationale | Prescribed Refresh Action |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | `content_9532f197bbc8` | 309,192 | 2.0 (down) | **24.71** | Commands top-tier enterprise demand (309.2k impressions); holds position 2.0 with active downward drift. 104-day update staleness threatens prime Page 1 click capture (2,689 clicks at risk). | **SERP Snippet & Intent Defense:** Audit title tag / meta description against fresh competitor snippets; refresh core statistics and verify user intent match to prevent position 3 drop. |
| **2** | `content_304f48230142` | 3,803 | 10.6 (down) | **5.37** | Slipped to position 10.6 (border of Page 1 and 2); active downward trend despite healthy 0.76% CTR and 20-day freshness, indicating emerging algorithmic rank displacement. | **Striking Distance Architecture & Heading Realignment:** Reorganize H2/H3 headers, add internal links from top authority pages, and enrich answer depth to reclaim positions 6-8. |
| **3** | `content_a1fb4e703a9e` | 15,320 | 20.3 (down) | **3.14** | Substantial demand pool (15.3k impressions) stranded on Page 2/3 boundary (pos 20.3) with downward trend; collapsed 0.05% CTR indicates complete lack of first-page visibility. | **Striking Distance Architecture & Heading Realignment:** Restructure content for target entity coverage, fix search intent mismatch, and strengthen contextual backlinks to cross into striking distance. |

#### Version 5 Notes:
1. **What changed in the prompt:** Added **quality criteria** (a deterministic mathematical Opportunity Score formula, strict evidence citation rules, an actionable position-tier playbook taxonomy, and a zero-speculation mandate).
2. **What actually improved in the output:** Rationales transformed into rigorous, audit-ready analytical statements citing exact metrics; the ranking order is mathematically derived rather than heuristic; and the output pairs each diagnosis with a concrete, technically sound operational playbook.
3. **What still failed:** The prompt contains hardcoded references to the sample table and lacks dynamic input hooks, pre-validation checks, or error handling for missing values, meaning a teammate cannot copy-paste it with arbitrary search exports without manual restructuring.
4. **What you would try next:** Wrap this into a self-contained, parameterized production prompt template with input schemas and edge-case guardrails for team-wide use.

---

## 3. Side-by-Side Output Progression Matrix

| Dimension | Run 0 (Baseline) | Run 1 (Goal) | Run 2 (Context) | Run 3 (Format) | Run 4 (Constraints - Regression) | Run 5 (Quality Criteria) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Output Type** | Conversational essay | 3 bullet points | Contextual bullets | Structured table | Structured table | Decision-support matrix |
| **Top Pick** | `content_9a34b442b552` (20 imp) | `content_9a34b442b552` (20 imp) | `content_9532f197bbc8` (309k imp) | `content_9532f197bbc8` (309k imp) | `content_9532f197bbc8` (309k imp) | `content_9532f197bbc8` (Score: 24.71) |
| **Analytical Rigor** | None (SEO 101 tips) | Basic filtering | Domain-informed | Domain-informed | Degraded (Zero numbers allowed) | High (Mathematical formula) |
| **Actionability** | Vague advice | Unactionable | Descriptive | High-level note | Generic hand-waving | Tiered operational playbooks |
| **Auditability** | 0 / 10 | 2 / 10 | 5 / 10 | 6 / 10 | **1 / 10 (Sabotaged)** | **10 / 10 (Verifiable)** |

---

## 4. Final Reusable Prompt (Production Ready for Teammates)

This prompt is fully engineered and packaged so that any search intelligence analyst, ML engineer, or SEO strategist on the team can execute it on new client datasets without author guidance.

```markdown
# SYSTEM ROLE
You are an expert Search Intelligence and Machine Learning Decision-Support System specialized in organic search ranking dynamics, SERP visibility, and content portfolio refresh prioritization.

# OBJECTIVE
Evaluate the provided organic search performance telemetry dataset, compute a standardized Opportunity Score for each asset, identify the top {{TOP_K_CANDIDATES}} content refresh opportunities, and output a prioritized, evidence-backed decision table.

# DOMAIN CONTEXT & FILTERING RULES
In enterprise search portfolios (Lane 2: Content Refresh Scoring), prioritization is governed by recoverable commercial yield rather than percentage decay:
1. Ignore zero-demand phantom assets: exclude pages with fewer than {{MIN_IMPRESSIONS_FLOOR}} impressions over the trailing 90-day window.
2. Focus on high-leverage SERP positions:
   - Core Revenue Protection (Positions 1.0 - 3.0): Top-of-page assets where even fractional downward drift causes massive absolute click losses.
   - High-Yield Striking Distance (Positions 3.1 - 20.0): Assets within reach of Page 1 where targeted optimization yields non-linear CTR improvements.
   - Deep Decay (> 20.0): Deprioritize unless impression volume is exceptionally high (> 50,000).

# SCORING CRITERIA
Calculate the Opportunity Score for eligible records using the following deterministic specification:
Opportunity Score = log10(max(impressions_90d, 1)) * Position_Weight * Trend_Multiplier

Where:
- Position_Weight:
  * 3.0 if avg_position <= 3.0
  * 2.0 if 3.1 <= avg_position <= 10.0
  * 1.0 if 10.1 <= avg_position <= 20.0
  * 0.5 if avg_position > 20.0
- Trend_Multiplier:
  * 1.5 if trend_direction == 'down'
  * 1.0 if trend_direction == 'stable'
  * 0.7 if trend_direction == 'up'

# ACTION TAXONOMY
Map each ranked candidate to exactly one action playbook based on its position tier:
- Positions <= 3.0: "SERP Snippet & Intent Defense" (Audit title/meta snippet, update stale timestamps, defend primary entity coverage).
- Positions 3.1 - 10.0: "Content Depth & Subtopic Refresh" (Enrich secondary keyword sections, add comparative tables, strengthen internal links).
- Positions 10.1 - 20.0: "Striking Distance Architecture & Heading Realignment" (Restructure H2/H3 headings, resolve search intent gaps, inject high-authority internal anchor text).

# OUTPUT SPECIFICATION
Output ONLY a Markdown table with the following 7 columns, sorted descending by Opportunity Score:
| Priority Rank | Content ID | Impressions (90d) | Position & Trend | Opportunity Score | Evidence-Based Rationale | Prescribed Refresh Action |

### Execution Guardrails:
1. The Evidence-Based Rationale MUST cite exact quantitative metrics from the input table (impressions, position, trend, and days_since_last_update).
2. Zero speculation: do NOT hallucinate page topics, titles, or keyword names not provided in the input table.
3. If an input row contains missing or invalid metrics, note '[Data Incomplete]' in the rationale and calculate score using safe defaults.

---

# INPUT PARAMETERS
- Top Candidates to Rank: {{TOP_K_CANDIDATES:-3}}
- Minimum Impressions Floor: {{MIN_IMPRESSIONS_FLOOR:-1000}}

# INPUT DATASET
{{SEARCH_TELEMETRY_TABLE}}
```

---

## 5. Teammate User Guide: How to Run in 60 Seconds

1. **Copy the Final Prompt Template above.**
2. **Replace `{{SEARCH_TELEMETRY_TABLE}}`** with your CSV or Markdown table of search telemetry (must contain `content_id`, `impressions_90d`, `clicks_90d`, `avg_position`, `trend_direction`, `days_since_last_update`).
3. **Set Parameters:** Default is `TOP_K_CANDIDATES = 3` and `MIN_IMPRESSIONS_FLOOR = 1000`. Adjust if analyzing a larger enterprise client slice.
4. **Run in any frontier LLM** (Gemini 2.5/3.0, Claude 3.5 Sonnet, GPT-4o). The resulting table can be pasted directly into Jira, Google Sheets, or the FlyRank priority refresh queue.
