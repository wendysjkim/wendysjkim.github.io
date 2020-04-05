---
layout: post
title:      "COVID-19: Importance of data quality in modeling"
date:       2020-04-05 02:04:55 +0000
permalink:  covid-19_importance_of_data_quality_in_modeling
---

An infectious disease that causes respiratory illness like flu, the novel coronavirus (COVID-19) has deeply affected our society all over the world. Despite the efforts of social distancing and increased personal hygiene, the countries have been struggling to ["flatten the curve"](https://coronavirus.jhu.edu/data/new-cases). In other to better combat the pandemic, many experts have been developing various models to predict the number of infections.

For example, [a New York Times article](https://www.nytimes.com/interactive/2020/03/25/opinion/coronavirus-trump-reopen-america.html) suggests that more than 82k people would die from the virus in the United States, assuming 60 days of social distancing. The model is built on many assumptions such as the incubation period, the infectious period, the recovery period, etc. 

As of April 4, 2020, [the IHME (Institute for Health Metrics and Evaluation)](http://www.healthdata.org/covid), a study referenced by the White House, projects there would be over 90k COVID-19 deaths in the United States by early August. 

![](https://imgur.com/FjA7K2f.png)

We should not take these predictions as granted, however, since every underlying assumption can significantly change the predicition statistics. 

Let's revisit the model from the New York Times article. The model predicts more than 82,300 deaths in the U.S. under these assumptions:
* ** 1% death rate**
* 60 days of moderate intervention starting March 13th
* Medium impact of warm weather
* Infectiousness of 2.5 people
* 10% share requiring hospitalization
* Incubation period of 5.2 days
* Infectious period of 2.9 days
* Recovery period of 11.1 days for mild cases and 28.6 days for severe cases
* Time to death of 32 days
* Delay of 5 days before infected patients visit a hospital.

How much a different death rate assumption, for instance, would affect the fatalities projected? According to [Johns Hopkins University Coronavirus Resource Center](https://coronavirus.jhu.edu/data/mortality), the case-fatality ratio varies significantly across different countries. As of April 3, case-fatality ratio of U.S., China, Italy, and South Korea are as follows.

* U.S. : 2.6%
* China : 4%
* Italy : 12.3% 
* South Korea: 1.7%

When we adjust the New York Times model to have above death rates (and other assumptions unchanged), the model suggests,

*  More than 214,100 people would die in the U.S., assuming the U.S.'s 2.6% death rate;
*  More than 329,300 people would die in the U.S., assuming China's 4% death rate; and
*  More than 140,000 people would die in the U.S., assuming South Korea's 1.7 death rate.

Moreover, even these raw data come with [many caveats](https://www.nytimes.com/2020/02/18/opinion/coronavirus-china-numbers.html). Are we accurately identifying the causes of deaths? Are we catpuring the infected population comprehensively? Do we have capacity to test everyone with symptoms? What about people who already contracted with the virus, but has no symptoms? Are the test kits reliable? Do we agree on what it means to be infected by the coronavirus?

![](https://ichef.bbci.co.uk/news/410/cpsprodpb/8E92/production/_110889463_optimised-daily_china_coronavirus_cases_hist_13feb-nc.png)

The pandemic has raised a myriad of public health, economic, and social concerns. Given the wide range of potential scenarios, the decisions that people and the governments make can significantly change the outcome of this pandemic. 
