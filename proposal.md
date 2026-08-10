# F&B Product Innovation Intelligence
## Consumer Feedback, Product Attributes & Innovation Opportunity Analysis

> **Project Type:** Portfolio Project — Simulated Business Scenario
>
> **Dataset:** Food.com Recipes and Reviews (Kaggle, irkaal)
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

**What the data cannot support:** Marketing cannot use this dataset to identify *consumer segments* (age, income, region, or any demographic split). The dataset contains `AuthorId`/`AuthorName` for the recipe's *creator*, not demographic attributes of the people who rated or reviewed it. Any claim about "which customer segment prefers what" is out of scope and will not be attempted.

However: **high review volume does not necessarily mean high satisfaction** — a recipe may simply have more exposure. Review volume and rating will not be treated as interchangeable evidence.

---

## 2.2 R&D Hypothesis — Testable

> **Hypothesis:** Certain combinations of recipe attributes (ingredients, category, nutrition profile, preparation time) may be associated with stronger consumer evaluation than others.

**What the data can support:** This is the hypothesis best matched to the available fields. `RecipeIngredientParts`, `RecipeCategory`, `CookTime`/`PrepTime`/`TotalTime`, and the nutrition columns (`Calories`, `FatContent`, `SodiumContent`, `ProteinContent`, etc.) can all be tested against `AggregatedRating`.

**Caveat:** The presence of an attribute in highly rated recipes does not by itself demonstrate that the attribute *causes* stronger preference. This distinction will be preserved throughout — especially because `ReviewCount` varies enormously across recipes (some have 1–2 reviews, others dozens), so small-sample ratings need to be treated with appropriate caution rather than compared at face value with high-volume ones.

---

## 2.3 Consumer Insights Hypothesis — Conditional, Not Yet Confirmed

> **Hypothesis:** Aggregate ratings may hide recurring product-level pain points and positive attributes that are only visible in written review text.

**Status: unconfirmed.** The Recipes table (RecipeId, Name, AuthorId, ..., AggregatedRating, ReviewCount) does **not** contain review text — it only contains an aggregated score and a count. Testing this hypothesis requires the separate **Reviews table** from the same dataset (which, per the dataset's public documentation, contains `ReviewId`, `RecipeId`, `AuthorId`, `Rating`, `Review` text, and submission date).

**This hypothesis will only be included in the analysis if the Reviews table is loaded and confirmed to contain a text field.** If it is not available or does not contain usable text, this hypothesis will be marked "Not Testable" in the final report rather than answered indirectly through the aggregated rating alone.

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

---

# 4. The Difficult Business Problem

> **Given recipe-level attributes and rating/review-volume data (and review text, if confirmed available), how can the company distinguish recurring and meaningful consumer signals from popularity effects, small-sample noise, or misleading associations — and identify which product attributes are sufficiently supported by evidence to warrant further R&D investigation?**

| Function | Hypothesis | Testable with this dataset? |
|---|---|---|
| Marketing | Preference concentrated around category/cuisine/ingredient/nutrition profile | Yes |
| Marketing | Preference varies by consumer segment (demographic) | **No — no demographic fields exist** |
| R&D | Attribute combinations associated with stronger evaluation | Yes |
| Consumer Insights | Written reviews reveal pain points hidden by aggregate rating | **Conditional — requires Reviews table with text** |
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

### Consumer Pain Points (conditional on Reviews table with text)
8. *[Pending confirmation of Reviews table]* What recurring issues appear in review text for recipes with high ratings but specific recurring complaints?

### Evidence Quality
9. Which of the above patterns hold up after controlling for category and review-count sample size?
10. Which apparent patterns should be marked "insufficient evidence" or "not testable"?

---

# 6. Evidence-First Principle

The project distinguishes between **Observation** (a pattern exists), **Association** (two variables move together), **Evidence** (the pattern survives basic robustness checks — e.g., controlling for category, excluding low-review-count outliers), and **Hypothesis** (a possible explanation, not yet supported).

> **Correlation will not be interpreted as causation. A pattern found in the Recipes table will not be extended into a claim about "customers" or "consumer segments" unless supported by a table that actually describes reviewers.**

---

# 7. Scope and Limitations (Confirmed Against Actual Schema)

Based on the columns provided (`RecipeId, Name, AuthorId, AuthorName, CookTime, PrepTime, TotalTime, DatePublished, Description, Images, RecipeCategory, Keywords, RecipeIngredientQuantities, RecipeIngredientParts, AggregatedRating, ReviewCount, Calories, FatContent, SaturatedFatContent, CholesterolContent, SodiumContent, CarbohydrateContent, FiberContent, SugarContent, ProteinContent, RecipeServings, RecipeYield, RecipeInstructions`), this analysis **cannot** support claims about:

- customer demographics or consumer segments (no such fields exist),
- actual commercial/manufacturing feasibility (this is home-recipe data),
- real sales, revenue, or purchase behavior (no transaction data),
- individual reviewer sentiment (unless the separate Reviews table with text is confirmed and loaded),
- causal explanations for why a recipe is rated the way it is (rating drivers can only be described as associative).
- External market context — macroeconomic conditions, competitor activity, or other brands/products — since this dataset contains only recipe and rating data from a single platform, with no fields describing anything outside that platform.
Where the data does not support a conclusion, the project will state **"insufficient evidence"** or **"not testable with available data"** rather than infer an answer.

---

# 8. Data Verification Before Analysis

Before analysis begins, the following will be confirmed directly from the loaded data (not assumed):

- exact row/column counts for the Recipes table, and for the Reviews table if used,
- whether a Reviews table with review text is actually available and joinable via `RecipeId`,
- missing-value structure across all fields used in analysis (e.g., `RecipeServings`, `RecipeYield`, and nutrition fields show `NA` in the sample rows already seen),
- the distribution of `ReviewCount` (to set a minimum-review threshold before comparing ratings across recipes),
- parsing requirements for list-like string columns (`RecipeIngredientParts`, `RecipeIngredientQuantities`, `Keywords`, `RecipeInstructions` are stored as R-style `c("...", "...")` strings and will need parsing before use).

---

## Project Principle

> **Do not start with the answer. Start with the competing explanations — and rule out the ones the data cannot actually test before analyzing the ones it can.**
