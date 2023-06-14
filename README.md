# Data-Lemur-SQL
This repository contains my answer of data lemur SQL practice. You can practice SQL for free from top IT companies on this website [Data Lemur](https://datalemur.com/).
I wrote this repository for learning purpose. 

**1. User's Third Transaction [Uber SQL Interview Question]**

***Question:***

Assume you are given the table below on Uber transactions made by users. Write a query to obtain the third transaction of every user. Output the user id, spend and transaction date.

<img width="241" alt="image" src="https://github.com/rindangchi/Data-Lemur-SQL/assets/10241058/39a7cc3f-27c5-4d09-b05a-ec563c229026">

***Answer:***

```sql
select user_id, spend, transaction_date
from(
select * FROM
(select * , row_number() OVER(PARTITION BY table1.user_id) row_number
from (
select * from transactions order by user_id, transaction_date) table1
) table2
where row_number = 3)
table3
```

<img width="370" alt="image" src="https://github.com/rindangchi/Data-Lemur-SQL/assets/10241058/b27c8ac0-4ca0-4383-95c0-4092d10e1a1f">


**2. Sending vs. Opening Snaps [Snapchat SQL Interview Question]**

***Question***:

Assume you're given tables with information on Snapchat users, including their ages and time spent sending and opening snaps.
Write a query to obtain a breakdown of the time spent sending vs. opening snaps as a percentage of total time spent on these activities grouped by age group. Round the percentage to 2 decimal places in the output.

Notes:
Calculate the following percentages:
time spent sending / (Time spent sending + Time spent opening)
Time spent opening / (Time spent sending + Time spent opening)
To avoid integer division in percentages, multiply by 100.0 and not 100.

<img width="405" alt="image" src="https://github.com/rindangchi/Data-Lemur-SQL/assets/10241058/0d34cbfe-c124-4e47-b91d-6cc3a7fb994c">

<img width="208" alt="image" src="https://github.com/rindangchi/Data-Lemur-SQL/assets/10241058/275bddb1-a62c-4657-b3f9-1da51e4a04f8">

***Answer:***

```sql

with open as 
(
SELECT act.user_id, age.age_bucket, sum(time_spent) open
FROM activities act
inner join age_breakdown age
using (user_id)
group by user_id, act.activity_type, age.age_bucket
having activity_type = 'open')

, send as 
(
SELECT act.user_id, age.age_bucket, sum(time_spent) send
FROM activities act
inner join age_breakdown age
using (user_id)
group by user_id, act.activity_type, age.age_bucket
having activity_type = 'send')

select y.age_bucket, round(100.0*(sum_send/(sum_open + sum_send)),2) send_perc,
round(100.0*(sum_open/(sum_open + sum_send)),2) open_perc

from(
select x.age_bucket, sum(open) sum_open, sum(send) sum_send
from(
select * from open
join send 
using (user_id, age_bucket)) x
group by age_bucket ) y

```

<img width="397" alt="image" src="https://github.com/rindangchi/Data-Lemur-SQL/assets/10241058/f1e3f63b-2605-4cc0-b9ad-19f7c28b5553">






