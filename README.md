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

1. Obtaining average (mean) rating for each recipe
- The recipes dataset contains unique recipes, while the interactions dataset contains user reviews. To obtain the average rating for each recipe, I had to match reviews to recipes based on the `id` column in recipes and the `recipe_id` column in interactions.
- To do this, I first left merged the recipes and interactions datasets together. This meant that all recipes were maintained and interactions were only added if the corresponding recipe existed in recipes.
- I filled all ratings of 0 with np.nan. When we fill out reviews on websites, we are usually given a scale from 1-5. Thus, a rating of 0 likely represents a missing value where a user did not submit a rating.
- I then grouped ratings by recipe id and calculated the mean, ultimately obtaining the average rating for each recipe. I added this data as a new column titled `rating` in the recipes dataset.
2. Adding `chicken_in_ingredients` to recipes
- In order to easily check whether or not chicken (or any chicken-based food) was an ingredient, I decided to create a `chicken_in_ingredients` column.
- The `ingredients` column in recipes was initially stored as a string representation of a Python list. Thus, a simple string-based check helped me quickly generate a `chicken_in_ingredients` column which checked whether or not the word 'chicken' was in `ingredients` (True if 'chicken' was a recipe ingredient, False if otherwise).
- I then added this column to the recipes dataset.
3. Adding `chicken_in_name` to recipes
- I took a similar approach to adding `chicken_in_name` to recipes in order to provide another opportunity to understand the different relationships between presence of chicken and recipe ratings.
- To implement the column, I did another string-based search for the phrase 'chicken' in the `name` column. The resulting column contained True if 'chicken' was present in the recipe's name and False if otherwise. I added this `chicken_in_name` column to the recipes dataset.
4. Adding `chicken_in_description` to recipes
- Like the previous two steps, I did a string-based search to check if the phrase 'chicken' was present in the `description` column of the recipes dataset. I added the resulting `chicken_in_description` column to the recipes dataset.
5. Converting the type of `submitted`
- The `submitted` column in the recipes dataset was initially stored as a string. In order to enable time-based analysis, I converted the `submitted` column from string values to datetime values.
6. Creating `year` and `month` columns
- After converting the `submitted` column into datetime format, I created `year` and `month` columns to enable time-based analysis. For instance, chicken (not traditionally a Thanksgiving meal) may be less popular in November compared to other recipes involving turkey, pumpkin pie, or mashed potatoes.
7. Converting `tags`, `steps`, and `ingredients` from strings to lists
- The `tags`, `steps`, and `ingredients` columns in the recipes dataset were initially stored as string representations of Python lists. For simplicity, I converted these into lists.
8. Explanding the `nutrition` column
- The `nutrition` column initially contains, calories, total fat, sugar, sodium, protein, saturated fat, and carbohydrates. Each value was stored as a string representation of a Python list containing the nutrition information.
- To extract this information, I first converted the strings into Python lists. I then stored each value within the list as a float and matched it to the corresponding nutrition label (such as calories or total fat).
- I then added these columns (`calories`, `total_fat`, `sugar`, `sodium`, `protein`, `saturated_fat`, and `carbohydrates`) to the recipes dataset.

My final cleaned dataset contained 83,782 rows and 24 columns.

Here is a preview of the first 5 rows. It is important to note that some columns have been removed as their values were too large to be effectively displayed on this site.

| name                                 |     id | submitted           |   n_steps |   n_ingredients |   rating | chicken_in_ingredients   | chicken_in_name   | chicken_in_description   |   year |   month |   calories |   total_fat |   sugar |   sodium |   protein |   saturated_fat |   carbohydrates |
|:-------------------------------------|-------:|:--------------------|----------:|----------------:|---------:|:-------------------------|:------------------|:-------------------------|-------:|--------:|-----------:|------------:|--------:|---------:|----------:|----------------:|----------------:|
| 1 brownies in the world    best ever | 333281 | 2008-10-27 00:00:00 |        10 |               9 |        4 | False                    | False             | False                    |   2008 |      10 |      138.4 |          10 |      50 |        3 |         3 |              19 |               6 |
| 1 in canada chocolate chip cookies   | 453467 | 2011-04-11 00:00:00 |        12 |              11 |        5 | False                    | False             | False                    |   2011 |       4 |      595.1 |          46 |     211 |       22 |        13 |              51 |              26 |
| 412 broccoli casserole               | 306168 | 2008-05-30 00:00:00 |         6 |               9 |        5 | True                     | False             | False                    |   2008 |       5 |      194.8 |          20 |       6 |       32 |        22 |              36 |               3 |
| millionaire pound cake               | 286009 | 2008-02-12 00:00:00 |         7 |               7 |        5 | False                    | False             | False                    |   2008 |       2 |      878.3 |          63 |     326 |       13 |        20 |             123 |              39 |
| 2000 meatloaf                        | 475785 | 2012-03-06 00:00:00 |        17 |              13 |        5 | False                    | False             | False                    |   2012 |       3 |      267   |          30 |      12 |       12 |        29 |              48 |               2 |

## Univariate Analysis

First, I explored the distribution of average (mean) recipe ratings to see if there was any pattern in the data.

<iframe
  src="assets/recipe_rating_distribution.html"
  width="700"
  height="425"
  frameborder="0"
></iframe>

Based on the plot, the distributions appears to be skewed to the left, with many recipes having an average rating close to or at 5.0. What is also interesting to note is the presence of small clusters around integers (1, 2, 3, 4, and 5), which could mean that many recipes only receive one rating (thus keeping the mean rating at an integer value).

## Bivariate Analysis

I also explored the relationship between the presence of chicken in a recipe and its rating. Specifically, I took a look at the distribution of average recipe ratings conditioned between recipes with 'chicken' in the name and recipes without 'chicken' in the name.

<iframe
  src="assets/average_rating_by_chicken_in_name.html"
  width="700"
  height="425"
  frameborder="0"
></iframe>

Based on the box plots, it looks like there may be a difference: the distribution for recipes with 'chicken' in their name has smaller values for both the first quartile and lower whisker. 

## Interesting Aggregates

I continued to explore the relationship between the presence of 'chicken' in a recipe's name and its rating. Below, I created a pivot table containing the mean of each recipe's average rating grouped by whether or not 'chicken' is in the recipe's name.

| chicken_in_name   |   rating |
|:------------------|---------:|
| False             |  4.62855 |
| True              |  4.59805 |

Based on this pivot table, recipes with 'chicken' in the name are rated around 0.03 less than recipes without 'chicken' in the name on average. Later on, I will explore the significance of this observation.

# Assessment of Missingness

My initial merged dataset (the recipes dataset after step 1 of the data cleaning process) contained three columns with missing data: `name`, `description`, and `rating`. Because `chicken_in_name` and `chicken_in_description` were derived from these columns, they are also missing data.

## NMAR Analysis
I believe that the missing data in the `rating` column is not missing at random (NMAR). People who do not feel strongly about a recipe may decide not to leave a review. By contrast, people who love or hate the recipe will feel more inclined to leave a rating on the website to show their feelings.

## Missingness Dependency

To explore further relationships within the data, I look at the missingness of the `rating` column in the dataset. Specifically, I looked at whether or not the missingness of `rating` is dependent on two other columns: `n_steps` (number of steps in a recipe) and `sodium` (PDV—percent daily value).

### Missigness of `rating` based on `n_steps`

**Null Hypothesis**: The missingness of `rating` does not depend on `n_steps`

**Alternate Hypothesis**: The missingness of `rating` does depend on `n_steps`

**Statistic**: The absolute difference between the mean of `n_steps` among recipes with missing ratings and the the mean of `n_steps` among recipes without missing ratings.

**Significance Level**: α = 0.05

To conduct this permutation test, I created a column to indicate whether or not `rating` is missing. First, I calculated the observed test statistic by calculating the difference in means of `n_steps` between the two groups (`rating` present and `rating` missing). Then, I shuffled the missing indicator column 1000 times and recalculated the difference in means.

To determine the p-value, I looked at how many sampled differences in means were as extreme or more extreme than the observed statistic.

<iframe
  src="assets/mar_perm_test_mean_diff_rating_n_steps.html"
  width="700"
  height="425"
  frameborder="0"
></iframe>

None of the 1000 permuted differences were as extreme as the observed difference. Thus, the p-value is 0.0.

As a result, we reject the null hypothesis. Our test suggests that the missigness of `rating` is dependent on `n_steps`.

### Missigness of `rating` based on `sodium`

**Null Hypothesis**: The missingness of `rating` does not depend on `sodium`

**Alternate Hypothesis**: The missingness of `rating` does depend on `sodium`

**Statistic**: The absolute difference between the mean of `sodium` among recipes with missing ratings and the the mean of `sodium` among recipes without missing ratings.

**Significance Level**: α = 0.05

To conduct this permutation test, I created a column to indicate whether or not `rating` is missing. First, I calculated the observed test statistic by calculating the difference in means of `sodium` between the two groups (`rating` present and `rating` missing). Then, I shuffled the missing indicator column 1000 times and recalculated the difference in means.

To determine the p-value, I looked at how many sampled differences in means were as extreme or more extreme than the observed statistic.

<iframe
  src="assets/mar_perm_test_mean_diff_rating_sodium.html"
  width="700"
  height="425"
  frameborder="0"
></iframe>

Some of the 1000 permuted differences in means fit the above criteria. Specifically, my permutation test yielded a p-value of 0.874.

Thus, we fail to reject the null. My test results do not provide sufficient evidence to support the idea that the missingness of `rating` is dependent on `sodium`.

# Hypothesis Testing

**Null Hypothesis**: The mean rating of recipes with chicken in the name is equal to the mean rating of all recipes.

I chose this null hypothesis because `chicken_in_name` is the best indicator of a chicken-based recipe (other recipes with chicken in the description or as ingredients may not be primarily chicken-based).

**Alternate Hypothesis**: The mean rating of recipes with chicken in the name is not equal to the mean rating of all recipes.

I chose this alternate hypothesis because we are interested in whether or not chicken-based recipes deviate from the population, not necessarily in a specific direction.

**Statistic**: Absolute difference between mean rating of samples with chicken in the name and all samples.

Because I am conducting a two-sided test, I chose to use the absolute difference in means.

**Significance Level**: α = 0.05

I chose 0.05 as the significance level because a Type-1 error (rejecting the null hypothesis when it is actually true) is not particularly harmful in our case.

To conduct this hypothesis test, I first calculated the difference in means between just chicken-based recipes and all recipes. I then created 1000 bootstrapped samples from the overall dataset. I evaluated the absolute difference between the sample mean and the overall mean and stored these in an array.

<iframe
  src="assets/hypothesis_test.html"
  width="700"
  height="425"
  frameborder="0"
></iframe>

Following these steps, 0 out of 1000 observations were as extreme as my observed statistic.

Thus, the p-value is 0.0. We reject the null hypothesis. The test suggests that the mean rating of recipes with chicken in the name is not equal to the mean rating of all recipes.

# Framing a Prediction Problem

I will be attempting to predict the average rating of a recipe using regression. I chose `rating` because it is the most well-defined metric for how "good" a recipe is. Additionally, the average rating of a recipe is something we know in the future, while we know other critical information (such as `n_steps`, `n_ingredients`, and most of the other columns in the dataset) at the time of recipe submission.

To evaluate my regression model, I have chosen to utilize mean squared error (MSE). The distribution of ratings is heavily skewed, with most recipes having an average rating of around 5. However, a model simply predicting 5 would not be useful for someone interested in seeing a predicted rating for their recipe. Therefore, it is important to penalize large deviations, making MSE an ideal evaluation metric. I will also utilize the r^2 score to better understand the effectiveness of the model.

# Baseline Model

For my baseline model, I utilized a random forest regressor with default hyperparameters.

First, I dropped any rows containing missing data in any column (for either my baseline or final model). I then split the data into a training set and a test set.

For the baseline model, I utilized three features: `chicken_in_name` (nominal), `chicken_in_ingredients` (nominal), and `minutes` (quantitative).

I one hot encoded `chicken_in_name` and `chicken_in_ingredients`. Because both of these columns only contain either True or False values, I dropped one of the encoded columns. I also utilized a quantile transformer to modify `minutes` due to the presence of extremely larger outliers in the distribution of `minutes`.

The mean squared error (MSE) for my test data is 0.408. Without another MSE metric for comparison, I chose to calculate the r^2 value for my predictions. This came out to a value of -0.00896. This means that the model is likely a poor representation of the data, because such a value means that a model based on a single constant would perform better.

# Final Model

## Features

In my final model, I utilized the following features:

1. `chicken_in_name`
- This feature tells us whether or not the phrase 'chicken' is in the name of a recipe. Based on a pivot table I had constructed, I observed higher ratings for recipes without 'chicken' in the name compared to recipes with 'chicken' in the name. Thus, I felt this could be a useful trend I could capture by one hot encoding the boolean values in the `chicken_in_name` column.
2. `chicken_in_ingredients`
- This feature tells us whether or not chicken or any chicken-related product is an ingredient of a recipe. Based on a pivot table I had constructed, I observed higher ratings for recipes without chicken-based products as an ingredient compared to recipes with chicken-based products. Thus, I felt this could be a useful trend I could capture by one hot encoding the boolean values in the `chicken_in_ingredients` column.
3. `sugar`
- This feature tells us the % daily value of sugar in each recipe. Based on a bar chart I had constructed, it looked like there was a trend where recipes with higher ratings had less sugar. Thus, I wanted to include this feature in case there was any useful information. I chose to use a quantile transformer to modify this column due to the presence of extreme outliers (other transformers like a standard scaler would be an inaccurate representation of the data).
4. `total_fat`
- This feature tells us the % daily value of total fat in each recipe. Based on a bar chart I had constructed, it looked like there was a trend where recipes with higher ratings had higher total fat. Thus, I wanted to include this feature in case there was any useful information. I chose to use a quantile transformer to modify this column due to the presence of extreme outliers.
5. `protein`
- This feature tells us the % daily value of sugar in each recipe. Based on a bar chart I had constructed, it looked like there was a trend where recipes with higher ratings had higher protein. Thus, I wanted to include this feature in case there was any useful information. I chose to use a quantile transformer to modify this column due to the presence of extreme outliers.
6. `sodium`
- This feature tells us the % daily value of sodium in each recipe. Based on a bar chart I had constructed, it looked like there was a trend where recipes with higher ratings had higher sodium. Thus, I wanted to include this feature in case there was any useful information. I chose to use a quantile transformer to modify this column due to the presence of extreme outliers.
7. `month`
- This feature tells us the % daily value of sugar in each recipe. Based on a bar chart I had constructed, it looked like there was a trend where recipes with higher ratings had less sugar. Thus, I wanted to include this feature in case there was any useful information. I chose to use a quantile transformer to modify this column due to the presence of extreme outliers.

## The Model

For the modeling algorithm itself, I looked at a variety of options (including LinearRegression, turning the regression problem into a classification problem with RandomForestClassifier, and others). However, RandomForestRegressor appeared to be the best option given the skew in rating data.

I utilized GridSearchCV to find the best hyperparameters for my model. The best combination of hyperparameters was max_depth=10, min_samples_split=30, and n_estimators=200.

The model itself ended up with a mean squared error (MSE) of 0.403 (the baseline model had an MSE of 0.408) and an r^2 value of -0.00896 (the baseline model had an r^2 value of 0.00304). Based on these values, it appears that the final model marginally improved upon the baseline model. However, the overall performance of the model still appears to be poor.

# Fairness Analysis

For my fairness analysis, I looked at model parity between two groups: high protein and low protein. To designate whether or not a recipe belongs to the high protein or low protein group, I compared its protein PDV (percent daily value) to the median protein PDV. If it was greater than or equal to the median, I added it to the high protein group. Otherwise, I added it to the low protein group. I chose to evaluate MSE (mean squared error) parity between the two groups because large deviations between actual rating and predicting ratings are significantly worse than minor deviations.

**Null Hypothesis**: The MSE of our model across recipes with high protein and low protein is roughly the same. The model achieves MSE parity across these two groups.

**Alternate Hypothesis**: The MSE of our model across recipes with high protein and low protein is not the same. The model does not achieve MSE parity across these two groups.

**Statistic**: Absolute difference between MSE of our final model for recipes with high protein and recipes with low protein.

**Significance Level**: α = 0.05

I chose 0.05 as the significance level because a Type-1 error (rejecting the null hypothesis when it is actually true) is not particularly harmful in our case.

To conduct this hypothesis test, I first calculated the difference in means between just chicken-based recipes and all recipes. I then created 1000 bootstrapped samples from the overall dataset. I evaluated the absolute difference between the sample mean and the overall mean and stored these in an array.

<iframe
  src="assets/missingness_test.html"
  width="700"
  height="425"
  frameborder="0"
></iframe>

My fairness analysis yielded a p-value of 0.009. Thus, we reject the null hypothesis. The test suggests that the MSE of our model across recipes with high protein and low protein is not the same. Based on the test, our model appears to be unfair.