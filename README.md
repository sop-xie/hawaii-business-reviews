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

We believe the **`price`** column is **NMAR (Not Missing At Random)**. About 81% of businesses have no price tier, and whether a price is recorded depends on what kind of place it is: parks, beaches, scenic lookouts, and free attractions simply have no price to report, whereas restaurants and stores do. The chance that `price` is missing depends on the "has a price at all" nature of the business — a property of the value itself — which is the definition of NMAR.

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

The missingness of `price` **depends strongly on `primary_category`**: categories such as *Park* and *Beach* make up a much larger share of the price-missing group. In contrast, it **does not depend on `is_touristy`** (≈81.4% missing in non-touristy vs. ≈81.8% in touristy areas). This is consistent with our NMAR reasoning — whether a price is recorded is driven by *what kind of business it is*, not where it is — and means price comparisons between the two area types are not biased by differential missingness.


## Hypothesis Testing

*Coming soon.*

## Framing a Prediction Problem

*Coming soon.*

## Baseline Model

*Coming soon.*

## Final Model

*Coming soon.*

## Fairness Analysis

*Coming soon.*
