# Cyclistic-Bike-Share-Analysis
## 1st Case Study of Google Data Analytics Professional Certificate
#### Austin Sebastian Marviano - 18/February/2022

### :clapper: Scenario
You are a junior data analyst working in the marketing analyst team at Cyclistic, a bike-share company in Chicago. The director
of marketing believes the company’s future success depends on maximizing the number of annual memberships. Therefore,
your team wants to understand how casual riders and annual members use Cyclistic bikes differently. From these insights,
your team will design a new marketing strategy to convert casual riders into annual members. But first, Cyclistic executives
must approve your recommendations, so they must be backed up with compelling data insights and professional data
visualizations.

### :question: Business Task
1. The goals to strategize converting casual riders into annual members
2. Understand how annual members and casual riders differ
3. Analyzing the Cyclistic historical bike trip data to identify trends
4. Design a new marketing strategy to convert casual riders into annual members

### :scroll: Library Used
```
install.packages("tidyverse")
install.packages("dplyr")
install.packages("lubridate")
install.packages("ggplot2")

library(tidyverse)
library(lubridate)
library(ggplot2)
library(dplyr)
```

### :information_source: Data Source
[Link to Datasets](https://divvy-tripdata.s3.amazonaws.com/index.html)<br />
Note: The datasets have a different name because Cyclistic is a fictional company. For the purposes of this case study,
the datasets are appropriate and will enable you to answer the business questions. The data has been made available by
Motivate International Inc. under this license.

Data used : 
- 2021, Month of 01-05 & 11-12
- 2020, Month of 06-10

Changes made :
1. Mixing the 2020 and 2021 due to too many invalid data on 2021 month of 06-10, at the "started at" & "ended at" column's rows filled with "######". 
2. Cleaning and Aggregate all of each month files into one file named "MasterTrip_1yr-mix_2020-2021.csv"
3. Added "ride_length" & "day_of_week" columns using "Google Spreadsheet'




```
# Storing data to data frame
master_trip<- read.csv("MasterTrip_1yr-mix_2020-2021.csv")


# Checking the columns and structure
colnames(master_trip)
str(master_trip)

# Making sure the categories & members are unique
ride_ctg <- unique(master_trip$rideable_type)
member_cat <- unique(master_trip$member_casual)
ride_ctg
member_cat
table(master_trip$rideable_type)
table(master_trip$member_casual)
```

### Adding date, day, month, year, day of week column
```
master_trip$date <- as.Date(master_trip$started_at) 
master_trip$day <- format(as.Date(master_trip$started_at), "%d") 
master_trip$month <- format(as.Date(master_trip$started_at), "%m")
master_trip$year <- format(as.Date(master_trip$started_at), "%Y") 
master_trip$day_of_week <- format(as.Date(master_trip$started_at), "%A") 
```

### Adding ride length in minutes column
```
master_trip$ride_length_mins <- difftime(master_trip$ended_at, master_trip$started_at, units = "mins")
master_trip$ride_length_mins <- as.numeric(as.character(master_trip$ride_length_mins))
```

### Bad Data Removal
Keeping the old data frame and creating a new data frame filled with cleaned data
```
master_trip_2 <- master_trip[!(master_trip$ride_length<=0),]
sum(master_trip$ride_length<=0)
sum(master_trip_2$ride_length<=0)
sum(master_trip$ride_length>=1440)
master_trip_2 <- master_trip_2[!(master_trip_2$ride_length>=1440),]
sum(master_trip_2$ride_length>=1440)
```
Summary : 
- <= 0 mins ride_length removed, 1
- more than 1 day ride_length removed, 2

###  Dataframe columns of master_trip_2, for visualization
```
[1] "ride_id"            "rideable_type"      "started_at"         "ended_at"           "start_station_name"
[6] "start_station_id"   "end_station_name"   "end_station_id"     "start_lat"          "start_lng"         
[11] "end_lat"            "end_lng"            "member_casual"      "ride_length"        "day_of_week"       
[16] "X"                  "X.1"                "X.2"                "date"               "day"               
[21] "month"              "year"               "ride_length_mins"  
```

### Descriptive Stastistics, all members type are included
```
mean(master_trip_2$ride_length_mins) 
median(master_trip_2$ride_length_mins) 
min(master_trip_2$ride_length_mins) 
max(master_trip_2$ride_length_mins) 
```
Result : 
1. Mean : 13.82
2. Median : 9.83
3. Min : 0.13
4. Max : 74.61

### Proportion of Casual Member
```
master_trip_2%>%
group_by(member_casual) %>%
summarise(number_of_users = n()) %>%
ggplot(aes(x="", y=number_of_users, fill=member_casual)) +
geom_col(color = "black") +
geom_text(aes(label = number_of_users), position = position_stack(vjust = 0.5),
            show.legend = FALSE) +
coord_polar(theta = "y")+
labs(title = "Proportion of members type")
```


### Types of Bikes Proportion
```
master_trip_2%>%
group_by(rideable_type) %>%
summarise(number_of_bikes = n()) %>%
ggplot(aes(x="", y=number_of_bikes, fill=rideable_type)) +
geom_col(color = "black") +
geom_text(aes(label = number_of_bikes), position = position_stack(vjust = 0.5),
show.legend = FALSE) +
coord_polar(theta = "y") +
theme_void() + 
labs(title = "Proportion of Three Different Types of Bikes")
```


### Comparison of each member type ride length
```
# Mean - Casual : 18.33 | Member : 12.50
aggregate(master_trip_2$ride_length_mins~ master_trip_2$member_casual, FUN = mean)

# Median - Casual : 14.41 | Member : 8.68
aggregate(master_trip_2$ride_length_mins~ master_trip_2$member_casual, FUN = median)

#Min - Casual : 0.71 | Member : 0.13
aggregate(master_trip_2$ride_length_mins~ master_trip_2$member_casual, FUN = min)

#Max - Casual : 74.611 | Member : 62.81
aggregate(master_trip_2$ride_length_mins~ master_trip_2$member_casual, FUN = max)
```

### Converting day_of_week from number to day
```
master_trip_2$day_of_week <- ordered(master_trip_2$day_of_week, levels 
= c("Sunday","Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"))
```

### Average ride length in minutes each day
```
aggregate(master_trip_2$ride_length_mins ~ master_trip_2$member_casual + master_trip_2$day_of_week, FUN =mean)
```
```
# Result
1                       casual                    Friday                      19.412745
2                       member                    Friday                      13.733838
3                       casual                    Monday                      18.800758
4                       member                    Monday                       7.629667
5                       casual                  Saturday                      22.405556
6                       member                  Saturday                      13.836792
7                       casual                    Sunday                      16.521053
8                       member                    Sunday                      14.407778
9                       casual                  Thursday                      13.607778
10                      member                  Thursday                      12.706011
11                      casual                   Tuesday                      22.141667
12                      member                   Tuesday                      11.570667
13                      casual                 Wednesday                      14.684848
14                      member                 Wednesday                      13.159016
```

### Average duration & number of ride each member type
master_trip_2 %>%
mutate(weekday = wday(started_at, label = TRUE)) %>%
group_by(member_casual, weekday) %>%
summarise(number_of_rides = n(), average_duration = mean(ride_length_mins)) %>%
arrange(member_casual, weekday)

### Viz - Each day average ride length in minutes
master_trip_2 %>%
mutate(weekday = wday(started_at, label = TRUE)) %>%
group_by(member_casual, weekday) %>%
summarise(number_of_rides = n(), average_duration = mean(ride_length_mins)) %>%
arrange(member_casual, weekday) %>%
ggplot(aes(x = weekday, y = average_duration, fill = member_casual)) +
geom_col(position = "dodge") +
ggtitle("Average Duration based on Rider Type")

{r weekday total rides viz}
master_trip_2 %>%
mutate(weekday = wday(started_at, label = TRUE)) %>%
group_by(member_casual, weekday) %>%
summarise(number_of_rides = n(), average_duration = mean(ride_length_mins)) %>%
arrange(member_casual, weekday) %>%
ggplot(aes(x = weekday, y = number_of_rides, fill = member_casual)) +
geom_col(position = "dodge") +
ggtitle("Number of Bikes based on Rider Type")


{r bike_type_user}
master_trip_2 %>%
mutate(master_trip_2$rideable_type, label = TRUE) %>%
group_by(rideable_type, member_casual) %>%
arrange(member_casual, rideable_type) %>%
summarise(number_of_rides = n()) %>%
ggplot(aes(x = rideable_type, y = number_of_rides, fill = member_casual)) +
geom_col(position = "dodge") +
ggtitle("Different Bike Types Usage")


{r generated_summary_report}
counts <- aggregate(master_trip_2$ride_length_mins ~ master_trip_2$member_casual + 
                      master_trip_2$day_of_week, FUN = mean)


{r exported_saved_csv}
write.csv(counts, file ="/Users/Marviano/Desktop/avg_ride_length.csv")

# FINDINGS
* Casual users occupied almost 40% of the total users population. 
* Docked bike was the most popular one (over 80%) among three bike types.
* Casual users mostly use the bike-share service on the weekends, but less often during the weekdays. 
  Number of rides from casual users on the weekdays was only half of number on the weekends. 
* Average duration from casual users were twice as much as annual members' over the whole weeks.

# CONCULSIONS / SUGGESTIONS
* Provide special membership offers (e.g lower price) for using docked bikes in order to convert huge casual docked bike users into annual members. 
* Offer "weekdays membership packages" to those who use Cyclistic bikes on weekdays. 
* Set a duration milestone and create cash-back or credit-bonus membership program to convert casual users into members because 
  they have a longer average duration than current members. 

# LIMITATION / FUTURE IMPROVEMENT
* There were 11126 rows that ride duration was equal to or less than 0 minutes, and 2863 rows that ride duration was greater than one day. 
  All of these were removed and we should figure out: 
  - whether technical errors have occurred in the system (e.g. negative ride duration records)
  - and the possibility of users rent Cyclistic over a day (e.g ride duration over 1440 minutes.)    
* There were huge number of rows that the station names were missing while the latitude and longitude information recorded. 
  I didn't remove or drop these rows, but I think the missing station name could be fixed by comparing latitude and 
  longitude information from other rows.
