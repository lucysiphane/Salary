# Salary Prediction
## Overview
This project implemts multlinear regression model to predict academic salaries based on several professional atrributes.
The model uses both numerical and categorical features to estimate salaries of faculity members.

### Features used
The model utilized the following predictors:

1.**Years since Phd(yrs.since.phd)**-
Continuous numerical feature
2.**Years of service(yrs.service)**_continuous numerical feature

3.**Academic Rank(rank)**-Categorical feature with 3 levels;
**Assistant Professor*
**Associate Professor*
**Full Professor*

4.**Sex**-binary categorical features

## Load the nesseccary libraries
```python
import pandas as pd
import numpy as np
from category_encoders import OneHotEncoder
from sklearn.linear_model import LinearRegression, Ridge
from sklearn.metrics import mean_absolute_error
from sklearn.pipeline import make_pipeline
import plotly.express as px
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.impute import SimpleImputer
```
## load the data
Detail loading of the data using panda
```python
data=pd.read_csv("salary.csv")
data.head(5)
data.info()
```
### Data Exploration
***Scatter plot***- it shows relationship between the years of service and salary.
```python
plt.scatter(x=data["yrs.service"], y=data["salary"])
plt.xlabel("yrs.service")
plt.ylabel("salary")
plt.title("Salary vs yrs.service")
```
***Correlation HeatMap**-it shows the correlation between the numerical feature matrix
```python
#correlation between variables in the feature matrix
corr=data.select_dtypes("number").drop(columns="salary").corr()
corr
sns.heatmap(corr)
```
***Histogram of Salary distribution**-It shows the general distibiution of the salary(target matrix)
```python
#histogram of target matrix
plt.hist(data["salary"])
plt.xlabel("salary")
plt.ylabel("frequency")
plt.title("Distribution of Salary")
```
### Split the Matrix
Split the data into Feature matrix and target matrix
```python
#split of data into training data
target="salary"
features=['rank','discipline', 'yrs.since.phd', 'yrs.service', 'sex']
y_train=data[target]
X_train=data[features]
X_train.shape
y_train.shape
```
### Make a baseline model
make a baseline model
```python
y_mean=y_train.mean()
y_pred_baseline=[y_mean]*len(y_train)
print("Mean salary:", mean_absolute_error(y_train,y_pred_baseline))
```
### Build the model
Pipeline Components:
OneHotEncoder: Handles categorical variables (rank, discipline, sex)

Ridge Regression: Regularized linear regression model
```python
model=make_pipeline(
    OneHotEncoder("use_cat_name=True"),
    Ridge()
)
model.fit(X_train,y_train)
```
### Model Evaluation 
Training MAE (Mean Absolute Error): Calculated during development

Baseline comparison against mean salary prediction
```python
y_pred_training= model.predict(X_train)
mae_training= mean_absolute_error(y_train,y_pred_training)
mae_training
```
### test the model
```python
X_test=pd.read_csv("salary.csv")[features]
y_pred_test=pd.Series(model.predict(X_test))
y_pred_test.head(5)
```
### Getting the intercept 
The intercept term represents the baseline salary prediction when all feature values are zero. In our Ridge regression pipeline, we access it through:
```python
intercept = model.named_steps['ridge'].intercept_
intercept
```
The intercept is the expected salary when:

yrs.since.phd = 0

yrs.service = 0

All categorical variables are at their reference levels:

rank = AsstProf (first alphabetically)

discipline = A

sex = Female
### Getting the coffiecient
The coefficients in our salary prediction model quantify the relationship between each feature and the target salary amount. We access them through:
```python
coefficients = model.named_steps['ridge'].coef_
feature_names = model.named_steps['onehotencoder'].get_feature_names()
feat_imp = pd.Series(coefficients, index=feature_names)
```
### Horizontal Bar Chart of Feature Importance
This visualization displays the most influential features in our salary prediction model, ranked by the absolute value of their coefficients.
```python
feat_imp.sort_values(key=abs).tail(15).plot(kind="barh")
plt.xlabel("Salary")
plt.ylabel("Feature")
plt.title("Feature Important for Salary")
```
### Interpretation Guide
#### What the Chart Shows
**Directionality**:

Bars extending right = Positive impact on salary

Bars extending left = Negative impact on salary

**Magnitude**:

Longer bars = Stronger influence on predicted salary

Shorter bars = Weaker influence
### Salary Prediction Function Documentation
#### make_predictions() Function
This function provides an interface to get salary predictions from our trained machine learning model using individual faculty characteristics.

**Function Definition**
```python

def make_predictions(rank, discipline, yrs_since_phd, yrs_service, sex):
    df = {
        "rank": [rank],
        "discipline": [discipline],
        "yrs.since.phd": [yrs_since_phd],
        "yrs.service": [yrs_service],
        "sex": [sex]
    }
    data = pd.DataFrame(df)
    predictions = model.predict(data).round(2)[0]
    return f"predicted salary: {predictions}"

````
### Prediction Function Testing
This section demonstrates how to test the trained salary prediction model using the make_predictions() function.
```python
#Test the model by using the function
result = make_predictions("Prof", "B", 15, 10, "Male")
print(result)
```

