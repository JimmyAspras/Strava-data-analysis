# Strava-data-analysis
Can default bike be predicted with ride data?

## Introduction

This analysis uses my personal Strava data from all rides from start to Dec 2021. I use Tableau, Postgres, and R to conduct some basic analyses and to see whether it might be possible to implement a “guess my bike” feature on the Strava app, which could replace the default bike feature for riders who frequently use one of many bikes. This would make uploading rides faster, eliminating or reducing the need to edit the default bike.

## SQL Analysis

### How many rides were completed by each bike? Sort in descending order of count

SELECT bike_id, COUNT(id) as "Number of Rides by Bike"

FROM strava

GROUP BY bike_id

ORDER BY "Number of Rides by Bike" DESC;


Most rides have taken place with bike 3996174 (Cannondale CAAD10), followed by 4226605 (Salsa Mukluk), 3996178 (Titanium Litespeed), and 3996175 (Cannondale Trail MTB). 8 rides have been done using unnamed bikes.
Most rides have taken place with bike 3996174 (Cannondale CAAD10), followed by 4226605 (Salsa Mukluk), 3996178 (Titanium Litespeed), and 3996175 (Cannondale Trail MTB). 8 rides have been done using unnamed bikes.
