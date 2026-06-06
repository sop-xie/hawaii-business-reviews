# Touristy vs. Non-Touristy Hawaiian Businesses: Ratings, Price, and Reviews

Author: Sophie Xie

## Introduction

Hawaii's economy is heavily tourism-driven, so we were wondering whether businesses in tourist areas differ in their online reputation, pricing, and visibility from other businesses elsewhere on the islands.

- **`locations`**: one row per business — **21,507** businesses — with columns:`gmap_id`, `address`, `avg_rating`, `num_of_reviews`, `price`, `category`.
- **`reviews`**: one row per individual review (~1.5 million reviews), with columns: `user_id`, `rating`, `gmap_id`, `time`.

The columns most relevant to this analysis:

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

## Assessment of Missingness

*Coming soon.*

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
