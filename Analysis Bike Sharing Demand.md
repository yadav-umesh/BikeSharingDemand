# Bike Sharing Demand
## Introduction
In this analysis the demand of bikes is predicted.
First the interpretation of the problem is stated, then the data is explored, first by looking at the data qualitatively and then quantitatively with some exploratory analyses. Out out these explorations assumptions of the behaviour are made.
Thereafter the prediction is performed. Also the different methods and tools used are explained. The results will be evaluated and recommendations for future work are given.

##Problem definition
"Bike sharing systems are a means of renting bicycles where the process of obtaining membership, rental, and bike return is automated via a network of kiosk locations throughout a city. Using these systems, people are able rent a bike from a one location and return it to a different place on an as-needed basis." [Kaggle](https://www.kaggle.com/c/bike-sharing-demand)
By predicting the demand of bicycles, the allocation of resources can be optimized.

Hourly rental data is provided with weather information. the data span 2011 to 2013. The train data consists of the first 19 days and the test data the rest of the month.

## Exploration
### Qualitative exploration
The dataset contains the following attributes:

|Attribute | Data level | Description|
---|---|---|
|datetime | Timestamp | Used to determine behaviour over time
|season | Nominal | In the summer and spring probably more bicycles rent
|holiday | dichotomous | When holiday is true probably more bicycles are rent
|workingday | dichotomous | In cities where the bikes are used for leisure less bikes are rent during working days, when mainly used by e.g. commuters, probably more bikes are used during working days
|weather | Ordinal | When the weather is better (1=best, 2=wost) more bikes are rent
|temp | Interval | Temp probably results in multicollinearity with atemp since they measure practically the same.
|atemp | Interval | Atemp is probably then a better predictor, since this has more influence on customers' behaviour
|humidity | Ratio | Probably the less humid more bikes are rent
|windspeed | Ratio | Probably the less windy more bikes are rent
|casual | Ratio |  number of non-registered user rentals initiated
|registered | Ratio | number of registered user rentals initiated
|count | Ratio | This is the number of bikes rent, the attribute that has to be predicted

Taking a look at the graph date vs. count, it can be seen that: the demand of bikes has a bit of a sinus behaviour, probably due to seasons, and the demand increases over time. (Or the weather conditions have changed significantly in two years, which, after checking, appears not the case) 

![Count of bike rent](img/CountOfBikeRent.png "Count of bike rent")

### Exploration with SPSS
To look at the behaviour of the data in relation to the count of bike rentals I used SPSS since it provides clear tables and descriptives.

A first glance at the count shows the following basics:
![Descriptives ](img/DescriptiveCount.png "Descriptives")

```sh

DESCRIPTIVES VARIABLES=count
  /STATISTICS=MEAN STDDEV VARIANCE RANGE MIN MAX SEMEAN.
```

I used further exploration to see in what way the attributes affect the bike count.

```sh
EXAMINE VARIABLES=count BY season
  /PLOT BOXPLOT STEMLEAF HISTOGRAM
  /COMPARE GROUPS
  /STATISTICS DESCRIPTIVES EXTREME
  /CINTERVAL 95
  /MISSING LISTWISE
  /NOTOTAL.
```

This roughly shows for example the seasonal change in bike demand and the correlation with the weather as explained in the previous section.
To examine the relevance of the attributes (the interval and ratio and the dichotomous attributes, see the table above) I started off with a simple linear regression.
This also gave insight in, for example the multicollinearity of 'temp' and 'atemp'. This means that they pretty much measure the same behaviour. As assumed in the table above 'atemp' is significantly a better predictor than 'temp'.

After determining the coefficients I used excel for calculating the predicted counts and the RMSLE. For testing the model I randomly picked a subset of train.csv: 67% training data and 33% testing data.

```sh
DATASET ACTIVATE DataSet4.
REGRESSION
  /MISSING LISTWISE
  /STATISTICS COEFF OUTS R ANOVA COLLIN TOL
  /CRITERIA=PIN(.05) POUT(.10)
  /NOORIGIN 
  /DEPENDENT count
  /METHOD=STEPWISE atemp humidity windspeed days hours atempS humidityS windspeedS daysS hoursS 
    spring summer fall winter better good bad worse.
```
I only included the dichotomous (dummy variables), interval and ratio attributes. The nominal attributes are split by indicator coding into: summer, winter, spring and fall, and better, good, bad and worse for season and weather respectively.
For simplicity I took the date as hours from the start of the dataset. Also I included hour of the day. I am aware of its cyclic behaviour but for now I took it as a (curve)linear relation.
For example 'hour of the day' has no linear relation. Therefore I tested those attributes on curvilinearity by squaring them.

![Regression models ](img/LinearRegression.png "Regression models")
This Model 11 shows all significant attributes. Those coefficients are used to calculate the predicted count of bike demand. 

This rough estimation gave already a Root Mean Squared Logarithmic Error (RMSLE) of 1.31.

### First conclusions
After some data preparation like correcting for nominal and cyclical attributes and taking e.g. the increase of demand into account their can be already said quite something about the bike demand. However, there is still a lot room for improvement.
I used only a simple linear regression model (with curvilinear attributes). More advanced models like random forest, a very popular algorithm in predictions, can be deployed to improve the predictions and decrease the RMSLE. There may be more relations I did not find with these exploration. Some interaction effects or other effect that are not caused by weather but does significantly affect the bike demand. For example coupons that are issued during the year, or some kind of 'happy hours'.

In the next part I'll improve the predictions.




