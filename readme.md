# Which recipe signals are actually strong enough to justify product development?

An F&B product team can't test every recipe attribute before deciding where to invest. This project asks a narrower, more useful question: **based on 10,000 recipes, is there strong enough signal in category, prep time, or nutrition profile to prioritize any of them for product development — or not yet?**

> **Business question:** Which recipe attributes show a strong enough link to consumer ratings to justify further investment (consumer research, sensory testing)?
> **Dataset:** 10,000-recipe sample from Food.com (Kaggle)
> **Decision:** Hold off on prioritizing category, prep-time, or nutrition-based product development from this dataset alone — the signal isn't strong enough yet.

**Headline findings:**
- 3 attribute groups tested (category, prep time, nutrition) — **none showed strong enough evidence** to differentiate consumer ratings
- 1 result looked promising at first (p=0.049) — **it didn't survive a second look**, and that's the most important finding in this project
- 48.3% of recipes have never been rated, and 71% of ratings that exist are a perfect 5.0 — both had to be accounted for before trusting any comparison
- Recommendation: **don't invest yet** — validate with better data first

<img width="649" height="414" alt="image" src="https://github.com/user-attachments/assets/0b62bc2c-1f3b-4b85-9217-a956e7040b14" />


---

## Executive Summary

**Problem:** An F&B R&D team wants to know if certain recipe attributes are worth prioritizing in product development, based on how recipes with those attributes tend to be rated.

**What I investigated:** Whether recipe category, preparation time, or nutrition profile (sugar, sodium, saturated fat) show a meaningfully different rating pattern — after accounting for how many reviews a recipe has, which category it's in, and who authored it.

**What I found:** None of the three attribute groups showed strong enough evidence to say they're linked to higher ratings. One result initially looked significant, but it disappeared under closer testing — which turned out to be the most useful finding of the project.

**What it means:** This doesn't mean these attributes have *no* effect on ratings. It means this dataset, at this sample size, doesn't provide strong enough evidence to justify betting on any of them yet.

**What I recommend:** Don't prioritize product development based on these signals alone. If the business wants to keep investigating, the next step is better data — not a bigger version of the same dataset (see [Limitations](#limitations) for why).

---

## Business Question

**Who's making the decision:** An R&D / Consumer Insights team deciding whether to invest further (consumer research, sensory testing) in a specific recipe attribute.

**What they need to decide:** Not "which product to launch" — a narrower, earlier question: *is there enough signal to justify spending more R&D time investigating a specific attribute?* This is a screening decision, not a launch decision.

**The analytical question:** Do recipes with certain category, prep-time, or nutrition characteristics get rated meaningfully differently than recipes without them — within the same category, to avoid comparing, say, desserts to salads?

**Scope:** 10,000 recipes, treated as their own dataset — not assumed to represent the full ~500K+ Food.com catalog (see [Data](#data--what-each-row-represents)).

**What this dataset can't answer:** It has no review text, no reviewer identity, and no purchase or transaction data. So this analysis can speak to *rating patterns*, not to customer demand, purchase intent, or "what people actually want" in a broader sense.

---

## Data & What Each Row Represents

Each row is **one recipe**, not one review or one customer. Ratings are aggregated — one average rating per recipe, contributed by however many people chose to rate it.

**What this means in practice:**
- The dataset represents *recipes that got posted and optionally rated on Food.com* — not a representative sample of consumers, and not purchase behavior.
- **48.3% of recipes have never been rated.** This isn't random — a recipe with a rating is one someone actually tried and cared enough to score. Any analysis using ratings only sees the subset that got engagement, which is a meaningfully different (and smaller) group than "all recipes."
- Review counts are heavily skewed — most rated recipes have just 1–4 reviews, so a recipe's rating from 2 reviews and one from 200 reviews don't carry the same reliability. This had to be controlled for before comparing anything.
- `AuthorId` identifies who *posted* the recipe, not who ate it — not a customer ID.

Full missingness and data-quality checks are in the [Full Report](FB_Full_Report.md), Phase 0.

---

## Analytical Approach

```
Business Question
      ↓
Data Understanding & Validity Checks   → confirm what the data can and can't answer
      ↓
Scoping & Variable Design              → decide what's testable given sample size
      ↓
Exploratory Analysis                   → surface confounders, measurement issues
      ↓
Hypothesis Testing                     → test category, prep-time, nutrition against rating
      ↓
Robustness Checks                      → stress-test any promising result before trusting it
      ↓
Evidence Assessment                    → grade findings by strength, not just p-value
      ↓
Recommendation
```

Each step exists because the previous one surfaced a reason it was needed — for example, testing at multiple review-count thresholds wasn't a default step, it's what caught the one result that looked significant but wasn't stable (see [The Important Negative Finding](#the-important-negative-finding)).

---

## Key Findings

**Rating scale for evidence strength used throughout:** Observation → Association → Robust Evidence, with **Insufficient Evidence** meaning a result didn't clear even the "Association" bar.

### Finding 1 — Category
**What:** Tested whether recipe category (e.g., Dessert, Breakfast, Chicken) is linked to rating differences, across the 19 categories with enough recipes to compare fairly.
**Evidence:** Insufficient Evidence. One threshold showed a borderline result — see the negative finding below for why that didn't hold up.
**Business meaning:** Category alone isn't a reliable signal for where to prioritize development, based on this data.

### Finding 2 — Preparation Time
**What:** Tested whether short/medium/long prep-time recipes get rated differently, within category.
**Evidence:** Insufficient Evidence, consistently across every check run.
**Business meaning:** No indication that recipes taking longer (or shorter) to prepare are rated better — at least not detectably at this sample size.

### Finding 3 — Nutrition Profile
**What:** Tested sugar, sodium, and saturated fat content separately (not combined into a single "health score" — see [Limitations](#limitations) for why) against rating, within category.
**Evidence:** Insufficient Evidence for all three.
**Business meaning:** No nutrition dimension tested here showed a detectable link to consumer ratings.

**Important:** "Insufficient Evidence" means this dataset didn't provide strong enough signal to detect an effect — it does not mean these attributes have been shown to have no effect. See below.

---

## The Important Negative Finding

Sometimes the most useful answer isn't what to invest in.

One result — category, at one specific review-count threshold — came back statistically significant (p=0.049). On its own, that's the kind of number that could get escalated as "category matters."

Before trusting it, I re-ran the same test at three other review-count thresholds, and again after removing the small group of highly prolific recipe authors who could be skewing the comparison. The result **disappeared** every time except the one where it first showed up.

That's not a coincidence — it's what a false positive looks like when you go looking for it. Reporting the first significant number without checking whether it holds up is how weak signals get sold as findings. This project is built to catch that before it happens, not after.

The takeaway isn't "we found nothing." It's: **tested attributes didn't show a differentiation strong enough to trust, and the dataset currently available isn't sized well enough to rule out that a real (if small) effect exists.** Those are two different claims, and this project keeps them separate on purpose.

---

## Recommendation

**Don't prioritize** category-, prep-time-, or nutrition-based product development based on this dataset alone. The signal isn't strong enough to justify allocating R&D resources — consumer research, sensory testing, or prototyping — to any of the three attribute groups tested.

**Do, if the business wants to keep pursuing this question:** validate with better-suited data before investing further — specifically, data that doesn't share this dataset's two structural limitations (see below). Growing the *same* dataset would not fix either one.

**Postpone:** any resourcing decision tied to "which recipe attribute drives ratings" until either (a) better data is available, or (b) the business decides the question isn't worth pursuing further, given what's achievable with data like this.

---

## Limitations

Stated plainly, not as an apology — these shaped what could and couldn't be concluded.

- **Ceiling effect:** 71% of rated recipes have a rating of exactly 5.0. When most of your outcome variable is clustered at the maximum, it's structurally hard for any attribute to show a detectable difference — this is a property of how people rate on this platform, not a flaw in the analysis.
- **Selection bias:** 48.3% of recipes have never been rated, and that's not random — it's recipes nobody tried hard enough to score. Every finding here applies to the subset of recipes that got rated, not to "all recipes."
- **Shrinking sample after validity filters:** After requiring a minimum review count and a large-enough category to compare fairly, the usable sample dropped to 13–14% of the original 10,000 for two of the three tests, and to 9% for the nutrition test specifically.

<img width="787" height="421" alt="image" src="https://github.com/user-attachments/assets/f5b26bea-3bee-456d-9695-f6deca9c9c12" />

- **Not causal:** Every result here is a tested association, not a causal claim. Recipes weren't randomly assigned their attributes.
- **No ROI or business-impact claim:** This dataset has no price, cost, or transaction data — nothing here quantifies revenue or savings, and none is claimed.
- **No composite "health score":** Nutrition dimensions were tested individually rather than combined into one score, because any combined score requires choosing weights — and there was no validated basis in this dataset for choosing them. See the [Decision Log](FB_Decision_Log.md) for the full reasoning.
- **A note on "just use more data":** because the ceiling effect and selection bias above are properties of *how people rate on this platform*, not of sample size, simply re-running this analysis on a larger slice of the same dataset likely would not resolve them — see the [Decision Log](FB_Decision_Log.md) and [Assumption Register](FB_Assumption_Register.md) for the full reasoning on why more of the same data isn't the obvious next step.

---

## How the Analysis Was Built

| File | What's in it |
|---|---|
| [`FB_Full_Report.md`](FB_Full_Report.md) | Full phase-by-phase analysis — every number, every decision, every check |
| [`FB_Recipe_Analysis_Notebook.ipynb`](FB_Recipe_Analysis_Notebook.ipynb) | Executable code, verified to reproduce every number in the report |
| [`FB_Decision_Log.md`](FB_Decision_Log.md) | Every branching decision made, the alternative considered, and why it was rejected |
| [`FB_Assumption_Register.md`](FB_Assumption_Register.md) | Every assumption the analysis had to make, and what breaks if it's wrong |

---

## Deep Dive / Technical Details

For readers who want the statistical detail behind the findings above:

- **Why not t-test/ANOVA:** the outcome variable (`AggregatedRating`) has a skew of -2.52 and 71% of values at the ceiling — normal-distribution assumptions are clearly violated, so rank-based tests were used instead.
- **Tests used:** Mann-Whitney U (two-group comparisons, e.g. high-sugar vs. low-sugar within category) and Kruskal-Wallis (category, and three-bucket prep-time comparisons), each run within category and combined across categories using Fisher's method.
- **Multiple testing correction:** Benjamini-Hochberg applied across all 5 tests run (category, prep-time, sugar, sodium, saturated fat) — no result survived correction.
- **Confounders addressed:** review-count reliability (minimum-threshold + multi-threshold robustness), author clustering (~30% of the sample comes from just 145 prolific authors — tested with and without them), and category-level rating-missingness differences (ranged 40–64% across categories, flagged as an unresolved limitation rather than something correctable with a simple covariate).
- **Effect sizes:** rank-biserial r and epsilon-squared, both consistently near zero (e.g. r = -0.003 to -0.033 across nutrition dimensions) rather than merely non-significant — this pattern of near-zero effect size holding steady across every robustness check is part of why the recommendation leans toward "the signal is genuinely weak," not just "underpowered."

<img width="692" height="365" alt="image" src="https://github.com/user-attachments/assets/b065b3b6-fe92-4b72-947f-52049df897bb" />


*Note: epsilon-squared (Category, Prep-time) and rank-biserial r (nutrition dimensions) are different metrics, not directly comparable on the same scale — shown together only to illustrate that every test, regardless of metric, landed near zero rather than near a conventional "small effect" threshold.*

Full statistical reasoning, assumption checks, and every audit correction made during the analysis are documented in the [Full Report](FB_Full_Report.md).

---

## What I Would Do Next

Prioritized by what would actually move the business question forward — not a generic wishlist:

1. **Review-level, not just recipe-level, data** — individual ratings instead of one aggregate per recipe would allow proper modeling of rating variance and reviewer reliability, instead of treating every aggregate rating as equally trustworthy.
2. **Review text** — even a sample of written reviews would let you test *why* people rate recipes the way they do, instead of inferring reasons indirectly.
3. **A rating scale that isn't saturated** — a platform or dataset where ratings aren't 71% clustered at the maximum would give any real effect actual room to show up.
4. **Controlled consumer testing** — for whichever attribute the business cares most about, a small controlled taste-test or survey would produce evidence this dataset structurally cannot.

I would not propose combination/interaction testing (e.g. "sweet + quick + dessert") as a next step — with sample sizes already down to 900–1,400 per single-attribute test, splitting further by combinations would be underpowered before it started.
