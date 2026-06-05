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

*Coming soon.*

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
