# SQL Questions

**Author: Wenxiao Jeremy Gu** 

<jeremygu86@gmail.com>

**Latest update:** Feb 18, 2016

This document talks about my interview experience in SQL for Statistician/Data scientist/BI analyst positions.

This document is also the reading notes for two books [Learning SQL](http://www.amazon.com/Learning-SQL-Alan-Beaulieu/dp/0596520832/ref=sr_1_1?s=books&ie=UTF8&qid=1455490909&sr=1-1&keywords=Learning+sql) and [Joe Celko's SQL Puzzles and Answers](http://www.amazon.com/Puzzles-Answers-Kaufmann-Management-Systems/dp/0123735963/ref=sr_1_1?s=books&ie=UTF8&qid=1455490957&sr=1-1&keywords=SQL+Puzzle).

#### Frequent interview questions


**[Pr14]** Telephone directory

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
```

*Business question*

The codes 'hom' and 'fax' indicate whether the number is the employee’s home phone number or a fax number. You want to print out a report with one line per employee that gives both numbers, and shows a NULL if either or both numbers are missing.

**Note that** 

- The FOREIGN KEY constraint on the Phones table means that you cannot list a telephone number for someone who is not an employee. The PRIMARY KEY looks a bit large until you stop and think about all the cases. Married personnel could share the same fax or home telephones, and a single line could be both voice and fax services.




**[Pr10]** Calculation of the recent xx months

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