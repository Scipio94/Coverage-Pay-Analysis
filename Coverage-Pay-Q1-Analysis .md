~~~ SQL
/* Coverage Pay Q1 Total*/
SELECT 
  SUM(Total_Compensation) AS Q1_Coverage_Pay
FROM `my-data-project-36654.FA_Coverage_Pay.Q1_Coverage_Pay`;
-- Coverage Pay Q1 Total $41,699.71

/* Q1 Monthly Coverage Pay Total*/
SELECT 
  DATE_TRUNC(Date_of_Coverage,month) AS Month, -- Creating Month Column
  ROUND(SUM(Total_Compensation),2) AS Q1_Monthly_Coverage_Pay_Total
FROM `my-data-project-36654.FA_Coverage_Pay.Q1_Coverage_Pay`
GROUP BY Month
ORDER BY Month;
-- September 2023 was the month with the highest coverage pay total $19,752.52
-- August 2023 was the month with the lowest coverage pay total $5,973.1

/*Q1 Campus Coverage Pay Total*/
SELECT 
  Campus, -- Creating Month Column
  ROUND(SUM(Total_Compensation),2) AS Q1_Monthly_Coverage_Pay_Total
FROM `my-data-project-36654.FA_Coverage_Pay.Q1_Coverage_Pay`
GROUP BY Campus
ORDER BY Q1_Monthly_Coverage_Pay_Total DESC ;
-- The Elementary campus had the highest campus coverage pay totals for Q1 totaling $21,465.23

/* Coverage Pay Q1 Total Individual*/
SELECT 
  Name,
  SUM(Total_Compensation) AS Q1_Coverage_Pay_Individual
FROM `my-data-project-36654.FA_Coverage_Pay.Q1_Coverage_Pay`
GROUP BY Name
ORDER BY Q1_Coverage_Pay_Individual;
-- DaNae Williams submitted had the highest indivudal Coverage Pay total in Q1 of $4,180
-- There were several staff members that had the lowest individual Coverage Pay total of $25 

/*Q1 Coverages Count Total */
SELECT 
  SUM(Number_of_Coverages) AS Q1_Coverages
FROM `my-data-project-36654.FA_Coverage_Pay.Q1_Coverage_Pay`;
-- There were 1,024 coverages submitted in Q1

/*Q1 Monthly Coverages Count Total */
SELECT 
  DATE_TRUNC(Date_of_Coverage,month) AS Month, -- Creating Month Column
  SUM(Number_of_Coverages) AS Q1_Monthly_Coverages_Total
FROM `my-data-project-36654.FA_Coverage_Pay.Q1_Coverage_Pay`
GROUP BY Month
ORDER BY Month;
-- September 2023 was the month with the greatest coverage count in Q1 with a total of 483

/*Q1 Campus Coverages Count Total */
SELECT 
  Campus,
  SUM(Number_of_Coverages) AS Q1_Campus_Coverages_Total
FROM `my-data-project-36654.FA_Coverage_Pay.Q1_Coverage_Pay`
GROUP BY Campus
ORDER BY Q1_Campus_Coverages_Total DESC;
-- The Elementary campus had the greatest number of coverages in Q1 with a total of 562

/* Q1 Coverages Count Individual Total*/
SELECT 
  Name,
  SUM(Number_of_Coverages) AS Q1_Coverages_Individual
FROM `my-data-project-36654.FA_Coverage_Pay.Q1_Coverage_Pay`
GROUP BY Name
ORDER BY Q1_Coverages_Individual;
-- DaNae Williams had the highest individual coverage count total of 145
-- There were several staff members that had the lowest individual coverage count total of 1 

/*MoM Coverage Pay Comparison*/
WITH Month_Coverage_Pay AS
(
SELECT 
  DATE_TRUNC(Date_of_Coverage,month) AS Month, -- Creating Month Column
  ROUND(SUM(Total_Compensation),2) AS Q1_Monthly_Coverage_Pay_Total
FROM `my-data-project-36654.FA_Coverage_Pay.Q1_Coverage_Pay`
GROUP BY Month
ORDER BY Month
),

Month_Coverage_Pay_Lag AS
(
SELECT 
  sub.month,
  LAG(Q1_Monthly_Coverage_Pay_Total) OVER (ORDER BY sub.month) AS Q1_Monthly_Coverage_Pay_Total_Lag
FROM
(SELECT 
  DATE_TRUNC(Date_of_Coverage,month) AS Month, -- Creating Month Column
  ROUND(SUM(Total_Compensation),2) AS Q1_Monthly_Coverage_Pay_Total
FROM `my-data-project-36654.FA_Coverage_Pay.Q1_Coverage_Pay`
GROUP BY month) AS sub -- subquery
)

SELECT 
  month,
  p.Q1_Monthly_Coverage_Pay_Total,
  COALESCE(l.Q1_Monthly_Coverage_Pay_Total_Lag,0) AS Q1_Monthly_Coverage_Pay_Total_Lag,
  COALESCE(ROUND(p.Q1_Monthly_Coverage_Pay_Total - l.Q1_Monthly_Coverage_Pay_Total_Lag,2),0) AS MoM_Coverage_Pay_Total_Change,
  COALESCE(ROUND((p.Q1_Monthly_Coverage_Pay_Total - l.Q1_Monthly_Coverage_Pay_Total_Lag)/ l.Q1_Monthly_Coverage_Pay_Total_Lag,2),0) AS MoM_Coverage_Pay_Total_Change_Pct
FROM Month_Coverage_Pay AS p
LEFT JOIN Month_Coverage_Pay_Lag AS l
USING (Month)
ORDER BY Month;

/*MoM Coverage Pay Count Comparison*/
WITH Monthly_Coverage_Count AS 
(
SELECT 
  DATE_TRUNC(Date_of_Coverage,month) AS Month, -- Creating Month Column
  SUM(Number_of_Coverages) AS Q1_Monthly_Coverages_Total
FROM `my-data-project-36654.FA_Coverage_Pay.Q1_Coverage_Pay`
GROUP BY Month
),

Monthly_Coverage_Count_Lag AS 
(
SELECT 
  sub.Month,
  LAG(sub.Q1_Monthly_Coverages_Total,1) OVER (ORDER BY sub.Month) AS Q1_Monthly_Coverages_Total_Lag
FROM 
  (SELECT 
  DATE_TRUNC(Date_of_Coverage,month) AS Month, -- Creating Month Column
  SUM(Number_of_Coverages) AS Q1_Monthly_Coverages_Total
FROM `my-data-project-36654.FA_Coverage_Pay.Q1_Coverage_Pay`
GROUP BY Month) AS sub
)

SELECT 
  m.month,
  m.Q1_Monthly_Coverages_Total,
  COALESCE(l.Q1_Monthly_Coverages_Total_Lag,0) AS Q1_Monthly_Coverages_Total_Lag,
  COALESCE(m.Q1_Monthly_Coverages_Total - l.Q1_Monthly_Coverages_Total_Lag,0) AS MoM_Coverage_Count_Total_Change,
  COALESCE(ROUND((m.Q1_Monthly_Coverages_Total - l.Q1_Monthly_Coverages_Total_Lag)/l.Q1_Monthly_Coverages_Total_Lag,2),0) AS MoM_Coverage_Count_Total_Change_Pct
FROM Monthly_Coverage_Count AS m
LEFT JOIN Monthly_Coverage_Count_Lag AS l
USING (Month)
ORDER BY m.month

~~~
