# Onboarding Experiment Investigation

## Summary

The dataset contains 14,000 signup users across five acquisition segments and two onboarding variants. I first checked data quality, then compared conversion rates overall and within each segment, calculated a population-mix-adjusted lift, and finally checked whether treatment assignment was balanced across segments.

## Question 1 — Overall (naive) lift

| Variant | Users | Conversions | Conversion rate |
|---|---:|---:|---:|
| Control | 7,136 | 1,414 | 19.815022% |
| Treatment | 6,864 | 1,814 | 26.427739% |

Naive lift:

`26.4277389277% - 19.8150224215% = +6.6127165062 percentage points`

**Answer: +6.6127165062 percentage points**, with **7,136 control users** and **6,864 treatment users**.

## Question 2 — Segment-level comparison

| Segment | Control n | Control CR | Treatment n | Treatment CR | Lift |
|---|---:|---:|---:|---:|---:|
| app_store | 925 | 8.7568% | 960 | 20.0000% | +11.2432 pp |
| influencer | 119 | 23.5294% | 131 | 16.7939% | -6.7355 pp |
| organic | 1,298 | 35.2851% | 2,917 | 35.0703% | -0.2148 pp |
| paid_search | 3,353 | 15.1804% | 1,459 | 14.3934% | -0.7870 pp |
| referral | 1,441 | 23.4559% | 1,397 | 26.2706% | +2.8146 pp |

The segment whose estimate I would **not trust as evidence of a real effect is influencer**. Its observed treatment-control difference is -6.7355 percentage points, but the segment contains only 250 users in total (119 control and 131 treatment). Its 95% confidence interval is approximately **-16.69 to +3.22 percentage points**, so it is wide and includes zero. Thus, the large magnitude of the observed difference is not sufficient evidence of a real treatment effect.

For reference, the other 95% confidence intervals were approximately:
- app_store: **+8.13 to +14.36 pp**
- organic: **-3.34 to +2.91 pp**
- paid_search: **-2.96 to +1.39 pp**
- referral: **-0.37 to +5.99 pp**

## Question 3 — Mix-adjusted overall lift

For each segment, I calculated:

`segment lift × (segment users / 14,000)`

| Segment | Users | Population share | Segment lift | Weighted contribution |
|---|---:|---:|---:|---:|
| app_store | 1,885 | 13.4643% | +11.2432 pp | +1.5138 pp |
| influencer | 250 | 1.7857% | -6.7355 pp | -0.1203 pp |
| organic | 4,215 | 30.1071% | -0.2148 pp | -0.0647 pp |
| paid_search | 4,812 | 34.3714% | -0.7870 pp | -0.2705 pp |
| referral | 2,838 | 20.2714% | +2.8146 pp | +0.5706 pp |

Summing the weighted contributions gives:

**Mix-adjusted overall lift = +1.63 percentage points**  
(exact calculation: **+1.6289429306 pp**)

This differs from the naive +6.6127 pp because treatment and control do not have the same segment composition. In particular, treatment contains a much larger share of organic users and a much smaller share of paid-search users, and those segments have different baseline conversion rates. The mix-adjusted calculation puts both variants on the same overall population mix, separating segment-composition effects from the within-segment treatment differences.

## Question 4 — Segment with a real, meaningful positive effect

**app_store**

The evidence is:
- Control conversion rate: **8.7568%**
- Treatment conversion rate: **20.0000%**
- Lift: **+11.2432 percentage points**
- Total segment sample: **1,885 users**
- 95% confidence interval for the lift: **+8.13 to +14.36 percentage points**

The confidence interval remains above zero, and the segment has substantially more observations than the influencer segment. This makes app_store the segment with the clearest evidence of a meaningful positive treatment effect in this dataset.

The referral segment also has a positive point estimate (+2.8146 pp), but its 95% confidence interval (**-0.37 to +5.99 pp**) includes zero, so the evidence is less conclusive.

## Question 5 — Treatment assignment by segment

| Segment | Control | Treatment | Control % | Treatment % |
|---|---:|---:|---:|---:|
| app_store | 925 | 960 | 49.07% | 50.93% |
| influencer | 119 | 131 | 47.60% | 52.40% |
| organic | 1,298 | 2,917 | **30.79%** | **69.21%** |
| paid_search | 3,353 | 1,459 | **69.68%** | **30.32%** |
| referral | 1,441 | 1,397 | 50.78% | 49.22% |

The overall experiment is close to 50/50, but assignment is strongly imbalanced within two segments: organic is 69.21% treatment, while paid_search is only 30.32% treatment. Under a 50/50-per-segment assignment assumption, these deviations are far too large to look like ordinary random variation; the other three segments are close to 50/50. This assignment imbalance is important because organic and paid-search users have very different baseline conversion rates and therefore can materially affect the naive overall comparison.

A binomial test against a 50% treatment probability gave extremely small two-sided p-values for organic and paid_search, reinforcing that the observed splits would be highly unusual under that specific 50/50 assumption. This does not by itself prove that the experiment was not randomized, because the actual assignment mechanism is not provided; it does establish that the treatment/control mix is highly segment-dependent.

## Investigation process

- Loaded the CSV with pandas and verified the dataset shape: 14,000 rows and 4 columns.
- Checked for missing values; none were present in the data used for the analysis.
- Checked for duplicate rows and duplicate user IDs; no duplicates were found.
- Validated the allowed segment, variant, and conversion values.
- Calculated the overall treatment/control conversion rates and the naive lift.
- Broke conversion rates down by acquisition segment and calculated treatment-control lifts.
- Checked confidence intervals for the segment-level differences to distinguish large point estimates from well-supported effects.
- Initially, the overall +6.61 pp result looked like the main conclusion, but this was a dead end because it ignored the different segment composition of treatment and control.
- Calculated the mix-adjusted lift using each segment's share of the full 14,000-user population.
- Checked treatment/control assignment shares within each segment and tested the observed deviations from a 50/50 allocation assumption.
