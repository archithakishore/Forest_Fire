## Introduction

We will be studying the Forest Fires Data Set from https://archive.ics.uci.edu/ml/datasets/forest+fires put together and studied by Cortez, P., & Morais, A. D. J. R. (2007) [1]. This dataset covers meteorological and spatio-temporal data for forest fires (between 2000 and 2003) in Portugal’s Montesinho Natural Park, with 13 attributes each for 517 such incidents. Our target attribute from these 13 is ‘area’ - total burned area in hectares (ha). The corresponding weather data is that which is registered by sensors at the moment the fire was detected (or first broke out). We have: temp, RH (realtive humidity), wind (speed), rain (accumulated precipitation over the last 30 minutes).
 
We also have DMC, DC, ISI and FFMC are components of the Canadian Forest Fire Weather Index (FWI) System [2], which measures the effects of fuel moisture and wind on fire behaviour. These have been calculated using consecutive daily observations of temperature, relative humidity, wind speed, and 24-hour rainfall - i.e. these are time-lagged and not instantaneous values, unlike the four weather variables. Roughly, the higher these components, the more the expected severity of the fire.

Spatio-temporal data includes the month and day of the week that the incident occurred, and X and Y co-ordinates of the incident with respect to the park. As noted in [1], smaller fires are much more frequent. This is the case in this dataset, as well as with incidences of wildfires around the world, making this a difficult regression problem.

### Problem and Metric

We work on the following regression problem:

Predicting the burned area of a forest fire, given the weather conditions and FWI components at the time the fire breaks out.
In the future, this data can be monitored or obtained in real-time, and is non-costly.
This prediction can provide instant feedback to the appropriate diaster response teams.
It could also be re-framed as a multi-class classification problem, by dividing the incidents based on severity - Small, Medium, Large - and predicting this, rather than the absoulute burned area.
As this is a regression problem, our main evaluation metric is “test” (or validation) RMSE

## Motivation 

The environmental damage, financial and infratstructure loss from forest fires (or wildfires) can be staggering. Reports from CoreLogic, a property and consumer information provider, estimate that the loss from the 2018 California Wildfires is between 15 and 19 billion USD [3]. With forest fires sweeping the globe and worsening forest fire seasons [4], there is need for a tool such as this to improve firefighting resource management and disaster response. For example, when there are simultaneous occurrences of wildfires, we would be able to prioritize and allocate resources appropriately: ground crew could respond to fires judged to be “smaller”, with air support diverted to locations of larger fires.

## Literature Review

The frame of reference for our regression task is work by Cortez, P., & Morais, A. D. J. R. (2007). They compared performance of SVM, NN and multiple linear regression and RF from the ‘rminer’ package, with the best performance from SVM using just the four weather variables - rain, wind, temperature and humidity. While this dataset hasn’t been studied extensively, similar work in this domain has been done by ?zbayoglu, A. M., & Bozer, R. (2012) [5]. They predicted burned area in hectares, as well as the size-class of the fire as big, medium or small, using SVM and multi-layer perceptrons, with data from 7,920 forest fires in Turkey between 2000 and 2009. However, this dataset wasn’t accessible to us. In addition to weather and meteorological data, they used geographic features like the type of trees and number of trees per unit area.

## Exploratory Data Analysis

Initial EDA and visualization to get a better idea of our next steps and the required data pre-processing.

```
library(tidyverse)
library(psych)
library(caretEnsemble)
library(caret)
library(corrplot)
library(ModelMetrics)

# load data
data <- read.csv(file = "forestfires.csv", stringsAsFactors = T)
```
```
ggplot(data, aes(data$area)) + geom_histogram()
```
<img src="https://github.com/archithakishore/Forest_Fire/blob/master/Images/figure-html/unnamed-chunk-2-1.png" alt="hi" class="inline"/>
 
We can see that the data is heavily skewed towards small forest fires. There are 247 entries with area = 0, for incidents where the resulting burned area was < 100m2. A logarithmic transofrmation of area might be useful in this case, and we can add 1 to the target column first since ln(0) approaches negative infinity.

Check correlation - there is some collinearity based on which we might drop features:
```
corPlot(Filter(is.numeric, data))
```
<img src="images/Emoticons/cool.png" alt="hi" class="inline"/>
[https://github.com/archithakishore/Forest_Fire/blob/master/Images/figure-html/unnamed-chunk-4-1.png]

Day/month wise incidents:
```
plot(data$day, col = "purple")
```
<img src="images/Emoticons/cool.png" alt="hi" class="inline"/>
https://github.com/archithakishore/Forest_Fire/blob/master/Images/figure-html/unnamed-chunk-3-3.png

```
plot(data$month, col = "purple")
```
<img src="images/Emoticons/cool.png" alt="hi" class="inline"/>
https://github.com/archithakishore/Forest_Fire/blob/master/Images/figure-html/unnamed-chunk-3-4.png

Day/Month wise area Burnt:
```
ggplot(data, aes(x = data$month, data$area)) + geom_boxplot(outlier.shape = NA) + geom_jitter(col = "red") + theme_bw()
 ```
 <img src="https://github.com/archithakishore/Forest_Fire/blob/master/Images/figure-html/unnamed-chunk-3-5.png" alt="hi" class="inline"/>

```
ggplot(data, aes(x = data$day, data$area)) + geom_boxplot(outlier.shape = NA) + geom_jitter(col = "red") + theme_bw()
```
<img src="https://github.com/archithakishore/Forest_Fire/blob/master/Images/figure-html/unnamed-chunk-3-6.png" alt="hi" class="inline"/>

## Data Preparation
# Outlier Detection

We see from the boxplot below that there are some outliers that may skew our results. These are removed, so that we are predicting burned areas of smaller to medium forest fires. This is reasonable, since those are the more frequently encountered ones in real-life scenarios as well, and the ones for which resource prioritization may matter more. We are left with 501 observations after this.

<img src="https://github.com/archithakishore/Forest_Fire/blob/master/Images/figure-html/unnamed-chunk-6-1.png" alt="hi" class="inline"/>




Syntax highlighted code block


# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/archithakishore/Forest_Fire/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
