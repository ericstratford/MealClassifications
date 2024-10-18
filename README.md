# Meal Classifications

by Eric Stratford (estratford@uscd.edu)

---

## Table of Contents
1. [Pre-introduction](#pre-introduction)
2. [Introduction (Framing the Problem)](#introduction-framing-the-problem)
3. [Baseline Model](#baseline-model)
   - [Feature Engineering](#feature-engineering)
   - [Model Design & Performance](#model-design--performance)
4. [Final Model](#final-model)
   - [Feature Engineering](#feature-engineering-1)
   - [Classification Model](#classification-model)
   - [Hyperparameter Tuning](#hyperparameter-tuning)
   - [Performance Evaluation](#performance-evaluation)
5. [Fairness Analysis](#fairness-analysis)
6. [Conclusion](#conclusion)

---

## Pre-introduction

This project builds upon an exploratory data analysis (EDA) that examined the structure and patterns within recipes. Through this EDA, we gained insights into how various attributes such as nutritional values, ingredients, and preparation times can be used to classify recipes into specific meal categories. The exploratory data analysis precursor to this prediction project can be found [here]

## Introduction (Framing the Problem)

**Can we predict what type of meal a recipe belongs to based on its other attributes?**

Over half of the recipes on Food.com are not tagged into one of the meal categories, and many others are poorly classified (e.g. a brownie recipe may be marked as 'lunch'). Can we resolve this issue and give the home cooks some more direction as they browse recipes?

Prediction Type: Multiclass Classification
The meal types for this problem are: 
*breakfast, brunch, lunch, dinner-party, main-dish, side-dish, snacks, beverages, cocktails, and condiments-etc*
This list was gathered through thorough data exploration and analysis to get as complete coverage as possible while retaining generalizability.

Response variable: meal-type
This sort of hidden variable is nested within each recipe's 'tags'. By extracting the meal-type from the 'tags' we are able to obtain our prediction target.
Browsing through food.com, I found that being able to correctly classify recipes is important. Furthermore, I felt that because of the hidden nature of meals within 'tags', this classification would be fairly unique and interesting.

Evaluation Metric: Accuracy
At first, I thought thought that F1-Score would be most applicable as my research detailed that F1-Score was just more useful than accuracy. More specifically, F1-Score is useful in uneven class distributions, which is very applicable to the uneven class distributions of 'meals'. However, Because I want to concentrate my predictions towards True positives and True negatives (or their respective interpretations in a multiclass problem), I decided to go with accuracy. I felt that for the specific scenario which I am dealing with, I don't want to overly punish false predictions compared to trues. Moreover, accuracy is simply more intuitive for my understanding.

Note:
A given 'meal' tag would be typically be published with an entire recipe, so it seems like there is little predicting. However, the true predictions occur in classifications and corrections as described above. This is especially useful when imputing misclassified or unfilled data related to the meal type.

Now that we have gone through an overview of the problem, let's dive into our first model.

---

## Baseline Model

The Decision Tree Classification

### Feature Engineering

Approaching this first model, I knew that there were many features to be extracted from this dataset. In the aforementioned data exploration project, the entirety of the analysis was based upon multiple extracted features which I wanted to implement into this model.

At first, I started with a cleaned dataset which had each recipe as well as its average rating. This cleaned dataset also had a column which classified each recipe according to a meal-type, extracted from the recipes' 'tags'. Some recipes were classified as multiple meals, such as 'dessert', 'snack', and 'main-dish'. For my extaction, I chose the first tag as it seemed to be the most accurate according to my data exploration. This transformation also included some other cleaning, such as removing unclassified recipes, imputing missing ratings, and removing all meal-types from the 'tags' column so as to not interfere with predictions.

As for the actual feature engineering...
- Nutrition Values
	- Extracting from a single column of percent daily values (PDV) for different nutritional categories (such as total fat, protein, etc.), I divided each value by the total calories from that recipe. This would essentially standardize the nutritional values without giving too much weight to outliers. For instance, if one recipe was a single chicken nugget, it's protein PDV would be miniscule in comparison to a recipe of an entire roasted turkey. However, I kept 'Calories' as is, because there are significant caloric differences across meals (e.g. beverages vs dinner). To finalize the nutritional feature engineering, I used StandardScaler() to standardize all of the prior nutritional values.
- Tags
	- After removing the meal classifications from the 'tags', I wanted to use the rest of the tags to predict the meal. If a meal had the '15-minutes-or-less' tag, it probably wouldn't be dinner. To encode the tags, I use a MultiHotEncoder, which essentially just one-hot-encodes the values stored within a series of lists.
- Other Features (see final model for more in-depth explanation of inclusion)
	- Minutes
		- Cook Time
	- \# Steps	
		- Number of steps in cooking process
	- \# Ingredients
		- Number of different ingredients in the recipe

The remaining unused variables were dropped for this model.

Feature Summary:
- Quantitative Features
	- Nutrition Values (Calories, Protein/Cal, etc.)
	- Minutes
	- \# Steps
	- \# Ingredients
- Nominal Features
	- Tags

### Model Design & Performance

For the initial model, I wanted to stick with simple and robust classification models before using more advanced ones for the final model. Sticking to this, I tested a K-Neighbors Classifier as well as a Decision Tree Classifier. The latter yielded the better validation performance, with a validation accuracy of 0.73.

This model did not involve any hyperparameter tuning.

I believe that this model is fairly strong, especially given the classification task as well as the fact that this is the preliminary model. In general, I don't expect there to be much success for many of the classifications. In my mind, I don't see how one could easily classify a recipe as 'lunch' as opposed to 'main-dish', so I find this accuracy to be successful.

---

## Final Model

Coming out of the baseline model, I was pretty satisfied with my validaiton accuracy, but I knew there was room for improvement in both the features and the model.

### Feature Engineering

'Ingredients' was one of the key predictors that I did not include in my baseline model, which I wanted to include for the final model. Furthermore, I wanted to make some adjustments to some of the quantitative variables used in the baseline model.

Features:
- Nutrition Values
	- Described more in depth in the baseline model, this include standardized caloric values of each recipe as well as standardized nutritional values per calorie (e.g. Sugar/Calorie)
	- This is useful as different meal types often have different nutrtional values (e.g. dessert vs lunch)
- Tags
	- One Hot Encoded tags such as '60-minutes-or-less' or 'mexican'
	- Tags can help differentiate between meal types. For example, a '15-minutes-or-less' tagged recipe might differentiate between 'snacks' and 'dinner-party'.
- Standardized Quantitative Variables
	- Average rating
		- Through some exploratory data analysis, I found that certain meal types have higher ratings than others. For instance, 'beverages' are on average rated 0.33 points higher than snacks.
	- \# Steps
		- Certain meal types are going to requires more cooking steps than others (e.g. desserts are likely going to be more complicated to make than beverages)
	- \# Ingredients
		- Meal types often involve different ingredient usage. For example, breakfast is probably going to involve less ingredients than a dinner-party dish.
- Minutes
	- Standardized through RobustScaler()
	- Through data exploration, I noticed that many recipes had outlandish cooking times, perhaps due to mistakes or because some recipes actually took multiple days (such as dry-aging). To account for these outliers while still retaining the value in the predictor, I used a robust scaler.
- Ingredients
	- What I believe to be the biggest different in feature engineering between the baseline model and the final model
	- The ingredients represent each different ingredient used in a recipe, regardless of quantity. To make use of this, I used a CountVectorizer() to track the importance of each ingredient as a word across the entire corpus of ingredients.
	- This feature engineering enables the model to examine the presence of certain ingredients to classify a recipe.
	- Originally, I wanted to use a TF-IDF vectorizer (term frequency-inverse document frequency) to examine the relative importance of each ingredient across all reicpes, however, I encountered dimensional issues which were resolved by using count vectorization instead.

The remaining unused variables were dropped.

Feature Summary:
- Quantitative:
	- Nutritional Values
	- Minutes
	- Average Rating
	- \# Steps
	- \# Ingredients
- Nominal
	- Ingredients
	- Tags

### Classification Model

While the decision tree classification worked well, I wanted to use a deeper, more advanced classifier for this model with developed features.

While I typically like to experiment with models such as gradient boosting, SVC (support vector classifier), and random forests, I had to simplify my options for a multi-class classification problem. For this I tested a random forest model and a support vector machine (SVM).

Using the same features for each and without hyperparameter tuning, I found random forest classification to yield the highest validation accuracy. For this reason, I performed the rest of my hyperparameter tuning and evaluation using a random forest classification.

Before any hyperparameter tuning, my validation accuracy averaged around 0.808

### Hyperparameter Tuning

The hyperparameters I would be tuning here are the n_estimators parameter and the max_depth parameter of the RandomForestClassifier.

My hyperparameter values were listed as such:
n_estimators: [100, 200, 300]
max_depth: [40, 60, 80, 100, 200, None]

To perform my hyperparameter tuning, I performed a segmented grid search with 5-fold cross-validation. Because my model takes relatively long to learn (around 5 minutes), it would have been unreasonable to perform the entire grid-search for the parameters at once. For this reason, I performed multiple grid searches with segments of the hyperparameters.

By the end of my hyperparameter tuning, my optimal parameters were:
max_depth: 60
n_estimators: 200

### Performance Evaluation

After implementing these hyperparameters into my model and re-evaulating the score, the validation accuracy improved to 0.816, showing a jump in 0.07 points.


Final Validation Accuracy:
0.816
(0.086 point improvement on baseline model)

Final Test Accuracy:
0.811

---

## Fairness Analysis

While our model seems to be performing pretty well overall, let's examine our model's performance within specific sub groups.

Firstly, let's examine the model's classification performance for each class:

<iframe src="Assets/meal_accuracy.html" width=800 height=600 frameBorder=0></iframe>

It makes sense that certain meals would be easier to classify. After all, it's easier to distinguish dessert from lunch than it is to distinguish breakfash from brunch.


Now, let's look at how well our model predicts recipes from year to year:

<iframe src="Assets/years_accuracy.html" width=800 height=600 frameBorder=0></iframe>

Here, we see an interesting trend. It seems as though more recently published recipes have a slightly declining accuracy. Could this be due to the fact that there is less data on newer recipes overall (see the count of recipes per year).

Let's perform a fairness analysis on this with a permutation test.

Null Hypothesis: The model is fair and has balanced predictions for all 'minutes' group, and any differences are due to random chance.

Alternative Hypothesis: The model is unfair and predicts certain 'minutes' groups better than others.

-Test Statistic: Difference in accuracy
-Significance level: 0.05

After performing a permutation test with 1000 samples, the resulting p-value was 0.894, meaning that we cannot reject the null hypothesis given this information.

---

## Conclusion

Our final model was able to achieve a meal-type classification accuracy of around 81%. However, we saw that the model is better at predicted certain types of meals than others. Nonetheless, using these prediction, we could theoretically improve the browsing experience on food.com, by giving its viewers more accurate direction when finding recipes.

---
