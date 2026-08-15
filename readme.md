# F&B R&D Analytics — Recipe Consumer Evaluation

> **Portfolio Project | R&D Analyst Simulation**
>
> An evidence-driven analysis of whether recipe attributes are associated with stronger consumer ratings, with a deliberate focus on data validity, analytical limitations, and robustness rather than forcing an actionable finding.

---

## Executive Summary

The business question was:

> **"Certain combinations of recipe attributes (ingredients, category, nutrition profile, preparation time) may be associated with stronger consumer evaluation than others."**

The analysis found **Insufficient Evidence** for the three attribute groups tested:

- Recipe Category
- Preparation Time
- Nutrition Profile

One result initially appeared statistically significant at **p = 0.049**, but the signal disappeared when tested under alternative review-count thresholds and after removing highly prolific recipe authors.

The final conclusion was therefore **not** that these attributes have no effect.

Instead:

> **This dataset does not provide strong enough evidence to justify prioritizing any of these attributes for further R&D investment.**

The project deliberately separates:

- "No detectable signal in this dataset"
- from
- "No effect exists"

That distinction is central to the analysis.

---

# Business Question

## Business Context

An R&D / Consumer Insights team wants to understand whether certain recipe characteristics are associated with stronger consumer evaluation.

The broader question is:

> **Do certain combinations of recipe attributes tend to receive stronger consumer ratings?**

However, before testing combinations, the first analytical question was narrowed to:

> **Do recipes with certain category, preparation-time, or nutrition characteristics receive meaningfully different ratings within the same category?**

The purpose was not to decide which product to launch.

It was a **screening decision**:

> Is there enough evidence to justify spending additional R&D resources on investigating a particular recipe attribute?

---

## Decision Context

The simulated stakeholder is an R&D / Consumer Insights team deciding whether to invest further in:

- Consumer research
- Sensory testing
- Product prototyping
- Additional investigation of a recipe attribute

The analysis therefore focuses on determining whether the available data contains a sufficiently credible signal to justify that next step.

---

# Data & What Each Row Represents

The dataset contains **10,000 recipes** sampled from the Food.com recipe dataset.

Each row represents:

> **One recipe, not one review or one customer.**

Ratings are aggregated at the recipe level.

## Important Data Characteristics

### 48.3% of Recipes Have No Rating

Almost half of the recipes have no `AggregatedRating`.

This missingness cannot reasonably be treated as random.

A recipe with a rating is a recipe that someone chose to engage with and rate.

Therefore:

> The analysis of ratings applies only to the subset of recipes that received ratings.

It should not automatically be generalized to all recipes.

---

### Review Count Is Highly Skewed

Among recipes with review information:

- Most recipes have only a small number of reviews.
- A small number have many reviews.
- Therefore, a rating based on 2 reviews does not carry the same reliability as one based on 200 reviews.

A minimum review-count threshold was therefore introduced before comparing ratings.

---

### `AuthorId` Is Not a Customer ID

`AuthorId` identifies the person who posted the recipe.

It does **not** identify the people who consumed or rated the recipe.

Therefore:

- Author-level clustering can affect recipe-level comparisons.
- But `AuthorId` cannot be interpreted as reviewer/customer identity.

---

## What the Dataset Cannot Answer

The dataset does not contain:

- Individual review text
- Reviewer identity
- Purchase transactions
- Revenue
- Product cost
- Customer demographics

Therefore, this analysis can speak about:

> **Recipe rating patterns**

It cannot directly measure:

- Customer demand
- Purchase intent
- Revenue impact
- Profitability
- What consumers "actually want" in a broader commercial sense

---

# Analytical Approach

The project followed a staged analytical process:

```text
Business Question
        ↓
Data Understanding & Validity Checks
        ↓
Scoping & Variable Design
        ↓
Exploratory Analysis
        ↓
Hypothesis Testing
        ↓
Robustness Checks
        ↓
Evidence Assessment
        ↓
Business Recommendation
