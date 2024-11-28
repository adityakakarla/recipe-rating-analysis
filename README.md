# The Poultry Effect: Understanding the Relationship Between Chicken and Recipe Ratings

Author: Aditya Kakarla

# Overview

In this project, I will explore the relationship between the presence of chicken in a recipe (either as text or as an ingredient) and the rating of a recipe.

# Introduction

Everyone loves food. On top of being a basic necessity, food provides the opportunity to explore cultures all over the world. In particular, staples such as chicken can take you on a culinary journey. From India's butter chicken to Japan's katsu, chicken can take you around the world.

According to the USDA's Economy Research Service, chicken leads US per person availability of meat over the past decade. Thus, it is critical to understand the relationship between chicken presence and average rating.

In particular, I investigated the question **"Is there a relationship between presence of chicken (in name, description, or ingredients) and average rating?"**

To investigate this question I will look at two datasets: recipes and interactions. The dataset contains recipes and ratings from food.com. It was originally scraped and used by the authors of [this paper](https://cseweb.ucsd.edu/~jmcauley/pdfs/emnlp19c.pdf)

The first dataset, **recipes**, contains **83,782 rows**. Each row is a specific recipe. It has **12 columns** as detailed below:

| Column          | Description                                                                                           |
|------------------|-------------------------------------------------------------------------------------------------------|
| `name`          | Recipe name                                                                                          |
| `id`            | Recipe ID                                                                                            |
| `minutes`       | Minutes to prepare recipe                                                                            |
| `contributor_id`| User ID who submitted this recipe                                                                     |
| `submitted`     | Date the recipe was submitted                                                                         |
| `tags`          | Food.com tags for the recipe                                                                          |
| `nutrition`     | Nutrition information in the form `[calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]`; PDV stands for “percentage of daily value” |
| `n_steps`       | Number of steps in the recipe                                                                         |
| `steps`         | Text for recipe steps, in order                                                                       |
| `description`   | User-provided description                                                                             |
| `ingredients`   | Text for recipe ingredients                                                                           |
| `n_ingredients` | Number of ingredients in the recipe                                                                   |

---

The second dataset, **interactions**, contains **731,927 rows**. Each row is a review a user review for a specific recipe. The columns included are:

| Column     | Description          |
|------------|----------------------|
| `user_id`  | User ID              |
| `recipe_id`| Recipe ID            |
| `date`     | Interaction Date     |
| `rating`   | Rating               |
| `review`   | Review text          |

The columns most relevant to my question are "name", "description", and "ingredients" in recipes, as well as "rating" in interactions.

# Data Cleaning and Exploratory Data Analysis

## Data Cleaning

merged to get avg rating
got out chicken in ingrdients
suchicken in name
chicken in description
submitted
year
month

convert tags, steps, and ingredients into list to string

convert nutrition into calories, total-fat, sugar, sodium, protein, saturated_fat, and carbohydrates

## Univariate Analysis

## Bivariate Analysis

## Interesting Aggregates

# Assessment of Missingness
# Hypothesis Testing
# Framing a Prediction Problem
# Baseline Model
# Final Model
# Fairness Analysis
