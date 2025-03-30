# sql_interview_answers


-- Find the top 25% customers from sales table in the last year --

with cust_sales as(
select 
user_id,
sum(purchase_amount) as total
from user_activity
where activity_date between "2023-01-01" and "2023-12-31"
group by user_id),

sales_rnk as (
select 
user_id,
total,
percent_rank() over(order by total desc) as ranking
from cust_sales
)

select 
user_id,
total
from sales_rnk
where ranking <= 0.25

-- Identify users when users reached 100 points --

select
user_id,
min(date) as date_to_100
from
(select 
user_id,
date,
sum(points)
over(partition by user_id
order by date) as total_points
from user_points) t
where total_points >= 100
group by user_id;

-- Calculate late deliveries percentage from orders table --

with cte as (
select 
order_id,
provider_id,
datediff(delivery_date,order_date) as delivery_days,
case 
	when datediff(delivery_date,order_date) 
	> 3 
	then 1 
	else 0 
	end as is_late
from orders_data)
select 
provider_id,
count(order_id) as total_orders,
sum(is_late) as late_deliveries,
round(sum(is_late)*1.0/count(order_id) *100, 2) as late_delivery_percent
from cte 
group by provider_id;

-- Select product with biggest black friday discount --
with cte as (
select 
product_name,
max(case when price_date = '2023-01-01'
	then price end )as original,
max(case when price_date = '2023-11-25'
	then price end) as black_friday
from products
group by product_name)
select 
product_name,
(original-black_friday)/original * 100
as discount
from cte
where black_friday is not null
order by 2 desc;

-- Find most recent customer transactions using subqueries
select
t.customerr_id,
t.transacton_date,
t.amount
from
(select 
*,
row_number() 
over(partition by customer_id order by transaction_date desc) as rnk
from orders) t
where rnk = 1

---------OR---------

with cte as (
select 
customer_id,
max(transaction_date) as max_date
from orders
group by customer_id)

select 
from orders t
join cte on
t.customer_id = t.customer_id
and t.transaction_date = t.max_date

-- Find managers with direct reports more than 0 --
select 
m.name,
count(em.id) as n_employees
from employee_manager m 
left join employee_manager em 
on m.id = em.manager_id
group by m.name
having count(em.id) > 0;

-- Total of the last- 5 transactions by affiliates

with cte as (
select 
a.affiliate_id, 
b.transaction_date,
b.amount,
row_number()
	over(partition by a.affiliate_id
	order by b.transaction_date) as r_num
from affiliate a 
join transactions b
on a.affiliate_id = b.affiliate_id)
 select
 affiliate_id,
 sum(amount) as total
 from cte
 where r_num <=5
 group by affiliate_id
 
 
