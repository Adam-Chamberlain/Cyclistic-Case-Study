# Cyclistic Case Study

## Introduction
In this case study, **Cyclistic** is a Chicago-based bike-share company. Within Cyclistic, there are two different types of consumers: **members**, who purchase annual memberships, and **casual riders**, who typically purchase a one-time bike pass. The director of marketing believed that selling as many annual memberships as possible is the key to maximizing profits and ensuring long-term success, so they wanted to figure out how to effectively convert casual riders into annual members.

In order to accomplish this goal, Cyclistic's marketing team wanted to create a new marketing strategy specifically aimed at casual riders. They identified three key questions to focus on:
- **How do annual members and casual riders use Cyclistic bikes differently?**
- Why would casual riders buy Cyclistic annual memberships?
- How can Cyclistic use digital media to influence casual riders to become members?

I was specifically instructed to analyze the first question: "How do annual members and casual riders use Cyclistic bikes differently?" Therefore, this is the main focus of my research, but it was also important to analyze the other two questions in order to understand the bigger picture. I used data from the past 12 months to analyze how these two groups of Cyclistic users differ.

### Business Task: Identify how Cyclistic's casual riders and annual members use bikes differently in order to create an effective marketing strategy that can convert casual riders into annual members to drive future growth.

## Data Preparation

**Sources Used:**
- Ride Data: https://divvy-tripdata.s3.amazonaws.com/index.html (Data made available by Motivate International Inc. under this license: https://www.divvybikes.com/data-license-agreement)
- Chicago Weather History: https://www.wunderground.com/history/daily/us/il/chicago/KMDW

I began by pulling data from the past 12 months (June 2023 - May 2024). This included the following CSV files:
```
- 202306_divvy-tripdata.csv
- 202307_divvy-tripdata.csv
- 202308_divvy-tripdata.csv
- 202309_divvy-tripdata.csv
- 202310_divvy-tripdata.csv
- 202311_divvy-tripdata.csv
- 202312_divvy-tripdata.csv
- 202401_divvy-tripdata.csv
- 202402_divvy-tripdata.csv
- 202403_divvy-tripdata.csv
- 202404_divvy-tripdata.csv
- 202405_divvy-tripdata.csv
```

Each file contained hundreds of thousands of rows, each containing accurate and unbiased information about a ride that took place. Each record included the following fields:
```
Row Name                Description
- ride_id               Unique Ride ID
- rideable_type         Type of Bike (Classic or Electric Bike)
- started_at            Exact Start Date & Time (YYYY-MM-DD HH:MM:SS)
- ended_at              Exact End Date & Time (YYYY-MM-DD HH:MM:SS)
- start_station_name    Name of Station that the bike started at
- start_station_id      Station ID
- end_station_name      Name of Station that the bike ended at
- end_station_id        Station ID
- start_lat             Start Location Latitude
- start_lng             Start Location Longitude
- end_lat               End Location Latitude
- end_lng               End Location Longitude
- member_casual         Rider Type (Member or Casual)
```
To start off, I opened each CSV file in Microsoft Excel and made some modifications in order to clean the data and discover more relevant information. I added the following rows:

- **ride_length**: To get the exact length of each ride, I subtracted started_at from ended_at. This showed me the difference in ride length between casual riders and annual members. Formatted as HH:MM:SS.
- **month**: I used the formula =MONTH(started_at) to get the month number of which the ride started. This was important to discover trends between months.
- **day**: I used the formula =DAY(started_at) to get the day number of which the ride started. This let me see how rides per day changed on a daily basis throughout a given month.
- **day_of_week**: Using the formula =WEEKDAY(started_at), I pulled the weekday of which each ride started on. (1 = Sunday, 2 = Monday, etc) This showed me the specific days of the week different types of users preferred to ride bikes.

I then had to make a few modifications to ensure the data was accurate and ready to be analyzed. I first noticed that in some rare cases, the **started_at** time took place *after* the **ended_at** time. I used conditional formatting to highlight these instances and then swapped the start and end date. I also converted the numbers in **day_of_week** using Find and Replace to make them the actual name of the date.

With those changes in place, this is how all of the files looked:
![image](https://github.com/Adam-Chamberlain/Cyclistic-Case-Study/assets/173857433/bfde9ef1-007a-4498-8750-91045f3c59a6)
I saved them as CSV files and began uploading them into Google Cloud Storage so that I could then merge them into one large database in Google BigQuery.

## Data Analysis

Before analysing the full dataset with BigQuery, I looked at data from the most recent month (May 2024) in Excel to find important trends that I could further explore using SQL. I analyzed the data using Pivot Tables and charts, and I quickly found some key differences between annual members and casual riders.
![image](https://github.com/Adam-Chamberlain/Cyclistic-Case-Study/assets/173857433/44a3085b-1572-4c45-b3de-69f94096174b)
I first looked at the number of rides that were taken on a daily basis. With this chart, I noticed three key things:
- Casual riders almost always had less daily rides than annual members
- Casual riders rode much more frequently on weekends
- There were two major drops in rides that did not have much explanation (5/9 and 5/26)

- Casual riders riding more on weekends showed that they are likely using the bikes mainly for recreational use. At the same time, members typically rode the most in the middle of the work week, which meant they likely used bikes to commute to work or school. I later looked into further trends in SQL to see if these theories were accurate.

As for the two days with major drops in both rider categories, I looked to see if the weather had anything to do with it. Sure enough, May 9 had rain in the morning as well as temperatures below 60°F, and May 26 had even more rain in the early afternoon.

#### Total Monthly Rides
At this point, I combined all of the monthly data files into one large database in BigQuery. With over 5.7 million total rows, SQL made this job much more efficient, as Excel would never be able to handle that many rows. Starting off, I wanted to get a better idea of how the number of monthly rides changed between months. I ran the following SQL query to pull this data:
```
SELECT 
month,
SUM(CASE WHEN member_casual='member' THEN 1 ELSE 0 END) AS members,
SUM(CASE WHEN member_casual='casual' THEN 1 ELSE 0 END) AS casuals,
COUNT(member_casual) as total
FROM `hallowed-pager-423020-s0.bike_data.bike_data`
GROUP BY month
ORDER BY month
```
This showed me the total amount of rides that happened each month from annual members, casual riders, the two groups combined per month. This allowed me to create the following charts:
![image](https://github.com/Adam-Chamberlain/Cyclistic-Case-Study/assets/173857433/693b36ed-391d-4d8e-ae38-4e4cbf7d11b8)

Overall, there were a major decrease in rides during winter months, as expected in Chicago, where it gets very cold and snows a lot during the winter. Members also consistently take about 100,000 more rides than casual riders every single month.
![image](https://github.com/Adam-Chamberlain/Cyclistic-Case-Study/assets/173857433/7385edc2-35e7-4906-9bde-dfeceddb14f8)

However, percentage of casual riders decreased significantly in winter months. This further added to the theory that casual riders use bikes for recreation, and annual members ride bikes daily for work commutes.

#### Rides per Weekday
I then looked at rides per weekday by using a similar SQL query:
```
SELECT 
DATE(started_at) AS dates,
SUM(CASE WHEN member_casual='member' THEN 1 ELSE 0 END) AS members,
SUM(CASE WHEN member_casual='casual' THEN 1 ELSE 0 END) AS casuals,
COUNT(DATE(started_at)) AS amount_per_day
 FROM `hallowed-pager-423020-s0.bike_data.bike_data`
GROUP BY dates
ORDER BY dates
```
![image](https://github.com/Adam-Chamberlain/Cyclistic-Case-Study/assets/173857433/8fb53a78-22c1-4f2d-ae22-578378a78ed0)

Members ride bikes much more frequently throughout weekdays, which is expected if they are using bikes to commute to work, and there is a massive increase in casual rides throughout the weekend, likely due to them using bikes for recreational purposes.
![image](https://github.com/Adam-Chamberlain/Cyclistic-Case-Study/assets/173857433/41ef0d89-c6a5-4ffb-b8bf-cb19c3f0fd44)

I also looked at the average ride length for each day and each group. Casual riders tend to ride bikes for a much longer period of time, which peaks during weekends. However, annual members ride for an extremely consistent amount of time from Monday through Friday; this showed that it is likely the same people that made the same commute every single day.

#### Rides per Hour
Finally, I looked at how the amount of rides differ throughout the day. I pulled this data using the following SQL query:
```
SELECT 
EXTRACT(HOUR FROM started_at) AS hour,
SUM(CASE WHEN member_casual='member' THEN 1 ELSE 0 END) AS members,
SUM(CASE WHEN member_casual='casual' THEN 1 ELSE 0 END) AS casuals,
COUNT(member_casual) AS total
FROM `hallowed-pager-423020-s0.bike_data.bike_data`
GROUP BY hour
ORDER BY hour
```
![image](https://github.com/Adam-Chamberlain/Cyclistic-Case-Study/assets/173857433/edc7846d-46b4-421f-a3fb-57c8f5338caa)

As expected, annual members see a huge spike in rides at 8 AM and 5 PM, which is when they are likely riding to and from work. Casual riders see a slow increase in the number of rides throughout the afternoon.
![image](https://github.com/Adam-Chamberlain/Cyclistic-Case-Study/assets/173857433/890e6444-d4b4-4a67-82b8-133e23a413a7)

I also looked at the data for Saturday only, which showed an extremely similar trend between the two rider groups. On Saturday, less people are working, so it is to be expected that annual members are not riding their bikes more frequently at 8 AM and 5 PM, unlike the prior chart.

#### Rides per Day
Lastly, I took a look at how the number of rides differed between all 366 days included in the data. I found some interesting outlier points, which I highlighted below:
![image](https://github.com/Adam-Chamberlain/Cyclistic-Case-Study/assets/173857433/b1f7dd3d-04ee-4e52-be15-61355492060f)
Most peaks were due to good weather, especially in the summer. There were also multiple peaks for annual members during weekdays in the fall, which were work days with great weather. Like the Total Monthly Rides chart, I found a major decrease in winter months, but annual members ride much more frequently than casual riders in these months.

Like the peaks, the low points are almost all due to weather, specifically rain. Nobody wants to bike in the rain, and this chart really shows it. It's also notable that bikes are ridden much less on holidays with colder weather, such as Halloween and Thanksgiving.

## Findings
Cyclistic's marketing team wanted to identify how their annual members and casual riders use bikes differently in order to understand what causes casual members to become annual members. Through analyzing this data, I was able to find these key differences:

#### Annual Members
- Used for work / school commutes: Annual members ride bikes much more frequently than casual riders, especially on work days.
- Consistent but short commutes
- Most rides occur around 8 AM and 5 PM
#### Casual Riders
- Used for recreation: Casual riders prefer to ride bikes during weekends and in the afternoon.
- Major increase in rides during summer months
- Longer rides
- Steady increase of the amount of rides throughout the early afternoon

## Next Steps
Based on my analysis, I made three key suggestions for how Cyclistic could use my findings to effectively convert casual riders to annual members:

#### 1. Advertise other ways that casual riders can use bikes
Most casual riders use bikes for recreation during weekends, but they may not know how they can be used in other ways. In downtown Chicago, bike-sharing can be a great alternative to public transit, and it's a much more eco-friendly option as well! Bikes are also a great way to get daily exercise, especially on days with sunny weather.
#### 2. Offer membership discounts to casual riders
Casual riders could be offered discounts on their first year of a membership, especially if they've ridden Cyclistic bikes many times prior. For example, if an annual membership costs $200 per year, a discount of $150 for the first year could be a great way to get new members to sign up.
#### 3. Offer monthly memberships
An annual membership is a big commitment, especially if a consumer does not want to ride during winter months. Monthly memberships can be more expensive in total, but they would be a great way to turn casual riders into members without them having to worry about a large commitment.
