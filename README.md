# Strava-data-analysis
Can default bike be predicted with ride data?

## Introduction

This analysis uses my personal Strava data from all rides from start to Dec 2021. I use Tableau, Postgres, and R to conduct some basic analyses and to see whether it might be possible to implement a “guess my bike” feature on the Strava app, which could replace the default bike feature for riders who frequently use one of many bikes. This would make uploading rides faster, eliminating or reducing the need to edit the default bike.

## SQL Analysis

### How many rides were completed by each bike? Sort in descending order of count
```
SELECT bike_id, COUNT(id) as "Number of Rides by Bike"
FROM strava
GROUP BY bike_id
ORDER BY "Number of Rides by Bike" DESC;
```

<img width="584" alt="image" src="https://user-images.githubusercontent.com/72087263/188246111-b890c749-e599-4402-b854-d0cd11821f05.png">

Most rides have taken place with bike 3996174 (Cannondale CAAD10), followed by 4226605 (Salsa Mukluk), 3996178 (Titanium Litespeed), and 3996175 (Cannondale Trail MTB). 8 rides have been done using unnamed bikes.

### Display min and max suffer score by bike
```
SELECT bike_id, MIN(suffer_score), MAX(suffer_score)
FROM strava
WHERE suffer_score>0
GROUP BY bike_id;
```

<img width="523" alt="image" src="https://user-images.githubusercontent.com/72087263/188247319-f2e4e027-883f-4593-87a9-29059ea05349.png">

This output gives a range of suffer scores, which in itself doesn’t reveal much. However, it shows that there could be an outlier for bike 3996174.

### Display the average suffer score by bike in descending order to see if there is a difference
```
SELECT bike_id, AVG(suffer_score) as "Average Suffer Score"
FROM strava
GROUP BY bike_id
HAVING AVG(suffer_score)>0
ORDER BY "Average Suffer Score" DESC;
```

<img width="540" alt="image" src="https://user-images.githubusercontent.com/72087263/188247386-f90649f1-d264-435b-b54b-adce5fce604a.png">

The average suffer score reveals somewhat more information. The CAAD10 has the highest average suffer score, followed by the Litespeed. These are both road bikes and compare to somewhat lower average suffer scores of 74 and 60.6 for my Mukluk and Trail bikes, respectively. This indicates that the road bikes generally have higher suffer scores and warrants further examination.

### Which day of the week is a ride most likely to take place?
```
SELECT start_day, COUNT(start_day) AS "Number of Rides by Day"
FROM strava
GROUP BY start_day
ORDER BY "Number of Rides by Day" DESC;
```

<img width="610" alt="image" src="https://user-images.githubusercontent.com/72087263/188247445-73f69c9a-dbf7-48a8-afec-d38d343c023b.png">

These rides are fairly evenly dispersed, but there are somewhat more rides that have taken place during the week versus the weekend. Interestingly, Saturday has seen the fewest rides.

## Tableau Analysis

### The first SQL query viewed as a treemap in Tableau shows just how dominant the CAAD10 and Mukluk are in this dataset.

<img width="802" alt="image" src="https://user-images.githubusercontent.com/72087263/188247538-da1c77c8-d113-4579-94fd-fb85f473b5c3.png">

### To see if there is a possible relationship between bike and speed, the bikes and their average speed were plotted on a simple bar graph. These speeds are calculated values and represent the average moving speed of each bike. Surprisingly, the Litespeed comes out on top of the CAAD10 (surprising because the CAAD10 is a race bike, and the Litespeed is a randonneuring bike).

<img width="364" alt="image" src="https://user-images.githubusercontent.com/72087263/188247604-e9220da3-2c6c-4d16-bcd7-e23549f32032.png">

### This chart reveals that there are a considerable amount of rides on the CAAD10 that are low distance/low speed. Not easily seen in this chart are that the Mukluk and Trail are grouped relatively similarly to each other, and the Litespeed data points are grouped more closely with the majority of CAAD10 values, which is to be expected given the previous bar chart.

<img width="617" alt="image" src="https://user-images.githubusercontent.com/72087263/188247625-d2d8c1a7-d822-4cec-9750-25dd667c19b4.png">

### Because the average suffer score calculated in SQL revealed some counterintuitive findings, it follows to investigate the relationship between suffer score and other variables. Here, the suffer score calculation is visualized, since MPH and bike are essentially synonymous.

<img width="702" alt="image" src="https://user-images.githubusercontent.com/72087263/188247668-21cea6b9-aaaf-4adf-93f9-b03e570978dc.png">

### There appears to be a modest relationship between moving time and suffer score, but the relationship is not exactly clear through a visual analysis (the Mukluk has many data points that seem to overlap the Litespeed, including some very high suffer scores).

<img width="697" alt="image" src="https://user-images.githubusercontent.com/72087263/188247694-26c5fa14-6b21-4726-b32a-19887260c281.png">

### Distance, though basically an analog of moving time, also appears to be a reliable indication of suffer score. 

<img width="687" alt="image" src="https://user-images.githubusercontent.com/72087263/188247761-c02c1f29-8937-43d0-8dcc-189db9cf7add.png">

## R Analysis

Data was imported to R and filtered down to numeric data and calculated variables include MPH and stopped time.

### Correlation Matrix and Regression Analysis

<img width="470" alt="image" src="https://user-images.githubusercontent.com/72087263/188252968-da3331b7-2636-457a-b001-e6a5855b53d2.png">

  Bike id is not strongly correlated with any particular numeric variable besides mph. In fact, suffer score seems less a function of bike ridden and more a function of moving time with an almost perfect correlation of 0.99. Suffer score can be ruled out as a valuable predictor of bike id. Despite this, it may be useful to run a regression on some of these variables and see if there is any predictive capacity.
  For the purpose of this analysis, we will attempt to see whether we can predict if the default bike was ridden. Using this as the dependent variable and distance, moving time, stopped time, mph, feet of gain per mile, and suffer score as the independent variables, a multiple regression model was constructed. All variables, with the exception of suffer score are statistically significant with p-values under 0.05.
  Mph and distance are the most important variables, both of which increase the probability of a bike other than the default when they increase in value. It can be concluded that with further analysis, it could be possible to construct a classification model to predict a user’s bike based on ride data.

<img width="476" alt="image" src="https://user-images.githubusercontent.com/72087263/188253127-de258bd6-3878-454b-b7f4-54b61e86de56.png">

## R Code
```
#Instructions for downloading Strava data
#https://scottpdawson.com/export-strava-workout-data/
#https://gist.github.com/scottpdawson/74f85f60a7cf7fcc8ee527592dadf498
#
#This analysis seeks to determine if it is possible to build a model that can
#predict whether the default bike was used for an activity, or if the app
#should prompt the user to select a bike. Such a feature would save the user time
#from having to manually edit the activity after it has been uploaded.


library(tidyverse)
library(readr)

ridedata=read_csv("/Users/jimmyaspras/Downloads/result.csv")
as_tibble(ridedata)

spec(ridedata)

#Compute mph
ridedata$mph=ridedata$distance/(ridedata$moving_time_raw/3600)
ridedata$mph<-as.numeric(ridedata$mph)

#Compute stopped time
ridedata$stopped_time=(ridedata$elapsed_time_raw-ridedata$moving_time_raw)/60

#Compute feet per mile, elevation gain
ridedata$feet_per_mile=ridedata$elevation_gain/ridedata$distance

#Run correlation to determine relationships
#Isolate numeric columns
ridedatanumeric<-ridedata[c("bike_id","distance","moving_time_raw","elapsed_time_raw",
                            "stopped_time","elevation_gain","suffer_score","mph")]
ridedatanumeric<-na.omit(ridedatanumeric)
library(corrr)
cormat<-correlate(ridedatanumeric)
cormat

#The most significant relationships are the most intuitive ones - suffer score/time
#distance/elevation gain, time/elevation gain.

#Can a model be built to predict the bike ridden based on some simple ride data?
#
#Code the dependent variable (bike_id) to binary categorical

ridedata$bike_id_binary<-ifelse(ridedata$bike_id=="4226605","Default","Other")
ridedata$bike_id_binary<-as.factor(ridedata$bike_id_binary)

#Binary model
library(tidymodels)
bikemod = logistic_reg() %>%
  set_engine("glm") %>%
  fit(bike_id_binary ~ distance + moving_time_raw + 
        stopped_time + mph + feet_per_mile + 
        suffer_score, data = ridedata)
bikemod
library(broom)
bikemod2<-tidy(bikemod)
bikemod2
glance(bikemod)
```
