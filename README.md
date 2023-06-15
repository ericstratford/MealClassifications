# Meal Classifications

by Eric Stratford (estratford@uscd.edu)

---

## Table of Contents
{: .no_tox .text-delta }

1. TOC
{:toc}

---

## Pre-introduction

*[TO DO]*
The exploratory data analysis precursor to this prediction project can be found [here]

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
- Other Features
	- Minutes
		- Cook Time
	- # Steps	
		- Number of steps in cooking process
	- # Ingredients
		- Number of different ingredients in the recipe

The remaining unused variables were dropped for this model.

Feature Summary:
*[TO DO]*
- Quantitative Features
	- Nutrition Values (Calories, Protein/Cal, etc.)
	- Minutes
- Ordinal Features
	- # Steps
	- # Ingredients
- Nominal Features
	- Tags

### Model Design & Performance

For the initial model, I wanted to stick with simple and robust classification models before using more advanced ones for the final model. Sticking to this, I tested a K-Neighbors Classifier as well as a Decision Tree Classifier. The latter yielded the better validation performance, with a validation accuracy of 0.73.

This model did not involve any hyperparameter tuning.

I believe that this model is fairly strong, especially given the classification task as well as the fact that this is the preliminary model. In general, I don't expect there to be much success for many of the classifications. In my mind, I don't see how one could easily classify a recipe as 'lunch' as opposed to 'main-dish', so I find this accuracy to be successful.

---

## Final Model

[In Progress]

### Feature Engineering

### Refining the Model

### Hyperparameter Tuning

### Performance Evaluation

---

## Fairness Analysis

[In Progress]

Graph 1:

<iframe src="Assets/meal_accuracy.html" width=800 height=600 frameBorder=0></iframe>

Graph 2:

<iframe src="Assets/minutes_accuracy.html" width=800 height=600 frameBorder=0></iframe>

---