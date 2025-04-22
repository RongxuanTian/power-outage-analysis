# power-outage-analysis

## Introduction

This report explores data on past major power outages collected by Sayanti Mukherjee, Roshanak Nateghi, and Makarand Hastak from their paper [link](https://www.sciencedirect.com/science/article/pii/S2352340918307182). The dataset contains 1,536 rows and 57 columns, where each row corresponds to an occurrence of a power outage in the continental U.S., and each column represents a unique feature of the event. The features span across domains such as temporal information, causes, economic characteristics of the region, population of the region, etc.

In this report, we focus specifically on the duration of each power outage and the urban population and economic information of the region in which the outage occurred. We aim to understand how the urban population and economic characteristics of the region affect the duration of the power outage.

Hence, we will be looking at the following columns/features:

- `OUTAGE.DURATION`: Duration of outage events (in minutes)  
- `POPDEN_URBAN`: Population density of the urban areas (persons per square mile)  
- `AREAPCT_URBAN`: Percentage of the land area of the U.S. state represented by the land area of the urban areas  
- `RES.SALES`: Electricity consumption in the residential sector (megawatt-hour)  
- `RES.PRICE`: Monthly electricity price in the residential sector (cents/kilowatt-hour)  
- `RES.CUST.PCT`: Percent of residential customers served in the U.S. state (in %)

The motivation behind selecting these features is that urban environment and density could affect outage risk due to infrastructure stress. Economic characteristics of electricity also inform us about the residential demand for electricity, especially in urban neighborhoods. Hence, investigating these factors seems crucial to understanding how urban areas might influence power outages.

## Data Cleaning and Exploratory Analysis

### Data Cleaning and Imputation

Two redundant columns were dropped using `.drop()`. The `variables` column, which contains only `NaN` values, was dropped. The `OBS` column, which only contains the index for each of the rows, was dropped as a column and used as the index for the DataFrame using `.set_index()`. The six features above were originally in simple objects, so they were converted to floats (`float64`).

The first row of the DataFrame contained units by which the column is measured; this was dropped to ensure the DataFrame only contained actual data.

We also checked whether there were any missing values from the features. `OUTAGE.DURATION`, `RES.SALES`, `RES.PRICE`, and `POPDEN_URBAN` all had missing values.

For `OUTAGE.DURATION`, out of the 1,534 rows containing actual data, 58 rows had `nan` values. Since `OUTAGE.DURATION` is the target variable we are trying to predict, we decided to drop these rows to avoid interfering with the prediction process.

For `RES.SALES`, `RES.PRICE`, and `POPDEN_URBAN`, we used probabilistic imputation to fill in the missing values based on the state of each region, since we know these features are mostly dependent on their location. We also avoided mean imputation to prevent artificial spikes.

Finally, we created a `DURATION_IN_HOURS` column from `OUTAGE.DURATION`, which was measured in minutes, since expressing outage duration in hours offers a more interpretable unit for analysis—especially when comparing longer outages across different states.

Below shows the head of the cleaned dataset:

### Univariate Graphs

We first visualized the distribution of outage duration in hours, `DURATION_IN_HOURS`.

(insert uni graph 1)

The pink dashed line indicates the 24-hour mark, when the outage duration lasts for a day. By the plot, it seems that over half of the outages lasted less than a day. As such, most of the outages are fixed very quickly. It is also worth noting that eight outages lasted longer than 480 hours; hence, they are excluded from the graph for better visualization purposes.

We also examined the distribution of electricity consumption in residential sectors.

(insert uni graph 2)

By the histogram, it seems that for most of the outages, the region in which they occur has fallen under consuming 5 million megawatt-hours. This perhaps suggests that outages may be more common in less energy-intensive residential areas and that the bulk of outages lasting less than 24 hours occur in regions with less than 5 million megawatt-hours of consumption.

## Bivariate Graphs

(insert Bi graph 1)

Looking at the scatter plot, there doesn’t seem to be much correlation between electricity consumption and outage duration as previously thought when examining the univariate graphs. A good number of outages that lasted more than 24 hours occurred in regions with more than 5 million megawatt-hours of consumption. This suggests that electricity consumption is not linearly correlated with the duration of outages.

For the second graph, we would like to see the distribution of the different causes in relation to outage duration.

(insert Bi graph 2)
