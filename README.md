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
