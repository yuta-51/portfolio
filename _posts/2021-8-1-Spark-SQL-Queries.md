---
layout: post
title:  "Spark SQL Queries"
info: "A collection of Spark SQL queries written for an Apache Spark course."
tech: "Spark SQL"
type: Coursework
thumbnail: https://upload.wikimedia.org/wikipedia/commons/thumb/f/f3/Apache_Spark_logo.svg/1200px-Apache_Spark_logo.svg.png
---

# Goal
To showcase some SQL queries written as coursework in the DataBricks Apache Spark SQL for Data Analysts course on Coursera.


## Filtering Array Data 
This problem deals with a mock data set from a group of data centers collecting sensor data (co2 levels, temperature, battery level). The co2 levels are captured as arrays, as measurements are taken multiple times a day but data is sent daily. This query shows all decvice IDs which have a CO2 level higher than 1400 in any of their readings using the FILTER function.

```sql
SELECT 
deviceId, highCO2
FROM (
  SELECT deviceId, FILTER(co2Level, i -> i > 1400) highCO2
  FROM DCDevices
) 
WHERE SIZE(highCO2) > 0
ORDER BY highCO2
```


## Using RollUp to Generate Subtotals
Computes averages of income grouped by itemName and month such that the results include averages across all months as well as a subtotal for an individual month from the sales table.

```sql
CREATE OR REPLACE TEMPORARY VIEW q6Results AS
  SELECT 
    COALESCE(itemName, "All items") AS itemName,
    COALESCE(month(date), "All months") AS month,
    ROUND(AVG(revenue), 2) as avgRevenue
  FROM sales
  GROUP BY ROLLUP (itemName, month(date))
  ORDER BY itemName, month;

SELECT * FROM q6Results;
```

## Creating a Table as Select with Subquery
The table ```purchaseEvents``` was created based on another table ```eventsRaw``` which contains mock data meant to replicate data from an ecommerce mattress seller. The raw data contains data of all sorts including structs and arrays. Here, we are creating a new table which includes event data with purchases.

```sql
DROP TABLE IF EXISTS purchaseEvents;

CREATE TABLE purchaseEvents
WITH explodeEventsRaw AS
(
   SELECT 
   ecommerce.purchase_revenue_in_usd AS purchase_usd,
   geo.city AS city,
   geo.state AS state, 
   CAST(CAST((event_timestamp / 1000000) AS TIMESTAMP) AS DATE) AS eventTimeStamp,
   CAST(CAST((event_previous_timestamp / 1000000) AS TIMESTAMP) AS DATE) AS eventPreviousTimeStamp,
   user_id
   FROM eventsRaw
   WHERE ecommerce.purchase_revenue_in_usd IS NOT NULL
)
SELECT 
  purchase_usd AS purchases,
  eventTimeStamp AS eventDate,
  eventPreviousTimeStamp AS previousEventDate,     
  city,    
  state,     
  user_id AS userId

FROM explodeEventsRaw
```


