# Motor-Vehicle-Thefts-Analysis 🚗🔍
## 🔍 Project Overview
This project analyzes stolen vehicle data using MySQL and Power BI to uncover trends, patterns, and insights. The analysis explores vehicle characteristics, regional variations, and potential factors influencing theft rates.

Problem Statement

Motor vehicle theft remains a persistent issue in New Zealand, affecting not only individuals, but also insurance companies and law enforcement agencies. Understanding the patterns and trends associated with stolen vehicles is crucial for developing preventive measures and improving recovery efforts.T his project aims to uncover patterns in stolen vehicle data, analyze regional theft trends, and identify the most and least targeted vehicle types. By leveraging MySQL for data analysis and Power BI for visualization, I hope to generate actionable insights that help authorities and vehicle owners mitigate theft risks.  

Project Objectives
1.  Identify trends in stolen vehicle data – Analyze the most stolen vehicle types, colors, and model years.  

2.  Determine regional variations in theft – Understand how theft rates vary by location, population, and density.  

3.  Assess the age of stolen vehicles – Investigate whether newer or older models are more frequently stolen.  

4.  Provide data-driven recommendations – Suggest strategies for reducing vehicle theft based on key findings.

     
📂 Dataset Description  

The dataset consists of multiple tables, including:

stolen_vehicles – Details of stolen vehicles (type, model year, color, date stolen).  

locations – Region, country, population, and density details.  

make_details -  Detail description on the make of the various vehicles (make_id, make_name, make_type)   

## Data Source 🗄️  

This dataset was downloaded from Maven Analytics Playground [https://app.mavenanalytics.io/datasets] 


### My SQL Codes   
[
[Upload
USE stolen_vehicles_db;
  ## Analyze the stolen_vehicles , location and make_details table.
 SELECT * 
 FROM stolen_vehicles;
 
 SELECT * 
 FROM locations;
 
 SELECT * 
 FROM make_details;
 
 
 # What is the total number of vehicles stolen?
 
 SELECT COUNT(*) as total_vehicle_stolen
 FROM stolen_vehicles;
 
 # What is the most stolen vehicles ?
  SELECT vehicle_type, COUNT(*) vehicle_count
  FROM stolen_vehicles
  GROUP BY vehicle_type
  ORDER BY vehicle_count DESC
  LIMIT 5 ;
 
 # What modle year of cars are the most stolen ? 
 SELECT model_year,COUNT(*) model_count
 FROM stolen_vehicles
 GROUP BY model_year  
 ORDER BY model_count DESC
 LIMIT 5 ;
 
 # What color of vehicles are the most stolen?
 SELECT color, COUNT(*) color_count
 FROM stolen_vehicles
 GROUP BY color
 ORDER BY color_count DESC
 LIMIT 5;
 
 # What is the most common theft dates?
 SELECT date_stolen, COUNT(*) date_count
 FROM stolen_vehicles
 GROUP BY date_stolen
ORDER BY date_count DESC
LIMIT 5 ; 
 
 # What days of the week are vehicles most and least often stolen?
SELECT DAYNAME(date_stolen) AS day_of_week, COUNT(*) AS day_count 
FROM stolen_vehicles 
GROUP BY day_of_week 
ORDER BY count DESC;
 
 # Which month had the most vehicles stolen ?
 SELECT MONTH(date_stolen) AS month, COUNT(*) AS month_count 
FROM stolen_vehicles 
GROUP BY month 
ORDER BY month_count DESC;
 
 # Wich year had the most and least vehicles stolen?
SELECT YEAR(date_stolen) AS year, COUNT(*) AS year_count 
FROM stolen_vehicles 
GROUP BY year
ORDER BY year_count DESC ;

# How many vehicles by descriptions were stolen ?
SELECT vehicle_desc, COUNT(*) AS description_count 
FROM stolen_vehicles 
GROUP BY vehicle_desc 
ORDER BY description_count DESC 
LIMIT 5;

# What make of cars are the most stolen ?
SELECT md.make_name,COUNT(*) as make_count
FROM stolen_vehicles sv
JOIN make_details md
	ON sv.make_id = md.make_id
GROUP BY md.make_name 
ORDER BY make_count DESC
LIMIT 5;

# What is the average age of the vehicles that are stolen? Does this vary based on the vehicle type?
### using CURDATE () does the calcultaion based on the current real world date  which is 2025
-- 1. What is the average age of the vehicles that are stolen?
SELECT AVG(YEAR(CURDATE()) - model_year) as avg_model_age
FROM stolen_vehicles
WHERE model_year IS NOT NULL;
  -- or (used 2022 becuz it is the max model year according to the dataset)
SELECT AVG(2022 - model_year) AS avg_model_age
FROM stolen_vehicles
WHERE model_year IS NOT NULL;

-- Q2. Does this vary based on the vehicle type?

SELECT vehicle_type, 
       AVG(2022 - model_year) AS avg_vehicletype_age
FROM stolen_vehicles
WHERE model_year IS NOT NULL
GROUP BY vehicle_type
ORDER BY avg_vehicletype_age DESC;

 # or 
SELECT vehicle_type, 
       AVG(YEAR(CURDATE()) - model_year) AS avg_vehicletype_age
FROM stolen_vehicles
WHERE model_year IS NOT NULL
GROUP BY vehicle_type
ORDER BY avg_vehicletype_age DESC;
 
 
 # Q3 What types of vehicles are most often and least often stolen? Does this vary by region? 
 
  SELECT vehicle_type, COUNT(*) vehicle_count
  FROM stolen_vehicles
  GROUP BY vehicle_type
  ORDER BY vehicle_count DESC
  LIMIT 5 ;

# Q2 Does it varies by region ? 
SELECT lr.region, vehicle_type, COUNT(*) as stolen_count
FROM stolen_vehicles as sv
JOIN locations as lr
 on sv.location_id  = lr.location_id
GROUP BY lr.region, sv.vehicle_type
ORDER BY lr.region, stolen_count DESC;

SELECT lr.region, vehicle_type, COUNT(*) as stolen_count
FROM stolen_vehicles as sv
JOIN locations as lr
 on sv.location_id  = lr.location_id
GROUP BY lr.region, sv.vehicle_type
ORDER BY lr.region, stolen_count DESC;

WITH RankedVehicles AS (
    SELECT lr.region, sv.vehicle_type, COUNT(*) AS stolen_count,
           RANK() OVER (PARTITION BY lr.region ORDER BY COUNT(*) DESC) AS most_rank,
           RANK() OVER (PARTITION BY lr.region ORDER BY COUNT(*) ASC) AS least_rank
    FROM stolen_vehicles sv
    JOIN locations lr ON sv.location_id = lr.location_id
    GROUP BY lr.region, sv.vehicle_type
)
SELECT region, vehicle_type, stolen_count, 'Most Stolen' AS category
FROM RankedVehicles WHERE most_rank = 1
UNION ALL
SELECT region, vehicle_type, stolen_count, 'Least Stolen' AS category
FROM RankedVehicles WHERE least_rank = 1;

# Which regions have the most and least number of stolen vehicles? What are the characteristics of the regions?
# Q1. 
(SELECT lr.region, vehicle_type, COUNT(*) as stolen_count
FROM stolen_vehicles as sv
JOIN locations as lr
 on sv.location_id  = lr.location_id
GROUP BY lr.region, sv.vehicle_type
ORDER BY lr.region, stolen_count DESC
LIMIT 1
)
	 UNION ALL
 (  SELECT lr.region, vehicle_type, COUNT(*) as stolen_count
FROM stolen_vehicles as sv
JOIN locations as lr
 on sv.location_id  = lr.location_id
GROUP BY lr.region, sv.vehicle_type
ORDER BY lr.region, stolen_count ASC
LIMIT 1
);  


# Q2  What are the characteristics of the regions?
SELECT lr.region, lr.country, lr.population, lr.density, COUNT(sv.vehicle_id) AS stolen_count
FROM stolen_vehicles sv
JOIN locations lr 
	ON sv.location_id = lr.location_id
GROUP BY lr.region, lr.country, lr.population, lr.density
ORDER BY stolen_count DESC
LIMIT 5;

# Q3 Find the relation between population density and theft in the various regions ?
SELECT lr.region, lr.density, COUNT(sv.vehicle_id) AS stolen_count
FROM stolen_vehicles sv
JOIN locations lr 
	ON sv.location_id = lr.location_id
GROUP BY lr.region, lr.density
ORDER BY lr.density DESC
LIMIT 5;
ing stolenvehicles-dataset.sql…]()


## 📊 Key Analyses & SQL Queries

Recommended Analysis  

1. What day of the week are vehicles most and least often stolen?
   
   (![Day](https://github.com/user-attachments/assets/d0a2c144-5651-43db-82c5-c34eab82d587)

  2. Which year had the most and least vehicles stolen?
     
 ( ![year](https://github.com/user-attachments/assets/e726f58d-3d95-429c-8785-d248502e4f09)


3. a What types of vehicles are most and least often stolen?   

  (![TypeMostStolen 1](https://github.com/user-attachments/assets/a4aaf613-59cf-4ec2-a841-a1c2dde99d91)

   
3.b  Does this vary by region?          
    (![Type 2](https://github.com/user-attachments/assets/4f43d71a-6947-4dd9-9055-dd311beb84f1)


4.i What is the average age of the vehicles that are stolen? 
    
   (![Avg 2](https://github.com/user-attachments/assets/6bfa1235-16e9-4bd5-851f-75ae603a9829)

4. ii  Does this vary based on the vehicle type?


    (![Avg 1](https://github.com/user-attachments/assets/2f721c04-c5bb-4124-8b91-179637642523) 
   

5.a Which regions have the most and least number of stolen vehicles?    

(![Region 1](https://github.com/user-attachments/assets/b1b4273f-8cd7-4050-9a11-e86792287dd9)  

5.b  What are the characteristics of the regions?    

(![Region2](https://github.com/user-attachments/assets/e81ebfc0-9141-42e9-8b36-668e33a79c23)

   
6.  What is the total number of vehicles stolen?
   
   (![Total no of cars](https://github.com/user-attachments/assets/ca0a0eac-4c9a-4ffe-89a7-42ce3f7ff24d)

7. What model year of cars are the most stolen?
   (![model stolen](https://github.com/user-attachments/assets/0e955937-ded8-4c4e-b751-68d028c154ea)


   
   
