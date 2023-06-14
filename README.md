# Data-Lemur-SQL
This repository contains my answer of data lemur SQL practice. You can practice SQL for free from top IT companies on this website [Data Lemur](https://datalemur.com/).
I wrote this repository for learning purpose, the anwers below are my own solution. 

**Contents:**

- [User's Third Transaction Uber SQL Interview Question](#user's-third-transaction-uber-sql-interview-question)
- [Sending vs Opening Snaps Snapchat SQL Interview Question](#sending-vs-opening-snaps-snapchat-sql-interview-question)
- [Tweets Rolling Averages Twitter SQL Interview Question](#tweets-rolling-averages-twitter-sql-interview-question)
- [Highest Grossing Items Amazon SQL Interview Question](#highest-grossing-items-amazon-sql-interview-question)
- [Duplicate Job Listings Linkedin SQL Interview Question](#duplicate-job-listings-linkedin-sql-interview-question)
- [Average Review Ratings Amazon SQL Interview Question](#average-review-ratings-amazon-sql-interview-question)



## User's Third Transaction Uber SQL Interview Question

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

<br></br>

## Sending vs Opening Snaps Snapchat SQL Interview Question

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
<br></br>

## Tweets Rolling Averages Twitter SQL Interview Question

***Question:***

Given a table of tweet data over a specified time period, calculate the 3-day rolling average of tweets for each user. Output the user ID, tweet date, and rolling averages rounded to 2 decimal places.
Notes:
A rolling average, also known as a moving average or running mean is a time-series technique that examines trends in data over a specified period of time.
In this case, we want to determine how the tweet count for each user changes over a 3-day period.

<img width="275" alt="image" src="https://github.com/rindangchi/Data-Lemur-SQL/assets/10241058/9d380cfb-e003-4410-8b05-27bd42aa116c">

***Answer:***

```sql
with CT as(
select user_id, tweet_date, tweet_count
from tweets
group by user_id, tweet_date, tweet_count
order by user_id, tweet_date)


select user_id,tweet_date,  
round(avg(tweet_count)
over
(partition by user_id order by tweet_date rows between 2 PRECEDING AND CURRENT ROW ),2) rolling_avg_3days
from CT 
```

<img width="407" alt="image" src="https://github.com/rindangchi/Data-Lemur-SQL/assets/10241058/15cdcf7d-e815-4db9-b997-4d1020db8a18">

<br></br>

## Highest Grossing Items Amazon SQL Interview Question

***Question:***

Assume you're given a table with information on Amazon customers and their spending on products in different categories, write a query to identify the top two highest-grossing products within each category in the year 2022. The output should include the category, product, and total spend.

<img width="404" alt="image" src="https://github.com/rindangchi/Data-Lemur-SQL/assets/10241058/e79a8b12-5c3a-46d8-aea5-ed82baa9f875">

***Answer:***

```sql
select category, product, total_spend
from(
select category,product,sum(spend) total_spend, row_number()
over(partition by category order by sum(spend) desc) 
from(
SELECT *
FROM  PRODUCT_SPEND   
where extract(year from transaction_date) = 2022
) table1
group by product,category) table2
where row_number in (1,2)
```

<img width="405" alt="image" src="https://github.com/rindangchi/Data-Lemur-SQL/assets/10241058/f494dfef-7937-4cd0-ac57-8f2bfc11c085">

<br></br>

## Duplicate Job Listings Linkedin SQL Interview Question

***Question:***

Assume you are given the table below that shows job postings for all companies on the LinkedIn platform. Write a query to get the number of companies that have posted duplicate job listings.
Clarification:
Duplicate job listings refer to two jobs at the same company with the same title and description.

<img width="428" alt="image" src="https://github.com/rindangchi/Data-Lemur-SQL/assets/10241058/841724f1-ee9d-4af2-8ab8-a0d15e61141b">

***Answer:***

```sql
select count(company_id) as duplicate_companies
from(
select company_id, title, description, count(*) as count_job
from job_listings
group by company_id, title, description
having count(*) > 1)
as table1
```

<img width="338" alt="image" src="https://github.com/rindangchi/Data-Lemur-SQL/assets/10241058/3bcae69a-b459-4fdc-a91a-7a4b3753f491">

<br></br>

## Average Review Ratings Amazon SQL Interview Question

***Question:***

Given the reviews table, write a query to retrieve the average star rating for each product, grouped by month. The output should display the month as a numerical value, product ID, and average star rating rounded to two decimal places. Sort the output first by month and then by product ID.

<img width="299" alt="image" src="https://github.com/rindangchi/Data-Lemur-SQL/assets/10241058/9608fc81-1f8e-476c-b8f7-67cd65c7f083">


***Answer:***

```sql
select extract(month from submit_date)as mth, product_id as product, round(avg(stars),2) as avg_stars
from reviews
group by mth, product
order by mth, product
```

<img width="349" alt="image" src="https://github.com/rindangchi/Data-Lemur-SQL/assets/10241058/db2278c5-c286-4ac5-aa20-52e637817e5b">



