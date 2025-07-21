---
description: >-
  Forecasting global temperature anomalies using SARIMA modeling and historical
  climate data to uncover long-term warming trends and future projections.
---

# Climate Forecast - SARIMA

## 🛠️ Overview

This project explores and models climate change trends using historical data on temperature anomalies and CO₂ emissions. The ultimate goal is to forecast future global temperature anomalies based on past emission patterns and long-term climate trends.

Using R and key data science libraries, I conducted an exploratory data analysis (EDA) to understand how these indicators have evolved over the last century. I then applied basic forecasting techniques to predict how temperatures might rise in the coming decades if current emission trends continue.

***

## 🔥 Problem Statement

By the end of this analysis we wish to answer the following questions:

1. One recent source of climate “contention” is the evidence of a “hockey stick” shape in global temperatures. That is from 1950-1975 global temperatures were relatively constant where as post-1975 temperatures have been increasing. Is there evidence of a significant “hockey stick” shape in global temperatures?
2. If life continues as normal, what is the projected anomalies in the next 5 years?

***

## 🧩 Project Pipeline

### 1. Import the packages

```r
# Load Packages
library(tidyverse) # Data Wrangling and plotting
library(vroom) # Read Data
library(patchwork) # Combine multiple plots in one visualization

data <- vroom("AnnAvgGlobalClimate.txt")
```

### 2. Exploratory Data Analysis (EDA)

Start the EDA by taking an overview of the data.

```r
# head(data)
skimr::skim(data)
```

<figure><img src=".gitbook/assets/Screen Shot 2025-07-21 at 11.59.50 AM.png" alt=""><figcaption></figcaption></figure>

It looks like my response variable have negative and positive numbers. I have 2 predictive variables, both are numeric variables, but I will change Month as a factor. I am working with 72 years of data and 864 observations.

***

Now we'll look at the distribution for each variable for shape and possible outliers.

```r
par(mfrow = c(1, 2))
hist(data$AnnAnom, main = "Temperature Annomaly Distribution")
hist(data$Year, main = "Year Distribution")
```

<figure><img src=".gitbook/assets/Screen Shot 2025-07-21 at 12.12.06 PM.png" alt=""><figcaption></figcaption></figure>

```r
data$Month <- as.factor(data$Month)
ggplot(data, aes(x = Month)) +
  geom_bar(stat = "count", fill = "skyblue") +
  ggtitle("Month Count") +
  xlab("Count") +
  ylab("Month") +
  theme_minimal()
par(mfrow = c(1, 1))
```

<figure><img src=".gitbook/assets/Screen Shot 2025-07-21 at 12.12.32 PM.png" alt=""><figcaption></figcaption></figure>

Temperature Anomaly looks right skewed, same with year distribution. Month counts look evenly distributed.

***

Now we will look at the relationship between the response and the predictive variables.

```r
round(cor(data %>% dplyr::select(AnnAnom, Year)), 2)
```

```
        AnnAnom Year
AnnAnom    1.00 0.93
Year       0.93 1.00
```

Looks like there is a positive relationship between Year and Temperature Anomaly.

```r
ggplot(data, aes(x = Year, y = AnnAnom, color = Month)) +
  geom_point() +
  geom_smooth(se = FALSE) +
  theme_minimal()
```

<figure><img src=".gitbook/assets/Screen Shot 2025-07-21 at 12.17.49 PM.png" alt=""><figcaption></figcaption></figure>

> #### Correlated Data
>
> Looks like there might be some correlation from one data point to another, because of we are dealing with data points over time.

So far, I would probably do a SARIMA model since it captures seasonality and correlation between my data points.

***

I created a new variable `YrMon` in my dataset to **represent a continuous time variable that accounts for both the year and the month**, which is very useful for **accurately plotting time series data** with monthly resolution.

```r
data$YrMon <- data$Year + (data$Month - 0.5)/12
```

Draw a time series plot of the temperature anomalies with YrMon along the x-axis with a smooth curve overlaid to emphasize the non-linear aspect of the data.

```
ggplot(data, aes(x =    YrMon, y = AnnAnom)) +
  geom_line() +
  geom_smooth()
```

<figure><img src=".gitbook/assets/Screen Shot 2025-07-21 at 12.27.50 PM.png" alt=""><figcaption></figcaption></figure>

Now that we have a continuous variable that account for time, we can calculate and draw an autocorrelation function (ACF) of the raw temperature anomalies to show the strong correlation between one data point to the other.

```r
# Show correlation from the past 48 months
ggAcf(x=data$AnnAnom, lag.max=48)
```

<figure><img src=".gitbook/assets/Screen Shot 2025-07-21 at 12.31.56 PM.png" alt=""><figcaption></figcaption></figure>

This ACF plot supports the hypothesis that **temperature anomalies are increasing over time in a dependent, predictable way**, which aligns with climate change trends.\
The strong autocorrelation (lines above the blue dashed line) justifies using models like **SARIMA** that capture both trend and seasonality.

### 4. Model Selection

#### Linea Spline

From the initial line graph, I noticed a clear change in the slope around **1975,** temperatures appeared to rise more sharply after that point. To capture this, I fit a **linear spline model** with a **knot at 1975**, allowing the trend to shift before and after that year. This helped model the “hockey stick” pattern often seen in climate data.

```r
model <- lm(AnnAnom ~ YrMon + pmax(YrMon - 1975, 0), data = data)

ggplot(data, aes(x = YrMon, y = AnnAnom)) +
  geom_line() + 
  geom_smooth(method = "lm", formula = y ~ x + pmax(x - 1975, 0), se = FALSE) + 
  labs(x = "Year", y = "Temperature Anomaly") +  
  ggtitle("Linear Spline Model of Annual Temperature Anomalies") +
  theme_minimal()
```

<figure><img src=".gitbook/assets/Screen Shot 2025-07-21 at 3.39.19 PM.png" alt=""><figcaption></figcaption></figure>

The smoothed line shows two linear trends: pre-1975 and post-1975.

***

Draw an ACF of the residuals from the Linear spline Model to verify that there is indeed still temporal correlation in the residuals that we will need to model.

```r
ggAcf(x=residuals(model), lag.max=12*3)
```

<figure><img src=".gitbook/assets/Screen Shot 2025-07-21 at 3.41.20 PM.png" alt=""><figcaption></figcaption></figure>

From this plot we can still see autocorrelation in residuals, indicating the linear model is insufficient, and time dependence must be modeled explicitly, using a time series model.&#x20;

***

#### Time Series Model

Check the earliest month recorded in the year 1950, to verify the correct **start point** for your time series.

```r
min(data %>% filter(Year == 1950) %>% dplyr::select(Month))
# Result is 1 suggesting it starts in January
```

Define a time series object of AnnAnom as use this object in all your analyses below.

```r
my.ts <- ts(data=data$AnnAnom, start=c(1950, 1), frequency=12)
```

Create an X matrix for a linear spline for YrMon with a knot location at 1975 and without an intercept (the Arima() function automatically adds in an intercept for you so you don’t need to include an intercept column in X

```r
X <- model.matrix(data$AnnAnom~YrMon-1+pmax(YrMon-1975, 0), data=data)
```

Using auto.arima() choose which p, d, q, P, D, Q to use by comparing AIC or BIC values for models with an order of 2 or less (i.e. max.p=max.q=max.Q=max.P=2) and a max difference of 1 (i.e. max.d=max.D=1). Make sure to use your linear spline as the xreg= value.

```r
test.arima <- auto.arima(my.ts,
                    max.p=2, max.q=2, max.P=2, max.Q=2, max.d=2, max.D=1,
                    ic="aic", stepwise=FALSE, xreg=X,
                    trace=TRUE)
                    
#Fit your chosen model and examine the model estimates.
my.sarima.model <- Arima(my.ts, order=c(2,0,1), seasonal=c(2,0,0), xreg=X)
my.sarima.model
```

<figure><img src=".gitbook/assets/Screen Shot 2025-07-21 at 3.56.18 PM.png" alt=""><figcaption></figcaption></figure>

Specifications of the model:

* Non-Seasonal Part `(p,d,q) = (2,0,1)`:
  * AR(2): The model uses the two most recent values of the series to help make the next prediction.
  * I(0): No differencing was applied, which suggests your time series was likely stationary to begin with (it wasn't showing a consistent upward or downward trend that needed to be flattened).
  * MA(1): The model looks at the one most recent forecast error to improve the next prediction.
* Seasonal Part `(P,D,Q)[m] = (2,0,0)[12]`:
  * SAR(2): The model uses the values from the same period in the last two seasons to help make its forecast. Since the period `[12]` is monthly, this means it's looking at the value from this month last year and the year before.
  * SI(0): No seasonal differencing was needed.
  * SMA(0): The model does not use past seasonal forecast errors.



### 5. Model Validation

#### ASSUMPSIONS

**Independence**

```r
acf(x=residuals(my.sarima.model), lag.max=12*3)
```

<figure><img src=".gitbook/assets/Screen Shot 2025-07-21 at 4.06.23 PM.png" alt=""><figcaption></figcaption></figure>

As we can see in this ACF after our first observation, we can appreciate that our model take in account the fact that they're correlated, therefore independence is met because there are no significant spikes beyond the confidence bands, suggesting that the residuals are independent.

**Equal Variance & Linearity**

```r
#fitted values vs. residual plot
fitted_values <- fitted(my.sarima.model)
residuals <- resid(my.sarima.model)
plot(fitted_values, residuals, xlab = "Fitted Values", ylab = "Residuals",
     main = "Fitted Values vs. Residuals")
```

<figure><img src=".gitbook/assets/Screen Shot 2025-07-21 at 4.08.21 PM.png" alt=""><figcaption></figcaption></figure>

This fitted values vs. residual plot helps to meet both assumptions, equal variance and linearity. Since the residuals are randomly scattered with no shape or curve, the model has properly captured the linear (or structured) part of the data. And since the spread of the residuals is consistent accross the range of fitted values, we have good equal variance.

**Normality**

Plot a histogram of the decorrelated residuals

```r
ggplot() +
  geom_histogram(mapping = aes(x =residuals(my.sarima.model)))
```

<figure><img src=".gitbook/assets/Screen Shot 2025-07-21 at 4.15.42 PM.png" alt=""><figcaption></figcaption></figure>

Thanks to the histogram, we can appreciate that the decorrelated residuals follows a normal distribution, meaning that normality is met.

#### PREDICTIONS

```r
num_test_periods <- 60
train_indices <- 1:(nrow(data) - num_test_periods)
test_indices <- (nrow(data) - num_test_periods + 1):nrow(data)

X_train <- X[train_indices, ]
X_test <- X[test_indices, ]
data_train <- data[train_indices, ]
data_test <- data[test_indices, ]

my.ts <- ts(data=data_train$AnnAnom, start=c(1950, 1), frequency=12)

my.sarima.model.2 <- Arima(my.ts, order=c(2,0,1), seasonal=c(2,0,0), xreg=X_train)

sarima_forecast <- forecast(my.sarima.model.2, h=60, xreg=X_test, level=.95)

plot(sarima_forecast)
```

<figure><img src=".gitbook/assets/Screen Shot 2025-07-21 at 4.17.53 PM.png" alt=""><figcaption></figcaption></figure>

```r
#Calculate RPMSE and coverage.
my.preds = sarima_forecast$mean
rpmse <- sqrt(mean((data_test$AnnAnom - my.preds)^2))
coverage <- mean(data_test$AnnAnom > sarima_forecast$lower & 
                 data_test$AnnAnom < sarima_forecast$upper)
```

```
rpmse =  0.07493193
coverage = 1
```

> **Root Mean Square Error (RMSE)** is a standard metric that measures how far off your model’s predictions are from the actual values — on average.

On average the sarima model is approx  0.074 of the mean of the actual values, and given that my mean annomaly temperature is 0.329 and we are dealing with climate context, it is an acceptable root mean square error.&#x20;

The 1 in coverage means that all observed data points fall within the prediction intervals generated by the model.

### 6. Statistical Inference

#### Calculate a p-value for a test

```r
a_t = t(c(0,0,0,0,0,0,0,1))
test_results = glht(my.sarima.model, linfct = a_t, alternative="greater")
summary(test_results)
```

```
Simultaneous Tests for General Linear Hypotheses

Fit: Arima(y = my.ts, order = c(2, 0, 1), seasonal = c(2, 0, 0), xreg = X)

Linear Hypotheses:
       Estimate Std. Error z value   Pr(>z)    
1 <= 0 0.017258   0.002423   7.121 5.35e-13 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
(Adjusted p values reported -- single-step method)
```

There is strong evidence to reject the null hypothesis in favor of the alternative hypothesis that the coefficient associated with B\_2\_hat is greater than 0, meaning that there does seem to be faster temperature increase since 1975.

#### Calculate a 95% confidence interval for β2

```r
coefficient <- coef(my.sarima.model)["pmax(YrMon - 1975, 0)"]
standard_error <- sqrt(diag(vcov(my.sarima.model))["pmax(YrMon - 1975, 0)"])
degrees_of_freedom <- nrow(data) - 8
alpha <- 0.05
t_score <-qt(1 - alpha / 2, df = degrees_of_freedom)
lower_bound <- coefficient - t_score * standard_error
upper_bound <- coefficient + t_score * standard_error

lower_bound
upper_bound
```

```
pmax(YrMon - 1975, 0) 
           0.01250113 
pmax(YrMon - 1975, 0) 
           0.02201404 
```

#### Predict the temperature anomalies forward 60 months (5 years)

To do this, you will have to set up an X matrix for the times you want to predict.

```r
last_YrMon <- tail(data$YrMon, 1)
new_YrMon <- seq(last_YrMon + 1/12, by = 1/12, length.out = 60)

new_data <- data.frame(
  Year = floor(new_YrMon),
  Month = rep(1:12, length.out = 60),
  YrMon = new_YrMon,
  AnnAnom = rep(0, 60)
)

X_extended <- model.matrix(AnnAnom ~ YrMon - 1 + pmax(YrMon - 1975, 0), data =new_data)

forecast_result.2 <- forecast(my.sarima.model, h = 60, xreg = X_extended, level = 0.95)

plot(forecast_result.2)
axis(1, at = seq(1950, 2029, by = 5), labels = seq(1950, 2029, by = 5))
```

<figure><img src=".gitbook/assets/Screen Shot 2025-07-21 at 4.41.24 PM.png" alt=""><figcaption></figcaption></figure>

***

## 📈 Conclusion

This project combined data exploration, statistical modeling, and forecasting to better understand long-term global climate patterns. Through careful analysis of temperature anomalies and CO₂ data, we confirmed a clear acceleration in global warming trends after 1975, often described as a "hockey stick" effect.&#x20;

Our SARIMA model effectively captured both trend and seasonal patterns, with strong diagnostic checks and reliable predictions. Despite the complexity of climate data, the model achieved an RMSE of just 0.074 and 100% prediction interval coverage on the test set — demonstrating good fit and reliable forecast accuracy.&#x20;

This work underscores the value of time series modeling in supporting climate awareness, policy planning, and long-term environmental forecasting.

***

Thanks for reading, and keep the learning going!
