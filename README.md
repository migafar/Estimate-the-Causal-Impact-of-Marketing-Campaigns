# Causal Impact of Marketing Campaigns

Most marketing reports answer the wrong question. They tell you how sales moved — not whether the campaign actually caused it. This notebook tries to answer the harder one.

Using a real marketing dataset, I built a full causal inference pipeline that goes from raw data to statistically credible estimates of campaign lift. Not just "sales went up after the campaign." But *how much* of that increase was the campaign, and how confident can we actually be?

---

## What's inside

The analysis runs through five distinct methods, each attacking the same question from a different angle:

**Difference-in-Differences (DiD)** — the backbone. I split cities into treatment and control groups based on campaign exposure, then compared how their sales trajectories diverged after the campaign launched. OLS regression with clustered standard errors handles the inference.

**Parallel Trends Check** — before trusting any DiD result, I verified that treatment and control groups were moving in sync pre-campaign. Slope comparison and visual inspection both. If the trends weren't parallel, the whole setup falls apart.

**Causal Impact (pycausalimpact)** — Google's Bayesian structural time series approach. The model builds a synthetic counterfactual using the control group as a covariate and estimates what would've happened without the campaign. The posterior distribution gives you actual probability intervals, not just point estimates.

**EconML Meta-Learners** — S-Learner, T-Learner, X-Learner, and LinearDML. These estimate *heterogeneous* treatment effects — meaning individual-level campaign impact rather than one average number for everyone. The X-Learner results are particularly interesting: the right tail of the CATE distribution shows a clear cluster of customers who responded much more strongly to the campaign.

**Cluster Bootstrap** — 500 iterations, sampling locations with replacement. Used to validate the DiD percentage estimate and produce honest 95% confidence intervals. If the lower bound crosses zero, the result is probably noise.

---

## The setup

The synthetic treatment effect was set at **+12% lift**, ramping up over 30 days post-launch. This gave me a ground truth to check each method against — useful for understanding which estimators are conservative and which ones overshoot.

Treatment/control split was done geographically by city, based on median campaign engagement. Sales were derived from `Clicks × Conversion_Rate × 50` with a sinusoidal seasonal adjustment baked in.

---

## Stack

```
pycausalimpact
econml
statsmodels
xgboost
pandas / numpy
matplotlib / seaborn
scikit-learn
```

---

## Results summary

All five methods converge on a statistically meaningful positive effect. The DiD OLS estimate recovers something close to the synthetic ground truth. The bootstrap confidence interval doesn't cross zero. The X-Learner and T-Learner both show a right-skewed CATE distribution — most customers show modest lift, but a small segment drives a disproportionate share of the campaign's impact.

Segment-level DiD (by channel and customer type) adds another layer: not all channels perform equally, and the effect size varies noticeably across customer segments. Worth knowing before you scale spend.

---

## Dataset

[Marketing Campaign Performance Dataset](https://www.kaggle.com/datasets/manishabhatt22/marketing-campaign-performance-dataset) — Kaggle

---

## Why this matters

Correlation-based reporting is everywhere in marketing analytics. Causal inference is not. This notebook is a practical example of what it looks like when you actually try to separate signal from coincidence — using tools that were built for that specific job.
