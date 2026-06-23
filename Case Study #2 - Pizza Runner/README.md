-----------------Data Cleaning & Transformation--------------

craete temp table customer_orders_temp as
select 
  order_id, 
  customer_id, 
  pizza_id, 
  case
	  when exclusions IS null or exclusions = 'null' then ''
	  else exclusions
	  end as exclusions,
  case
	  when extras is null or extras = 'null' then ''
	  else extras
	  end as extras,
	order_time
from customer_orders;

select * from customer_orders_temp;

create temp table runner_orders_temp as
select 
  order_id, 
  runner_id,  
  case
	  when pickup_time = 'null' then null
	  else pickup_time
	  end as pickup_time,
  case
	  when distance = 'null' then null
	  when distance like '%km' then trim('km' from distance)
	  else distance 
    end as distance,
  case
	  when duration = 'null' then null
	  when duration like '%mins' then trim('mins' from duration)
	  when duration like '%minute' then trim('minute' from duration)
	  when duration like '%minutes' then trim('minutes' from duration)
	  else duration
	  end as duration,
  case
	  when cancellation = '' or cancellation like 'null' then null
	  else cancellation
	  end as cancellation
from runner_orders;

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

1.How many pizzas were ordered?

select count(pizza_id) as pizzas_ordered
from customer_orders_temp;


2.How many unique customer orders were made?

select count(distinct order_id) as unique_orders
from customer_orders_temp;


3.How many successful orders were delivered by each runner?

select runner_id, count(order_id) as successful_orders
from runner_orders_temp
where cancellation is null
group by 1;


4.How many of each type of pizza was delivered?

select p.pizza_name, 
  count(c.pizza_id) as delivered_pizzas
from customer_orders_temp as c
join runner_orders_temp AS r
  on c.order_id = r.order_id
join pizza_names as p
  on c.pizza_id = p.pizza_id
where r.cancellation is null
group by 1;


5.How many Vegetarian and Meatlovers were ordered by each customer?

select c.customer_id, p.pizza_name, 
  count(p.pizza_name) as pizzas_ordered
from customer_orders_temp as c
join pizza_names as p
  on c.pizza_id= p.pizza_id
group by 1, 2
order by 1;


6.What was the maximum number of pizzas delivered in a single order?

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


7.For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

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


8.How many pizzas were delivered that had both exclusions and extras?

select  
sum(case when exclusions !='' and extras !='' then 1
    else 0 end) as with_exclusions_extras
FROM customer_orders_temp as c
join runner_orders_temp as r
  on c.order_id = r.order_id
where r.cancellation is null;


9.What was the total volume of pizzas ordered for each hour of the day?

select extract(hour from order_time) AS hour_of_day, 
count(order_id) as pizza_count
from customer_orders_temp
group by 1
order by 1;


10.What was the volume of orders for each day of the week?

select to_char(order_time, 'Day') as order_day,
count(order_id) as pizzas_ordered
from customer_orders_temp
group by 1;
