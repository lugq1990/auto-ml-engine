# auto-ml-cl

How to create a machine learning and deep learning models with just a few lines of code by just provide data, then framework will get best trained models based on the data we have? We don't need to care about `Data Loading`, `Feature Engineering`, `Model Training`, `Model Selection`, `Model Evaluation` and `Model Sink`, even `RESTful` with best trained model. Now this is **auto_ml_cl** comes in.

This repository is based on **scikit-learn** and **TensorFlow** to create both machine learning models and nueral network models with few lines of code by just providing a training file, if there is a test file will be nicer to evaluate trained model without any bias, but if with just one file will also be fine. Currently this repository is only support with `Classification` problem.

Key features highlights:
 - `machine learning` and `neural network models` are supported.
 - `Automatically data pre-processing` with missing, unstable, categorical various data types.
 - `Ensemble logic` to combine models to build more powerful models.
 - `Nueral network models search` with `kerastunner` to find best hyper-parameter for specific type of algorithm.
 - `Cloud files` are supported like: `Cloud storage` for GCP or local files.
 - `Logging` different processing information into one date file for future reference.
 - `Processing monitoring` for each algorithm training status.
 - `RESTful API` for API call to get prediction based on best trained model.

Sample code to use `auto_ml` package by using `Titanic` dataset from Kaggle competion, as this dataset contain different kinds of data types also contain some missing values with different threasholds.
```python

from auto_ml.automl import ClassificationAutoML, FileLoad

file_load = FileLoad(r"C:\auto_ml\test\train.csv", label_name='Survived')

auto_cl = ClassificationAutoML()
auto_cl.fit(file_load=file_load, val_split=0.2)

# Get prediction based on best trained models
file_load_test = FileLoad(r"C:\auto_ml\test\test.csv")

pred = auto_cl.predict(file_load=file_load_test)
```

Then we could get whole trained models' evaluation score for each trained model score, we could get best trained model based on validation score if we would love to use trained model for production, one important thing is that these models are stored in local server, we could use them any time with RESTFul API calls.
![Evalution result](https://github.com/lugq1990/auto_ml/blob/master/test/diff_model_score.png)
    
If we want to use GCP cloud storage as a data source for train and test data, what needed is just get the service account file with proper authority, last is just provide with parameter: `service_account_name` and file local path: `service_account_file_path` to `FileLoad` object, then training will start automatically.

```python
file_name="train.csv"
file_path = "gs://bucket_name"
service_account_name = "service_account.json"
service_account_file_path = r"C:\auto_ml\test"

file_load = FileLoad(file_name, file_path, label_name='Survived', 
    service_account_file_name=service_account_name, service_account_file_path=service_account_file_path)

auto_cl = ClassificationAutoML()
auto_cl.fit(file_load=file_load)
```

If we have data `in memory`, we could also use memory objects to train, test and predict with `auto_ml` object, just like `scikit-learn`.

```python
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split

x, y = load_iris(return_X_y=True)
xtrain, xtest, ytrain, ytest = train_test_split(x, y, test_size=.2)

auto_cl = ClassificationAutoML()
auto_cl.fit(xtrain, ytrain)

score = auto_cl.score(xtest, ytest)
pred = auto_cl.predict(xtest)
prob = auto_cl.predict_proba(xtest)
```

Current supported algorithms:
 - Logistic Regression
 - Support vector machine
 - Gradient boosting tree
 - Random forest
 - Decision Tree
 - Adaboost Tree
 - K-neighbors
 - XGBoost
 - LightGBM
 - Deep nueral network

Also supported with `Ensemble` logic to combine different models to build more powerful model by adding model diversity:
 - Voting
 - Stacking

For raw data file, will try with some common pre-procesing steps to create dataset for algorithms, currently some pre-processing algorithms are supported:
 - Imputation with statistic analysis for continuous and categorical columns, also support with KNN imputaion for categorical columns.
 - Standarize with data standard data
 - Normalize 
 - OneHot Encoding for categorical columns
 - MinMax for continuous columns to avoid data volumn bias
 - PCA to demension reduction with threashold
 - Feature selection with variance or LinearRegression or ExtraTree


Insight for logics to `auto` machine learning training steps.    
    
1. Load data from file or memory for both training and testinig with class `FileLoad`, support with GCP's `GCS` files as source file.
2. Build processing pipeline object based on data.
    
    (1). `Imputation` for both categorical and numerical data with different logic, if data missing column is over a threshold, will delete that column. Support with algorithm `KNNImputer` to impute data or `SimpleImputer` to fill missing data.
    
    (2). `OneHot Encoding` for categorical columns and add created columns into original data.
    
    (3). `Standardize` data to avoid data range, also benefit for some algorithms like `SVM` etc.
    
    (4). `MinMax` data to keep data into a 0-1 range.
    
    (5). `FeatureSelection` to keep features with a default threshold or using algorithm with `ExtraTree` or `LinearRegreesion` to select features.
    
    (6). `PCA` to reduce dimenssion if feature variance over a threshold and just keep satisfied features.
3. Build a `Singleton` backend object to do file or data related functions.
4. Build training pipeline to instant each algorithm with a `factory` class based on pre-defined used algorithms.
5. Build a `SearchModel` class for each algorithm to find best parameters based on `RandomSearch` or `GridSearch`.
6. Pre-processing pipeline `fit` and `tranform`, save trained pipeline into disk for future use.
7. Start `training` with training pipeline with processed data with doing parameters search to find `best parameter's model`, also combined with Neural network search to find best neural models. If need `validation` will use some data to do validation that will reduce training data size, or could use trainded `auto_ml` object to do validation will also be fine.
8. Use `Ensemble` logic to do `voting` or `stacking` to combine trained models as a new more diverse model based on best trained model.
9.  `Evaluate` each trained models based on validation data and return a ditionary with `training model name`, `training score` and `validation score`.
10.  Support to `export trained models into a new folder` that we want.
