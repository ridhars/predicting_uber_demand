## Project 5 Summary - Predicting Uber Demand

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

![Seasonalities](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 1")

The daily seasonality is shown by the hourly fluctuations of the trips. Looking at the orange plot, it peaks at 8AM (morning rush hour) and 7PM (evening rush hour) -- which is something expected. However, we have different behavior during weekends, shown by blue plot. It has no morning peak, but rather builds up slowly until it reached the peak at 7PM. These two different behaviour suggest we have weekly seasonality, or in particular weekday vs. weekend pattern.

#### 2. Preprocessing

Aside from the generic preprocessing steps, one that rather need to be informed here is the imputation part. During EDA, it was also found that we are missing some hours of data on 26 - 27 Jan. This date corresponds to the blizzard storm which occured in New York. Therefore I imputed all data during this date with the *average* of *week-before and week-after* data.

#### 3. Modelling

Two models were experimented, SARIMAX and Facebook Prophet. Both models allow the usage of additional features such as weather to be included as regressor. After experimenting, 


