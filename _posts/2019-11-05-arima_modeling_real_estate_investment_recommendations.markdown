---
layout: post
title:      "ARIMA Modeling: Real Estate Investment Recommendations"
date:       2019-11-05 07:42:11 +0000
permalink:  arima_modeling_real_estate_investment_recommendations
---

In this blog post, I will discuss how I analyzed zipcode-level median housing sales data from Zillow. I used ARIMA modeling to identify 5 zipcodes with highest predicted rate of returns in the three biggest cities (by number of zipcodes).

There were 14,723 zipcodes in total. I focused on top 3 cities with the most number of zipcodes. The top 3 cities with the most number of zipcodes are New York, Los Angeles and Houston.

The data is consistent with [the population size](http://worldpopulationreview.com/us-cities/) of each city. 

**Largest Cities in the US by Population (2019)**
* New York City, NY (Population: 8,601,186)
* Los Angeles, CA (Population: 4,057,841)
* Chicago, IL (Population: 2,679,044)
* Houston, TX (Population: 2,359,480)

![](https://imgur.com/oTyoZ7p.png)

I further filtered down the samples sizes to 100 zipcodes, by looking at the growth rates in the past 3 years.

```
top3_growth = top3.copy()
top3_growth['Growth'] = (top3_growth['2018-04']/top3_growth['2015-04'])-1
```

### EDA

There were 4 outliers in NYC whose prices were significantly higher than the rest of the regions. More interestingly, these zipcodes didn't seem to be affected by the 2008 financial crisis. Rather, their median housing sales value increased!

[](https://imgur.com/sDKHaY4.png)

Not surprisingly, these 4 zipcodes turned out to be one of the most expensive areas in Manhattan (Upper East Side, Chelsea, and Greenwich Village / Soho). After excluding these outliers, the housing price trend seemed much more reasonable.

![](https://imgur.com/dSjDj3o.png)
![](https://imgur.com/fA2rYN1.png)

### ARIMA Grid Search

I ran ARIMA grid search for each zipcode, to identify optimal ARIMA parameters. I selected ARIMA paramters as those with the smallest mean squared errors.

I used SARIMAX with no seasonal order and constant specified. If the constant is specified (i.e. trends='c'), the results should be the same as using the statsmodels ARIMA funciton.

I decided to use SARIMAX without the constant specified after few tries. SARIMAX also has a great method (results.plot_diagnostics) to visaulize the model output!

```
#evaluate an ARIMA model for a given order (p,d,q)
def evaluate_arima(X, arima_order, plots=False):
    '''
    For given dataframe and specified order (p, d, q),
    returns mean squared error, AIC, and ARIMA model output.
    Uses SARIMAX with no seasonal order and constant specified.
    When plots=True, shows SARIMAReslts.plot_diagnostics.
    '''
    # prepare training dataset
    train_size = int(len(X) * 0.66)
    train, test = X[0:train_size], X[train_size:]
    history = train
    
    # make predictions
    # SARIMAX function would return same result as ARIMA function, when specified trend='c'
    predictions = []
    for t in range(len(test)):
        model = sm.tsa.statespace.SARIMAX(history,
                                      order=arima_order,
                                      freq = 'MS',
                                      enforce_stationarity = False,
                                      enforce_invertibility= False)
        output = model.fit(disp=0)
        yhat = output.forecast()[0]
        predictions.append(yhat)
        history = history.append(test.iloc[t])
        
    # calculate out of sample error
    error = mean_squared_error(test, predictions)
    aic = output.aic
  
    # show plots
    if plots == True:
        print(output.summary())
        
        output.plot_diagnostics(figsize=(10,8)).show()

        plt.figure(figsize=(8,5))
        plt.plot(test, label='Actual')
        plt.plot(test.index, predictions, color='red', label='Prediction')
        plt.legend()
        plt.show()

    return error, aic, output
```

```
# evaluate combinations of p, d and q values for an ARIMA model
# select parameters based on smallest MSE
def evaluate_models(data, p_values, d_values, q_values):
    '''
    For given data, lists of p, d, and q values,
    evaluates combinations of p, d, and q values for an ARIMA model.
    Optimal parameters are chosen based on smallest MSE.
    Returns chosen parameters, MSE, AIC, and output of the fitted model.
    '''
    data = data.astype('float32')
    best_score, best_cfg, best_aic = float('inf'), None, None
    best_output = None
    
    for p in p_values:
        for d in d_values:
            for q in q_values:
                order = (p, d, q)
                try:
                    mse = evaluate_arima(data, order)[0]
                    aic = evaluate_arima(data, order)[1]
                    output = evaluate_arima(data, order)[2]
                    if mse < best_score:
                        best_score, best_cfg, best_aic = mse, order, aic
                        best_output = output
                    print('ARIMA{} MSE={} AIC={}'.format(order, mse, aic))
                except:
                    continue
    print('Best ARIMA{} MSE={} AIC={}'.format(best_cfg, best_score, best_aic))
    return best_cfg, best_score, best_aic, best_output
```

```
# subset data
def subset_data(zipcode, date_since=None):
    test = top3.loc[top3['RegionID'] == zipcode]
    test_melted = melt_data(test)
    
    if date_since == None:
        data = test_melted
    else:
        data = test_melted.loc[date_since:]
        
    return data
```

For the 100 fast-growing zip codes I chose, I ran ARIMA grid search with p values of [0, 1, 2, 4, 6, 8, 10] and d & q values of [0, 1, 2]. Since the data is available monthly, I iterated through more p values (up to 10) to capture any sort of seaonality with the AR component of the ARIMA model. 

```
#############################################################################
p_values = [0, 1, 2, 4, 6, 8, 10]
d_values = range(0,3)
q_values = range(0,3)
#############################################################################

results = []
i = 1
for zipcode in top3_growth['RegionID']:
    print()
    print('{}. RegionID {} Running.....'.format(i, zipcode))
    cfg, mse, aic, output = evaluate_models(subset_data(zipcode, date_since='2009-07-01'), 
                             p_values, 
                             d_values, 
                             q_values)
    results.append((zipcode, cfg, mse, aic, output))
    i += 1
    
    if i % 5 == 1:
        pd.DataFrame(results).to_csv('ARIMA_results_range0_2.csv')
        
results = pd.DataFrame(results)
results.columns = ['RegionID', 'Parameters', 'MSE', 'AIC', 'Output']
results.to_csv('ARIMA_results_range0_2.csv')
results.head()
```

### Results

Among the top 100 fast-growing zipcodes, I identified 30 zipcodes (10 zipcodes in each city) with the best performing models (i.e. smallest MSEs). Then, I calculated predicted growth rate in 1-year, 3-year and 5-year time periods.

```
def to_tuple(string):
    return tuple(int(x) for x in string[1:-1].split(','))
```

```
def get_predictions(data):
    '''
    For given dataframe, loops through each zipcode and optimal ARIMA parameters
    and calculates projected values (and ROIs) in 1, 3 and 5 years.
    Returns results as dataframe.
    '''
    futures = []
    for i in range(len(data)):
        zipcode = data['RegionID'].iloc[i]
        X = subset_data(zipcode, date_since='2009-07-01')
        arima_order = to_tuple(data['Parameters'].iloc[i])

        model = sm.tsa.statespace.SARIMAX(X,
                                      order=arima_order,
                                      freq = 'MS',
                                      enforce_stationarity = False,
                                      enforce_invertibility= False)
        output = model.fit(disp=0)
        future = output.get_forecast(steps=60).predicted_mean
        futures.append([zipcode, 
                       arima_order,
                       X.value.iloc[-1],
                       future.loc['2019-04-01'], 
                       future.loc['2021-04-01'],
                       future.loc['2023-04-01']])


    futures = pd.DataFrame(futures)    
    futures.columns = ['RegionID', 'Parameters', '2018-04-01', '2019-04-01','2021-04-01', '2023-04-01']
    
    futures['ROI_1'] = futures['2019-04-01']/futures['2018-04-01'] - 1
    futures['ROI_3'] = futures['2021-04-01']/futures['2018-04-01'] - 1
    futures['ROI_5'] = futures['2023-04-01']/futures['2018-04-01'] - 1
    
    return futures
```

Then, I identified top 5 zipcodes with highest 1-year return on investment. I decided to rely on 1-year ROI, as the uncertainty grows as we go further in the future.

* Top 5 ROI 1 year RegionID : {95984, 61779, 61780, 61781, 62107}
* Top 5 ROI 3 years RegionID: {96040, 95983, 95984, 61779, 96028}
* Top 5 ROI 5 years RegionID: {96040, 95983, 95984, 61779, 96028}

### Conclusion

The 5 chosen zipcodes are as follows. Among the 5 zipcodes, 4 are in NY and 1 is in LA.
['10303', '11421', '10305', '90003', '10304']

The 1-year ROI is expected to range 10-16%.

![](https://imgur.com/42SKWD8.png)
![](https://imgur.com/QGwi0xZ.png)
![](https://imgur.com/izldtNL.png)

