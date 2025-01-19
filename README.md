# Supply Chain Case Study
- Analyzing the data set that was provided and to get solutions for the problem statements.

## Problem Statements and their solutions

Q1. Get the number of orders by the Type of Transaction. Please exclude orders shipped from Sangli and Srinagar. Also, exclude the SUSPECTED_FRAUD cases based on the Order Status. Sort the result in the descending order based on the number of orders.

```sql
select count(Order_Id) as ord_num , Type
from orders
where Order_City not in ('Sangli', 'Srinagar') and Order_Status != 'SUSPECTED_FRAUD'
group by Type
order by ord_num desc;
```
![WhatsApp Image 2025-01-17 at 21 54 44_23b8ed10](https://github.com/user-attachments/assets/acaa33a8-32c4-4017-b0e3-480da731e221)



Q2. Get the list of the Top 3 customers based on the completed orders along with the following details:
Customer Id, Customer First Name, Customer City, Customer State, Number of completed orders, Total Sales
```sql
select Id, First_Name, City, State, count(Order_Id) as ord_num, sum(Sales) as total_sales
from customer_info c 
join orders o
on  c.ID = o.Customer_Id
join ordered_items i
using(Order_Id)
where order_status = 'Complete'
group by Id, First_Name, City, State
order by ord_num desc limit 3;
```
![WhatsApp Image 2025-01-17 at 21 55 58_d975855f](https://github.com/user-attachments/assets/3feeba34-1cc3-449d-a280-07ded07c969c)

Q3. Get the order count by the Shipping Mode and the Department Name. Consider departments with at least 40 closed/completed orders.
```sql
select count(distinct Order_id) as ord_count, Shipping_Mode, Name
from orders
join ordered_items o
using(order_id)
join product_info p 
on o.Item_Id = p.Product_Id
join department d
on p.Department_Id = d.Id
where order_status in ('Closed', 'Complete') 
group by Shipping_Mode, Name
having ord_count > 40;
```
![WhatsApp Image 2025-01-17 at 21 56 49_733a7f9f](https://github.com/user-attachments/assets/9de87765-c6d2-4e36-88f3-7fd2cb9b1ae1)

Q4. Create a new field as shipment compliance based on Real_Shipping_Days and Scheduled_Shipping_Days. It should have the following values:
- Cancelled shipment - If the Order Status is SUSPECTED_FRAUD or CANCELED
- Within schedule - If shipped within the scheduled number of days 
- On time - If shipped exactly as per schedule
- Upto 2 days of delay - If shipped beyond schedule but delay upto 2 days
- Beyond 2 days of delay - If shipped beyond schedule with delay beyond 2 days<br>Which shipping mode was observed to have the greatest number of delayed orders?
```sql
with new_table as (
select order_id, Shipping_Mode, 
case 
	when Order_Status = 'SUSPECTED_FRAUD' or 'CANCELED' then 'Cancelled shipment'
    when Real_Shipping_Days = Scheduled_Shipping_Days then 'On time'
    when Real_Shipping_Days < Scheduled_Shipping_Days then 'Within Schedule'
    when Real_Shipping_Days - Scheduled_Shipping_Days <= 2 then 'Upto 2 days of delay'
    else 'Beyond 2 days of delay '
end as shipment_compliance
from orders
)
select count(order_id) as ord_count, Shipping_Mode 
from new_table
where shipment_compliance = 'Upto 2 days of delay' or 'Beyond 2 days of delay' 
group by Shipping_Mode
order by ord_count desc;
```
![WhatsApp Image 2025-01-17 at 21 57 09_c9efbec7](https://github.com/user-attachments/assets/10d1936c-c28b-42f7-8cb7-71518e74a163)


Q5. An order is cancelled when the status of the order is either cancelled or SUSPECTED_FRAUD. Obtain the list of states by the order cancellation % and sort them in the descending order of the cancellation %.<br> 
Definition: Cancellation % = Cancelled order / Total Orders.
```sql
WITH cte AS(
SELECT order_state, COUNT(order_id) AS num_canclled_ord
FROM orders
WHERE Order_Status='SUSPECTED_FRAUD' OR Order_Status='CANCELED'
GROUP BY order_state    
)
SELECT order_state, (num_canclled_ord/total_orders)*100 AS 'Cancellation percentage'
FROM cte 
JOIN (
SELECT order_state, COUNT(order_id) as total_orders
FROM orders 
GROUP BY order_state) T
USING (order_state)
ORDER BY 'Cancellation percentage' DESC;
```
![WhatsApp Image 2025-01-17 at 21 58 13_9ecd50cc](https://github.com/user-attachments/assets/c112e514-2f05-4fb2-9d7f-905b4f390880)
![WhatsApp Image 2025-01-17 at 21 58 17_0a1ee2b2](https://github.com/user-attachments/assets/e5fa94fd-49c3-474f-b4fa-7c209733c823)


