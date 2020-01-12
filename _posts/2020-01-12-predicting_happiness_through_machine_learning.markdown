---
layout: post
title:      "Predicting Happiness Through Machine Learning"
date:       2020-01-12 09:08:14 +0000
permalink:  predicting_happiness_through_machine_learning
---

On March 20, 2019, Fortune reported that [the U.S. is the unhappiest it's ever been](https://fortune.com/2019/03/20/u-s-unhappiest-its-ever-been/).

This project started with this news article and the annual World Happiness Report referenced therein. 

Using the 2018 data from the [General Social Survey](https://gss.norc.org), which was also cited in the [2019 World Happiness Report](https://worldhappiness.report/ed/2019/the-sad-state-of-happiness-in-the-united-states-and-the-role-of-digital-media/), I will try to better understand America's unhappiness.

### Target variable
Despite the large number of total observations, the target variable, `happy`, suffered from class imbalance problem, with only 14% of respondents who are `not too happy`.

![](https://imgur.com/6fOXV83.png)

### Feature Selection
I only considered variables that had less than 1% missing observations. The missing threshold is very strict for the following reasons:
1. The target variable has a class imbalance problem, and the missing observations in the minority classes would worsen the class imbalance problem.
2. Some features have missing observations by construction. More specifically, if a respondent does not have a job, he/she won't be asked the follow-up question on how many hours he/she worked in the past week. Therefore, such features with missing observations are likely to have another features with less missing observations that have already covered the same topic, but with less detailed information.
3. The row dataset has 2,348 features. Considering how most of the variables are categorical variables, the train data set would increase exponentially when the categorical variables are converted to dummy variables. Considering only features with the most comprehensive information would reduce the computation time. 

Ideally, I would review and analyze each question asked, account for missing values, and even create new features accordingly.

For feature selection, I first looked at 20 continuous and 20 categorical variables with highest correlation with the target variable.

Calculating correlation for continous variables is straightfoward:
```
corr = df['happy'].corr(df[col])
```

For categorical variables, I calculated correlation ratio, i.e. weighted variance of the mean of each category divided by the variance of all samples. The underlying calculations are well documented in [this article](https://towardsdatascience.com/the-search-for-categorical-correlation-a1cf7f1888c9).

Additionally, I identified potentially important variables after manually reviewing the data dictionary.

Finally, I included variables with high feature importances in the optima vanilla model for:
1. All features that satisfy 1% missing observations threshold.
2. (1) but restricted to those whose target variable is either `very happy` or `not too happy`.
I followed the same steps as outlined in the section below.

### Choosing Optimal Algorithm
After one-hot-encoding the categorical variables and min-max scaling the dataset, I also accounted for class imbalance problem by using SMOTE.

Then, I ran multiple vanilla models on the resampled train dataset.
```
# oversampled data, features selected by correlation
pipe_tree = Pipeline([('clf', DecisionTreeClassifier(random_state=123))])
pipe_forest = Pipeline([('clf', RandomForestClassifier(random_state=123))])
pipe_adaboost = Pipeline([('clf', AdaBoostClassifier(random_state=123))])
pipe_gbt = Pipeline([('clf', GradientBoostingClassifier(random_state=123))])
pipe_xgb = Pipeline([('clf', XGB.XGBClassifier(random_state=123))])

# List of pipelines, List of pipeline names
pipelines = [pipe_tree, pipe_forest, pipe_adaboost, pipe_gbt, pipe_xgb]

# Loop to fit each of the three pipelines
for pipe in pipelines:
     print(pipe)
     pipe.fit(X_train_resampled, y_train_resampled)
```

![](https://imgur.com/P3b1PT6.png)

Interestingly, random forest performed better than any of the gradient boosting algorithms. 

### Refine the model through GridSearchCV

I performed multiple grid searches for efficiency. I ran initial grid search of parameters with wide ranges, then narrowed down the possibilities further.

###### 1. Initial grid search of the parameters
```
corr_param1 = {
    'clf__criterion': ["gini", "entropy"],
    'clf__n_estimators': [50, 100, 200, 300], 
    'clf__max_features': ['auto', 'sqrt', 0.2],
    'clf__min_samples_leaf': [1,5,10,50,100]
}

gs_corr1 = GridSearchCV(estimator = pipe_forest,
                       param_grid = corr_param1,
                       scoring='f1_weighted',
                       cv=5, verbose=2, n_jobs=-1,
                       return_train_score = True)

gs_corr1.fit(X_train_resampled, y_train_resampled)
 
print(gs_corr1.best_score_)
gs_corr1.best_params_
```
```
0.7771727413874162
{'clf__criterion': 'gini',
 'clf__max_features': 'auto',
 'clf__min_samples_leaf': 1,
 'clf__n_estimators': 300}
```

The optimal value for n_estimators is 300. Therefore, the model could possibly improve with higher n_estimators value.

Since the optimal min_samples_leaf is 1, I would also double check the values between 1 and 5.

###### 2. Further tune n_estimators and min_samples_leaf
```
corr_param2 = {
    'clf__criterion': ["gini"],
    'clf__n_estimators': [250, 300, 350, 500], 
    'clf__max_features': ['auto'],
    'clf__min_samples_leaf': [1,2,3,4]
}

gs_corr2 = GridSearchCV(estimator = pipe_forest,
                      param_grid = corr_param2,
                      scoring='f1_weighted',
                      cv=5, verbose=2, n_jobs=-1,
                      return_train_score = True)

gs_corr2.fit(X_train_resampled, y_train_resampled)
print(gs_corr2.best_score_)
gs_corr2.best_params_
```

```
0.7802364330331396
{'clf__criterion': 'gini',
 'clf__max_features': 'auto',
 'clf__min_samples_leaf': 1,
 'clf__n_estimators': 500}
```

The optimal min_samples_leaf is still 1. On the other hand, the optimal n_estimators is the largest value in the range, so I would further tune n_estimators.

##### 3. Further tune n_estimators
```
corr_param3 = {
    'clf__criterion': ["gini"],
    'clf__n_estimators': [400, 500, 700, 1000], 
    'clf__max_features': ['auto'],
    'clf__min_samples_leaf': [1]
}

gs_corr3 = GridSearchCV(estimator = pipe_forest,
                      param_grid = corr_param3,
                      scoring='f1_weighted',
                      cv=5, verbose=2, n_jobs=-1,
                      return_train_score = True)

gs_corr3.fit(X_train_resampled, y_train_resampled)
print(gs_corr3.best_score_)
gs_corr3.best_params_
```

```
0.7808123505678607
{'clf__criterion': 'gini',
 'clf__max_features': 'auto',
 'clf__min_samples_leaf': 1,
 'clf__n_estimators': 400}
```

With n_estimators of 400, the best score improved slightly. 

##### 4. Tune max depth and min_samples_split
```
corr_param4 = {
    'clf__criterion': ["gini"],
    'clf__n_estimators': [400], 
    'clf__max_features': ['auto'],
    'clf__min_samples_leaf': [1],
    'clf__max_depth': [None, 3, 5, 7, 9],
    'clf__min_samples_split': range(1,6)
}

gs_corr4 = GridSearchCV(estimator = pipe_forest,
                      param_grid = corr_param4,
                      scoring='f1_weighted',
                      cv=5, verbose=2, n_jobs=-1,
                      return_train_score = True)

gs_corr4.fit(X_train_resampled, y_train_resampled)
print(gs_corr4.best_score_)
gs_corr4.best_params_
```

```
0.7823370725644987
{'clf__criterion': 'gini',
 'clf__max_depth': None,
 'clf__max_features': 'auto',
 'clf__min_samples_leaf': 1,
 'clf__min_samples_split': 3,
 'clf__n_estimators': 400}
```

The default max_depth value turned out to be the best. I used these results as parameters for the final model.

### Results
```
# Final random forest model with optimal parameters
rf_corr_final = RandomForestClassifier(criterion='gini',
                                  n_estimators=400,
                                  max_depth=None,
                                  max_features='auto', 
                                  min_samples_leaf=1,
                                  min_samples_split=3,
                                  random_state=123)

rf_corr_final.fit(X_train_resampled, y_train_resampled)
rf_corr_final_pred = rf_corr_final.predict(X_test_scaled)
```

The model performance is not ideal, with accuracy at 0.64. However, considering the class imbalance problem and how the model is trying to predict the entire nation, it is impressive how no prediction is off by two categories. No 'very happy' observations were predicted to be 'not too happy' and vice versa.

![](https://imgur.com/EqMnlN8.png)

Furthermore, the model still provides us insight on what features are important in predicting happiness. Financial/social satisfaction level, mental health, and cohabitation status could explain happiness level to some extent.

![](https://imgur.com/alCozqv.png)

### Results for extreme cases
I did similar exercise only considering the minority classes, `very happy` and `not too happy`. After all, it is important to know what differentiates very happy people from not so happy people. This way, the class imbalance problem is not as serious as well.

Unlike when I considered all three levels of happiness, the best model was using XGBoost.

```
# Final XGB model with optimal hyperparameters
xgb_bi_final = XGB.XGBClassifier(learning_rate = 0.1,
                                 n_estimators = 70,
                                 max_depth = 4,
                                 min_child_weight = 1,
                                 gamma = 0.3,
                                 subsample = 0.8,
                                 colsample_bytree = 0.55,
                                 reg_alpha = 1e-05,
                                 scale_pos_weight = 1,
                                 random_state=123)

xgb_bi_final.fit(X_train_resampled_bi, y_train_resampled_bi)
xgb_bi_final_pred = xgb_bi_final.predict(X_test_scaled_bi)
```

The model performance increasedsignificantly, with accuracy at 0.89.

![](https://imgur.com/2UUEPc0.png)

Financial/social satisfaction level, mental health, and cohabitation status were still important features to explain happiness.

![](https://imgur.com/thhAJ6E.png)

