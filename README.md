<img src="https://user-images.githubusercontent.com/64495168/129553804-9baec55b-e3bf-407c-a5f5-8b229490bd27.png" alt="Rossmann logo" title="Rossmann" align="right" height="60" class="center"/>


# Rossmann stores - 6 weeks sales forecast

**Disclaimer**: this project was inspired by the "Rossmann Store Sales" challenge published on kaggle (https://www.kaggle.com/c/rossmann-store-sales). It is a fictitious project but with all the steps of a real project.

## Business scenario
The sales director of the Rossmann stores wants to estimate the sales forecast for the next 6 weeks on its different units spread across Europe.

## Solution methodology
The resolution of the challenge was carried out following the CRISP (CRoss-Industry Standard Process for data mining) methodology, which is a cyclical approach that streamlines the delivery of value.

![crisp_cycle](/img/crisp_cycle.png)

## Data collection and understanding

The first step was to collect (from kaggle) and understand the data; soon after, the cleaning of the database and treatment of missing values took place. <br>
There are 1017209 sales records for 1115 different stores, containing different attributes such as: "date", "store_type", "customers", "assortment", "school_holiday", "open", "promo2", "sales", among others. The explanation of each of the attributes is available on the notebook. <br>
To complete the data understanding step, features that will not be available at the time of the forecast were removed, such as the number of customers, which will only be known on the day of sales and, therefore, it would be impractical to train the model with such variable.

## Exploratory data analysis guided by a mind map of hypotheses 

The next step was to perform exploratory data analysis (EDA). But right before that, a mind map of hypotheses was made in order to guide the EDA, to generate insights and to understand a little more about the database and the most important attributes. 

![MindMapHypothesis](/img/MindMapHypothesis.png)

With the feature diagram above, several hypotheses were generated; the ones that were judged to be most relevant were selected (listed below) and then the EDA actually started.

1. Stores with a larger assortment should sell more
2. Stores with closer competitors should sell less
3. Stores with longer-standing competitors should sell more
4. Stores where products cost less for longer (active promotions) should sell more
5. Stores with more promotion days should sell more
6. Stores with more extended promotions should sell more
7. Stores open during Christmas holiday should sell more
8. Stores should sell more over the years
9. Stores should sell more in the second half of the year
10. Stores should sell more after the 10th day of each month
11. Stores should sell less on weekends
12. Stores should sell less during school holidays
13. Stores that open on Sundays should sell more

The discussion of each hypothesis to validate or refute it is found in the notebook file. <br>
Below are the summary of the analysis of hypotheses 1, 10 and 13:

### 1. Stores with a larger assortment should sell more: **true**

By plotting the moving average of the last 4 weeks over the entire period available in the database, it is noted the better performance, in sales, of stores that have 'extra' assortment, compared to the 'extended', which in turn is better than the 'basic'.

![h1](/img/h1.png)

It is also noted that there seems to be a tendency to accentuate the difference of stores with extra assortment as time goes on.

### 10. Stores should sell more after the 10th day of each month: **false**

After analyzing the composition of the 4 graphs related to this hypothesis, it is noted that before the 10th, on average, stores sell more. 

![h10](/img/h10.png)

The confusion matrix reveals a slight trend for the number of the day to be inversely correlated with the sales result.

### 13. Stores that open on Sundays should sell more: **true**

After identifying the different stores that open on Sundays (which most stores remain closed), the average sales performance between these stores that never open on Sundays vs stores that always open on Sundays was compared; to be a fair comparison, the Sunday sales of these last stores were excluded, so only the performance of Monday to Saturday sales was compared.

![h13](/img/h13.png)

Perhaps this analysis is an insight and investigation of such superior performance is recommended. <br>
One possible reason could be the store's location (which makes sense to open it on Sundays); another reason may be that such stores that do not close on any day of the week result in increased confidence by customers in "finding the doors open" and therefore looking for such stores regardless of the day of the week. <br>
Of course, it could still be another factor or combination of them.

## Data preparation (standardization and feature selection)

After EDA, data preparation was performed, where rescaling and encondings were applied. <br>
As a highlight, there is the transformation of cyclical data (such as day and week) using the sine and cosine functions to leave such variables correctly spaced according to the calendar, for example approximating the end and beginning of the month (image below) or the end and the beginning of the year.
 
![sin_cos](/img/sin_cos.png)

The next step was to identify the most relevant features for training machine learning models.
For this, in addition to the knowledge acquired during EDA, the Python implementations of the Boruta R package (https://github.com/scikit-learn-contrib/boruta_py) was used. <br>
The features chosen by Boruta are described in the notebook.

## Machine learning modeling

Four different models (linear regression, regularized linear regression - lasso, random forest and XGBoost ) were evaluated using the cross-validation on a rolling basis, schematically represented below.

![ts_cross_validation](/img/ts_cross_validation.png)

It started with a reduced portion of the training database, whose last 6 weeks were separated for validation; then, the model was trained and its performance was calculated. <br>
New iterations were performed, each time increasing the training dataset and always separating the last 6 weeks for the test. <br>
The cross-validation performance was the average of each of these iterations.

The results in terms of Mean Absolute Error (MAE), Mean absolute percentage error (MAPE) and Root Mean Square Error (RMSE) were:

|Model|MAE|MAPE|RMSE|
|-----------------------------|------------------|-------------|------------------|
|Random forest regressor      |853.71 +/- 257.13 |0.12 +/- 0.03|1297.01 +/- 400.11|
|XGBoost regressor            |1068.62 +/- 165.13|0.15 +/- 0.02|1531.94 +/- 235.94|
|Linear regression            |2081.69 +/- 295.28|0.3 +/- 0.02 |2952.57 +/- 468.48|
|Regularized linear regression|2116.43 +/- 341.25|0.29 +/- 0.01|3057.75 +/- 503.93|
	
Although the random forest model was the best, the model chosen to go ahead with the tuning of the hyperparameters was XGBoost. The reason for this is that it is a much lighter model to operate in production and does not have a significant difference in performance; the operability in production is an extremely important requirement in this project.

## Hyperparameter tuning
Using the random search precedure,  with different values for the parameters "n_estimators", "eta", "max_depth", "subsample", "colsample_bytree" and "min_child_weight", 25 different iterations of XGBoost were performed, all evaluated using cross-validation. 
The values of MAE, MAPE and RMSE are detailed in the notebook for all the iterations. <br>
The performance of the chosen model, considering performance and size(keeping in mind the operability in production), was:

|Model|MAE|MAPE|RMSE|
|----------------|----------------|-------------|-----------------|
XGBoost regressor|972.0 +/- 166.53|0.13 +/- 0.02|1409.8 +/- 247.92|

And then the model was trained with the entire training data: 

```python
model_xgb_tuned = xgb.XGBRegressor(objective = 'reg:squarederror',
                                   n_estimators = 500,
                                   eta = 0.03,
                                   max_depth = 9,
                                   subsample = 0.7,
                                   colsample_bytree = 0.9,
                                   min_child_weight = 15).fit(x_train,y_train)
```

The performance of the test data was:

|Model|MAE|MAPE|RMSE|
|----------------|------|----|-------|
XGBoost regressor|803.56|0.12|1176.21|

## Business performance

Finally, with the model trained, it's time to translate model performance into business performance.
Considering the MAE obtained in the forecast for each store, during the test period, the best and worst sales scenarios for each store were projected.

Below, the expected business performance of the first 5 stores:

|store|predictions|MAE|MAPE|days|worst_scenario|best_scenario|
|-|---------|------|----|--|---------|---------|
|1|169305.70|308.78|0.07|37|157881.01|180730.40|
|2|182122.09|395.43|0.08|37|167491.11|196753.09|
|3|258817.89|538.91|0.08|37|238878.23|278757.55|
|4|338596.25|944.09|0.09|37|303664.96|373527.54|
|5|174173.37|388.33|0.09|37|159805.20|188541.55|

The overall performance of the model can be represented in the graphs below, where: <br>
<br>
error_rate = predictions/sales <br>
error = sales - predictions <br>

<br>

![performance](/img/performance.png)

Overall, the model performed well. <br>
But it is always possible to improve it; following the CRISP methodology, if a new round is needed, it may be considered to train stores individually or even a smaller group of them, for example. Another possibility is to explore other machine learning models.  <br>
However, the deadline for delivery of forecasts and the performance of the model already in production must be taken into account. Something very heavy or time-consuming is also impractical, even if it performs exceptionally well. <br>
It is a trade-off that must be closely aligned with the company's management. <br>
Further details on business performance are available on the notebook.

## Model in production

The model was finally put into production and operated via a Telegram chatbot. For this, in addition to the final trained model, a class in python was created with the entire data processing pipeline, an API handler and an application to manage the messages. All files were hosted on Heroku (https://www.heroku.com/); the production data was also stored in its cloud. <br>

The following scheme represents all these files.

![app](/img/app.png)

A demonstration of the app: <br>

![rossmann](/img/rossmann.gif) <br />
<p>&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp chatbot id: @rossmann_rnf_bot  </p>
     






