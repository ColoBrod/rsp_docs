### Account Overview

#### Issues

##### UNASSIGNED

function tep_count_unassigned_orders() in general.php

~~~ sql
select o.order_id, o.date_schedualed, o.order_total, ot.name as order_type_name, o.order_status_id, os.order_status_name, a.house_number, a.street_name, a.city, o.order_issue from orders o, order_types ot, orders_statuses os, addresses a, users u where  o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id > 0 and o.order_status_id != '3' and o.order_status_id != '4' and o.address_id = a.address_id
~~~

Selects all orders, where order_status_id more than 0 (table 'orders') and order_status_id is not 3 and order_status_id is not 4

There is no 'count' in SQL-query. After executing SQL, counts number of result row inside of a function.

##### RED FLAG
account_overview.php || admin_service_stats (also exists pretty much the same code)

Selects all orders from these 5 tables:
orders, addresses, order_types, orders_statuses, users

where order_issue = 1 (table 'orders')

~~~ sql
select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.address_id = a.address_id and o.order_issue = '1'
~~~

##### ON HOLD

Selects all orders from these 5 tables:
orders, addresses, order_types, orders_statuses, users

where order_status_id = 5 (table 'orders')

~~~ sql
select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id = '5' and o.address_id = a.address_id
~~~

#### Miss Unity

##### Open:       

~~~ sql
select count(o.order_id) as count from orders o left join orders_miss_utility omu on (o.order_id = omu.order_id) where o.order_status_id < 3 and omu.contacted = 0 and not (omu.agent_requested = 0 and (omu.has_gas_lamp = 0 or omu.has_lamp = 0))
~~~

Selects all orders, where orders_status_id < 3 and orders_miss_utility.contacted = 0 and not (orders_miss_utility.agent_requested = 0 (orders_miss_utility.has_gas_lamp = 0 or orders_miss_utility.has_lamp = 0))

##### Called:     

~~~ sql 
select count(o.order_id) as count 
from orders o 
left join orders_miss_utility omu on (o.order_id = omu.order_id) 
where o.order_status_id < 3 and omu.contacted = 1 
and not (omu.agent_requested = 0 and (omu.has_gas_lamp = 0 or omu.has_lamp = 0))
~~~

Selects orders, where order_status_id < 3 and orders_miss_utility.contacted = 1
and not (orders_miss_utility.agent_requested = 0 and (orders_miss_utility.has_gas_lamp = 0 or orders_miss_utility.has_lamp = 0) )

##### Completed:  

~~~ sql
select count(o.order_id) as count from orders o left join orders_miss_utility omu on (o.order_id = omu.order_id) where o.order_status_id = 3 and omu.contacted = 1 and not (omu.agent_requested = 0 and (omu.has_gas_lamp = 0 or omu.has_lamp = 0))
~~~

Selects orders, where orders.order_status_id = 3 and orders_miss_utility = 1 and not (orders_miss_utility.agent_requested = 0 and (orders_miss_utility.has_gas_lamp = 0  or orders_miss_utility.has_lamp = 0) )

##### Percentage: 

What is bgdn???

~~~ sql
select count(order_miss_utility_id) as count from orders_miss_utility
~~~

$miss_utility_all_bgdn = 13884

~~~ sql
select count(order_id) as count from orders WHERE order_id > 109892
~~~

$orders_all_bgdn = 78393

Percentage is get via this formula:
(13884 * 100) / 78393 = 17.71%

some useless code, that doesn't do anything useful?! (I have to review code one more time from line 706)

#### ORDERED TODAY:

##### Installs

~~~ sql
select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.address_id = a.address_id and o.order_type_id = '1' and o.date_added > 0 and o.date_added >= NOW()
~~~

##### Removals

~~~ sql
select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.address_id = a.address_id and o.order_type_id = '3' and o.date_added > 0 and o.date_added >= NOW()
~~~

##### Service Calls

~~~ sql
select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.address_id = a.address_id and o.order_type_id = '2' and o.date_added > 0 and o.date_added >= NOW()
~~~

#### RESCHEDULED TODAY:

##### Removals Rescheduled

~~~ sql
SELECT o.order_id, rh.new_scheduled_date, rh.old_scheduled_date FROM reschedule_history rh JOIN orders o ON (o.order_id = rh.order_id) WHERE rh.rescheduled_date >= 1569013200 AND o.order_type_id = 3 ORDER BY o.order_id, rh.rescheduled_date
~~~

Instead of 1569013200 we use 'midnight' built-in - strtotime("midnight")

##### Pushed Back: TODO (798 - 807 in account_overview.php)
##### Moved Up:    TODO

#### Post Total Change for Yesterday:

It takes the time for Yesterday between 00:00:01 and 23:59:59

~~~ sql
select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id = '3' and o.address_id = a.address_id and o.date_completed >= 1568926801 and o.date_completed < 1569013199 and o.order_type_id = '1'
~~~

#### Post Total Change for Last Week:

Pretty much the same as previous, but for DateStartLastWeek:
strtotime('midnight - 6days 00:00:01')
And for the DateEndLastWeek:
strtotime('today 23:59:59')

#### Overdue Orders: 

##### Pending:

SQL-query takes the time, that was exactly 2 days ago.

~~~ sql
select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id = '1' and o.address_id = a.address_id and o.date_schedualed > 0 and o.date_schedualed < 1569045051
~~~

##### Scheduled:  

This SQL-query is absolutely the same as previous one, except orders.order_status_id = 2

~~~ sql
select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id = '2' and o.address_id = a.address_id and o.date_schedualed > 0 and o.date_schedualed < 1569045051
~~~

#### Current Active Orders:

##### Pending (before today + 2):

###### Installs

Counts orders where orders.order_status_id = '1' and orders.order_type_id = '1'.
Only orders until tomorrow midnight

~~~ sql
select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id = '1' and o.address_id = a.address_id and o.date_schedualed < '1569099600' and o.order_type_id = '1'
~~~

Replace 1569099600 with strtotime('midnight +1 days')

###### Removals

Counts orders where orders.order_status_id = '1' and orders.order_type_id = '3'
Only orders until tomorrow midnight

~~~ sql
select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id = '1' and o.address_id = a.address_id and o.date_schedualed < '1569099600' and o.order_type_id = '3'
~~~

###### Service Calls

Counts orders where orders.order_status_id = '1' and orders.order_type_id = '2'
Only orders until tomorrow midnight

~~~ sql
select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id = '1' and o.address_id = a.address_id and o.date_schedualed < '1569099600' and o.order_type_id = '2'
~~~

###### Total

Just sums Installs, Removals, Service Calls

##### Schedule (before today + 1):

The time is before now! Not like written???

###### Installs

Counts all orders before now where orders.order_status_id = '2' and orders.order_type_id = '1'

~~~ sql
select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id = '2' and o.address_id = a.address_id and o.date_schedualed < '1569058378' and o.order_type_id = '1'
~~~

###### Removals

Counts all orders before now where orders.order_status_id = '2' and orders.order_type_id = '3'

~~~ sql
select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id = '2' and o.address_id = a.address_id and o.date_schedualed < '1569058378' and o.order_type_id = '3'
~~~

###### Service Calls

Counts all orders before now where orders.order_status_id = '2' and orders.order_type_id = '2'

~~~ sql
select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id = '2' and o.address_id = a.address_id and o.date_schedualed < '1569058378' and o.order_type_id = '2'
~~~

###### Total

Just sums Install, Removals and Service Calls

#### Completed Orders:

In account_overview.php the order is opposite. Today is at the and Yesterday is under Today.

> Completed Yesterday:

date_completed (table 'orders') is between the begining of yesterday and the begining of today (midnights)

>> Installs:

  Counts all orders, where order_status_id = '3' and order_type_id = '1' (table 'orders')

  select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id = '3' and o.address_id = a.address_id and o.date_completed >= '1568926800' and o.date_completed < '1569013200' and o.order_type_id = '1'

>> Removals:

  Counts all orders, where order_status_id = '3' and order_type_id = '3' (table 'orders')

  select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id = '3' and o.address_id = a.address_id and o.date_completed >= '1568926800' and o.date_completed < '1569013200' and o.order_type_id = '3'

>> Service Calls:

  Counts all orders, where order_status_id = '3' and order_type_id = '2' (table 'orders')

  select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id = '3' and o.address_id = a.address_id and o.date_completed >= '1568926800' and o.date_completed < '1569013200' and o.order_type_id = '2'

>> Total:

  Just sums Installs, Removals and Service Calls.

> Completed Today:

orders.date_completed from today's midnight and to this moment (now)

>> Installs:
  
  Counts all orders, where order_status_id = '3' (table 'orders') and order_type_id = '1' (table 'orders')

  select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id = '3' and o.address_id = a.address_id and o.date_completed >= '1569013200' and o.date_completed < '1569058954' and o.order_type_id = '1'

>> Removals:

  Counts all orders, where order_status_id = '3' and order_type_id = '3' (table 'orders' both)

  select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id = '3' and o.address_id = a.address_id and o.date_completed >= '1569013200' and o.date_completed < '1569058954' and o.order_type_id = '3'

>> Service Calls:

  Counts all orders, where order_status_id = '3' and order_type_id = '2' (table 'orders' both)

  select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id = '3' and o.address_id = a.address_id and o.date_completed >= '1569013200' and o.date_completed < '1569058954' and o.order_type_id = '2'

>> Total:
  Just sums Installs, Removals and Service Calls

#### Future Orders:




#### Agents to be made Inactive:


#### Agencies to be made Inactive:


#### Posts in the Field: FFX, MD, PA, Total:


#### Total Operational Posts (Installed + Pending + Scheduled + Available):


#### Pending/Scheduled Removals:


#### New/Active Agencies:


#### MONEY STATISTICS:


#### Current Year:



#### Previous Year (Completed):



#### Installer Information:



































