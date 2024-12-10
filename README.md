
# Holiday Food Analysis : Recipes and Rating Dataset


### Investigation on Holiday Foods Versus Non-Holiday Foods in Terms of Nutrition and Ratings

Authors: Aryan Kanuparti


## Introduction

In most cultures, the holidays are a time to be grateful for what we have, cherish moments with family and friends, and, perhaps most importantly, enjoy delicious food. Holiday dishes often carry special significance, with recipes passed down through generations or designed to capture the festive spirit. However, the indulgent nature of holiday foods raises intriguing questions about their nutritional value and popularity. Do holiday recipes, often laden with sugar and fats, receive higher ratings simply due to the nostalgia and festive atmosphere, or do health-conscious individuals rate them lower because of their nutritional content?

This data science project, conducted at UCSD, investigates the relationship between a recipe's holiday food status—determined by whether it was posted between October and December—and its ratings and nutritional values. By analyzing datasets containing recipes and ratings posted on food.com since 2008, originally curated for the research paper Generating Personalized Recipes from Historical User Preferences by Majumder et al., we aim to uncover trends that reveal how the festive season influences culinary preferences and perceptions.



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

   - This step helps match the unique recipes with their rating and review.

1. Check data types of all the columns.

   - This step helps us evaluate what data cleaning steps are appropriate for the dataset and if we need to conduct data type conversion.
   - | Column             | Description |
     | :----------------- | :---------- |
     | `'name'`           | object      |
     | `'id'`             | int64       |
     | `'minutes'`        | int64       |
     | `'contributor_id'` | int64       |
     | `'submitted'`      | object      |
     | `'tags'`           | object      |
     | `'nutrition'`      | object      |
     | `'n_steps'`        | int64       |
     | `'steps'`          | object      |
     | `'description'`    | object      |
     | `'ingredients'`    | object      |
     | `'n_ingredients'`  | int64       |
     | `'user_id'`        | float64     |
     | `'recipe_id'`      | float64     |
     | `'date'`           | object      |
     | `'rating'`         | float64     |
     | `'review'`         | object      |

1. Fill all ratings of 0 with np.nan.

   - Rating is generally on a scale from 1 to 5, 1 meaning the lowest rating while 5 means the highest rating. With that being said, a rating of 0 indicates missing values in rating. Thus, to avoid bias in the ratings, we filled the value 0 with np.nan.

1. Add column `'average_rating'` containing average rating per recipe.

   - Since a recipe can have numerous ratings from different users, we take an average of all the ratings to get a more comprehensive understanding of the rating of a given recipe.

1. Split values in the nutrition column to individual columns of floats.

   - Even though the values in the nutrition column look like a list, they are actually objects, which act like strings. Given by the description of the columns of the recipe dataset, we know what each individual values inside the brackets mean. To split up the values, we applied a lambda function then converted the columns to floats. It will allow us to conduct numerical calculations on the columns.

1. Convert submitted and date to datetime.

   - These two columns are both stored as objects initially, so we converted them into datetime to allow us conduct analysis on trends over time if needed.

1. Add `'is_dessert'` to the dataframe

   - `'is_dessert'` is a boolean column checking if the tags of recipes contain 'dessert' since desserts typically have a higher amount of sugar. This step separates the recipes into two groups, ones that are dessert and ones that are not. This provides us another way to compare ratings of recipes with more sugar and ones with less sugar.

1. Add `'prop_sugar'` to the dataframe
   - prop_sugar is the proportion of sugar of the total calories in a recipe. To calculate this, we use the values in the sugar (PDV) column to divide by 100% to get it in the decimal form. Then, we multiply by 25 to convert the values to grams of sugar since 25 grams of sugar is the 100% daily value (PDV). We got this value of 25 grams from experimenting on food.com with different amounts of sugar in a recipe. The experimentation allows us to understand the nutrition formula used on the website for recipes. Lastly, we multiply by 4 since there are 4 calories in 1 gram of sugar. After all these conversions, we end up with the number of calories of sugar, so we can divide by the total amount of calories in the recipe to get the proportion of sugar of the total calories. This data cleaning step is critical to allow us to make parallel comparisons on the amount of sugar in a recipe without concerns of extremely large values since all the values will be between 0 and 1.

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
| `'average rating'`      | object         |
| `'calories (#)'`        | float64        |
| `'total fat (PDV)'`     | float64        |
| `sugar (PDV)'`          | float64        |
| `'sodium (PDV)'`        | float64        |
| `'protein (PDV)'`       | float64        |
| `'saturated fat (PDV)'` | float64        |
| `'carbohydrates (PDV)'` | float64        |
| `'is_dessert'`          | bool           |
| `'prop_sugar'`          | float64        |


Our cleaned dataframe ended up with 234429 rows and 27 columns. Here are the first 5 rows of ~unique recipes of our cleaned dataframe for illustration. Since there is a lot of columns for the merged dataframe, we selected the columns that are most relevant to our questions for display. Scroll right to view more columns.

| name                                 |     id |   minutes | submitted           |   rating |   average rating |   calories (#) |   sugar (PDV) | is_dessert   |   prop_sugar |
|:-------------------------------------|-------:|----------:|:--------------------|---------:|-----------------:|---------------:|--------------:|:-------------|-------------:|
| 1 brownies in the world    best ever | 333281 |        40 | 2008-10-27 00:00:00 |        4 |                4 |          138.4 |            50 | True         |    0.361272  |
| 1 in canada chocolate chip cookies   | 453467 |        45 | 2011-04-11 00:00:00 |        5 |                5 |          595.1 |           211 | False        |    0.354562  |
| 412 broccoli casserole               | 306168 |        40 | 2008-05-30 00:00:00 |        5 |                5 |          194.8 |             6 | False        |    0.0308008 |
| millionaire pound cake               | 286009 |       120 | 2008-02-12 00:00:00 |        5 |                5 |          878.3 |           326 | True         |    0.371172  |
| 2000 meatloaf                        | 475785 |        90 | 2012-03-06 00:00:00 |        5 |                5 |          267   |            12 | False        |    0.0449438 |


### Univariate Analysis

<iframe
  src="assets/holiday_season_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis



### Interesting Aggregates


## Assessment of Missingness

Three columns, `'date'`, `'rating'`, and `'review'`, in the merged dataset have a significant amount of missing values, so we decided to assess the missingness on the dataframe.

### NMAR Analysis


### Missingness Dependency


## Hypothesis Testing



## Framing a Prediction Problem


## Baseline Model


## Final Model



## Fairness Analysis

