
# Holiday Food Analysis : Recipes and Rating Dataset

### Investigation on Holiday Foods Versus Non-Holiday Foods in Terms of Nutrition and Ratings

Author: Aryan Kanuparti


## Introduction

In most cultures, the holidays are a time to be grateful for what we have, cherish moments with family and friends, and, perhaps most importantly, enjoy delicious food. Holiday dishes often carry special significance, with recipes passed down through generations or designed to capture the festive spirit. However, the indulgent nature of holiday foods raises intriguing questions about their nutritional value and popularity. Do holiday recipes, often laden with sugar and fats, receive higher ratings simply due to the nostalgia and festive atmosphere, or do health-conscious individuals rate them lower because of their nutritional content?

This data science project, conducted at UCSD, investigates the relationship between a recipe's holiday food status—determined by whether it was posted between October and December—and its ratings and nutritional values. By analyzing datasets containing recipes and ratings posted on food.com since 2008, I aim to uncover trends that reveal how the festive season influences culinary preferences and perceptions.



The primary dataset, `recipes`, contains 83782 rows -- 83782 unique recipes -- with 12 columns recording the following information:

| Column             | Description                                                                                                                                                                                       |
| :----------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `'name'`           | Recipe name                                                                                                                                                                                       |
| `'id'`             | Recipe ID                                                                                                                                                                                         |
| `'minutes'`        | Minutes to prepare recipe                                                                                                                                                                         |
| `'contributor_id'` | User ID who submitted this recipe                                                                                                                                                                 |
| `'submitted'`      | Date recipe was submitted                                                                                                                                                                         |
| `'tags'`           | Food.com tags for recipe                                                                                                                                                                          |
| `'nutrition'`      | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” |
| `'n_steps'`        | Number of steps in recipe                                                                                                                                                                         |
| `'steps'`          | Text for recipe steps, in order                                                                                                                                                                   |
| `'description'`    | User-provided description                                                                                                                                                                         |
| `'ingredients'`    | Text for recipe ingredients                                                                                                                                                                       |
| `'n_ingredients'`  | Number of ingredients in recipe                                                                                                                                                                   |

The second dataset, `interactions`, contains 731927 rows with each one representing one review that a user left on a specific recipe. Each recipe could have multiple interactions and reviews. The columns it includes are:

| Column        | Description         |
| :------------ | :------------------ |
| `'user_id'`   | User ID             |
| `'recipe_id'` | Recipe ID           |
| `'date'`      | Date of interaction |
| `'rating'`    | Rating given        |
| `'review'`    | Review text         |


**Given the datasets, I am investigating whether people rate holiday recipes and non-holiday recipes similarly.** To answer this  question, I identified recipes submitted during the holiday season (October to December) and created a new column, 'is_holiday_season', to indicate whether a recipe falls into this category. Additionally, I extracted values from the 'nutrition' column into separate columns such as 'calories (#)', 'total fat (PDV)', 'sugar (PDV)', and others. 
Information  about these relationships could help contributors produce better recipes during the festive season. Additionally, these findings could lead to future work exploring how nutritional factors or seasonal sentiments influence ratings and preferences for holiday foods

## Data Cleaning and Exploratory Data Analysis

I followed this outline to clean and explore my data before getting into my testing and modeling.

1. Left merge the recipes and interactions datasets on id and recipe_id.

1. Fill all ratings of 0 with np.nan.

   - Ratings in this dataset range from from 1 to 5. 1 being the lowest rating and 5 being highest rating. Furthermore a rating of 0 does not really make sense with this scale, so to minmize bias I am going to fill all the 0 ratings with null values.
1. Add column `'rating_avg'` containing average rating per recipe.

   - Since each recipe could potentially hae multiple rating from different users, aggregating all the ratings will help ii analyzing the trends associated with the rating of eah recipe.

1. Extract values in the nutrition column to individual columns of floats.

   - Based on the description of the columns of the recipe dataset, I know what each individual values inside the brackets of the nutrition columns means. In order to prerfom analysis with those values, I split up the nutrition list for each recipe and put those floats into thier own respective columns. 

1. Convert submitted date column to datetime.

   - These two columns are both stored as objects initially, so we converted them into datetime to allow us conduct analysis on trends over time if needed.

1. Drop unused columns

   - I then dropped some columns that I decided I would not be using going forward, ['user_id', 'recipe_id'], recipe id is reduntant with id already being a column, and the userId of whoever submitted the review bears no significance in terms of this analysis.

1. Add `is_holiday_season` to the dataframe
   - Added on a column of boolean values that represented whether each recipe was posted during a holiday month (October - December). False for recipes that were posted in the other nine months and true for the rest. I calculated this column using the clean datetime values in the submitted column.

#### Result
Here are all the columns of the cleaned df.

| Column                  | Description    |
| :---------------------- | :------------- |
| `'name'`                | object         |
| `'id'`                  | int64          |
| `'minutes'`             | int64          |
| `'contributor_id'`      | int64          |
| `'submitted'`           | datetime64[ns] |
| `'tags'`                | object         |
| `'nutrition'`           | object         |
| `'n_steps'`             | int64          |
| `'steps'`               | object         |
| `'description'`         | object         |
| `'ingredients'`         | object         |
| `'n_ingredients'`       | int64          |
| `'user_id'`             | float64        |
| `'recipe_id'`           | float64        |
| `'date'`                | datetime64[ns] |
| `'rating'`              | float64        |
| `'review'`              | object         |
| `'rating_avg'`          | object         |
| `'calories (#)'`        | float64        |
| `'total fat (PDV)'`     | float64        |
| `sugar (PDV)'`          | float64        |
| `'sodium (PDV)'`        | float64        |
| `'protein (PDV)'`       | float64        |
| `'saturated fat (PDV)'` | float64        |
| `'carbohydrates (PDV)'` | float64        |
| `'is_holiday_season'`   | bool           |



My cleaned dataframe ended with 234429 rows and 23 columns. Here are the first 5 rows of of the merged dataset ( there are some duplicated recipes due to the initial left merge and the fact that each recipe could potentially have multiple differnt reviews and ratings) Since there are a lot of columns for the merged dataframe, I selected the most relevant ones to display.

| name                                 |     id |   rating_avg |   calories (#) |   total_fat (PDV) |   sugar (PDV) | is_holiday_season   |
|:-------------------------------------|-------:|-------------:|---------------:|------------------:|--------------:|:--------------------|
| 1 brownies in the world    best ever | 333281 |            4 |          138.4 |                10 |            50 | True                |
| 1 in canada chocolate chip cookies   | 453467 |            5 |          595.1 |                46 |           211 | False               |
| 412 broccoli casserole               | 306168 |            5 |          194.8 |                20 |             6 | False               |
| 412 broccoli casserole               | 306168 |            5 |          194.8 |                20 |             6 | False               |
| 412 broccoli casserole               | 306168 |            5 |          194.8 |                20 |             6 | False               |

### Univariate Analysis
For the single variable analysis, I wanted to explore ratio of holiday to non-holiday foods as well as the general shape of the distribution for some of the more pivotal nutrition columns 
<iframe
  src="assets/holiday_season_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

It appears that there are far more non-holiday recipes than holiday ones, which will make our future analysis a little challenging. 


<iframe
  src="assets/log_scaled_histogram_sugar.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

In looking at the shape of the sugar (PDV), it is skewed right with a fairly even log-scaled distribution which will be useful for training models in the future.

### Bivariate Analysis
For the bivariate analysis, I wanted to study the relationship between sugar and fat becasue I believe that recipes with more of both of these are likely unhealthy and thus taste good enough to garner higher ratings.


<iframe
  src="assets/scatter_sugar_vs_fat.html"
  width="1000"
  height="800"
  frameborder="0"
></iframe>

It appears that there isnt any strong noticable trend, other than most of the values being smaller rather than larger, with some relatively massive outliers for both sugar and fat
Furthermore, to ties this back into the holiday food question, I wanted to determine some baseline information like the mean sugar (PDV) in holiday and non-holiday foods.

<iframe
  src="assets/mean_sugar_by_season.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Based on the graph, I can see that holiday foods have marginally more sugar on average, which is a good sign for my initial hypothesis but requires further analysis.

### Interesting Aggregates
For this section, I wanted to investigate similar relationships between the holiday status of foods and other aggreates like average rating and total_fat (PDV) / sugar (PDV)

| is_holiday_season   | rating_avg mean | rating_avg median |
|:--------------------|--------:|---------:|
| False               | 4.67989 |  4.85714 |
| True                | 4.66112 |  4.875   |


| is_holiday_season   |   ('sugar (PDV)', 'mean') |   ('sugar (PDV)', 'median') |   ('total_fat (PDV)', 'mean') |   ('total_fat (PDV)', 'median') |
|:--------------------|--------------------------:|----------------------------:|------------------------------:|--------------------------------:|
| False               |                   62.6315 |                          22 |                       31.6395 |                              20 |
| True                |                   69.3588 |                          24 |                       33.201  |                              20 |

These pivot tables clearly highlight the marginal differnces between holiday and non-holiday food. It is interesting to note that the average rating of holiday foods is actually slightly lower then non-holiday foods, while the other nutritional variables follow the opposite trend.

## Assessment of Missingness

`'date'`, `'rating'`, and `'review'`, have a significant amount of missing values.

### NMAR Analysis

Out of these three, I believe it to be most likely that the `'review'` column is NMAR. In other words the missigness of `'review'` column depends on the values or natures of the review themselves. The logic is that if people are apathetic about the recipe they are less likely to write out a review. If they really truly enjoyed the recipe they would take the time to write out a review, if not they might just click the rating and leave. Knowing somethig like the amount of time the reviewers spend on the sight might shed some light on the missigness of the review column. For exmaple, the longer someone spends leaving the review and rating the recipe, the more likely they were to type out a response is one hypothesis that could tested.


### Missingness Dependency
For this section, I'll pick the `'rating'` columns as I beleive to have nontrivial missigness. Recall earlier that I replace all the ratings of 0 ( from a 1-5 scale) to null/missing values. My initial thoughts about the dependcnay of the missignems of this column is that number of steps and cooking time might be influential here. The logic is that if someone finds a recipe and sees a long cook time, or complicated and long number of steps, they might be inclined to not even make the recipe and thus not rate it which would explain the missignss of the rating column.

To investigate this, I conducted two permutation tests to assess whether the missingness of ratings is dependent on these factors.
### Hypothesis Testing Report: Missingness of Ratings

### Number of Steps and Rating
- **Null Hypothesis (H₀):** The missingness of ratings does not depend on the number of steps in the recipe.
- **Alternative Hypothesis (H₁):** The missingness of ratings does depend on the number of steps in the recipe.
- **Test Statistic:** The absolute difference in the mean number of steps between the group with non-missing ratings and the group with missing ratings.
- **Significance Level (α):** 0.05 (A fair significance level, stringent evidence not required to draw connections)

After running 1,000 permutations by shuffling the missingness of the 'rating' column, we calculated the simulated mean differences under the null hypothesis. The observed statistic, **4.3145**, is shown by the red vertical line on the graph. The p-value for this test was **0.0**, which is less than the significance level of 0.05.

<iframe
  src="assets/permutation_test_n_steps.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Conclusion
Since the p-value is less than the threshold, we reject the null hypothesis. This indicates that the missingness of ratings is dependent on the number of steps in the recipe.

---

### Cooking Time (Minutes) and Rating
- **Null Hypothesis (H₀):** The missingness of ratings does not depend on the cooking time of the recipe in minutes.
- **Alternative Hypothesis (H₁):** The missingness of ratings does depend on the cooking time of the recipe in minutes.
- **Test Statistic:** The absolute difference in the mean cooking time between the group with non-missing ratings and the group with missing ratings.
- **Significance Level (α):** 0.05 (A fair significance level, stringent evidence not required to draw connections)

Outliers in cooking time posed a challenge for examining the distributions, but the test proceeded by considering the absolute mean differences. After running 1,000 permutations, the observed statistic was **51.4524**, and the p-value was **0.12**. This is greater than the significance level of 0.05.

<iframe
  src="assets/permutation_test_minutes.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Conclusion
Since the p-value is greater than the threshold, we fail to reject the null hypothesis. This suggests that the missingness of ratings is not dependent on the cooking time of the recipe.

## Hypothesis Testing

## Introduction
This study examines whether recipes submitted during holiday seasons differ significantly from non-holiday recipes in terms of average ratings and sugar content. Holiday seasons may influence both the types of recipes shared and how they are received due to cultural or seasonal preferences. I hypothesize that holiday recipes may have higher average ratings and distinct sugar content distributions compared to non-holiday recipes.

To test these hypotheses, I conducted two-sample Kolmogorov-Smirnov (K-S) tests for the distributions of ratings and sugar content between holiday and non-holiday recipes.
---

### Hypotheses
#### Test for Ratings:
- **Null Hypothesis (H₀):** The distributions of ratings for holiday and non-holiday recipes are the same.
- **Alternative Hypothesis (H₁):** The distributions of ratings for holiday recipes differ significantly from those for non-holiday recipes.

#### Test for Sugar Content:
- **Null Hypothesis (H₀):** The distributions of sugar content for holiday and non-holiday recipes are the same.
- **Alternative Hypothesis (H₁):** The distributions of sugar content for holiday recipes differ significantly from those for non-holiday recipes.

---

### Test Details

- **Test Statistic:** Kolmogorov-Smirnov Test Statistic (K-S Statistic)
- **Significance Level (α):** 0.01  
We used the Kolmogorov-Smirnov test as it is a non-parametric method that compares the cumulative distributions of two samples to determine if they are from the same population.

---

### Results

#### Test for Ratings:
- **K-S Test Statistic:** 0.0193  
- **p-value:** (1.72 x 10^{-11})

#### Test for Sugar Content:
- **K-S Test Statistic:** 0.0254  
- **p-value:** (8.33 x 10^{-20})

---

### Interpretation of Results

1. **Ratings:**  
   The p-value (1.72 x 10^{-11}) is far below the significance level of 0.01. Therefore, we reject the null hypothesis and conclude that the distributions of ratings for holiday and non-holiday recipes are significantly different. One potential reason could be that holiday recipes are perceived as more celebratory or indulgent, leading to higher average ratings.

2. **Sugar Content:**  
   The p-value (8.33 x 10^{-20}) is also far below the significance level of 0.01. Hence, we reject the null hypothesis and conclude that the distributions of sugar content for holiday and non-holiday recipes are significantly different. This difference may arise because holiday recipes are often sweeter due to cultural traditions and seasonal preferences.

---

### Conclusion
Our analysis demonstrates significant differences in both ratings and sugar content between holiday and non-holiday recipes. Holiday recipes tend to be rated differently, likely due to their cultural or celebratory context. Additionally, the higher sugar content in holiday recipes aligns with expectations of indulgence during festive periods. These findings suggest that seasonality has a meaningful impact on both the composition and reception of recipes.

## Framing a Prediction Problem

### Predicting Average Rating of Recipes

After conducting my initial exploratory data analysis (EDA) and running the above hypothesis tests, I would like to attempt to predict the average rating of recipes based on a number of features, with the most important being the **is_holiday_season** column. The response variable for this prediction is **rating_avg**, which is a scalar quantitative value ranging from 1 to 5.

Since the **rating_avg** is a continuous numerical value, I have decided to use regression models rather than classification. This is because the average rating is not an ordinal categorical variable, and rounding it would limit its precision. Thus, I aim to predict the average rating as a real-valued number within the range [1, 5].

To build the regression model, I will use the **is_holiday_season** column, which indicates whether the recipe was submitted during a holiday season, and **sugar (PDV)**, the sugar content of the recipe. These features are likely to have an impact on the recipe’s rating, and they provide valuable input for the model.

To evaluate the performance of my regression models, I will use two metrics:
- **RMSE (Root Mean Squared Error):** RMSE scales the error back to the target variable, which makes it easier to interpret. A lower RMSE indicates better predictive performance.
- **R² (Coefficient of Determination):** The R² value represents the proportion of the variance in the target variable (avg_rating) that can be explained by the model. The closer the R² value is to 1, the better the model explains the variance in the data.

Since I am predicting a continuous value, regression will be employed, and I will focus on how well the **is_holiday_season** and **sugar (PDV)** features help explain the variance in the **avg_rating** of recipes.


## Baseline Model

### Model Description

In this initial model, I aimed to predict the average recipe rating (`rating_avg`) using two features: `is_holiday_season` and `sugar (PDV)`. Below is a breakdown of the features and how they were processed:

### Features:
1. **is_holiday_season**:
   - Type: **Nominal (Categorical)**
   - Encoding: This feature was one-hot encoded to convert the categorical values into binary variables (0 or 1), with the "first" category dropped to avoid multicollinearity.

2. **sugar (PDV)**:
   - Type: **Quantitative (Numerical)**
   - Encoding: No encoding was necessary since this feature is already in a numerical format.

### Data Preprocessing:
- **Missing Value Handling**: For both the features and the target variable (`rating_avg`), missing values were handled using **mean imputation**. This strategy replaces missing values with the mean of the column.
- **Feature Scaling**: No explicit scaling was performed on the numerical feature (`sugar (PDV)`), as linear regression models can generally handle unscaled features, but this is something to consider in future iterations.

### Model:
- **Model Type**: The model used was **Linear Regression**, a common method for predicting continuous values.
- **Evaluation Metrics**: The model's performance was evaluated using several metrics:
  - **Mean Squared Error (MSE)**: 0.241
  - **Root Mean Squared Error (RMSE)**: 0.491
  - **Mean Absolute Error (MAE)**: 0.339
  - **R² Score**: 0.0003

### Model Performance

The R² score of approximately **0.0003** suggests that the model is not effectively predicting the target variable, `rating_avg`. In fact, it performs almost as poorly as predicting the mean of the target for all samples. The RMSE and MAE also indicate substantial error in the predictions.

Given that the R² value is close to zero, it suggests that the current model barely performs better than a simple baseline model that predicts the average rating for every recipe. This indicates that the linear regression model, with its current features and preprocessing steps, is not capturing the underlying relationships between the features and the target variable effectively.

#### Ideas for Improvement

After making this initial model and reevaluting my methods, I compiled some thoughts to potentially improve this currently unusable model.

1. **Probabilistic Imputation**:
   - **Current Limitation**: Mean imputation assumes that the data is missing completely at random (MCAR), which might not be the case.
   - **Improvement**: I plan to explore **probabilistic imputation**, where missing values are imputed based on a probability distribution derived from observed data. This could result in more realistic imputations and improve model performance.

2. **Inclusion of More Quantitative Features**:
   - **Current Limitation**: The model currently uses only two features: `is_holiday_season` (which is categorical) and `sugar (PDV)` (which is quantitative).
   - **Improvement**: I will consider including more quantitative features, such as `cooking_time` or `num_steps`, as these might provide additional predictive power. I will also explore the possibility of including interactions between features, which could improve the model’s ability to capture more complex relationships.

3. **Alternative Regression Models**:
   - **Current Limitation**: Linear regression may not be the best choice for this problem, especially given the poor performance observed.
   - **Improvement**: I will experiment with other regression models, such as **Random Forest Regression** or **Gradient Boosting Machines** which may better capture the relationships between the features and target variable. These models can also handle more complex, non-linear patterns.

4. **Feature Engineering**:
   - **Improvement**: I will explore feature engineering techniques, such as creating new features or transforming existing ones, to improve the model's performance. For example, I could introduce interaction terms between features like sugar (PDV) and total_fat (PDV) which togather might be more correlated with rating_avg. 



While the initial linear regression model offers a baseline, the poor performance (as indicated by the near-zero R² score) suggests that significant improvements are needed. The next steps will focus on improving the feature set, exploring better imputation techniques, and trying alternative regression models to enhance predictive accuracy. Given the current results, I do not consider this model to be "good," -- in fact it is really bad -- but I see clear paths for improvement.



## Final Model



## Fairness Analysis

