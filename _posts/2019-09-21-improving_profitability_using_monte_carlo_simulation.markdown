---
layout: post
title:      "Improving Profitability: Using Monte Carlo Simulation"
date:       2019-09-21 06:44:56 +0000
permalink:  improving_profitability_using_monte_carlo_simulation
---



In this blog post, I will discuss how I analyzed the Northwind Database using [Monte Carlo Simulation](https://en.wikipedia.org/wiki/Monte_Carlo_method). 

The [Northwind database](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/northwind-pubs) is a free, open-source dataset created by Microsoft containing data from a fictional company. I investigated various aspects such as revenue or quantity sold by discount level, product category, and shipping region. 

I performed several hypothesis tests to see how Northwind could potentially improve profitability, either by increasing revenue or decreasing costs. Although t-tests might be a quicker and easier way to run hypothesis tests, obtaining p-values through Monte Carlo Simulation was extremely helpful, especially because almost all of variables I investigated were heavily skewed to the right (which make sense considering the nature of the data I'm looking at -- prices and quantities, for example). 

### Setup

I used following libraries for my analysis.

```
import time
import pandas as pd
import sqlite3
import matplotlib.pyplot as plt
import numpy as np
from statsmodels.stats.power import TTestIndPower, TTestPower
```

### Defining Monte Carlo Simulation Function

I defined the `monte_carlo` function as follows. Given lists of 2 sample, I can specify how many simulations to run (with default set as 10,000), and what kind of tests to run (i.e. is mean of sample1 bigger? or vice versa?). I also added two features where I could (a) plot histogram of sample mean differences and (b) print the simulation progress.

```
def monte_carlo(sample1, sample2, n=10000, greater='sample1', plot=False, print_progress=False):
    '''
    Returns p-value
    Performs Monte Carlo simulation of given 2 samples, where
    sample1 : sample (list) to perform the simulation
    sample2 : sample (list) to perform the simulation
    n : number of simulations
    greater : sample that is expected to have greater mean, either sample1 or sample2
    plot : if True, outputs histogram of sample mean differences from the simulation
    print_progress : if True, prints simulation number when n is at 0%, 20%, 40%, 60%, 80% and 100% of given n
    '''
    # identify sample with bigger size
    if len(sample1) >= len(sample2):
        a = list(sample1)
        b = list(sample2)
        if greater == 'sample1':
            test_if_greater = 'a'
            test = 'Testing if sample1 is greater than sample2 where difference = mean(sample1) - mean(sample2)'
        else:
            test_if_greater = 'b'
            test = 'Testing if sample2 is greater than sample1 where difference = mean(sample1) - mean(sample2)'
    else:
        a = list(sample2)
        b = list(sample1)
        if greater == 'sample1':
            test_if_greater = 'b'
            test = 'Testing if sample1 is greater than sample2 where difference = mean(sample2) - mean(sample1)'
        else:
            test_if_greater = 'a'
            test = 'Testing if sample2 is greater than sample1 where difference = mean(sample2) - mean(sample1)'
    
    # difference of sample means
    mu_diff = np.mean(a) - np.mean(b)

    counter = 0
    combined = a + b
    sample_diffs = []
    
   
    # Run simulations
    print(' ', test)
    if print_progress:
        print('Simulation starting')
    for i in range(n):
        ai = np.random.choice(combined, size=len(a), replace=False)
        bi = combined.copy()

        for element in ai:
            bi.remove(element) #bi is a complement of ai

        mu_diff2 = np.mean(ai) - np.mean(bi)
        sample_diffs.append(mu_diff2)
        
        if test_if_greater == 'a':
            if mu_diff2 > mu_diff:
                counter += 1
        else:
            if mu_diff2 < mu_diff:
                counter += 1
                
        if print_progress:        
            if i in list(np.linspace(0, n, 5)): # print simulation progress 
                t1 = time.time()
                print('Simulation #{}'.format(i))
            
    if print_progress:
        print('Simulation done')
        %%time
    
    p_val = counter / n
    
    if plot:
        plt.figure(figsize=(10,6))
        plt.hist(sample_diffs, label='sample mean diffs')
        plt.axvline(mu_diff, color='k', label='mean diff')

        plt.xlabel('Mean Differences')
        plt.ylabel('Counts')
        plt.suptitle('Distribution of Sample Mean Differences', fontsize= 18)
        plt.title('P-value that {} is greater is {}'.format(greater, np.round(p_val,4)))
#         plt.title(test)
        plt.legend(bbox_to_anchor=(1, 1))
        plt.show()

#     print(test)
    print('  P_value is', np.round(p_val,2))
    return p_val
```

### Compare multiple groups using Monte Carlo Simulation

With the `monte_carlo` function, I can easily run multiple hypothesis tests among different groups. For example, I analyzed if the revenue generated in each shipping region was significantly different. I compared revenues of each pair of shipping regions and stored the results to a dataframe. I also reported effect sizes and powers for more comprehensive review.


```
# run monte carlo simulation to compare different shipping regions
results = []
alpha = 0.05

regions_num = list(order_ship['ShipRegion_num'].unique()) 
regions = list(order_ship['ShipRegion'].unique())

for reg1 in regions_num:
    print('----')
    for reg2 in regions_num:
        print('comparing {} and {}'.format(regions[int(reg1-1)], regions[int(reg2-1)]))
        
        sample1 = order_ship[order_ship['ShipRegion_num'] == reg1]['Revenue']
        sample2 = order_ship[order_ship['ShipRegion_num'] == reg2]['Revenue']

        p_val = monte_carlo(sample1, sample2, n=10000, greater='sample1', plot=False, print_progress=False)
        if p_val  < alpha:
            concl = 'Rejected'
        else:
            concl = 'Failed to Reject'


        effect = cal_cohens_d(sample1, sample2)
        power = cal_power(sample1, sample2, effect[0], alpha=alpha)
        results.append([regions[int(reg1-1)], regions[int(reg2-1)],
                        reg1, reg2, 
                        result[1], 
                        concl, 
                        effect[0], effect[1], 
                        power])
    
results_df = pd.DataFrame(results, columns=['Group_1', 'Group_2', 
                                            'Group_1_Num', 'Group_2_Num', 
                                            'P_Value','Conclusion',
                                            'Effect Size','Effect Size Flag','Power'])
```

I calculated effect size and statistical power as follows:

##### Effect Size

```
# Investigate effect size
def cal_cohens_d(sample1, sample2):
    '''
    Calculates effect size (Cohen's D) of given 2 samples. 
    Returns Cohen's d and if the effect size is small, medium, or large.
    
    d = (mu1 - mu2) / (sqrt(std1**2 + std2**2) / 2)
    Where
    mu1 = mean of sample 1
    mu2 = mean of sample 2
    std1 = standard deviation of sample 1
    std2 = standard deviation of sample 2
    
    Small effect size: 0 < d < 0.4
    Medium effect size: 0.4 <= d < 0.8
    Large effect size: d >= 0.8
    '''
    d = (np.mean(sample1)- np.mean(sample2))/(np.sqrt(np.std(sample1)**2+np.std(sample2)**2)/2)
    
    if d == 0:
        size = 'No Effect'
    elif np.abs(d) < 0.4:
        size = 'Small'
    elif np.abs(d) >= 0.8:
        size = 'Large'
    else:
        size = 'Medium'
        
    return (d, size)
```

##### Statistical Power
```
# Investigate statistical power
power_analysis = TTestIndPower()

def cal_power(sample1, sample2, effect, alpha=0.05):
    '''
    Calculates statistical power using TTestIndPower function from statsmodels library,
    given effect size and alpha.
    '''
    n1 = len(sample1)
    n2 = len(sample2)
    power = power_analysis.solve_power(nobs1=n1, ratio=n2/n1, effect_size=effect, alpha=alpha)
    
    return power
```

### Results

My `results_df` showed that differences in revenue were statistically significant for a fair amount of regions, with meidum to large effect sizes. 

![](https://imgur.com/W4MYGno.png)

In particular, it was interesting to see how British Isles and Southern Europe were significantly different, considering how the total revenues in these two regions are quite similar:
![](https://imgur.com/llO0xz8.png)


The results look interesting, but there were too many comparisons to see the big picutre. Then I thought, what if I visualize these results as a scatter plot that looks like a 2D matrix?

```
# monte carlo results (sample 1 > sample 2)
plt.figure(figsize=(4,3.5))
group1 = results_df.loc[results_df_4m['Conclusion'] == 'Rejected']
group2 = results_df.loc[results_df_4m['Conclusion'] == 'Failed to Reject']

plt.scatter('Group_1_Num', 'Group_2_Num', data=group1, marker='s', color='turquoise', s=300, label='Rejected')
plt.scatter('Group_1_Num', 'Group_2_Num', data=group2, marker='x', color='green', s=25, label='Failed to Reject')
plt.plot(list(range(1,10)), list(range(1,10)), 'r--', alpha=0.3)

plt.yticks(ticks=np.arange(1,10,1), labels=regions, size=13)
plt.xticks(ticks=np.arange(1,10,1), labels=regions, rotation=90, size=13)
plt.legend(bbox_to_anchor=(1, -0.65))
plt.xlabel('Group 1', weight='heavy')
plt.ylabel('Group 2', weight='heavy')
plt.title('Monte Carlo Simulation Results \n Testing Group 1 > Group 2 \n 10,000 Simulations', 
          size=14, weight='heavy')
plt.show()
```
		
![](https://imgur.com/kHTICRp.png)

I believe the scatter plot is quite effective in summarizing the results in a concise manner. The plot allows not only region-to-region level comparison, but also more wholistic comparison (i.e. which region differs the most from other regions). The results indicate that Western Europe and North America have larger revenue than other regions. Similarly, Eastern Europe seems to be struggling to generate revenue.



