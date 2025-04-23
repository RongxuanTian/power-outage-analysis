# Predicting outage duration from the urban population and economic characteristics of the region?

Rongxuan Tian
rxtian@umich.edu

## Introduction

This report explores data on past major power outages in the U.S. collected by Sayanti Mukherjee, Roshanak Nateghi, and Makarand Hastak from their [paper](https://www.sciencedirect.com/science/article/pii/S2352340918307182). The dataset contains 1,536 rows and 57 columns, where each row corresponds to an occurrence of a power outage in the continental U.S., and each column represents a unique feature of the event. The features span across domains such as temporal information, causes, economic characteristics of the region, population of the region, etc.

In this report, we focus specifically on the duration of each power outage and the urban population and economic information of the region in which the outage occurred. We aim to understand **how the urban population and economic characteristics of the region affect the duration of the power outage**.

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

<iframe src="assets/clean_table.html" width="800" height="400" frameborder="0">
</iframe>

### Univariate Graphs

We first visualized the distribution of outage duration in hours, `DURATION_IN_HOURS`.

<iframe
 src="assets/uni1.html"
 width="800"
 height="400"
 frameborder="0"
 ></iframe>

The pink dashed line indicates the 24-hour mark, when the outage duration lasts for a day. By the plot, it seems that over half of the outages lasted less than a day. As such, most of the outages are fixed very quickly. It is also worth noting that eight outages lasted longer than 480 hours; hence, they are excluded from the graph for better visualization purposes.

We also examined the distribution of electricity consumption in residential sectors.

<iframe
 src="assets/uni2.html"
 width="800"
 height="400"
 frameborder="0"
 ></iframe>

By the histogram, it seems that for most of the outages, the region in which they occur has fallen under consuming 5 million megawatt-hours. This perhaps suggests that outages may be more common in less energy-intensive residential areas and that the bulk of outages lasting less than 24 hours occur in regions with less than 5 million megawatt-hours of consumption.

### Bivariate Graphs

<iframe
 src="assets/biv1.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>

Looking at the scatter plot, there doesn’t seem to be much correlation between electricity consumption and outage duration as previously thought when examining the univariate graphs. A good number of outages that lasted more than 24 hours occurred in regions with more than 5 million megawatt-hours of consumption. This suggests that electricity consumption is not linearly correlated with the duration of outages.

For the second graph, we would like to see the distribution of the different causes in relation to outage duration.

<iframe
 src="assets/biv2.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>

It seems that different causes do have correlation with outage duration. Intentional attack, equipment failure, operability failure all have lower outage duration times since their range falls under 50 hours. As such, if the outage duration is significantly longer, we can quite reliably deduce that it is most probably either fuel supply emergency or severe weather. As such, causes of outage could potentially be a reliable indicator of outage duration. More to note, seven outliers lie outside of five hundred hours hence are ignored from the plot for better visualization purposes.

### Aggregate Table

The below aggregate table shows the mean value for all of the features: 'RES.SALES', 'RES.PRICE', 'POPDEN_URBAN', 'RES.CUST.PCT', 'AREAPCT_URBAN' for each U.S. state. It appears that for most features, there is little drastic variation for most U.S. states, which suggests that perhaps each state where the outage occurs might not provide the most salient information.

 <iframe src="assets/agg_table.html" width="800" height="400" frameborder="0">
</iframe>

The below graph demonstrates the urban area distribution for each state:

<iframe
 src="assets/biv3.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>


## Prediction Problem

We would like to know how long the duration of the outage will last based on information about urban population and economic characteristics. As such, the goal of the model is to predict outage duration `OUTAGE.DURATION` using urban population density `POPDEN_URBAN`, percentage of urban land area of the total state area `AREAPCT_URBAN`, residential electricity consumption `RES.SALES`, monthly electricity price in the residential sector `RES.PRICE`, and percent of residential customers served `RES.CUST.PCT`.  

The response variable is `OUTAGE.DURATION`, measured in minutes. This is one of the most pertinent variables that inform us about the severity and effect of an outage. This is a regression problem since we are trying to predict duration, a single, continuous, numerical variable. 

The metric in which we choose to evaluate our model is Mean Squared Error since it is the most appropriate to capture the average distance between the prediction and the actual data for a linear regression model. The higher the difference between the predicted and the actual, the more penalty is placed—this is something we would like to emphasize. Using mean absolute error would not emphasize this difference. We would also pay attention to the R² score of the model which informs us how well our features are able to capture the actual outage duration.

At the time of the prediction, we would know all of the population and economic information regarding the region as they would remain the same before and after the outage. Hence using the features to predict outage duration is feasible in the real world. Data such as number of customers affected and demand loss, which can be collected after the outage has occurred, will not be included.


## Baseline Model

Our initial, baseline model is a multilinear regression model trained on five features:

- `POPDEN_URBAN`: quantitative data - urban population density  
- `AREAPCT_URBAN`: quantitative data - percentage of urban land area of the total state area  
- `RES.SALES`: quantitative data - residential electricity consumption  
- `RES.PRICE`: quantitative data - monthly electricity price in the residential sector  
- `RES.CUST.PCT`: quantitative data - percent of residential customers served  

We then calculated the MSE for both predictions based on training data and testing data. The size of the MSE depends on the actual values from the data, hence MSE by itself does not offer much insight into the exact accuracy of our prediction. To better assess the model's performance, we compared the MSEs of our predictions to the variances of the actual outage durations in the training and testing sets. The variances of the training and testing data can be considered as the MSE of the constant function that always predicts the mean outage duration. Therefore, if our linear regression model were to yield meaningful predictions, the MSE of the predictions based on the training and testing data should be significantly lower than the variances of the actual outage durations from the training and testing data.

We also examined the R² value for the model to observe how strongly our selected features correlate with the actual values. Since all of our five features are quantitative, we didn’t need to include any encodings which would have been necessary for categorical features.

The MSE obtained from comparing the prediction from training data and the actual power outage durations from training data is about `35282348.03`, the variance of the outage durations from the training data set is `35433163.42`. There is no noticeable difference between the variance and the MSE. The baseline regression model doesn’t seem to produce any meaningful predictions since it appears that it doesn’t do any better than the constant function that predicts the mean.

Looking at the results from the testing data, the MSE obtained from comparing the prediction and the actual outage durations from testing data is about `34875044.02` and the variance of the outage durations from testing data is about `34798811.40`. Once again, this confirms that our regression baseline did not meaningfully capture anything and did not do any better than a constant model.

Examining the R² value for our model, it also shows the features have very little correlation with the target variable as the R² value is `0.00425`, which is extremely weak.


## Final Model

From the R² value of the baseline model, we can tell that the features we have selected are not the best for predicting duration. Hence, for the final model, we would like to change certain features and include new ones. 

The eventual features that were used were:

- `CUSTOMERS.AFFECTED`: quantitative data  - Numer of residential customers affected
- `CAUSE.CATEGORY`: nominal data  - Types of causes
- `AREAPCT_URBAN`: quantitative data - percentage of urban land area of the total state area  
- `RES.SALES`: quantitative data - residential electricity consumption  
- `RES.CUST.PCT`: quantitative data - percent of residential customers served  

`RES.PRICE` was excluded since it bears similarity to `RES.SALES`. `CAUSE.CATEGORY` is included since, as shown from the bivariate graph, different causes seem to have variation on the duration of the outage, hence it could have more correlation with outage duration. `CUSTOMERS.AFFECTED` is also included since it bears relation to urban population density—as the higher the density, the more customers are expected to be affected by the outage. 

It's key to note that `CUSTOMERS.AFFECTED` had several missing rows. We eventually decided that it was not appropriate to impute these values as:  
1) There is no good method to impute—unlike previous probabilistic imputations, `CUSTOMERS.AFFECTED` is not state-dependent.  
2) Since this is the value we are trying to use for prediction, imputing it could lead to misalignment with the real world.  

As such, we dropped the rows in which `CUSTOMERS.AFFECTED` had missing values.

Since we are using a nominal variable this time, we used `OneHotEncoder` to encode the variable later for regression. Moreover, we first fitted `CUSTOMERS.AFFECTED`, `RES.CUST.PCT`, `RES.SALES`, `AREAPCT_URBAN` to polynomial features. Previous baseline attempts at fitting the features using linear regression didn’t work well. This could suggest that the shape of the features is too complex and that the relationship between the features is non-linear. Therefore, to increase complexity in fitting, we selected `PolynomialFeatures()` to better accommodate their non-linearity. After applying transformations on these features, we applied linear regression to fit the data.

The hyperparameter we varied is the degree of the polynomial features applied to the four quantitative features. We tried degrees between 1 and 20. The higher the degree, the more concavity there is to the relationship between the four quantitative features. We performed 5-fold cross-validation to evaluate the hyperparameters and we used negative mean absolute error for scoring. The result showed that a polynomial of degree 3 best fits the data, which shows that the relation between the four quantitative features is not extremely intricate.

Same as the baseline model, we observed the variance of the actual data and MSE between prediction and the actual data as well as the R² score. The variance of the training data is `20485458.09` and the MSE between prediction and the actual training data is `15976629.97`, demonstrating a noticeable decrease compared to the baseline model. The variance of the testing data is `17280608.55` and the MSE between prediction and the actual testing data is `15955072.49`, demonstrating a decrease less than that of the training data but still better compared to the baseline. The R² value of the prediction is `0.22`. This is not a very high score, but still an improvement from the baseline.

Since incorporating the `CUSTOMERS.AFFECTED` feature caused us to clean the data again, this means that we have changed the training and testing data that we had for the baseline. As such, we re-fitted the baseline model with the new training data and made predictions.

The new R² value for the baseline prediction is `0.012`, much lower than the R² value for the final model.  
The MSE of the prediction for training data is `20232003.54` and the MSE of testing data is `17181913.56`. Both of which are much closer to the variance of the actual data than the final model.
