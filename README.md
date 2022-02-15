# CASE STUDY: Chicago Cyclist Bike-Share Analysis

##### Author: Emi Ly

##### Date: October 1, 2021

##### [Tabealu Dashboard](https://public.tableau.com/app/profile/emily.liang7497/viz/CyclisticBikeShareAnalysisDashboard/GiantDashboard)

##### [Tableau Story Presentation to Skateholders](https://public.tableau.com/app/profile/emily.liang7497/viz/CyclistBikeShareAnalysis/Story1)

#

_The case study follows the six step data analysis process:_

###  ❓ [Ask](#1-ask)
### 💻 [Prepare](#2-prepare)
### 🛠  [Process](#3-process)
### 📊 [Analyze](#4-analyze)
### 📋 [Share](#5-share)
### 🚲 [Act](#6-act)

## Scenario
In 2016, Cyclistic launched a successful bike-share offering. The company’s future success depends on maximizing the number of annual memberships. Therefore, your team wants to understand how casual riders and annual members use Cyclistic bikes differently. From these insights, your team will design a new marketing strategy to convert casual riders into annual members.

## 1. Ask
💡 **BUSINESS TASK: Analyze Divvy's riding data to develop digital marketing strategies to convert casual riders into annual members.**

Primary stakeholders: The director of marketing Lily Moreno and Cyclistic executive team.

Secondary stakeholders: Cyclistic marketing analytics team.

## 2. Prepare 
Data Source: 12 Month (Aug 2020 to August 2021) of Cyclistic trip Data from Motivate International Inc: [data source link](https://divvy-tripdata.s3.amazonaws.com/index.html) with [license](https://www.divvybikes.com/data-license-agreement).

The dataset has 12 CSV, 13 columns and 4.9 million rows. The data also follow a ROCCC approach:

- Reliability: the data includes complete and accurate ride data from Divvy. Divvy is program of the Chicago Department of Transportation (CDOT), which owns the city’s bikes, stations and vehicles
- Original: the data is from Motivate International Inc, which operates the City of Chicago’s Divvy bicycle sharing service.
- Comprehensive: The data incudes type of bikes, start and end station name, start and end time, station ID, station longtitude and latitude, membership types.
- Current: data is up to date to August 2021
- Cited: the data is cited and under current [license](https://www.divvybikes.com/data-license-agreement) agreement.

⛔ The dataset has limitations:

- Personally identifiable information: the dataset has a restriction of personally identifiable information, so we have no data if that a ride is by an unique rider or the same rider who ride more than once as a casual rider or a member. 
- NA values: after checking `sum(is.na(bike_data))`, we see the dataset has 1893790 NA values, such as in starting_station_id, end_station_id. Further investigation we noticed the NA values are mostly under rideable type: electric bike. Future investigations may be needed by the station names are not entered for electric bike. 

  ```
  head(count(bike_data, start_station_name, member_casual,  rideable_type, sort= TRUE))
  
  head(count(bike_data, end_station_name, member_casual,  rideable_type, sort= TRUE))
  ```

## 3. Process

Examine the data:

```
head(bike_data)
dim(bike_data)
colnames(bike_data)
summary(bike_data)
```

Indentify unnecessary data and remove those columns:

```
bike_data <- bike_data %>% select(-c(start_lat, start_lng, end_lat, end_lng))
```

Add two columns: ride length and day of the week:
```
bike_data <- bike_data %>% mutate(ride_length = ended_at - started_at) %>% mutate(day_of_week = weekdays(as.Date(bike_data$started_at)))

#Convert ride_length from from seconds into minutes
bike_data$ride_length <- as.numeric(bike_data$ride_length)
bike_data$ride_length <- as.numeric(bike_data$ride_length/60)
```

⛔ The started and ended time is in a yyyy-mm-dd hh-mm-ss format. We can further divide this into two columns: date and time. This step is optional.
```
bike_data <- separate(bike_data,"started_at",into=c('start_date','start_time'), sep=' ')
bike_data <- separate(bike_data,"ended_at",into=c('start_date','start_time'), sep=' ')
```

Remove data error:

```
#check for data with negative ride length:
bike_data <- bike_data[bike_data$ride_length>0,]

#check for data with ride length  more than 1 day (86400 seconds or 1440 mins):
sum(bike_data$ride_length > 1440)
```

Clean the data to prepare for analysis in 4. Analyze!

## 4. Analyze


Check min, max, mean, median and any outlier on the ride length. 

```
summary(bike_data$ride_length)
```

Aggregate the data based on user types.
```
aggregate(bike_data$ride_length ~ bike_data$member_casual, FUN = mean)
aggregate(bike_data$ride_length ~ bike_data$member_casual, FUN = median)
aggregate(bike_data$ride_length ~ bike_data$member_casual + bike_data$day_of_week, FUN = mean)
```
![000005](https://user-images.githubusercontent.com/62857660/135518463-b62936bd-ae6a-479d-9613-412ef341bfca.png)



Analyze ridership by user types and day of the week.
```
bike_data %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>%  
  group_by(member_casual, weekday) %>%  
  summarise(number_of_rides = n()							
  ,average_duration = mean(ride_length)) %>% 		
  arrange(member_casual, weekday)								
 ```
 ![000003](https://user-images.githubusercontent.com/62857660/135518560-3169ab87-8a83-41d3-aad2-136483a6d188.png)

 
 ⛔ For the complete R code and analyze the data using ggplot for graphical interpretation, please view the rmd file on this [R code link](https://github.com/xtenix88/Google-Data-Analytic-Capstone/blob/main/Cyclist-Data-Analysis-Google-Capstone.Rmd)!
 

## 5. Share 
🎨 **[Tabealu Dashboard on Bike-Sharing Case Study](https://public.tableau.com/app/profile/emily.liang7497/viz/CyclisticBikeShareAnalysisDashboard/GiantDashboard)**

![dash](https://user-images.githubusercontent.com/62857660/136834696-39ee5b7e-71ca-43f7-b7f9-a5d4c2fd53d6.PNG)

🎨 **[Tableau Presentation on Cyclistic Bike-Sharing Case Study](https://public.tableau.com/app/profile/emily.liang7497/viz/CyclistBikeShareAnalysis/Story1).**

![1](https://user-images.githubusercontent.com/62857660/136473917-62988816-3893-4ee1-b2ce-fdb17f9a6552.JPG)
![2](https://user-images.githubusercontent.com/62857660/136473929-be13f89d-6ebe-43f0-96c0-9dc7cb7d1c7b.JPG)




## 6. Act
Conclusion based on our analysis:
- Casual riders rides mostly during the weekends.
- Casual riders ride longer duration, but least total trips. 
- Casual riders rides longer on docked bike, but least total trips.
- Most popular station for casual riders are: Streeter Dr & Grand Ave, Lake Shore Dr & Monroe St, Millennium Park.
- Most active months for casual riders are from June to August.

Marketing recommendations to convert casual riders into members:

##### 🚩  Marketing effort on the top 5 most popular stations for the causal riders. It can be a booth, print media on the bike or the locking station area, or social media post on contest starting from the most popular stations. 

##### ⛱  Promotional short term membership offer during the summer months.

##### 🚴‍♂️ Promotional weekend term membership for the weekends.

##### 🎁 Point-award incentive system for riding more trips in a membership format to receive discount and partnership offers. 




