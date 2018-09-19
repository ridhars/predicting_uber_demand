## Project 5 Summary - Predicting Uber Demand

[MVP](https://docs.google.com/document/d/1c5xfZa7yJ5WqXNHfWXPiEJN56KZQmXC-7Y8Oqn8O_Ao/edit)
[Presentation Deck](https://docs.google.com/presentation/d/1njmqChgvC210yJwYXLsWl53AuuQIwZqihHa1YBk9FsA/edit?usp=sharing)

### Project Design

Today, ride-hailing service has become considered a common option of transportation. Particulary in short to medium remote-to-remote trips, ride-hail service provider such as Uber is increasingly popular. This in turn attracts more and more people to become Uber drivers, which reportedly to have grown at exponential rate since 2014 according to Uber site. For customers, it made finding Uber rides much more convenient. But for drivers, especially today with over-saturated Uber drivers -- not to mention competition -- it made finding customers significantly more difficult.

As someone who occasionally drives Uber, the most common question that often came up is; when and where should I drive to maximize my trips? 

In this project, I aim to gain information on estimated trips given a specific time in the future by building a time series forecasting model. The output of the model is the number of estimated trips at hourly rate on neighborhood level.
  
  
### Data & Tools

To build the model, 2015 New York historical Uber trips were used from publicly available NYC Taxi & Limousine Commision. The dataset contains 14.3 million trips from Jan - June 2015, each described by pickup timestamp and pickup geolocation. To improve prediction power, historical weather data were also taken from Open Weather Map. The data contains hourly temperature, wind speed, and weather condition flags.

Data are stored in PostgresSQL database in AWS while preprocessing & model building are carried out locally in iPython Jupyter Notebook. To aid visualization, dashboard were created in Tableau.
  
  
### Algorithms

#### 1. EDA

The most important information inferred during EDA is the time series' seasonality. As shown below, we can recognize two type of seasonalities; daily seasonality and weekly seasonality. 

![Seasonalities](https://github.com/ridhars/predicting_uber_demand/blob/master/image/Screen%20Shot%202018-09-18%20at%2011.04.31%20PM.png?raw=true)

The daily seasonality is shown by the hourly fluctuations of the trips. Looking at the orange plot, it peaks at 8AM (morning rush hour) and 7PM (evening rush hour) -- which is something expected. However, we have different behavior during weekends, shown by blue plot. It has no morning peak, but rather builds up slowly until it reached the peak at 7PM. These two different behaviour suggest we have weekly seasonality, or in particular weekday vs. weekend pattern.

#### 2. Preprocessing

Aside from the generic preprocessing steps, one that rather need to be informed here is the imputation part. During EDA, it was also found that we are missing some hours of data on 26 - 27 Jan. This date corresponds to the blizzard storm which occured in New York. Therefore I imputed all data during this date with the *average* of *week-before and week-after* data.

#### 3. Modelling

Two models were developed, SARIMAX and Facebook Prophet. Both models allow the usage of additional features such as weather to be included as regressor. Initially three types of weather data were used; temperature, wind speed, and one-hot encoded weather description flags. After experimenting, only temperature and ‘heavy intensity rain’ flag were kept for the final model as these two features were the ones that improved prediction for both SARIMAX and Prophet.

Between SARIMAX and Prophet, the best result was achieved by using Prophet. 

Under the hood, SARIMAX only supports one seasonality. Two workarounds were attempted; first by using only weekly seasonality (lags of 7 * 24 hourly data points) and secondly using daily seasonality (lags of 24 hourly data points) + day-of-week flags. The first method was very computationally heavy to the point it consistently crashed the machine. The second method only improved the prediction marginally.

On the other hand, Prophet can be customised with as many seasonalities as needed. Additionally, it allows exclusion of outliers (e.g. public holidays) from the seasonality pattern. This flexibility allows Prophet to produce forecasting result as shown below.

![Forecast](https://github.com/ridhars/predicting_uber_demand/blob/master/image/Screen%20Shot%202018-09-18%20at%2011.03.41%20PM.png?raw=true)

The model predicts the hourly fluctuations quite well while also able to distinguish weekdays vs. weekends pattern. However, by Sunday afternoon, it fails to predict a surge. Upon checking historical data, this surge does not typically occur in Sunday afternoon. A quick googling revealed that this date corresponds to the Puerto Rican Day Parade in New York, which explains the irregular surge.

**Final Model**

For one week forecast, which predicts 7 * 24 future data points, the best model achieved 498 trips/hour RMSE and 15% MAPE. The next step is to breakdown this model into neighbourhood level. In total, there are 262 New York neighborhoods. The result is then visualised into Tableau dashboard, as shown below.

![Dashboard](https://github.com/ridhars/predicting_uber_demand/blob/master/image/Screen%20Shot%202018-09-18%20at%2011.14.40%20PM.png?raw=true)

### Improvements

#### 1. Predicting irregular surge

The Sunday afternoon surge we’ve seen earlier was related to event that occurred on that particular day. In order to overcome this, I plan to include local event / calendar data as additional features. However event flag alone will not be enough, as the model requires continuous data to predict the magnitude of the event. One that I can think of is to use venue’s capacity as a proxy, but this would still not solve the Puerto Rican Day Parade we have seen before — because the venue is the city itself.

#### 2. Weather forecast as features

Currently in this model I simply included historical weather data as features. Realistically, these data will not be available until after we’ve gone past the day. The weather will also need to be forecasted first, then included as features in the main model.

#### 3. Sub-Neighborhood granularity

Currently, the model can only forecast on neighbourhood level. For some larger neighbourhoods, this may not be suitable because drivers need to ensure they are in a location within the reach of incoming rides. By clustering blocks within neighbourhood, we can create a lower-level prediction. To do this properly, the first step is to learn the maximum radius that a driver can receive incoming rides. Afterwards, we can adjust the clustering area accordingly.

