# F&B Product Innovation Intelligence
## Consumer Feedback, Product Attributes & Innovation Opportunity Analysis

> **Project Type:** Portfolio Project — Simulated Business Scenario
>
> **Dataset:** Food.com Recipes and Reviews (Kaggle, irkaal)
>
> **Actual working file:** `recipes_10k.csv` — a 10,000-row sample. RecipeId range (70–541,366) and gap-pattern analysis (mean gap ≈54, median ≈37, non-uniform) are consistent with random sampling from the full ~500K+ dataset, but the exact sampling method (simple random vs. conditional/stratified) has not been confirmed. **All scope, sample-size language, and conclusions in this project refer to this 10,000-row sample — findings will not be extrapolated to the full ~500K+ public dataset.**
>
> **Scope:** This project simulates an F&B product innovation analytics problem using publicly available recipe and review data. It does not represent internal company data, actual commercial decisions, or the operations of any specific F&B company.

---

# 1. Business Context

An F&B company operates a large and diverse product portfolio. The company has accumulated a substantial amount of consumer feedback and recipe-level information, but the R&D team faces a recurring problem:

> **There is a large amount of recipe and review data, but it is unclear which signals represent meaningful consumer preferences, genuine product pain points, or opportunities worth further R&D investigation.**

The company does not want product innovation decisions to be driven simply by the highest-rated recipes, the most frequently reviewed recipes, or the most common ingredients.

The challenge is that different functions interpret the same signals differently — and, importantly, **not every function's hypothesis can actually be tested with the data available.** Part of this project's job is to be explicit about that before any analysis begins.

---

# 2. The Internal Debate

## 2.1 Marketing Hypothesis — Testable

Marketing observes that some recipes receive substantially more reviews and stronger ratings than others.

> **Hypothesis:** Customer response may be concentrated around particular recipe categories, cuisines, ingredients, nutritional profiles, or preparation characteristics.

**What the data can support:** `RecipeCategory`, `Keywords`, `RecipeIngredientParts`, and the nutrition fields can be compared against `AggregatedRating` and `ReviewCount` to test whether certain categories or ingredient patterns are associated with stronger evaluation.

**What the data cannot support:** Marketing cannot use this dataset to identify *consumer segments* (age, income, region, or any demographic split). The dataset contains `AuthorId`/`AuthorName` for the recipe's *creator* (confirmed at Phase 0: 4,618 unique AuthorId across 10,000 recipes, ~2.2 recipes/author on average), not demographic attributes of the people who rated or reviewed it. Any claim about "which customer segment prefers what" is out of scope and will not be attempted.

However: **high review volume does not necessarily mean high satisfaction** — a recipe may simply have more exposure. Review volume and rating will not be treated as interchangeable evidence.

---

## 2.2 R&D Hypothesis — Testable

> **Hypothesis:** Certain combinations of recipe attributes (ingredients, category, nutrition profile, preparation time) may be associated with stronger consumer evaluation than others.

**What the data can support:** This is the hypothesis best matched to the available fields. `RecipeIngredientParts`, `RecipeCategory`, `CookTime`/`PrepTime`/`TotalTime`, and the nutrition columns (`Calories`, `FatContent`, `SodiumContent`, `ProteinContent`, etc.) can all be tested against `AggregatedRating`.

**Caveat:** The presence of an attribute in highly rated recipes does not by itself demonstrate that the attribute *causes* stronger preference. This distinction will be preserved throughout — especially because `ReviewCount` varies enormously across recipes (confirmed at Phase 0: median = 2, mean = 5.1, max = 542, among the 5,297 recipes that have at least one review), so small-sample ratings need to be treated with appropriate caution rather than compared at face value with high-volume ones.

**Ingredient-combination risk:** `RecipeIngredientParts` is a list-like field with a very large number of distinct ingredients across 10,000 recipes. Testing arbitrary *combinations* of ingredients directly risks a combinatorial explosion — most specific combinations will appear in only 1-2 recipes, which is the same small-sample problem already flagged for `ReviewCount`, just at the ingredient level. To keep this tractable and avoid overfitting: analysis will start at the **single-ingredient level** (frequency and association with rating, within category), and only move to co-occurrence/combination analysis using simple frequency-lift comparisons (high-rating vs. low-rating recipes within the same category) — not complex ML — and only where sample size at that level is adequate. This scoping decision will be revisited if Phase 4 (EDA) shows single-ingredient analysis is insufficient to answer the R&D hypothesis.

---

## 2.3 Consumer Insights Hypothesis — NOT TESTABLE (confirmed at Phase 0)

> **Original hypothesis:** Aggregate ratings may hide recurring product-level pain points and positive attributes that are only visible in written review text.

**Status: NOT TESTABLE.** Phase 0 data verification confirmed that no Reviews table or review-text field exists in this project. `recipes_10k.csv` contains only the aggregated `AggregatedRating` and `ReviewCount` columns — no reviewer-level text, no `ReviewId`, no free-text field of any kind. `RecipeInstructions` was checked directly and confirmed to be cooking instructions written by the recipe's author, not customer review text.

**This hypothesis is excluded from the analysis.** Any business question requiring "what customers actually said," "why they liked or disliked a recipe," or any sentiment/text-based signal cannot be answered with this dataset. If review-level text becomes available later, this hypothesis can be reopened — until then it is out of scope, not merely deprioritized.

---

## 2.4 Product / Commercial Hypothesis — Testable, Reframed

> **Original framing (rejected):** "...requires greater preparation complexity... less practical for commercial development."

**Why this framing is rejected:** This dataset contains home-cook recipes submitted to a public recipe platform. It has no data on manufacturing cost, production scalability, ingredient sourcing at commercial volume, or industrial feasibility. Framing any finding as being about "commercial development" would overstate what the data supports.

**Corrected hypothesis:**

> **A recipe that appears attractive to reviewers (based on rating) may also require substantial preparation effort or a large number of ingredients, which could make it less practical for a home cook to actually complete — independent of how well-rated it is.**

**What the data can support:** `PrepTime`, `CookTime`, `TotalTime`, and ingredient count (derived from `RecipeIngredientParts`) compared against `AggregatedRating` and `ReviewCount`. This tests home-preparation practicality, not commercial manufacturing feasibility — the two are not the same claim, and only the first is within scope.

---

# 3. Core Business Pain Point

The company has a large number of recipes, ratings, and (potentially) reviews, but:

> **The problem is not a lack of data. The problem is too much information with insufficient prioritization, and not every apparent signal is actually testable with what's available.**

A high rating may indicate satisfaction, or it may reflect a very small number of reviewers. A frequently occurring ingredient may indicate a real pattern, or it may simply be common across the category regardless of rating. None of these signals is sufficient on its own — and part of this project's discipline is separating hypotheses the data can test from ones it cannot.

**Additional pain point confirmed at Phase 0:** nearly half the dataset (4,827 of 10,000 recipes, 48.3%) has no `AggregatedRating` at all — meaning these recipes have never been reviewed. Any analysis using rating as an outcome variable will necessarily exclude this ~48%, and this exclusion must be treated as a potential selection-bias issue, not silently dropped.

---

# 4. The Difficult Business Problem

> **Given recipe-level attributes and rating/review-volume data, how can the company distinguish recurring and meaningful consumer signals from popularity effects, small-sample noise, or misleading associations — and identify which product attributes are sufficiently supported by evidence to warrant further R&D investigation?**

| Function | Hypothesis | Testable with this dataset? |
|---|---|---|
| Marketing | Preference concentrated around category/cuisine/ingredient/nutrition profile | Yes |
| Marketing | Preference varies by consumer segment (demographic) | **No — no demographic fields exist** |
| R&D | Attribute combinations associated with stronger evaluation | Yes |
| Consumer Insights | Written reviews reveal pain points hidden by aggregate rating | **No — confirmed not testable at Phase 0, no Reviews table exists** |
| Commercial | High-rated recipes may still be impractical for a home cook to prepare | Yes (reframed from "commercial development") |

---

# 5. Analytical Questions (Revised to Match Available Data)

### Consumer Preference (Recipes table)
1. Are certain recipe categories or cuisines associated with higher `AggregatedRating`?
2. Are specific ingredients or ingredient combinations associated with higher ratings, after accounting for category?
3. Are apparent high-rating patterns driven by recipes with very few reviews (small-sample effects)?

### Product Attributes vs. Preparation Burden
4. Is there a relationship between `TotalTime`/ingredient count and `AggregatedRating`?
5. Do highly rated recipes tend to require more or less preparation effort than lower-rated ones in the same category?

### Nutrition Profile
6. Are nutrition attributes (calories, fat, sugar, protein) associated with rating differences, and does this vary by category?

### Review Volume vs. Rating
7. Does `ReviewCount` correlate with `AggregatedRating`, or are they independent signals (popularity vs. satisfaction)?

### Consumer Pain Points
8. ~~*Pending confirmation of Reviews table*~~ **[REMOVED — confirmed NOT TESTABLE at Phase 0; no Reviews table or review text exists in this project]**

### Evidence Quality
9. Which of the above patterns hold up after controlling for category and review-count sample size?
10. Which apparent patterns should be marked "insufficient evidence" or "not testable"?

---

# 6. Evidence-First Principle

The project distinguishes between **Observation** (a pattern exists), **Association** (two variables move together), **Robust Evidence** (the pattern survives basic robustness checks — e.g., controlling for category, excluding low-review-count outliers), and **Hypothesis** (a possible explanation, not yet supported). "Hypothesis" is not itself a level of evidence — it sits on a separate track (Explanation Status) from Evidence Strength.

> **Correlation will not be interpreted as causation. A pattern found in the Recipes table will not be extended into a claim about "customers" or "consumer segments" unless supported by a table that actually describes reviewers.**

---

# 7. Scope and Limitations (Confirmed Against Actual Loaded Data)

Based on the 28 columns confirmed in `recipes_10k.csv` (`RecipeId, Name, AuthorId, AuthorName, CookTime, PrepTime, TotalTime, DatePublished, Description, Images, RecipeCategory, Keywords, RecipeIngredientQuantities, RecipeIngredientParts, AggregatedRating, ReviewCount, Calories, FatContent, SaturatedFatContent, CholesterolContent, SodiumContent, CarbohydrateContent, FiberContent, SugarContent, ProteinContent, RecipeServings, RecipeYield, RecipeInstructions`), this analysis **cannot** support claims about:

- Customer demographics or consumer segments (no such fields exist),
- Commercial or manufacturing feasibility (this is home-recipe data, not production data),
- Actual sales, revenue, or purchase behavior (no transaction data of any kind is present),
- Individual reviewer sentiment or written feedback of any kind (**confirmed at Phase 0: no Reviews table, no text field exists**),
- Causal explanations for why a recipe is rated the way it is (rating drivers can only be described as associative, never causal),
- External market context — macroeconomic conditions, competitor activity, or other brands/products — since this dataset contains only recipe and rating data from a single platform, with no fields describing anything outside that platform,
- **Generalization beyond this 10,000-row sample** to the full ~500K+ public Food.com dataset, since the exact sampling methodology has not been confirmed (only "consistent with random sampling" based on RecipeId distribution — see Data Verification below).

Where the data does not support a conclusion, the project will state **"insufficient evidence"** or **"not testable with available data"** rather than infer an answer.

---

# 8. Data Verification (Completed at Phase 0)

The following has been confirmed directly from the loaded `recipes_10k.csv` — not assumed:

- **Row/column counts:** 10,000 rows, 28 columns. Grain confirmed as 1 row = 1 recipe (10,000 unique `RecipeId`, no duplicates).
- **Reviews table:** confirmed NOT present in this project. No review text exists anywhere in the available data.
- **Missing-value structure** (of 10,000 rows):

  | Field | Missing | % |
  |---|---|---|
  | RecipeYield | 6,786 | 67.9% |
  | AggregatedRating | 4,827 | 48.3% |
  | ReviewCount | 4,703 | 47.0% |
  | RecipeServings | 3,453 | 34.5% |
  | CookTime | 1,598 | 16.0% |
  | Keywords | 328 | 3.3% |
  | RecipeCategory | 13 | 0.1% |

  `AggregatedRating`/`ReviewCount` missingness is `NaN` (never reviewed), confirmed distinct from a true zero. 124 recipes have `ReviewCount` present but `AggregatedRating` missing — a minimum-review filter must not assume "has ReviewCount → has Rating."

- **ReviewCount distribution** (n=5,297 recipes with ≥1 review): median = 2, mean = 5.1, max = 542, std = 15.4 — strongly right-skewed. A minimum-review threshold or weighted/shrinkage approach is required before comparing ratings across recipes at face value.
- **Data types:** all nutrition/rating fields are `float64`. `CookTime`/`PrepTime`/`TotalTime` are stored as ISO 8601 duration strings (e.g., `PT1H20M`) and require parsing before use — not directly numeric.
- **Parsing requirement confirmed:** list-like string columns (`RecipeIngredientParts`, `RecipeIngredientQuantities`, `Keywords`, `RecipeInstructions`) are stored as R-style `c("...", "...")` strings and need parsing before use.
- **Nutrition outliers observed** (not yet treated): e.g., Calories mean ≈472 vs. max 19,380; Sodium mean ≈749 vs. max 113,198. Likely batch-size recipes (e.g., a recipe yielding a large batch rather than a single serving) or data-entry errors.

  **Working principle for treatment (to be applied at EDA, not before):** outliers will be flagged relative to their **own `RecipeCategory`**, not the whole dataset — a high-calorie value may be normal for one category (e.g., a large-batch dessert) and extreme for another (e.g., a single-serving salad). A percentile-based flag within category (e.g., beyond the 1st/99th percentile) will be used to identify candidates for exclusion or separate treatment, and any exclusion will be accompanied by a sensitivity check comparing results with and without the flagged rows — not a silent, one-way removal.

---

# 9. Deliverable & Success Criteria

**This is framed as a decision-support project, not a descriptive analysis exercise.** The output is not simply "here are some patterns in the data" — it is a structured answer to: *which product attributes are worth further R&D investigation, and how confident should R&D be in each one?*

**Deliverable (final form to be confirmed once findings exist, not designed in advance):** a short evidence-based summary per testable hypothesis (Marketing, R&D, Commercial), each finding tagged with its Evidence Strength (Observation / Association / Robust Evidence / Insufficient Evidence) and Explanation Status, plus a recommended next step drawn only from: Monitor / Investigate further / Consumer test / Prototype / No action. Whether this ships as a written memo, a simple table, or a dashboard is a presentation decision to be made after Phase 8 (Evidence Synthesis) — it will not be decided now, before any finding exists, to avoid designing the output before knowing what it needs to communicate.

**What this project deliberately will NOT produce:** a numeric Impact × Confidence × Feasibility prioritization score. Scoring "Feasibility" or "Impact" numerically would require cost, resourcing, or market data this dataset does not contain — doing so would repeat the exact mistake this project exists to avoid (manufacturing a number that looks decisive but isn't supported by evidence). Prioritization language, if used, will stay qualitative (e.g., "this finding has stronger evidence than that one, but both would compete for the same R&D attention").

**Success criteria for this project:** not "how many opportunities were found," since finding zero robust opportunities is a legitimate and useful outcome. Success means: every one of the 3 testable hypotheses (Marketing, R&D, Commercial) has a clear, evidence-backed answer — including "insufficient evidence" where that is the honest answer — such that a reader knows exactly what is and isn't known after reading it, and why.

---

## Project Principle

> **Do not start with the answer. Start with the competing explanations — and rule out the ones the data cannot actually test before analyzing the ones it can.**
