
# Holiday Food Analysis : Recipes and Rating Dataset


### Investigation on Holiday Foods Versus Non-Holiday Foods in Terms of Nutrition and Ratings

Authors: Aryan Kanuparti


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

   - I then dropped some columns that I decided I would not be using going forward, ['user_id', 'recipe_id'], recipe id is reduntant with id already being a column and the userid of who submitted the review bears no significance in this analysis

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



My cleaned dataframe ended with 234429 rows and 23 columns. Here are the first 5 rows of of the merged dataset ( there are some duplciated recipes due to the initial left merge and the fact that each recipe could potentially have multiple differnt reviews and ratings) Since there are a lot of columns for the merged dataframe, I selected the most relevant ones to display.

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
  width="800"
  height="600"
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

