# SQL Questions

**Author: Wenxiao Jeremy Gu** 

<jeremygu86@gmail.com>

**Latest update:** Feb 18, 2016

This document talks about my interview experience in SQL for Statistician/Data scientist/BI analyst positions.

This document is also the reading notes for two books [Learning SQL](http://www.amazon.com/Learning-SQL-Alan-Beaulieu/dp/0596520832/ref=sr_1_1?s=books&ie=UTF8&qid=1455490909&sr=1-1&keywords=Learning+sql) and [Joe Celko's SQL Puzzles and Answers](http://www.amazon.com/Puzzles-Answers-Kaufmann-Management-Systems/dp/0123735963/ref=sr_1_1?s=books&ie=UTF8&qid=1455490957&sr=1-1&keywords=SQL+Puzzle).

##### Frequent interview questions

```sql
-- Small problem 1
--- Calculate the percentge 

create table ex1_expense(
item varchar(255),
spent decimal(10,2)
);

insert into ex1_expense values ('eat',500.50),('flight',2000.10),('housing',900.41),('taxi',129.55);
select item, spent, CONCAT(round(spent/b.sum_s*100,1), '%') percentage from ex1_expense A, (select sum(spent) sum_s from ex1_expense) B;


-- Small problem 2
-- Group by, adding columns, left outer join

create table inventory (
	brand varchar(255),
	location_id int,
	type varchar (255) check ( type IN ('new', 'user')),
	inventory int 
);

insert into inventory values 
('BMW',9856,'new',63),
('BMW',79285,'used',31),
('Chevrolet',2505,'new',125),
('Honda',4663,'new',133),
('Jeep',7931,'used',22),
('Toyota',1113,'new',100),
('Toyota',3136,'used',45),
('Toyota',48753,'new',120);
select * from inventory;

-- get the inventory for each brand and for each type

select distinct c.brand, a.count_new, b.count_old from inventory C

left join 
(select brand, sum(inventory) count_new from inventory I
where type = 'new'
group by brand) a  on a.brand = c.brand

left join (
select brand, sum(inventory) count_old from 
inventory I
where type = 'used'
group by brand) b on c.brand = b.brand;


-- Small problem 3 Play with datetime
-- Calculate time interval (Age: years, minutes)
-- Calcualte how many days/months/week ago
-- Calculate how many days/months/weeks later
-- Get the day, month, week of the year
-- Get the day of week
-- Get the day (M,T,...)

-- datetimes don't change if we change time zone.
-- timestamps change if we change time zone.
drop table ex3_datetime;
create table ex3_datetime (
	name varchar(255),
	born_datetime DATETIME DEFAULT NULL,
	graduation_datetime DATETIME DEFAULT NULL,
	creation_dt DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
	creation_ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

insert into ex3_datetime(name, born_datetime, graduation_datetime) values 
("GWX","1988-03-20 05:05:05","2014-06-10 15:05:05");

SELECT A.*,
now() today, 
TIMESTAMPDIFF(YEAR,born_datetime,now()) age,
TIMESTAMPDIFF(MINUTE,born_datetime,now()) age_MINS,
DATE_ADD(born_datetime,INTERVAL 29 YEAR) YEAR_30_LATER2,
DATE_ADD(born_datetime,INTERVAL -30 YEAR) YEAR_30_AGO,
DAYOFWEEK(born_datetime) born_day
FROM ex3_datetime A;
-- Calculate time interval (Age: years, minutes)
-- Calcualte how many days/months/week ago
-- Calculate how many days/months/weeks later
-- Get the day, month, week of the year
-- Get the day of week
-- Get the day (M,T,...)
```


##### **[Pr14]** Telephone directory

**outter join and NULL value**

Suppose you are trying to set up an office telephone directory with your new database publishing system, and you have the following tables:

```sql
    CREATE TABLE Personnel
        (emp_id INTEGER PRIMARY KEY,
         first_name CHAR(20) NOT NULL,
         last_name CHAR(20) NOT NULL);
    CREATE TABLE Phones
        (emp_id INTEGER NOT NULL,
         phone_type CHAR(3) NOT NULL
                 CHECK (phone_type IN ('hom', 'fax')),
         phone_nbr CHAR(12) NOT NULL,
         PRIMARY KEY (emp_id, phone_type),
         FOREIGN KEY emp_id REFERENCES Personnel(emp_id));
         
                  
         -- http://sqlfiddle.com/ code
```

*Business question*

The codes 'hom' and 'fax' indicate whether the number is the employee’s home phone number or a fax number. 

You want to print out a report with one line per employee that gives both numbers, and shows a NULL if either or both numbers are missing.

**Note that** 

- The FOREIGN KEY constraint on the Phones table means that you cannot list a telephone number for someone who is not an employee. The PRIMARY KEY looks a bit large until you stop and think about all the cases. Married personnel could share the same fax or home telephones, and a single line could be both voice and fax services.


** First Try **

1. temporary table A: Join the two tables (outer join)
```sql
	with A as (
		select personnel.*, phones.* from personnel
		outer join phones on personnel.emp_id = phones.emp_id
	)
```
2. calculate the nulls per emp_id. if ==2 report 1; if <2 report 2.

** Standard solution 1: look at each of the phone types; then combine**

Left outer join because we want the people to be employees! (Because you want to see all the personnel, you need an OUTER JOIN)

```sql
-- PARTIAL solution
SELECT E1.last_name, E1.first_name,
       H1.phone_nbr AS Home,
       F1.phone_nbr AS FAX
 FROM Personnel AS E1
        
 LEFT OUTER JOIN
        Phones AS H1
        ON E1.emp_id = H1.emp_id
           AND H1.phone_type = 'hom' -- only home number
 LEFT OUTER JOIN
       Phones AS F1  -- outer join fax number
       ON E1.emp_id = F1.emp_id
          AND F1.phone_type = 'fax';
-- My solution
with A as (
select E1.last_name, E1.first_name,
		H1.phone_nbr as Home
		F1.phone_nbr as FAX
from 

Personnel E1

left outer join phones H1
on E1.emp_id = F1.emp_id AND H1.phone_type = 'hom'

left outer join phones F1 -- extra columns wrt row
on E1.emp_id = F1.emp_id AND F1.phone_type = 'fax'
)


select last_name, first_name from A
where HOME is null or FAX is null
group by 1,2
;

select last_name, first_name from A
where HOME is not null and FAX is not null
group by 1,2;

```

##### **[Pr10]** Calculation of the recent xx months WAGES OF S-I-N

**This question talks about how to look at data ar row level.**

- The SIN column is the Social Insurance Number, which is something like the Social Security Number (SSN) used in the United States to identify taxpayers. 

- The pen_year column is the calendar year of the pension.

- The month_cnt column is the number of months in that year the person worked.

- The earnings is the person’s total earnings for the year.

*Business question*

The problem is to find the total earnings of each employee for the **most recent 60 months** of month_cnt in **consecutive** years. This number is used to compute the employee’s pension. The shortest period going back could be 5 years, with 12 months in each year applying to the total month_cnt. The longest period could be 60 years, with 1 month in each year. Some people might work four years and not the fifth, and thus not qualify for a pension at all.

The reason this is a beast to solve is that "most recent" and "consecutive" are hard to write in SQL.

**HINT**: _For each employee in each year, insert a row even in the years in which the employee did not work. It not only makes the query easier, but you also have a record to update when you get in new information._


```sql
CREATE TABLE Pensions
        (sin CHAR(10) NOT NULL,
         pen_year INTEGER NOT NULL,
         month_cnt INTEGER DEFAULT 0 NOT NULL
             CHECK (month_cnt BETWEEN 0 AND 12),
         earnings DECIMAL (8,2) DEFAULT 0.00 NOT NULL);         
```

Analysis on the solutions:

**1. Solution 1**

- 1) Make intervals. SIN, start_year, end_year, total_earn 

- 2) Whether consecutive. if_consec

- 3) Structure of the code (at row level)


```
WITH TMR AS (
SELECT T1.SIN, T1.PEN_YEAR START_YEAR, T2.PEN_YEAR END_YEAR,
	(SELECT SUM(earnings) 
		FROM Pensions T3
		WHERE T3.SIN = T1.SIN
		AND T3.PEN_YEAR between  T1.PEN_YEAR and T2.PEN_YEAR) total_earning

FROM Pensions T1, Pensions T2
WHERE T1.SIN = T2.SIN
	AND T2.PEN_YEAR >= P1.PEN_YEAR - 4 -- most recent 60 months >= years 
	AND 0<ALL (SELECT month_cnt from Pensions T4 where
	T4.SIN  = T1.SIN and T4.PEN YEAR BETWEEN  T1.PEN_YEAR and T2.PEN_YEAR) -- between start_year and end_year, month_cnt > 0 in each year
	AND 60 <= (SELECT SUM(month_cnt) from Pensions T4 where
	T4.SIN  = T1.SIN and T4.PEN YEAR BETWEEN  T1.PEN_YEAR and T2.PEN_YEAR)
)

SELECT * FROM TMR
WHERE END_YEAR = (SELECT MAX(END_YEAR)  -- FOR EACH SIN
				 FROM TMR AS T5 
				 WHERE T5.SIN = TMR.SIN)
;
```


##### Appendix 


###### Common mistakes

Missing group by

###### Good resources

[Learning SQL](http://www.amazon.com/Learning-SQL-Alan-Beaulieu/dp/0596520832/ref=sr_1_1?s=books&ie=UTF8&qid=1455490909&sr=1-1&keywords=Learning+sql)

- Lost notes.

[Joe Celko's SQL Puzzles and Answers](http://www.amazon.com/Puzzles-Answers-Kaufmann-Management-Systems/dp/0123735963/ref=sr_1_1?s=books&ie=UTF8&qid=1455490957&sr=1-1&keywords=SQL+Puzzle).

[Online SQL interviews](http://www.toptal.com/sql/interview-questions)

[Online SQL: SQLfiddle](http://sqlfiddle.com/)

###### Hive Syntax

```sql
SELECT [ALL | DISTINCT] select_expr, select_expr, ...
FROM table_reference
[WHERE where_condition]
[GROUP BY col_list]
[HAVING having_condition]
[CLUSTER BY col_list | [DISTRIBUTE BY col_list] [SORT BY col_list]]
[LIMIT number]
;
```