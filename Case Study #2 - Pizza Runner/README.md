CREATE TEMP TABLE customer_orders_temp AS
SELECT 
  order_id, 
  customer_id, 
  pizza_id, 
  CASE
	  WHEN exclusions IS null OR exclusions = 'null' THEN ''
	  ELSE exclusions
	  END AS exclusions,
  CASE
	  WHEN extras IS NULL or extras = 'null' THEN ''
	  ELSE extras
	  END AS extras,
	order_time
FROM pizza_runner.customer_orders;
select * from customer_orders_temp;

CREATE TEMP TABLE runner_orders_temp AS
SELECT 
  order_id, 
  runner_id,  
  CASE
	  WHEN pickup_time LIKE 'null' THEN null
	  ELSE pickup_time
	  END AS pickup_time,
  CASE
	  WHEN distance LIKE 'null' THEN null
	  WHEN distance LIKE '%km' THEN TRIM('km' from distance)
	  ELSE distance 
    END AS distance,
  CASE
	  WHEN duration LIKE 'null' THEN null
	  WHEN duration LIKE '%mins' THEN TRIM('mins' from duration)
	  WHEN duration LIKE '%minute' THEN TRIM('minute' from duration)
	  WHEN duration LIKE '%minutes' THEN TRIM('minutes' from duration)
	  ELSE duration
	  END AS duration,
  CASE
	  WHEN cancellation = '' or cancellation LIKE 'null' THEN null
	  ELSE cancellation
	  END AS cancellation
FROM pizza_runner.runner_orders;

alter table runner_orders_temp
alter column pickup_time type TIMESTAMP
using pickup_time::TIMESTAMP;

alter table runner_orders_temp
alter column distance type float
using distance::float;

alter table runner_orders_temp
alter column duration type int
using distance::int;

select * from runner_orders_temp;

------------------------Pizza Metrics-------------------------
select count(pizza_id) as pizzas_ordered
from customer_orders_temp;

select count(distinct order_id) as unique_orders
from customer_orders_temp;

select runner_id, count(order_id) as successful_orders
from runner_orders_temp
where cancellation is null
group by 1;

select p.pizza_name, 
  count(c.pizza_id) as delivered_pizzas
from customer_orders_temp as c
join runner_orders_temp AS r
  on c.order_id = r.order_id
join pizza_names as p
  on c.pizza_id = p.pizza_id
where r.cancellation is null
group by 1;

select c.customer_id, p.pizza_name, 
  count(p.pizza_name) as pizzas_ordered
from customer_orders_temp as c
join pizza_names as p
  on c.pizza_id= p.pizza_id
group by 1, 2
order by 1;

with cte as
(
  select c.order_id, count(c.pizza_id) as pizza_per_order
  from customer_orders_temp as c
  join runner_orders_temp as r
    on c.order_id = r.order_id
  where r.distance != 0
  group by c.order_id
)
select max(pizza_per_order) as pizza_count
from cte;

select c.customer_id,
sum(case when c.exclusions != ' ' or c.extras != ' ' then 1 else 0 end) as changed_pizzas,
sum(case when c.exclusions = ' ' and c.extras = ' ' then 1 else 0 end) 
as no_changes
from customer_orders_temp as c
join runner_orders_temp as r
  on c.order_id = r.order_id
where r.cancellation is null
group by 1
order by 1;

select  
sum(case when exclusions !='' and extras !='' then 1
    else 0 end) as with_exclusions_extras
FROM customer_orders_temp as c
join runner_orders_temp as r
  on c.order_id = r.order_id
where r.cancellation is null;

select extract(hour from order_time) AS hour_of_day, 
count(order_id) as pizza_count
from customer_orders_temp
group by 1
order by 1;

select to_char(order_time, 'Day') as order_day,
count(order_id) as pizzas_ordered
from customer_orders_temp
group by 1;
