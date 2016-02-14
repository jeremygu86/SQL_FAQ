# SQL functions

**Author: Wenxiao Jeremy Gu** 

<jeremygu86@gmail.com>

**Latest update:** Feb 14, 2016

``` SQL
/****** 10 Frequently asked SQL Query Interview Questions   ******/
/** 1. **/ -- Top 5 highest
SELECT top 5 [pi_amount] FROM [ENT_DSS_DW].[dbo].[view_fact_sales_opportunity] 
ORDER BY [pi_amount] DESC;

-- Top 5 loweset in default order
SELECT TOP 5 [pi_amount] FROM [ENT_DSS_DW].[dbo].[view_fact_sales_opportunity] 
ORDER BY pi_amount ;

-- Top 5 loweset in desc order: cant select from select. Use select from where in (select)
select [pi_amount] from [ENT_DSS_DW].[dbo].[view_fact_sales_opportunity] 
where [pi_amount] in (select top 5 [pi_amount] from [ENT_DSS_DW].[dbo].[view_fact_sales_opportunity] order by [pi_amount])
order by [pi_amount] desc

-- Second Highest
SELECT MAX([pi_amount]) FROM [ENT_DSS_DW].[dbo].[view_fact_sales_opportunity] 
WHERE [pi_amount] not in (select max([pi_amount]) from [ENT_DSS_DW].[dbo].[view_fact_sales_opportunity])

/** 2.SQL Query to find Max Salary from each department. **/ 
SELECT max([pi_amount]),dim_sales_territory_key 
FROM [ENT_DSS_DW].[dbo].[view_fact_sales_opportunity]  
GROUP BY dim_sales_territory_key
ORDER BY dim_sales_territory_key 

/** 3.DATE. **/ 
-- Calculate today
select getdate() -- 2014-07-29 14:33:50.807
SELECT YEAR(GETDATE()) as "Year"; -- 2014
-- Calculate the day of today
--select DATEADD(ss, DATEDIFF(ss, 0, getdate()), 0) -- 2014-10-14 10:40:46.033
select DATEADD(mi, DATEDIFF(mi, 0, getdate()), 0) -- 2014-10-14 10:41:00.000
select DATEADD(hh, DATEDIFF(hh, 0, getdate()), 0) -- 2014-10-14 10:00:00.000
select DATEADD(dd, DATEDIFF(dd, 0, getdate()), 0) -- 2014-10-14 00:00:00.000
select DATEADD(wk, DATEDIFF(wk, 0, getdate()), 0) -- 2014-10-13 00:00:00.000
select DATEADD(mm, DATEDIFF(mm, 0, getdate()), 0) -- 2014-10-1 00:00:00.000
select DATEADD(yy, DATEDIFF(yy, 0, getdate()), 0) -- 2014-1-1 00:00:00.000

select distinct top 13 fin_quarter from view_dim_date where calendar_date>=DATEADD(dd, DATEDIFF(dd, 0, getdate()), 0)  order by fin_quarter
--select DATEADD(dd, DATEDIFF(dd, 0, getdate()), 0) -- 2014-07-29 00:00:00.000
--select DATEADD(dd, 0, 0) -- 1900-01-01 00:00:00.000
--select DATEDIFF(dd, 0, getdate()) --41847
/** 4. Write an SQL Query to check whether date passed to Query is date of given format or not. **/ 
select isdate(getdate()) -- 1
/** 5.Write a SQL Query to print the name of distinct employee whose DOB is between 01/01/1960 to 31/12/1975. **/ 
-- SELECT DISTINCT EmpName FROM Employees WHERE DOB  BETWEEN ‘01/01/1960’ AND ‘31/12/1975’;
/** 9. find all Employee records containing the word "Joe", regardless of whether it was stored as JOE, Joe, or joe.**/ 
-- SELECT  * from Employees  WHERE  upper(EmpName) like upper('joe%');
```
