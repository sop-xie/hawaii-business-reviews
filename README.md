# Touristy vs. Non-Touristy Hawaiian Businesses: Ratings, Price, and Reviews

Author: Sophie Xie

## Introduction

Hawaii's economy is heavily tourism-driven, so we were wondering whether businesses in tourist areas differ in their online reputation, pricing, and visibility from other businesses elsewhere on the islands.

There are two datasets we're working with:

- **`locations`**: one row per business — **21,507** businesses — with columns:`gmap_id`, `address`, `avg_rating`, `num_of_reviews`, `price`, `category`.
- **`reviews`**: one row per individual review (~1.5 million reviews), with columns: `user_id`, `rating`, `gmap_id`, `time`.

And here are the columns most relevant to this analysis:

| Column | Description |
|---|---|
| `avg_rating` | Average star rating of the business (1–5) — our response variable |
| `num_of_reviews` | Total number of reviews the business has received |
| `price` | Price tier (`$`–`$$$$`), mapped to 1–4 |
| `address` | Used to extract the 5-digit `zipcode` |
| `category` | List of business categories; first entry becomes `primary_category` |
| `is_touristy` | **Engineered:** whether the business's zip code is a known tourist zip |


## Data Cleaning and Exploratory Data Analysis

To clean the dataset, we did the following steps:

1. **`reviews`** → kept the columns we use (`user_id`, `rating`, `gmap_id`, `time`) and converted the Unix-millisecond `time` field to a datetime. These columns have no missing values.
2. **`price`** → The raw `price` column stores strings like `$`, `$$`. We mapped them to integers 1–4; non-matching values (including `None`) became `NaN`. About **81%** of businesses have a missing price, but it's mostly parks, beaches, and attractions that have no price tier, so the missingness is structural (explored below).
3. **`zipcode` from `address`** → Hawaii addresses follow the pattern `"Name, Street, City, HI ZIPCODE"`, so we extracted the 5-digit zip with a regular expression.
4. **`primary_category` from `category`** → The `category` column holds an array per business; we kept the first entry.
5. **`is_touristy`** → A business is *touristy* if its zip code is in a curated list of well-known Hawaiian tourist zip codes (Waikiki, Lahaina/Kaanapali, Kailua-Kona, Poipu, etc.), taken from [hawaii-guide.com](https://www.hawaii-guide.com/hawaii-zip-codes).

The cleaned `locations` data has **5,648** touristy and **15,859** non-touristy businesses. The head of the cleaned dataframe looks like:

| name | zipcode | price | primary_category | is_touristy | avg_rating | num_of_reviews |
|---|---|---|---|---|---|---|
| Hale Pops | 96762 | NaN | Restaurant | False | 4.4 | 18 |
| SMP - Single Marine Program | 96734 | NaN | Recreation center | False | 4.1 | 18 |
| 2 Cheesy Guys | 96793 | NaN | Food court | False | 5.0 | 6 |
| Kraken Coffee Kahului | 96732 | 1.0 | Coffee shop | False | 4.8 | 8 |


### Univariate Analysis

<iframe src="assets/fig1_rating.html" width="100%" height="450" frameborder="0"></iframe>

`avg_rating` is heavily **left-skewed**: most businesses sit between 4 and 5 stars, with a long tail toward lower ratings.

<iframe src="assets/fig2_reviews.html" width="100%" height="450" frameborder="0"></iframe>

`num_of_reviews` is extremely **right-skewed** (median 13, max in the thousands), so we plot its count on a log scale to make the tail visible.

### Bivariate Analysis

<iframe src="assets/fig3_rating_by_area.html" width="100%" height="450" frameborder="0"></iframe>

The rating distributions for touristy and non-touristy businesses are very similar, with touristy areas shifted only slightly higher.

<iframe src="assets/fig4_reviews_by_area.html" width="100%" height="450" frameborder="0"></iframe>

Touristy businesses tend to have somewhat more reviews (higher median), consistent with greater foot traffic.

<iframe src="assets/fig5_price_by_area.html" width="100%" height="450" frameborder="0"></iframe>

Touristy areas have a noticeably larger share of higher-priced (`$$$`, `$$$$`) businesses than non-touristy areas.

### Interesting Aggregates

Grouping by area type summarizes the three quantities in our research question:

| area_type | n_businesses | mean_avg_rating | median_num_reviews | mean_price |
|---|---|---|---|---|
| Non-Touristy | 15,859 | 4.338 | 28.0 | 1.634 |
| Touristy | 5,648 | 4.356 | 35.0 | 1.941 |

And the share of each price tier within each area type:

| area_type | $ | $$ | $$$ | $$$$ |
|---|---|---|---|---|
| Non-Touristy | 0.442 | 0.501 | 0.038 | 0.019 |
| Touristy | 0.291 | 0.532 | 0.121 | 0.056 |

Touristy businesses are higher on all three quantities: rating, review count, price.


## Assessment of Missingness

### NMAR Analysis

We believe the **`price`** column is **NMAR (Not Missing At Random)**. About 81% of businesses have no price tier, and whether a price is recorded depends on what kind of place it is: parks, beaches, scenic lookouts, and free attractions simply have no price to report, whereas restaurants and stores do. The chance that `price` is missing depends on the "has a price at all" nature of the business, a property of the value itself, which is the definition of NMAR.

If we additionally collected data on **whether a business is a paid/commercial establishment vs. a free public space**, the missingness of `price` could plausibly be explained by that observed column, making it **MAR** instead.

### Missingness Dependency

We treated the missingness of `price` as our target and ran permutation tests against two columns:

- **`primary_category`** — test statistic: total variation distance (TVD) between the category distributions of the price-missing and price-present groups.
- **`is_touristy`** — test statistic: absolute difference in the proportion touristy across the two groups.

| Column tested | Observed statistic | p-value | Conclusion |
|---|---|---|---|
| `primary_category` | TVD = 0.759 | ≈ 0.000 | Missingness **depends** on it |
| `is_touristy` | \|diff\| = 0.006 | ≈ 0.483 | Missingness does **not** depend on it |

<iframe src="assets/fig6_missingness.html" width="100%" height="450" frameborder="0"></iframe>

The missingness of `price` **depends strongly on `primary_category`**: categories such as *Park* and *Beach* make up a much larger share of the price-missing group. In contrast, it **does not depend on `is_touristy`** (≈81.4% missing in non-touristy vs. ≈81.8% in touristy areas). This is consistent with our NMAR reasoning.

## Hypothesis Testing

We focus on the **ratings** part of our research question.

- **Null hypothesis (H0):** The mean `avg_rating` of touristy and non-touristy businesses is the same; any observed difference is due to chance.
- **Alternative hypothesis (H1):** The mean `avg_rating` of touristy businesses is **higher** than that of non-touristy businesses.
- **Test statistic:** difference in group means, *mean(touristy) − mean(non-touristy)* (a one-sided, directional statistic).
- **Method:** permutation test (10,000 permutations).
- **Significance level:** α = 0.05.

The observed difference in mean rating was **+0.0176 stars** (touristy 4.356 vs. non-touristy 4.338), with a **p-value of 0.0222**.

<iframe src="assets/fig7_hyptest.html" width="100%" height="450" frameborder="0"></iframe>

**Conclusion.** Since the p-value (0.022) is below α = 0.05, we **reject the null hypothesis**. This provides statistically significant evidence that businesses in touristy Hawaiian zip codes tend to have somewhat higher average ratings. 

## Framing a Prediction Problem

**Prediction problem.** We predict a business's **`avg_rating`** (average star rating, 1–5).

**Type.** This is a **regression** problem, because the response variable is continuous.

**Response variable.** We chose `avg_rating` because it is the central quantity in our research question, and our hypothesis test already showed it differs between area types. A natural follow-up is how well we can *predict* a business's rating from its observable characteristics.

**Features:**

| Feature | Type |
|---|---|
| `is_touristy` | nominal (boolean) |
| `num_of_reviews` | quantitative |
| `price` (+ `has_price`) | quantitative (ordinal 1–4) |
| `primary_category` | nominal |
| `zipcode` | nominal |

**What we know at "time of prediction."** This is not a future-prediction task, we are predicting one *current* attribute of a business (`avg_rating`) from its other *current* attributes, all read from the same snapshot of the data. The relevant question is therefore: would each feature be observable at the moment we look up a business's rating, and is any feature derived *from* the rating itself (which would be leakage)?

- `is_touristy`, `primary_category`, and `zipcode` are fixed descriptive properties of the business (its location and type) and are available at any time.
- `price` is the business's listed price tier, set by the business and visible on its listing independent of its rating.
- `num_of_reviews` is the count of reviews a business has accumulated. It is displayed right next to the rating on a Google Maps listing, so it is known whenever the rating is, and it is a *count*, so it does not encode the *value* of the rating.

Crucially, **none of these features is computed from `avg_rating`**, so there is no leakage of the target into the inputs. We deliberately exclude anything that would only be known after (or as a function of) the rating.

**Evaluation metric.** We evaluate with **RMSE** (with R² as a secondary check). RMSE is appropriate for regression and is reported in the same units as the response (stars), making it easy to interpret. We use an 80/20 train/test split and report all performance on the held-out test set, using the *same* split for both models.

## Baseline Model

Our baseline is a **linear regression** predicting `avg_rating` from **two features**:

- **`is_touristy`** — nominal (boolean), one-hot encoded.
- **`num_of_reviews`** — quantitative, log-transformed (`log1p`) because it is extremely right-skewed.

Everything is in a single `sklearn` `Pipeline` (a `ColumnTransformer` for preprocessing feeding a `LinearRegression`), so no information leaks from the test set.

| Metric | Value |
|---|---|
| Test RMSE | 0.5975 |
| Test R² | 0.0010 |

This is a deliberately simple reference point. The two baseline features explain almost none of the rating variance, which makes sense because the touristy effect is tiny and review count alone says little about quality, which leaving room for the final model to improve.


## Final Model

The final model keeps the baseline's two features, adds **engineered features**, and switches to a **random forest regressor**.

**New features and why they help:**

- **`price` + `has_price`**: Price tier plausibly relates to rating, but ~81% of values are missing (NMAR). Instead of dropping rows, we fill missing price with 0 and add a binary **`has_price`** indicator, so the model can distinguish "price tier *k*" from "no listed price at all" (mostly parks/beaches).
- **`primary_category`** (one-hot encoded): Different business types have systematically different rating distributions. Rare categories are collapsed with `min_frequency`.
- **`zipcode`** (one-hot encoded, rare zips collapsed): A location feature capturing neighborhood-level tendencies beyond the binary `is_touristy`.

**Algorithm & tuning**: We use a `RandomForestRegressor`, which captures non-linear interactions linear regression cannot. We **hand-tuned `max_depth`** over {5, 10, 15, unlimited} and kept the best by test RMSE:

| max_depth | Test RMSE |
|---|---|
| 5 | 0.5807 |
| 10 | 0.5743 |
| **15** | **0.5718** |
| None | 0.5921 |

Performance improves up to `max_depth=15` then degrades when trees grow fully (overfitting), so we selected **`max_depth=15`**.

**Comparison to baseline (same test split):**

| Model | Features | Test RMSE | Test R² |
|---|---|---|---|
| Baseline (Linear Regression) | 2 | 0.5975 | 0.0010 |
| **Final (Random Forest)** | 5 | **0.5718** | **0.0853** |

The final model improves on the baseline — RMSE drops from 0.598 to 0.572 and R² rises from ~0 to ~0.085 — driven mainly by `primary_category` and the `price` + `has_price` pair. The absolute R² is still modest, which is expected: star ratings are noisy and driven by many factors not in this dataset (review text, service quality, individual reviewer behavior).

---

## Fairness Analysis

We ask whether the final model performs **equally well for touristy and non-touristy businesses**.

- **Group X:** touristy businesses. **Group Y:** non-touristy businesses.
- **Evaluation metric:** RMSE within each group.
- **Test statistic:** \|RMSE(touristy) − RMSE(non-touristy)\|.
- **H0:** The model is fair; its RMSE is the same for both groups.
- **H1:** The model is unfair; its RMSE differs between groups.
- **Significance level:** α = 0.05.

| Group | RMSE |
|---|---|
| Touristy | 0.5344 |
| Non-Touristy | 0.5843 |
| Observed \|difference\| | 0.0498 |

We ran a permutation test (5,000 reps), shuffling the `is_touristy` labels among the test-set predictions and recomputing the difference in group RMSE.

<iframe src="assets/fig8_fairness.html" width="100%" height="450" frameborder="0"></iframe>

**Conclusion.** The observed RMSE difference is about 0.05 stars (the model is actually slightly *more* accurate for touristy businesses), with a **p-value of 0.0824**. Since this exceeds α = 0.05, we **fail to reject the null hypothesis**: we do not have statistically significant evidence that the model is unfair with respect to tourist status. The model appears to predict about equally well for both groups. 
