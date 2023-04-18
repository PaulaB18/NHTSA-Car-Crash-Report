# NHTSA-Car-Crash-Report
According to NHTSA, the United States experienced 10 billion crashes in 2021, resulting in 38,000 fatalities involving 55,000 vehicles, with over 9,000 of those fatalities involving drunk drivers.

To enhance road safety, NHTSA conducted an extensive analysis to determine the factors contributing to most accidents, particularly external factors such as weather conditions, light conditions, intersection types, and more.

This report provides an in-depth analysis of each factor, helping NHTSA identify the most effective strategies to improve road safety.

I will provide a step-by-step explanation of the analysis process:

First, using SQL to clean up the data, here is the SQL syntax


```
--Cleaning the table 
--Timezone conversion
--Excluding the data recorded in 2022

create table clean_crash as
select * from
(
select 	*,
		case
			when state_name in ('Alaska')	
				then timestamp_of_crash at time zone 'akst'
			when state_name in ('Hawaii')	
				then timestamp_of_crash at time zone 'hst'
			when state_name in ('California', 'Nevada', 'Oregon', 'Washington')	
				then timestamp_of_crash at time zone 'pst'	
			when state_name in ('Arizona', 'Colorado', 'Idaho', 'Montana', 'New Mexico', 'Utah', 'Wyoming')	
				then timestamp_of_crash at time zone 'mst'
			when state_name in ('Alabama', 'Arkansas', 'Illinois', 'Iowa', 'Kansas', 'Louisiana', 'Minnesota', 'Mississippi', 'Missouri', 'Nebraska', 'North Dakota', 'Oklahoma', 'South Dakota', 'Texas', 'Wisconsin')	
				then timestamp_of_crash at time zone 'cst'
			when state_name in ('Connecticut', 'Delaware', 'District of Columbia', 'Florida', 'Georgia', 'Indiana', 'Kentucky', 'Maine', 'Maryland', 'Massachusetts', 'Michigan', 'New Hampshire', 'New Jersey', 'New York', 'North Carolina', 'Ohio', 'Pennsylvania', 'Rhode Island', 'South Carolina', 'Tennessee', 'Vermont', 'Virginia', 'West Virginia') 
				then timestamp_of_crash at time zone 'est'
			else now()
			end timestamp_of_crash_conversion_per_state
from crash
) x
where timestamp_of_crash_conversion_per_state between '2021-01-01 00:00:00+07' and '2021-12-31 23:59:59+07';	

update clean_crash
set city_name='NOT REPORTED'
where city_name='Not Reported';

```
Next, start to find the factors that caused the accidents during the year of 2021. There are several factors that we analyze as follow:

Analyze the States with the highest accident rates using the below SQL syntax (for the effectiveness, we only limit to the top 10):

```
select rank () over (order by percentage DESC), x.state_name, x.total_accident, x.percentage
from 
(
select state_name, count(consecutive_number) total_accident, 
count(consecutive_number)/sum(count(consecutive_number)) over()*100 percentage
from clean_crash
where functional_system_name not in ('Unknown', 'Not Reported', 'Trafficway Not in State Inventory')
group by state_name
) x
order by total_accident desc
limit 10;
```
Next, we want to find out the hours when the accidents happened most with the below SQL syntax:

```
SELECT* FROM clean_crash;

CREATE TABLE hourly_crash AS SELECT 
cast(timestamp_of_crash AS DATE) AS crash_date, 
sum(number_of_vehicle_forms_submitted_all) AS daily_crash_number,
sum(number_of_vehicle_forms_submitted_all)/24 AS crash_per_hour
FROM crash
GROUP BY crash_date
ORDER BY crash_date ASC;

select* from hourly_crash;

SELECT AVG(crash_per_hour) AS avg_crash_hr FROM hourly_crash;
```
We want to find out the percentage of accidents caused by drunk drivers with the following syntax:

```
select	case 
			when number_of_drunk_drivers > 0 
				then 'Yes' 
		else 'No' 
		end drunk_driver,
		count(1) as jumlah_kecelakaan,
		round(((count(1)/sum(count(1)) over()) * 100), 1) as prosentase_kecelakaan
from clean_crash
where functional_system_name not in ('Unknown', 'Not Reported', 'Trafficway Not in State Inventory')
group by 1


```
We assume that the accidents happened more in the city compared to the village, so we used the below syntax to find out:

```
select* from clean_crash;

select
land_use_name,
SUM(number_of_vehicle_forms_submitted_all) AS total_case
FROM clean_crash
GROUP BY land_use_name
ORDER BY total_case DESC;
```
Days might be play an important role in the accidents, we assumed there are days where the accidents happened more compared to other days of the week, so we use the following syntax to find out:

```
SELECT* FROM clean_crash;

SELECT 
cast(timestamp_of_crash AS DATE) AS crash_date, 
TO_CHAR(timestamp_of_crash, 'Day') AS crash_day,
number_of_vehicle_forms_submitted_all
FROM clean_crash;

SELECT 
TO_CHAR(timestamp_of_crash, 'Day') AS crash_day,
SUM(number_of_vehicle_forms_submitted_all) AS total_crash
FROM clean_crash
group by crash_day
order by total_crash desc;
```

We also want to analyze how the weather and the atmospheric conditions affected the safety on the roads using the following syntax:

```
SELECT* FROM clean_crash;

SELECT atmospheric_conditions_1_name, COUNT(16) AS Risk_4 FROM clean_crash Group by Atmospheric_conditions_1_name ORDER BY Risk_4 DESC;
```

```
SELECT* FROM clean_crash;

SELECT light_condition_name, COUNT(15) AS Risk_3 FROM clean_crash Group by light_condition_name ORDER BY Risk_3 DESC;
```
Next, we hypothesize that the type of intersection and functional system roads may play a significant role in accidents. Therefore, we employ the following syntax to retrieve data on which types of intersections the accidents mostly occurred:

```
SELECT* FROM clean_crash;

SELECT type_of_intersection_name, COUNT(13) AS Risk_2 FROM clean_crash Group by type_of_intersection_name ORDER BY Risk_2 DESC;
```

```
SELECT functional_system_name, COUNT(11) AS Risk_1 FROM clean_crash Group by functional_system_name ORDER BY Risk_1 DESC;

```

## My Findings

#### Traffic Volume

The volume of traffic on roads is a significant contributing factor to the high incidence of car crashes. Available data indicates that accidents tend to occur more frequently during the daytime, when the number of vehicles on the road is at its peak. This is likely due to the increased activity of people during the day, which amplifies the likelihood of accidents. Additionally, the data shows that weekend record more crashes, especially Sundays record the highest number of car crashes. Since Sundays are usually considered a holiday, people tend to travel more frequently during the day, leading to heightened road activity. Unlike weekdays when traffic is primarily concentrated around commuting hours, Sundays tend to be bustling with activity throughout the day, making it a prime time for accidents to occur. The accidents also occur more in the states with more population.

#### Human Error

Based on our analysis of the data, we believe that human error is the second leading cause of accidents. This conclusion is supported by several key findings. Firstly, the majority of accidents occur during daylight hours and when the sky is clear that is typically considered to be safe driving conditions. This suggests that drivers may be less vigilant when they perceive conditions to be less risky. Conversely, accidents are less frequent during dusk, rain, and freezing conditions, as drivers tend to be more cautious during these times.

Additionally, our analysis indicates that traffic volume is a contributing factor to accidents. When there are more vehicles on the road, the risk of human error causing an accident increases. Even a single instance of human error can have a significant impact on road safety, and the risk of accidents escalates when multiple drivers make errors simultaneously.

However, our data only accounts for accidents caused by drunk drivers, which represents just one type of human error. It's important to note that other factors, such as distracted or drowsy driving, phone behind the wheel, being sick while driving may also contribute to accidents. Future research that accounts for these factors may provide a more comprehensive understanding of the role of human error in accidents.

#### Speed

One possible contributing factor to the occurrence of crashes is the speed of the vehicles involved. While we don't have direct data on the speed of the crashes, we can infer that speed may have played a role based on the available information. The data indicates that crashes are most common on interstates, where the speed limit allows for driving at up to 70mph, compared to other roads where the speed limit is between 50 to 60mph.  Local roads, where the speed limit is typically around 50mph, ranked second least in crash incidence. This may suggest that lower speed limits on local roads may help reduce the likelihood of crashes. Similarly, residential areas, where the speed limit is even lower, between 20 to 40mph, also experienced relatively fewer crashes, indicating that lower speed limits may be an effective means of reducing crash rates.


## RECOMMENDATION

To improve the safety, there are a few recommendations that can be implemented in the future:

* Increase police patrols on the roads, particularly during peak hours, days, and seasons, such as Christmas and New Year, when more people are traveling. This can help deter dangerous driving behavior and enforce traffic laws, reducing the likelihood of accidents.

* Increase the number of CCTV cameras on the roads to detect and identify drivers who break the law and put others at risk. This can help police arrest those drivers and protect others on the road.

* Increase fines and consider other punishments for those who break the law. This can serve as a deterrent and discourage reckless driving behavior.

* Launch a safety campaign that promotes responsible driving practices, such as avoiding phone use while behind the wheel, not driving when feeling drowsy or unwell, and following traffic rules and regulations.

* Introduce a policy that requires routine medical check-ups verified by police to ensure drivers are both physically and mentally fit to drive. This can help identify any medical conditions that could affect a driver's ability to operate a vehicle safely and reduce the risk of accidents caused by 
