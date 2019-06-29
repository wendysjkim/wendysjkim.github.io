---
layout: post
title:      "EDA and Visualization, from plt to folium"
date:       2019-06-26 19:31:42 -0400
permalink:  eda_and_visualization_from_plt_to_folium
---


In this blog post, I will discuss several EDA and data visualizataions I performed while analyzing house sales data in King County, WA.

#### Setup

First of all, matplotlib offers [various styles sheets](https://matplotlib.org/3.1.0/gallery/style_sheets/style_sheets_reference.html) that can be easily changed using following code. I decided to use the 'ggplot' style.

```
plt.style.use('ggplot')
```

Also, I created a dictionary of column names and descriptions, which expediated the formatting process.

```
# Create dictionary of column descriptions
dict = {'id': 'ID',
       'date': 'Date Sold',
       'price': 'Price',
       'bedrooms': 'Number of Bedrooms',
       'bathrooms': 'Number of Bathrooms',
        'sqft_living': 'Square Footage of the House',
        'sqft_lot': 'Square Footage of the Lot', 
        'floors': 'Total Floors in House', 
        'waterfront': 'House With Waterfront View', 
        'view': 'Number of Times Viewed', 
        'condition': 'Condition', 
        'grade': 'Overall Grade',
       'sqft_above': 'Sqaure Footage of House Other Than Basement', 
        'sqft_basement': 'Square Footage of the Basement', 
        'yr_built': 'Built Year', 
        'yr_renovated': 'Renovated Year', 
        'zipcode': 'Zipcode',
        'lat': 'Latitude', 
        'long': 'Longitude', 
        'sqft_living15': 'Living Space Square Footage \nFor Nearest 15 Neighbors', 
        'sqft_lot15':'Land Lots Square Footage \nFor Nearest 15 Neighbors'}
```

#### Get started

When thinking of housing price, the most important factors that came to my mind were the size and the location of the house. So I started by graphing a scatter plot that shows the relationship between square footage of a home and that of houses in nearby neighborhoods. Do people who live in big houses live in a neighborhood with big houses too? 

#### Size - Bedrooms, Bathrooms and Living Space

```
plt.figure(figsize=(8,6))
plt.scatter(x=df['sqft_living'], y=df['sqft_living15'], alpha=0.5, label='Square Footage of Houses')

plt.legend()
plt.title('Square Footage of Houses in Nearby Neighborhoods', fontsize=15)
plt.xlabel(dict['sqft_living'], fontsize=13)
plt.ylabel(dict['sqft_living15'], fontsize=13)
plt.show()
```

![](https://imgur.com/qN4mV0t.png)

It seems like people who live in bigger houses tend to live with others who also live in bigger houses. The scatter plot shows that square footage of living space in a house is positively correlated with square footage of living spaces in nearest 15 neighborhood. However, the plot didn't give me more insightful information.

Then I became curious, what does number of "rooms" or "floors" mean in terms of house sizes? Does higher number of bedrooms mean higher number of bathrooms? Or, does higher number of floors mean the house has more living space?

I decided to look into distributions of certain variables by grouping them. For example, how does the number of bathrooms vary given specific number of bedrooms in a house?

```
df.boxplot(column='bathrooms', by ='bedrooms')
plt.suptitle('')
plt.title('Distribution of Number of Bathrooms by Number of Bedrooms', fontsize=15)
plt.xlabel(dict['bedrooms'], fontsize=13)
plt.ylabel(dict['bathrooms'], fontsize=13)
plt.show()
```

![](https://imgur.com/3ZtskUT.png)

Interestingly enough, as the number of bedrooms increases, the median number of bathrooms does not increase as much. In other words, although houses with more bedrooms are more likely to have more bathrooms, houses have equal or less number of bathrooms than bedrooms on average.

Then, I looked into the relationship between the price and square footage of living space. Below figure also shows the relationship between square footage of living space and number of floors in a house.

```
fig, ax = plt.subplots(ncols = 2, nrows = 1, figsize = (20,8))

# sqft_living vs. price
ax1 = plt.subplot(1, 2, 1)
plt.scatter(x=df['sqft_living'], y=df['price'], alpha=0.5)
plt.title('Square Footage of Houses and The Prices', fontsize=15)
plt.xlabel(dict['sqft_living'], fontsize=13)
plt.ylabel(dict['price'], fontsize=13)

# sqft_living vs. floors
ax2 = plt.subplot(1, 2, 2)
sns.boxplot(x='floors', y='sqft_living', data=df)
plt.title('Distribution of Living Space by Total Number of Floors', fontsize=15)
plt.xlabel(dict['floors'], fontsize=13)
plt.ylabel(dict['sqft_living'], fontsize=13)

plt.show()
```

![](https://imgur.com/3EDeDRO.png)

It is not surprising that square footage of living space and housing prices have strong positive correlation, as shown in the subplot on the left. Yet, the relationship between the square footage of living space and total number of floors in the house is pretty interesting. The boxplot on the right shows that having higher number of floors doesn't necessarily mean the house has bigger living space. In fact, the median square footage of living space does not vary much by the number of floors in the house.

#### Geographical Location

Going back to the very first scatter plot that shows positive correlation between living space of a house and living spaces in nearby neighborhoods, it is reasonable to expect the housing prices to depend on neighborhoods, especially if size of the house affects the price.

Fortuantely, I was able to easily visualize geographical locations of the houses on a scatter plot, using longitude and latitude information in the dataset. In addition, I played around with colormap option, to see how the prices are distributed and where the houses with waterfront view are located.

```
fig, ax = plt.subplots(ncols = 2, nrows = 1, figsize = (20,10))

# housing price
ax1 = plt.subplot(1, 2, 1)
plt.scatter(df['long'], df['lat'], c=df['price'], s=15, cmap='Set2', alpha=0.6, label='Price')
plt.title('King County Housing Prices', fontsize=15)
plt.xlabel('Longitude', fontsize=13)
plt.ylabel('Latitude', fontsize=13)
plt.legend()
plt.colorbar(orientation="horizontal")

# waterfront
ax2 = plt.subplot(1, 2, 2)
plt.scatter(df['long'], df['lat'], c=df['waterfront'], s=10, cmap='cool', alpha=0.5, label='waterfront')
plt.title('King County Houses With Waterfront', fontsize=15)
plt.xlabel('Longitude', fontsize=13)
plt.ylabel('Latitude', fontsize=13)
plt.legend()
plt.colorbar(orientation="horizontal")

plt.show()
```

![](https://imgur.com/bUO9uWg.png)

It looks like geographical location plays a significant impact on the housing prices in King County. The subplot on the left shows housing prices based on the longitude and latitude of each houses. The subplot on the right shows whether these houses have waterfront view or not. Not surprisingly, houses with waterfront view tend to be in the area where prices are higher.

The violinplot of housing prices grouped by waterfront variable below confirms this observation. The median price is higher for those with waterfront view.

```
plt.figure(figsize=(8,6))
sns.violinplot(y='price', x ='waterfront', data=df)

plt.title('Distribution of Housing Prices by Waterfront View', fontsize=15)
plt.xlabel(dict['waterfront'], fontsize=13)
plt.ylabel(dict['price'], fontsize=13)
plt.show()
```

![](https://imgur.com/icJiiG2.png)

It would be much cooler if I could see these data points on an actual map. I learned that the folium library makes this task very easy.

```
import folium
from folium.plugins import MarkerCluster, FastMarkerCluster, HeatMap
MarkerCluster()
```

When plotting a lot of data points on a map, marker clusters come in very handy -- when you zoom out, the markers would gather to bigger clusters so that it's easier to see the distribution on the map.

When I tried to plot the entire dataset on the map, it kept returning blank map because there were too many observations. To solve this problem I used `FastMarkerCluster`. As its name suggests, `FastMarkerCluster` plots the data in a second, by creating marker clusters.

In addition, I created a heat map based on the prices to see which area in the King County have higher housing prices. In order to do so, I identified and classified the `price` variable into deciles. 

```
# create buckets for prices based on quartiles -- for HeatMap
df['price_q'] = pd.qcut(df['price'], 10, labels=False)
df[['price','price_q']].head()
```

In order to plot data points on the graph, I simply stored the coordinates as list. I set the location of the map, which specifies which cooridnate the map should be "started", as the middle element of the `locationlist`.

```
locations = df[['lat','long']]
locationlist = locations.values.tolist()

map_start = locationlist[len(locationlist) //2]
len(locationlist)
```

```
map1 = folium.Map(location = map_start, tiles='CartoDB positron', control_scale=True,zoom_start=9)
```

Then, I mapped all data points using FastMarkerCluster, and added heatmap based on the variable `price_q` (deciles of `price`.

```
# add all points from the dataset to the map using FastMarkerCluster
marker_cluster = map1.add_child(FastMarkerCluster(locationlist))
map1.add_child(HeatMap(df[['lat','long','price_q']].values, radius=15))

map1
```

![](https://imgur.com/ZJUFAN4.png)

Alternatively, I could also map individual data points as markers. The cool feature about the code below is that I can use "tooltip" or "popup" option, by which I can specify a text to be displayed when hovering over or clicking the marker.

```
# add markers with price 
# for point in range(len(locationlist)):
#     folium.CircleMarker(locationlist[point], tooltip=df['price'][point], radius=0.11).add_to(marker_cluster)
# map1
```

#### Next Steps

When I tried plotting all of the data points using `folium.CircleMarker()` method, it slowed down my jupyter notebook a lot. I haven't figured out how to add markers -- more specifically, tooltips or popups -- using `FastMarkerCluster` method yet. I would like to keep exploring how to use folium and be able to make my maps more interactive.








