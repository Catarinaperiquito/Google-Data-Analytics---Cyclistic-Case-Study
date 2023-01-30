# **The Cyclistic Case Study - Google Data Analytics Capstone**
---
### Author: Catarina Periquito
### Date: 2023-01-16
---

![](https://miro.medium.com/max/586/0*X1O2EZvkAkyX9PWf.webp)

# **Cyclistic bike-share** 
***

Cyclistic is a bike-share company in Chicago, founded in 2016 that features more than 5,800 bicycles and 600 docking stations. 
It sets itself apart by making bike-share more inclusive to people with disabilities 
There are two types of customers: annual members and casual customers.
They have a flexibility pricing plans: single-ride passes, full-day passes, and annual memberships.
Cyclistic’s finance analysts have concluded that annual members are much more profitable than casual riders.

# **Scenario**
***
The Cyclistic Case Study is one of the capstone projects of Google Data Analytics, a professional certificate program offered on Coursera. 
I roleplayed as a data analyst working on the marketing analyst team at Cyclistic. 
Lily Moreno, The director of marketing and my manager assigned me to explore the behavior of casual riders and annual members to understand how they use Cyclistic bikes differently. 


# 1. Ask
***
## 1.1 Identify the business task:
* How do annual members and casual riders use Cyclistic bikes differently?


## 1.2 Questions 

* How many casual vs membership rides take place?
* Is the number of rides seasonable?
* When do different users ride throughout the day? and Week?
* What is the duration of each user type rides? Does it vary per week day or month?
* Do members and casual users have different bike type preferences?
* Where does each type of user start and end their trip?


## 1.3 Consider key stakeholders:

#### Primary stakeholders
  
  * **Cyclistic executive team:** The notoriously detail-oriented executive team will decide whether to approve the
recommended marketing program.
  * **Lily Moreno:** The director of marketing and your manager. Moreno is responsible for the development of campaigns
and initiatives to promote the bike-share program. These may include email, social media, and other channels.
  
#### Secundary stakeholders

  * Cyclistic marketing analytics team
  
  
# 2. Prepare
***

## 2.1 Data source used:

* **Data source link:** [Cyclistic’s Data](https://divvy-tripdata.s3.amazonaws.com/index.html)
 
* **Data information:** Public Data, dataset made available through Motivate International Inc. under a [license](https://ride.divvybikes.com/data-license-agreement) from divvybikes.com. 
	+ Cyclistic is a fictional company. For the purposes of this case study, the datasets are appropriate and will enable to answer the business questions.
  + Since this is a public data or open source information, we can use this for the capstone project

## 2.2 Data credibility:
* The **ROCCC** method which stands for **R**eliable, **O**riginal, **C**omprehensive, **C**urrent, and **C**ited, will be used to determine the credibility and integrity of the data source provided.
	+ The data we are working on is **first-party type**: data collected and used by Cyclistic. 
	+ The data integrity was checked and deemed unbiased.

## 2.3 Dataset analysed:
The Cyclistic’s dataset has the historical trip data from 2022, separeted in csv. files to analyze and identify trends. 

### Loaded packages in R programing language

* Step 1 - Started by installing your required packages.
```{r}
install.packages("tidyverse")
install.packages("readr")
install.packages("tidyr")
install.packages("janitor")
install.packages("skimr")
install.packages("dplyr")
install.packages("lubridate")
```

* Step 2 - Once packages are installed, we can load them
```{r}
library(tidyverse)
library(readr)
library(tidyr)
library(janitor)
library(skimr)
library(dplyr)
library(lubridate)
```

* Step 3 - Import data using use the `read_csv()` function

```{r}
cyclistic_01 <- read_csv("202201-divvy-tripdata.csv")
cyclistic_02 <- read_csv("202202-divvy-tripdata.csv")
cyclistic_03 <- read_csv("202203-divvy-tripdata.csv")
cyclistic_04 <- read_csv("202204-divvy-tripdata.csv")
cyclistic_05 <- read_csv("202205-divvy-tripdata.csv")
cyclistic_06 <- read_csv("202206-divvy-tripdata.csv")
cyclistic_07 <- read_csv("202207-divvy-tripdata.csv")
cyclistic_08 <- read_csv("202208-divvy-tripdata.csv")
cyclistic_09 <- read_csv("202209-divvy-publictripdata.csv")
cyclistic_10 <- read_csv("202210-divvy-tripdata.csv")
cyclistic_11 <- read_csv("202211-divvy-tripdata.csv")
cyclistic_12 <- read_csv("202212-divvy-tripdata.csv")

```



## **Process**
***

In order to process the data I also used the R programing language. 

* The separated .csv files were combined by row vertically, since all datasets had the same fields and data types.
* Duplicates were cleaned.
* The Timestamp data type was transformed into new time descriptive columns: weekday, month and time, generating new columns. 
* 4 columns were removed(start_lat , start_lng, end_lat, end_lng) since they aren't going to help with this case study. 
* NAs or missing values weren't removed because they could also tell us something about the customers' behavior.
* Created a column to calculate the average ride lenght 
* Removed trips that had zero or negative time in the ride lenght column as I assumed they were incorrect
* In the above transformations, I removed almost 1.3 million rows to be left with a clean combined table with 4.369.052 rows.

#### Combining and Modifying data

```{r}
all_trips_raw1 <- rbind(cyclistic_01, cyclistic_02, cyclistic_03, cyclistic_04, cyclistic_05, cyclistic_06, cyclistic_07, cyclistic_08, cyclistic_09, cyclistic_10, cyclistic_11, cyclistic_12)
```

#### Changing columns name to be more legible
```{r}
all_trips_raw1 <-all_trips_raw1 %>% 
                  rename(bike_type=rideable_type,
                   start_time=started_at,
                   end_time=ended_at)
```


#### Remove the latitute and longitude fields

```{r}
all_trips_raw <- all_trips_raw1 %>%
  select(-c(start_lat, start_lng, end_lat, end_lng))
```

#### Confirm final columns of dataset

```{r}
colnames(all_trips_raw)
```

#### Data Cleaning 

```{r}
all_trips_raw <- all_trips_raw [!duplicated(all_trips_raw), ] 
```

the variable ride_id has no duplicates, it is also a primary key. Each Id has 16 characters and each row represents a unique bike ride. It's important to understand if ride_id is the user id or if a new ride generates a new ride id.

#### Adding columns for date, day of the week, month and time

```{r}
all_trips_raw$date <- as.Date(all_trips_raw$start_time) 
all_trips_raw$weekday <- format(as.Date(all_trips_raw$date), "%A") 
all_trips_raw$month <- format(as.Date(all_trips_raw$date), "%B")
all_trips_raw$hour <- hour(all_trips_raw$start_time)
```

#### Add calculated field for ride_length in minutes

```{r}
all_trips_raw<-all_trips_raw %>%
mutate (ride_length=difftime(end_time, start_time, unit="mins"))
```

After verifying the previews dataset, I spotted trips with zero minutes and negative time. I decided to remove them for the analysis and created a new dataset, as I assumed these were registered incorrectly.

```{r}
all_trips_raw[all_trips_raw<=0] <- NA
all_trips <- na.omit(all_trips_raw)
```


#### Confirm final dataset

```{r}
head(all_trips)
```

 
## **Analyse/share**
***
To analyse my dataframe I'm combining R programming language with SQL and create data visualization using Tableau. 

```{r}
install.packages("sqldf")
library(sqldf)
```

So fist we want to know how many annual members vs casual users we have. What can we tell from the findings?

### Is Rider_id the user id or is it the id of a new ride?

```{r}
ride_user <- sqldf("select count (distinct ride_id), count (ride_id) from all_trips ")
```

The number of unique ride_id (4369052) is the same as the count of all ride_id (4369052), which means that ride_id can never be a user identifier. If it was each member only used this service once a year.
 

### **How many casual vs membership rides take place?**

```{r}
members_vs_casual <- sqldf("select member_casual, count (distinct ride_id) from all_trips 
group by member_casual")

```
![](https://github.com/Catarinaperiquito/Google-Data-Analytics---Cyclistic-Case-Study/blob/16a7d9b74bc6a8e570ca31f594a8a7dc6a2d6e05/Captura%20member_ride.png)

the data shows that we have more rides from annual members (59,03%) than casual users(40.9%) in 2022.
Even though we have more usage from members we still have a lot activity from casual users. More analysis is needed.


### **Is the number of rides seasonable?**

```{r}
rides_month <- sqldf("select member_casual, month, count (distinct ride_id) from all_trips 
group by member_casual, month")
```
![](https://github.com/Catarinaperiquito/Google-Data-Analytics---Cyclistic-Case-Study/blob/16a7d9b74bc6a8e570ca31f594a8a7dc6a2d6e05/Captura%20ride_month.png)

We can see by the results that both type of customers are more active during the warmer month. So this is not a distinguishable factor. Nonetheless it is clear that casual users ride more than member during spring and summer. This way we can conclude that seasons affect the amount of membership rides. 


### **What is the duration of each user type rides? Does it vary per week day or month?**

* Calculations with ride_length

```{r}
mean(all_trips$ride_length)
median(all_trips$ride_length)
max(all_trips$ride_length)
min(all_trips$ride_length)
```
Right away I spotted something interesting in the max ride length. It seems that we have some outliers with trips of more than 24 hours (1440 min)  After some analysis I decided to leave this data in the dataset. The starting and ending ride time were in different days, which means that a customer had the bike for more than one day.  

```{r}
Ride_time <- sqldf("select month, weekday, member_casual, avg(ride_length) from all_trips 
group by month, weekday, member_casual")
```
![](https://github.com/Catarinaperiquito/Google-Data-Analytics---Cyclistic-Case-Study/blob/16a7d9b74bc6a8e570ca31f594a8a7dc6a2d6e05/Captura%20Ride_time_week.png)

![](https://github.com/Catarinaperiquito/Google-Data-Analytics---Cyclistic-Case-Study/blob/16a7d9b74bc6a8e570ca31f594a8a7dc6a2d6e05/Captura%20Ride_time_month.png)
Casual users ride longer than members. This can mean that their is no limit time for casual customers or they have more money to spend, maybe tourists. Also Cyclistic doesn't have a pricing limit per ride. they might be losing annual memberships because of the full-day passes.
Members have a more consistent rides throughout week and month, with a slight increase during the weekend and warmer months, during these times they might have more free time.

### **When do different users ride throughout the day? and week?**

```{r}
rides_day <- sqldf("select member_casual, hour, weekday, count(distinct ride_id) from all_trips group by member_casual, hour, weekday")
```
![](https://github.com/Catarinaperiquito/Google-Data-Analytics---Cyclistic-Case-Study/blob/16a7d9b74bc6a8e570ca31f594a8a7dc6a2d6e05/data%20viz%20-%20histogram%20(2).png)

By the data visualization it seems that annual members use Cyclistic mostly to commute to work or during rush hours and during lunch time. Casual riders use this service more sporadically, but mostly from 10-19h.

![](https://github.com/Catarinaperiquito/Google-Data-Analytics---Cyclistic-Case-Study/blob/16a7d9b74bc6a8e570ca31f594a8a7dc6a2d6e05/Captura%20rides_week.png)
 
 Once again the assumption that annual members use Cyclistic services to ride to work is even more clear. They use it especially from monday to friday which are normal work days. In the other hand casual users ride are more active during the weekend. And during saturday rides almost double meaning that casual customers use this service especially during freetime and for leisure purposes.

### **Do members and casual users have different bike type preferences?**

```{r}
rides_bike <- sqldf("select member_casual, bike_type, count (distinct ride_id) from all_trips group by member_casual, bike_type ")
```

![](https://github.com/Catarinaperiquito/Google-Data-Analytics---Cyclistic-Case-Study/blob/main/Captura%20rides_bike.png)

Both groups prefer to ride in classic bikes, which doen't show much diffence between the two groups. 
However only casual riders use docked bikes. This might indicate that this type of customers don't care about their starting and ending point. This evidence points, again, to the idea of casual users being tourists.
In the other hand Members don't go for docked bikes because they want the freedom to park their bikes when they go to work.

### **Where does each type of user start and end their trip?**

```{r}
station_users <-  sqldf("select start_station_name, member_casual, count(ride_id) from all_trips group by  start_station_name, member_casual")
```


# **Conclusion**
***

* The analysis results seem to show that annual members and casual users are two different customer groups.
 
* More than half (59,76%) of rides are made by Cyclistic annual members. 

* The average ride time for members is considerably lower than casual riders, which seems to indicate that members use Cyclistic bikes for different purposes comparing to casual riders

* The members are probably locals who use the bikes daily  to commute to and from their workplaces, while casual customers are most likely tourists and use the bikes for leisure and sightseeing around Chicago.

* Cyclistic can try to build a campaign to turn casual customers to members, especially during spring and summer but it will most likely be difficult, since they have such diferent purposes.

* The amount of casual rides (more then 1.7 million) is still considerable, and we might have some oportunities to convert them to members


# **Suggestions and recommendations**
***

**Change pricing plans** :
* Cyclistic doesn't have a pricing limit per ride duration. they might be losing new annual memberships because of the full-day passes. 
* Maybe creating a weekend membership can allow casual riders to became members to allow Cyclistic getting more value from them

**Membership Referral campaign**
* To increase the membership base the company might consider a compaign giving additional benefits or discounts to the existing members with a referral program. 
* Specially during spring and summer.

# **Presentation**
***

[Click here](https://github.com/Catarinaperiquito/Google-Data-Analytics---Cyclistic-Case-Study/blob/main/The%20Cyclistic%20Case%20Study%20(1).pdf)
