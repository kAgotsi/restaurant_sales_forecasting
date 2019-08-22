# Art in the Age of Machine Learning
## Predictive Analytics for Restaurants

### Project Scope

The restaurant industry operates with notoriously thin operating margins, typically dropping around 5% of total revenue to their bottom lines. For every $1.00 in sales, approximately $0.30 - $0.35 will likely go to cost of goods (e.g. raw ingredients, dry goods, alcohol) while an additional $0.30 - $0.35 will go to labor, leaving roughly $0.30 - $0.40 to cover overhead (rent, insurance, utilities, equipment rentals, waste management, accounting, marketing) and provide a return on capital, to the extent there is any excess after covering all the costs.

An accurate sales forecast provides several avenues of cost saving + revenue expansion to restaurants:

* **Labor Optimization** | Minimize labor hours on slow nights while maintaining appropriate staffing levels on busier nights.
* **Minimize Food Waste** | More precise daily prep levels to help limit food waste.
* **Lean Inventory** | Inventory is a drag on cash flow and high levels of inventory of fresh goods can lead to spoilage and food waste.
* **Marketing Initiatives** | Maximize marketing dollars by targeting slow nights and weeks.

Using real sales data from a two-star restaurant located in Brooklyn, NY, this is a pilot project exploring what an end-to-end restaurant sales forecasting tool utilizing machine learning would entail. 

### Data Sources
* **Restaurant Sales Data** | Restaurant sales and guest counts ("covers") were downloaded directly from the restuarant's Point of Sale ("POS") on a check-by-check basis from 1/1/2017 through 06/30/2019, then aggregated into nightly totals covering the Dinner period only, and for the moment, excluding the outdoor area revenue center. 

* **Reservations & Covers Data** | Reservations and covers data was downloaded directly from the restaurant's Resy platform. Resy data was considered the ground truth for covers data, however only aggregate data was available. Inside and outside cover counts were imputed from POS & Resy data. Total covers is equal to total Resy cover counts.

* **Weather Data** | Weather data was accessed via the DarkSky API, as of 7:30 PM each day. The source code for the API call can be found [here](https://github.com/maks-p/restaurant_sales_forecasting/blob/inside-sales-only/weather.py "weather.py"). The latitude and longitude for the restuarant, required to access the weather data, is accessed via the Yelp API - the source code for this API call is found [here](https://github.com/maks-p/restaurant_sales_forecasting/blob/master/restaurant_info.py "restaurant_info.py"). 

### Repository Guide
* [Final Model](https://github.com/maks-p/restaurant_sales_forecasting/blob/master/final_model.ipynb "Final Model")
* [EDA Notebook](https://github.com/maks-p/restaurant_sales_forecasting/blob/master/final_eda.ipynb "EDA")
* [AWS RDS Set Up](https://github.com/maks-p/restaurant_sales_forecasting/blob/master/load_aws_rds.ipynb "AWS RDS")
* [Yelp API Call](https://github.com/maks-p/restaurant_sales_forecasting/blob/master/restaurant_info.py)
* [DarkSky API Call](https://github.com/maks-p/restaurant_sales_forecasting/blob/master/weather.py)


### Restaurant Background

The subject is a two-star restaurant located in Brooklyn, NY. It has a patio that adds a substantial amount of seats. The subject is performing extremely well, earning $5.14 MM in revenue on 64,360 covers on indoor sales only (indoor revenue centers include the dining room, bar area, and a private dining room). For completeness, the outside area earned $733,000 of revenue in 2018, however as noted, the current analysis only covers the inside area.

The following two charts demonstrate the restaurant's performance on a monthly basis:

<p float="left">
  <img src="https://user-images.githubusercontent.com/42282874/60995179-ffa7a000-a31f-11e9-80ce-464b11728c0f.png" width="425" />
  <img src="https://user-images.githubusercontent.com/42282874/63281291-5b980980-c27a-11e9-84b6-1f893896886f.png" width="425" /> 
</p>

The restaurant is busy all year, with minor seasonal effects (as noted, the restaurant does additional business outside during the warmer months). And like most restaurants and bars, the restaurant is busiest on weekends.

<p float="left">
  <img src="https://user-images.githubusercontent.com/42282874/63281372-88e4b780-c27a-11e9-9a1a-072843260557.png" width="425" />
  <img src="https://user-images.githubusercontent.com/42282874/63281500-d82ae800-c27a-11e9-829c-6bc75dafb879.png" width="425" /> 
</p>

The following heatmap shows Average Sales for the restaurant by Day of Week & Month:

![heatmap](https://user-images.githubusercontent.com/42282874/63281562-f7297a00-c27a-11e9-930f-656c418bc3f7.png)

In 2018, the last full year of operations, the subject averaged nightly sales of $14,319 inside, with a standard deviation of $1,703.

All charts and scoring as of June 30th, 2019.

### Sales Distribution by Day
![sales_distribution](https://user-images.githubusercontent.com/42282874/63281684-39eb5200-c27b-11e9-8ae9-0405db45587a.png)

Weekends have a bit of a wider distribution than the rest of the week, reflecting a higher Standard Deviation.

### Baseline Multilinear Regression

A simple Mulilinear Regression Model that accounts for the outside space being open, closed days, and day of week (as dummy variables) performs fairly well, and provides a jumping off point for more extensive feature engineering, including weather data.

```
Root Mean Squared Error:  1224.49
Mean Absolute Error:  954.59 
```

### Correlation

There is a small, but present, correlation between the apparent temperature and inside sales, though we have not yet isolated the effect of temperature versus the effect of seasonality.

![sales_temp_ corr](https://user-images.githubusercontent.com/42282874/63281818-8afb4600-c27b-11e9-987b-21afed92ba11.png)

### Outliers

Sales and covers values three standard deviations or more from the mean were imputed with the median value for that particular day (i.e. if an outlier fell on a Wednesday, the value was replaced with the Wednesday median value). Overall, 12 outlier days were imputed for Sales data (1.3% of total).

### Feature Engineering

The following features were engineered as part of the modeling process:

* **Month Clusters** | K-Means Clustering was used to reduce dimensionality by clustering months together into four separate groups on the basis of historical sales central tendencies (median, standard deviation and max).

* **Sales Trend** | The ratio of 7-Day Moving Average over the 28-Day Moving Average reflects short term sales momentum.

* **Temperature Bins** | Temperature was converted from a continuous variable to a categorical one using KBinsDiscretizer (KMeans).

* **Precipitation While Open** | True if the maximum precipitation for the day occurred during service hours (above a minimum threshold). A proxy for rain during service.

* **Other Weather Features** | Humidity, Precipitation Probability (as of 7:30 PM).

* **Holiday & Sunday Three Day Weekend** | Calendar features.

* **Interaction Variables** | Between Temperature Bins & Outside Boolean.

All Feature Engineering is wrapped in custom transformers and included in either a "Pre Processing Pipeline" or "Post Processing Pipeline."


### Modeling Process

#### Multilinear Regression
A Multilinear Regression Model with Lasso Regularization after Feature Engineering performed better than the Baseline Linear Regression:

```
Root Mean Squared Error:  1202.13
Mean Absolute Error:  907.57
```

#### Feature Importance & Residuals Check - Multilinear Regression with Lasso Regularization
<p float="left">
  <img src="https://user-images.githubusercontent.com/42282874/63528876-bbd4b880-c4d1-11e9-88d5-fc1f7255ce82.png" width="425" />
  <img src="https://user-images.githubusercontent.com/42282874/63528875-bbd4b880-c4d1-11e9-999d-333f5707d2ed.png" width="425" /> 
</p>
The residuals are well distributed with no discernible pattern

#### XGBoost Regression
XGBoost is a popular algorithm built on a gradient boosting framework. Gradient boosting is a sequential ensemble method built on the idea that weak predictors (or "learners") can learn from the "mistakes" (or residuals) of previous iterations.

An XGBoost Regressor with GridSearchCV parameter tuning built the best model:

```
Root Mean Squared Error:  1130.60
Mean Absolute Error:  857.40
```

The best esimator after Grid Search:
```
Grid Search Best Estimator:  XGBRegressor(base_score=0.5, booster='gbtree', colsample_bylevel=1,
             colsample_bytree=0.775, gamma=0, importance_type='gain',
             learning_rate=0.02, max_delta_step=0, max_depth=3,
             min_child_weight=2, min_impurity_decrease=0.0001, missing=None,
             n_estimators=275, n_jobs=1, nthread=None, objective='reg:linear',
             random_state=0, reg_alpha=0, reg_lambda=1, scale_pos_weight=1,
             seed=None, silent=True, subsample=1)
```

#### Feature Importance & Residuals Check - XGBoost with GridSearchCV
<p float="left">
  <img src="https://user-images.githubusercontent.com/42282874/63530325-40283b00-c4d4-11e9-8540-b3584048bd75.png" width="425" />
  <img src="https://user-images.githubusercontent.com/42282874/63530328-41f1fe80-c4d4-11e9-93c7-467143c7291f.png" width="425" /> 
</p>

Random Forest Regression and Time Series Analyses were also performed as part of this project. The XGBoost Regression was the best model on the basis of RMSE & MAE.

### Model Evaluation

The following table shows the Mean Absolute Error by day of week:
	
| day_of_week   | mae    |	
| ------------- |:-------:|
| 0	            | 712.64	|
|1	            |  868.38	|
|2	            |  920.18	|
|3	            |  684.04	|
|4	            |  937.02|
|5	            |  1000.91	|
|6	            |  874.07 |

The MAE overall was $857.40, or 5.70%.

### Predictions

Predictions are made for the upcoming night & week:

		
| date			|	predicted_inside_sales		|
|---------|:---------:|
2019-07-01	| 12981.33
2019-07-02	| 12830.47
2019-07-03	| 13217.37
2019-07-04	| 13295.29
2019-07-05	| 14842.32
2019-07-06	| 15908.46
2019-07-07	| 14242.5

### Planned Upgrades

My intention is to build a model that can theoretically be productionized and applied to a generalized restaurant population. The first challenge is to build a robust data ingestion process that can handle different revenue centers (inside, outside, etc.), and day parts (lunch, dinner).
