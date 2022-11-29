# Analysis of Subreddit Questions for Women and Men

## Problem Statement

Data is now crucial in every part of our lives thus the usage use of surveys has skyrocketed. A research study is being done on whether questions that are geared toward women or men are distinguishable and their different properties. Using the subreddits: askwomen and askmen, two subreddits dedicated to asking questions of the two groups, a classifier model will be built to correctly identify questions that are specifically geared towards men or women. This model will be evaluated based on it's accuracy and it's ability to identify key terms or phrases that are used in the two question types.

## Data and Collection

### Size

- askmen_posts.csv (2063, 6)
- askwomen_posts.csv (2041, 6)

#### Data Dictionary


|Feature|Type|Dataset|Description|
|---|---|---|---|
|id|object|both|Id of the post|
|title|object|both|Title of the post (Question)|
|text|object|both|Text of the post|
|created_utc|int|both|Time created in UTC format|
|url|object|both|URL of the post|
|subreddit|object|both|Subreddit the post belongs to|


### Collection Notes

- Used the pushshift reddit api to collect posts from subreddits 
- Filtered removed posts by using the checking for the 'removed_by_category' tag
- Some submissions were removed, but didn't have the 'removed_by_category'

---


## Data Cleaning and EDA

## Early Cleaning

    - Concatenated askmen_posts.csv and askwomen_posts.csv
    - Dropped columns [created_utc, url]
    - Set all strings to lowercase

## Removed terms

    - Checked and removed links in the 'text' column
    - Removed posts where the 'text' was from the moderators
    - Removed the word "title" from text column since it was only used to refer to the title of the post
    - Removed all punctuation and numbers from both 'title' and 'text'
    - Changed nan values in 'text' to blank strings
    
## Title and Text Stats

    - Created new columns for the length of the 'title' and 'text'
    - Longest 'title' and 'text' found were 300 and 3461 characters long
    
<img src="./Images/histogram_men_titles.png" style="height: 400px width: 500px">

<img src="./Images/histogram_women_titles.png" style="height: 400px width: 500px">

- The distribution of the lengths in the titles for both men and women are similarly right skewed.

## Modeling

### Baseline

- After cleaning the data the percentages between posts from the askwomen and askmen subreddits were 50.23% and 49.76% respectively.

### Train Test Split

- X = 'title' and 'text' were combined into one variable
- y = subreddit labeled 0 or 1 for men or women
- random_state = 42
- data was stratified

---

### Logistic Regression

- CountVectorizer was used to transform the texts
- CountVector parameters: max_features, stop_words, ngram_range
- Logistic Regression parameters: C

#### Best Parameters

- cvec__max_features: 3630 - 3650
- cvec__ngram_range: (1,2)
- cvec__stop_words: None
- est__C: 0.09


#### Metrics

- Training Accuracy: ~86%
- Testing Accuracy: ~79%
- Recall: ~90%
- Precision: ~73%

<img src="./Images/lgr_confusion_matrix.png" style="height: 400px width: 500px">

#### Examining the Coefficients

<img src="./Images/top_cofficient_coefs.png" style="height: 400px width: 500px">

- For the terms ['ladies', 'women of', 'women who', 'women'] make sense. It is interesting to note that the word women is used instead of 'girl'
- Also interesting that the words 'love' and 'family' are found more in questions specified towards women

<img src="./Images/bot_cofficient_coefs.png" style="height: 400px width: 500px">

- The terms ['men', 'guys', 'man', 'men of', 'guy', 'wife'] make sense in terms of questions geared towards men
- The use of 'my' however is interesting as it may pertain to how males refer to their partners when asking about them. Also the importance of the word 'attractive' is interesting to see.

#### Looking at incorrectly classified questions

- The misclassified questions were pulled from the test data
- Then they were filtered for the top 30 most important terms
- After filtering and looking at the questions without the top key terms most of the questions that remained were gender neutral and not able to be classified even by a person.

---

### Decision Trees

- Second model was built on decision trees because of it's interpretability.
- However it's accuracy was a bit below the Logistic Regression and Random Forest Classifier

#### Best Parameters:

- cvec__max_features: 3450 - 3500
- cvec__ngram_range: (1,1)-(1,2)
- cvec__stop_words: None
- est__min_samples_leaf: 1
- est__min_samples_split:30-33

#### Metrics

- Training Accuracy: ~91%
- Testing Accuracy: ~75%
- Recall: ~81%
- Precision: ~72%

- Overall the metrics are below the Logistic Regression with the recall score being ~10% worse.

<img src="./Images/dt_confusion_matrix.png" style="height: 400px width: 500px">

#### Notes

- After optimizing on the parameters the Decision Tree performed worse in almost every category compared to the Logistic Regression model.

--- 

### Random Forest Classifier

#### Best Parameters:

- cvec__max_features: 3630 - 3650
- cvec__ngram_range: (1,1)-(1,2)
- cvec__stop_words: None
- est__min_samples_leaf: 1
- est__min_samples_split:38-40
- est__n_estimators: 40-45


#### Metrics

- Training Accuracy: ~94%
- Testing Accuracy: ~80%
- Recall: ~90%
- Precision: ~75%

<img src="./Images/rfc_confusion_matrix.png" style="height: 400px width: 500px">


#### Notes

- Althought this model had the highest metrics, since its an ensemble model it doesn't have the interpretability that DecisionTrees and Logistic Regression have. If the model had performed better by a sizeable amount there could have been consideration in using it over LogisticRegression.

## Conclusions

Overall the most optimal model was the logistic regression model as it had the most interpretability and equal or better results in accuracy, precision, and recall. One thing not accounted for in the data were completely neutral messages that didn't were asked in both subreddits. To work around this all neutral of the neutral questions could be filtered out. Another way would be to classify the neutral questions and create a non binary classification model to classify neutral, men, and women questions. The majority of keywords related to gender as expected, but there were a few key words such as family, love, and attractive that stood out. These findings could be useful for identifying behavioral patterns between men and women. 
