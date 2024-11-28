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

To enable analysis of the data, I took multiple steps to clean the given datasets.

1. Obtaining average rating for each recipe

The recipes dataset contains unique recipes, while the interactions dataset contains user reviews. To obtain the average rating for each recipe, I had to match reviews to recipes based on the `id` column in recipes and the `recipe_id` column in interactions.

To do this, I first left merged the recipes and interactions datasets together. This meant that all recipes were maintained and interactions were only added if the corresponding recipe existed in recipes.

I filled all ratings of 0 with np.nan. When we fill out reviews on websites, we are usually given a scale from 1-5. Thus, a rating of 0 likely represents a missing value where a user did not submit a rating.

I then grouped ratings by recipe id and calculated the mean, ultimately obtaining the average rating for each recipe. I added this data as a new column titled `rating` in the recipes dataset.

2. Adding `chicken_in_ingredients` to recipes

In order to easily check whether or not chicken was an ingredient, I decided to create a `chicken_in_ingredients` column.

The `ingredients` column in recipes was initially stored as a string representation of a Python list. Thus, a simple string-based check helped me quickly generate a `chicken_in_ingredients` column which checked whether or not the word 'chicken' was in `ingredients` (True if 'chicken' was a recipe ingredient, False if otherwise).

I then added this column to the recipes dataset.

3. Adding `chicken_in_name` to recipes

I took a similar approach to adding `chicken_in_name` to recipes in order to provide another opportunity to understand the different relationships between presence of chicken and recipe ratings.

To implement the column, I did another string-based search for the phrase 'chicken' in the `name` column. The resulting column contained True if 'chicken' was present in the recipe's name and False if otherwise. I added this `chicken_in_name` column to the recipes dataset.

4. Adding `chicken_in_description` to recipes

Like the previous two steps, I did a string-based search to check if the phrase 'chicken' was present in the `description` column of the recipes dataset. I added the resulting `chicken_in_description` column to the recipes dataset.

5. Converting the type of `submitted`

The `submitted` column in the recipes dataset was initially stored as a string. In order to enable time-based analysis, I converted the `submitted` column from string values to datetime values.

6. Creating `year` and `month` columns

After converting the `submitted` column into datetime format, I created `year` and `month` columns to enable time-based analysis. For instance, chicken (not traditionally a Thanksgiving meal) may be less popular in November compared to other recipes involving turkey, pumpkin pie, or mashed potatoes.

7. Converting `tags`, `steps`, and `ingredients` from strings to lists

The `tags`, `steps`, and `ingredients` columns in the recipes dataset were initially stored as string representations of Python lists. For simplicity, I converted these into lists.

8. Explanding the `nutrition` column

The `nutrition` column initially contains, calories, total fat, sugar, sodium, protein, saturated fat, and carbohydrates. Each value was stored as a string representation of a Python list containing the nutrition information.

To extract this information, I first converted the strings into Python lists. I then stored each value within the list as a float and matched it to the corresponding nutrition label (such as calories or total fat).

I then added these columns (`calories`, `total_fat`, `sugar`, `sodium`, `protein`, `saturated_fat`, and `carbohydrates`) to the recipes dataset.

My final cleaned dataset contained 83,782 rows and 24 columns.

Here is a preview of the first 5 rows. Note that some columns have been removed as their values were too large to be effectively displayed on this site.

<div style="overflow-x:auto;">
| name                                 |     id | submitted           |   n_steps |   n_ingredients |   rating | chicken_in_ingredients   | chicken_in_name   | chicken_in_description   |   year |   month |   calories |   total_fat |   sugar |   sodium |   protein |   saturated_fat |   carbohydrates |
|:-------------------------------------|-------:|:--------------------|----------:|----------------:|---------:|:-------------------------|:------------------|:-------------------------|-------:|--------:|-----------:|------------:|--------:|---------:|----------:|----------------:|----------------:|
| 1 brownies in the world    best ever | 333281 | 2008-10-27 00:00:00 |        10 |               9 |        4 | False                    | False             | False                    |   2008 |      10 |      138.4 |          10 |      50 |        3 |         3 |              19 |               6 |
| 1 in canada chocolate chip cookies   | 453467 | 2011-04-11 00:00:00 |        12 |              11 |        5 | False                    | False             | False                    |   2011 |       4 |      595.1 |          46 |     211 |       22 |        13 |              51 |              26 |
| 412 broccoli casserole               | 306168 | 2008-05-30 00:00:00 |         6 |               9 |        5 | True                     | False             | False                    |   2008 |       5 |      194.8 |          20 |       6 |       32 |        22 |              36 |               3 |
| millionaire pound cake               | 286009 | 2008-02-12 00:00:00 |         7 |               7 |        5 | False                    | False             | False                    |   2008 |       2 |      878.3 |          63 |     326 |       13 |        20 |             123 |              39 |
| 2000 meatloaf                        | 475785 | 2012-03-06 00:00:00 |        17 |              13 |        5 | False                    | False             | False                    |   2012 |       3 |      267   |          30 |      12 |       12 |        29 |              48 |               2 |
</div>

## Univariate Analysis

## Bivariate Analysis

## Interesting Aggregates

# Assessment of Missingness
# Hypothesis Testing
# Framing a Prediction Problem
# Baseline Model
# Final Model
# Fairness Analysis
