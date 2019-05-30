---
layout: post
title:      "Why data science?"
date:       2019-05-30 22:43:48 +0000
permalink:  why_data_science
---


I currently work at an economic consulting firm, which involves analyzing various financial data and presenting the findings to an audience with no technical background. 

Last year, I worked on a project that required me to cleaning and collapsing monthly remittance data into a time-series data. More specifically, I had to compile monthly delinquency history - i.e. if or how much a borrower had been late or overdue on a payment over time - of each loan from the raw data.



#### Seemingly simple task
The basic idea was simple. In the raw data, each row contained monthly delinquency status of a loan as a string with length of 1. For example, a delinquency status of `3`  would mean the loan has been delinquent for 30 days, `6` would mean delinquent for 60 days, `F` would mean foreclosure and so on. Like this:  


| Loan | Month | Delinquency Status |
| - | - | - |
| A | 1 | 3 |
| A | 2 | 6 |
| B | 1 | 9 |
| B | 2 | F |

The desired output would have one row for each unique loan in the raw data. So, for the simplified example above, I would want:

| Loan | Delinquency History |
| - | - |
| A | 36 |
| B | 9F |



#### Obstacles
At first, I thought this could be easily done through a simple while loop in SAS. However, I ran into obstacles soon enough. The raw data looked more like the table below.

| Loan | Month | Delinquency Status |
| - | - | - |
| A | 1 | 3 |
| A | 2 | 6 |
| A | 4 | 6 |
| B | 3 | 9 |
| B | 6 | F |

The main obstacles were:
  1. The raw dataset had over 2 billion rows, which took me few hours just to import the data.
  2.  Each loan had available data for different periods in time. Therefore, each loan would have `Delinqunecy History` of different lengths.
  2. For many months, data was missing for some months. Therefore, the length of `Delinquency History` would not always match the number of months between the first and last months of available data. Moreover, if a loan had missing months, the `Delinquency History` would become meaningless as I can't figure out delinquency status of a given month.


#### Process and Results
After multiple late nights and weekends, I was able to compile Delinquency History dataset as desired.

First of all, I started working with a small subset of the raw data so that I don't waste hours running my program, just to realize a typo in my code.

Then, I created a column called `Delinquency Status_New` that fills in delinquency statuses of missing months with 'X', as shown in the table below. The program would go through each row, and if `Month` increase by more than 1 month compared to the previous row, I would pad 'X's to `Delinquency Status` by the increment (i.e. the number of missing months). If  `Month` increase by 1 month compared to the previous row, `Delinquency Status_New` would be same as `Delinquency Status`.         

| Loan | Month | Delinquency Status | Delinquency Status_New |
| - | - | - | - |
| A | 1 | 3 | 3 |
| A | 2 | 6 | 6 |
| A | 4 | 6 | X6 |
| B | 3 | 9 | 9 |
| B | 6 | F | XXF |

Then, I created a `Delinquency History` column using `Delinquency Status_New` column. The program also identified the first and last months of available data for each loan, so that it is clear for which month each letter in the `Delinquency History` column represents. I also calculated the `Period` between the first and last months of available data, which became useful for sanity checking if `Delinquency History` had correct length.  

| Loan | First Month | Last Month | Period | Delinquency History |
| - | - | - | - | - |
| A | 1 | 4 | 4 | 36X6 |
| B | 3 | 6 | 4 | 9XXF |


Looking back, the task does not appear as challenging and I feel like I should have been able to figure it out sooner. Nevertheless, I enjoyed the entire process of problem solving and I became more interested in coding and data. It was my first time working with such big dataset, and I became curious about how data scientists work with and draw insights from big datasets. Had I written my program in Python, not SAS, would it be easier or more efficient? Rather than filling in 'X's for missing months, would it be possible to recognize delinquency patterns in the raw data estimate delinquency statuses of the missing months? If so, would it be possible to go one step further and predict delinquency statuses of each loan in the future?

I'm excited to begin my journey as a data scientist and solve a myriad of interesting real-world questions through data. 


 

