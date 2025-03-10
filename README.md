# Credit-Card-Transaction-Analysis

---

## **Project Overview**

This project involves analyzing credit card transaction data using SQL queries. It provides insights into spending patterns, top-performing cities, card type preferences, and customer demographics. The analysis is designed to help businesses optimize their credit card offerings and enhance customer experience

An ERD diagram is included to visually represent the database schema and relationships between tables.

---


## **Database Setup & Design**

### **Schema Structure**

```sql
-- create database
create database project;
use project;

-- start creating  schema

CREATE TABLE [dbo].[credit_card_transaction](
	[transaction_id] [smallint] NOT NULL,
	[city] [nvarchar](50) NOT NULL,
	[transaction_date] [date] NOT NULL,
	[card_type] [nvarchar](50) NOT NULL,
	[exp_type] [nvarchar](50) NOT NULL,
	[gender] [nvarchar](50) NOT NULL,
	[amount] [int] NOT NULL
)

-- end of schemas
```
---

## **Objective**

•The primary goal of this project is to extract meaningful insights from credit card transactions by performing various SQL queries. These insights include:

•Identifying top cities based on spending.

•Determining monthly spending trends by card type.

•Finding cumulative spending milestones.

•Analyzing spending contributions by gender and expense type.

•Evaluating the fastest-growing card and expense type combination.

•Examining transaction behavior on weekends.

•Identifying cities with rapid transaction growth.

## **Identifying Business Problems**

Key business problems identified:

1.High-Spending Locations: Which cities contribute the most to total credit card spends?

2.Monthly Trends: Which months see the highest credit card usage for each card type?

3.Card-Specific City Spending: Which city has the lowest spend percentage for a particular card type?

4.Expense Type Trends: What are the highest and lowest expense categories in each city?

5.Gender-Based Spending: What percentage of spending is contributed by female cardholders across different expense types?

6.Transaction Growth Speed: Which city reached 500 transactions in the shortest time?

---

## **Solving Business Problems**

### Solutions Implemented:

select * from credit_card_transaction; 


1- write a query to print top 5 cities with highest spends and their percentage contribution 
of total credit card spends 
```
with total_spent_cte as 
(
select sum(cast(amount as bigint)) as total_spent from credit_card_transaction
)
select top 5 city,sum(amount) as expense,total_spent,
cast(sum(amount)*1.0 /total_spent * 100 as decimal(5,2)) as percentage_contribution
from credit_card_transaction inner join total_spent_cte on 1=1
group by city,total_spent order by expense desc;
```


2- write a query to print highest spend month and amount spent in that month for each card type
```
with cte as
(
select card_type,sum(amount) as monthly_expense,datepart(month,transaction_date) as month,
datepart(year,transaction_date) as year from credit_card_transaction group by card_type,
datepart(month,transaction_date),datepart(year,transaction_date)
)
select * from
(select *,rank() over(partition by card_type order by monthly_expense desc) as rn from cte) A
where rn=1
```



3- write a query to print the transaction details(all columns from the table) for each card type when
--it reaches a cumulative of 1000000 total spends(We should have 4 rows in the o/p one for each card type)
```
select * from 
(select *, rank() over(partition by card_type order by cum_sum) as rn  from 
(select  * ,sum(amount) over(partition by card_type order by transaction_date,transaction_id)
as cum_sum from credit_card_transaction)A
where cum_sum >=1000000 )B
where rn=1
```


4- write a query to find city which had lowest percentage spend for gold card type
```
select city,sum(amount) as total_spend,
sum(case when card_type='Gold' then amount else 0 end) as gold_spend,
sum(case when card_type='Gold' then amount else 0 end)*1.0/sum(amount) *100 as gold_contribution
from credit_card_transaction 
group by city having sum(case when card_type='Gold' then amount else 0 end) > 0
order by  gold_contribution
```


5- write a query to print 3 columns:  city, highest_expense_type , lowest_expense_type 
(example format : Delhi , bills, Fuel)
```
with cte as 
(
select city,exp_type,sum(amount) as total_spend from credit_card_transaction
group by city,exp_type
),
 cte2 as
(
select * ,
rank() over(partition by city order by total_spend desc)as rn_high,
rank() over(partition by city order by total_spend )as rn_low from cte
)
select city,
max(case when rn_high=1 then exp_type end) as highest_expense_type,
max(case when rn_low=1 then exp_type end) as lowest_expense_type
from cte2
where rn_high=1 or rn_low=1
group by city;
```


6- write a query to find percentage contribution of spends by females for each expense type
```
select exp_type,sum(amount) as total_spend,
sum(case when gender='F' then amount else 0 end) as female_spend,
sum(case when gender='F' then amount else 0 end)*1.0/sum(amount)*100 as female_contribution
from credit_card_transaction
group by exp_type
order by female_contribution 
```


7- which card and expense type combination saw highest month over month growth in Jan-2014
```
with cte as 
(
select card_type,exp_type,datepart(year,transaction_date) yt
,datepart(month,transaction_date) mt,sum(amount) as total_spend
from credit_card_transaction
group by card_type,exp_type,datepart(year,transaction_date),datepart(month,transaction_date)
)
select  top 1 *, (total_spend-prev_mont_spend) as mom_growth
from (
select *
,lag(total_spend,1) over(partition by card_type,exp_type order by yt,mt) as prev_mont_spend
from cte) A
where prev_mont_spend is not null and yt=2014 and mt=1
order by mom_growth desc;
```



8- during weekends which city has highest total spend to total no of transcations ratio 
```
select city, sum(amount)*1.0/count(*) as ratio 
from credit_card_transaction
where datepart(weekday,transaction_date) in (1,7)
group by city
```


9- which city took least number of days to reach its 500th transaction after the first 
transaction in that city
```
with cte as 
(
select *
, row_number() over(partition by city order by transaction_date,transaction_id) as rn
from credit_card_transaction
)
select city, min(transaction_date) as first_transaction,max(transaction_date) as last_transaction,
datediff(day,min(transaction_date),max(transaction_date)) as days_to_500
from cte
where rn in (1,500)
group by city
having count(*)=2
order by days_to_500 asc
```
---

## **Learning Outcomes**

This project enabled me to:
- Design and implement a normalized database schema.
- Clean and preprocess real-world datasets for analysis.
- Use advanced SQL techniques, including window functions, subqueries, and joins.
- Conduct in-depth business analysis using SQL.
- Optimize query performance and handle large datasets efficiently.

---

## **Conclusion**

This project leverages SQL to analyze credit card transactions and extract business insights. These queries help financial institutions understand spending trends, optimize card offerings, and enhance customer targeting strategies.

By completing this project, I have gained a deeper understanding of how SQL can be used to tackle complex data problems and drive business decision-making.

---


