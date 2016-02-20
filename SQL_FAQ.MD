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

-- Small problem 4!! Very good and comprehensive

create table ex4_users (
user_id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
username varchar(255)
);

create table training_details(
user_training_id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
user_id int,
training_id int,
training_date date
);

insert into ex4_users(username) values ("John Doe");
insert into ex4_users(username) values ("Jane Don"),("Alice Jones"),("Lisa Romero");

insert into training_details(user_id, training_id, training_date) values 
(1,1,"2015-08-02"),
(2,1,"2015-08-03"),
(3,2,"2015-08-02"),
(4,2,"2015-08-04"),
(2,2,"2015-08-03"),
(1,1,"2015-08-02"),
(3,2,"2015-08-04"),
(4,3,"2015-08-03"),
(1,4,"2015-08-03"),
(3,1,"2015-08-02"),
(4,2,"2015-08-04"),
(3,2,"2015-08-02"),
(1,1,"2015-08-02"),
(4,3,"2015-08-03");


-- Write a query to to get the list of users who took the a training lesson more than once in the same day, 
-- grouped by user and training lesson,
-- each ordered from the most recent lesson date to oldest date.

-- Official solution
SELECT
      u.user_id,
      username,
      training_id,
      training_date,
      count( user_training_id ) AS count
  FROM ex4_users u JOIN training_details t ON t.user_id = u.user_id
  GROUP BY user_id,
           training_id,
           training_date
  HAVING count( user_training_id ) > 1
  ORDER BY training_date DESC;

-- My solution
-- Be careful when grouping. Group the samllist level
select u.*, training_date,training_id, count(user_training_id) from training_details t
-- select u.*, training_date, count(training_id) from training_details t
join ex4_users u ON t.user_id = u.user_id
group by 1,2,3,4
having count(user_training_id)>1
order by training_date desc;

-- Small problem 5. Rank and partition
-- (Not supported by MySQL)
create table ex5_grade_rank (
name varchar(255),
math int,
english int,
handover_datetime  DATETIME DEFAULT NULL);

-- if total is the same, higher math, higher rank. If the same, who handover early, the higher rank.
 

insert into ex5_grade_rank values 
("Mei",95,95,"2015-05-01 10:30:25"),
("Fei",95,95,"2015-05-01 10:50:20"),
("Ming",100,90,"2015-05-01 10:20:25"),
("Lei",90,100,"2015-05-01 10:40:25"),
("Fang",100,75,"2015-05-01 10:10:25"),
("Ning",98,92,"2015-05-01 9:55:25"),
("Han",100,80,"2015-05-01 11:00:25");

CREATE TABLE ex5_person (id int, first_name varchar(20), age int, gender char(1));

INSERT INTO ex5_person VALUES (1, 'Bob', 25, 'M');
INSERT INTO ex5_person VALUES (2, 'Jane', 20, 'F');
INSERT INTO ex5_person VALUES (3, 'Jack', 30, 'M');
INSERT INTO ex5_person VALUES (4, 'Bill', 32, 'M');
INSERT INTO ex5_person VALUES (5, 'Nick', 22, 'M');
INSERT INTO ex5_person VALUES (6, 'Kathy', 18, 'F');
INSERT INTO ex5_person VALUES (7, 'Steve', 36, 'M');
INSERT INTO ex5_person VALUES (8, 'Anne', 25, 'F');

select * from ex5_person;


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