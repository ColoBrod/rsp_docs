#  class Issues

#### Issues::$unassigned
:memo: Selects all orders, where order_status_id more than 0 (table 'orders') and order_status_id is not 3
There is no 'count' in SQL-query. After executing SQL, counts number of result row inside of a function.
~~~ sql
select o.order_id, o.date_schedualed, o.order_total, ot.name as order_type_name, o.order_status_id, os.order_status_name, a.house_number, a.street_name, a.city, o.order_issue from orders o, order_types ot, orders_statuses os, addresses a, users u where  o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id > 0 and o.order_status_id != '3' and o.order_status_id != '4' and o.address_id = a.address_id
~~~
:exclamation: function tep_count_unassigned_orders() in general.php
#### Issues::$redFlag
:memo: Selects all orders from these 5 tables:
orders, addresses, order_types, orders_statuses, users
where order_issue = 1 (table 'orders')  
~~~ sql
select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.address_id = a.address_id and o.order_issue = '1'
~~~
:exclamation: account_overview.php || admin_service_stats (also exists pretty much the same code)

#### Issues::$onHold
:memo: Selects all orders from these 5 tables:
orders, addresses, order_types, orders_statuses, users
where order_status_id = 5 (table 'orders')
~~~ sql
select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id = '5' and o.address_id = a.address_id
~~~

# class MissUtility

#### MissUtility::$open
:memo: Selects all orders, where orders_status_id < 3 and orders_miss_utility.contacted = 0 and not (orders_miss_utility.agent_requested = 0 (orders_miss_utility.has_gas_lamp = 0 or orders_miss_utility.has_lamp = 0))
~~~ sql
select count(o.order_id) as count from orders o left join orders_miss_utility omu on (o.order_id = omu.order_id) where o.order_status_id < 3 and omu.contacted = 0 and not (omu.agent_requested = 0 and (omu.has_gas_lamp = 0 or omu.has_lamp = 0))
~~~

#### MissUtility::$called
:memo: Selects orders, where order_status_id < 3 and orders_miss_utility.contacted = 1
and not (orders_miss_utility.agent_requested = 0 and (orders_miss_utility.has_gas_lamp = 0 or orders_miss_utility.has_lamp = 0) )
~~~ sql 
select count(o.order_id) as count 
from orders o 
left join orders_miss_utility omu on (o.order_id = omu.order_id) 
where o.order_status_id < 3 and omu.contacted = 1 
and not (omu.agent_requested = 0 and (omu.has_gas_lamp = 0 or omu.has_lamp = 0))
~~~

#### MissUtility::$completed
:memo: Selects orders, where orders.order_status_id = 3 and orders_miss_utility = 1 and not (orders_miss_utility.agent_requested = 0 and (orders_miss_utility.has_gas_lamp = 0  or orders_miss_utility.has_lamp = 0) )
~~~ sql
select count(o.order_id) as count from orders o left join orders_miss_utility omu on (o.order_id = omu.order_id) where o.order_status_id = 3 and omu.contacted = 1 and not (omu.agent_requested = 0 and (omu.has_gas_lamp = 0 or omu.has_lamp = 0))
~~~

#### MissUtility::$percentage
:memo: This property is counted in private method MissUtility::calcPercentage()
~~~ sql
select count(order_miss_utility_id) as count from orders_miss_utility
~~~
In following SQL-query statement order_id > 109892 is used just in order to avoid all old orders - that were added before 2012 approximately. I used pretty much same statement ()
~~~ sql
select count(order_id) as count from orders WHERE order_id > 109892
~~~
:exclamation: What is bgdn???



$orders_all_bgdn = 78393

Percentage is get via this formula:
(13884 * 100) / 78393 = 17.71%

some useless code, that doesn't do anything useful?! (I have to review code one more time from line 706)

# ORDERED TODAY:

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

# RESCHEDULED TODAY:

##### Removals Rescheduled

~~~ sql
SELECT o.order_id, rh.new_scheduled_date, rh.old_scheduled_date FROM reschedule_history rh JOIN orders o ON (o.order_id = rh.order_id) WHERE rh.rescheduled_date >= 1569013200 AND o.order_type_id = 3 ORDER BY o.order_id, rh.rescheduled_date
~~~

Instead of 1569013200 we use 'midnight' built-in - strtotime("midnight")

##### Pushed Back: TODO (798 - 807 in account_overview.php)
##### Moved Up:    TODO

# Post Total Change for Yesterday:

It takes the time for Yesterday between 00:00:01 and 23:59:59

~~~ sql
select count(o.order_id) as count from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id = '3' and o.address_id = a.address_id and o.date_completed >= 1568926801 and o.date_completed < 1569013199 and o.order_type_id = '1'
~~~

# Post Total Change for Last Week:

Pretty much the same as previous, but for DateStartLastWeek:
strtotime('midnight - 6days 00:00:01')
And for the DateEndLastWeek:
strtotime('today 23:59:59')

# Overdue Orders: 

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

# Current Active Orders:

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

# Completed Orders:

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

# Future Orders:




Agents to be made Inactive:
=============================

Agencies to be made Inactive:
=============================



Posts in the Field:
===================

__FFX:__
__MD:__
__PA:__
__Total:__

Total Operational Posts:
========================

__Installed:__
__Pending:__
__Scheduled:__
__Available:__

Pending/Scheduled Removals:
===========================

New/Active Agencies:
====================

MONEY STATISTICS:
=================

Current Year:
=============

### Today (placed):

__Time:__
Counts orders from the beginning of current day.

__# of Installs:__
__$ value of orders placed today:__

Both are received from 1 SQL-statement.

Counts orders where order_status_id IS NOT '4' (Cancelled) and order_type_id = '1' (table 'orders'), Sums the column 'order_total' as value.

_SQL example:_
~~~sql
select count(o.order_id) as count, sum(order_total) as value from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id != '4' and o.address_id = a.address_id and o.order_type_id = '1' and o.date_added > 0 and o.date_added >= '1569470400'
~~~

_Variables:_
~~~php
['today']['number_of_installs'] => $count_install_today
['today']['value_of_orders_placed'] => number_format($value_install_today,2)
~~~

__$ value / # of installs:__

Divides "Value of installs" / "Number of installs".

_Variables:_
~~~php
['today']['value_number_of_installs'] => ($count_install_today>0 ? number_format(($value_install_today/$count_install_today),2) : "0.00")
~~~
__% of CC Orders__

Counts orders where order_status_id IS NOT '4' (Cancelled) and order_type_id = '1' and billing_method_id = '1' (table 'orders')
Sums the column 'order_total'

_SQL example:_
~~~sql
select count(o.order_id) as count, sum(order_total) as value from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id != '4' and o.address_id = a.address_id and o.order_type_id = '1' and o.billing_method_id = '1' and o.date_added > 0 and o.date_added >= '1569470400'
~~~

_Variables:_
~~~php
$countCC, $valueCC
['today']['amount_of_cc_orders'] => number_format($valueCC,2)
~~~

__% of Invoice Orders:__

Counts orders where order_status_id IS NOT '4' (Cancelled) and order_type_id = '1' and billing_method_id IS '2' OR '3'
Sums the column 'order_total'

_SQL example:_
~~~sql
select count(o.order_id) as count, sum(order_total) as value from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id != '4' and o.address_id = a.address_id and o.order_type_id = '1' and o.billing_method_id IN (2,3) and o.date_added > 0 and o.date_added >= '1569470400'
~~~

_Variables:_
~~~php
$countIO, $valueIO
['today']['value_of_invoice_orders'] => number_format($valueIO,2)
~~~

### Month (placed):

Counts orders from the beginning of month (e.g. 01-09-2019 00:00:00) and until the beginning of tomorrow (e.g. 27:09:19 00:00:00).

__# of Installs:__
__$ value of orders placed this month:__

Both are received from 1 SQL-statement.
Counts orders where order_status_id IS NOT '4' ('Cancelled') AND order_type_id = '1' _(Thats the Count of orders)_
Sums the column 'order_total' _(Thats the Value of orders)_

_SQL example:_
~~~sql
select count(o.order_id) as count, sum(order_total) as value from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id != '4' and o.order_status_id = os.order_status_id and o.address_id = a.address_id and o.order_type_id = '1' and o.date_added > 0 and o.date_added >= '1567310400' and o.date_added < '1569556800'
~~~

_Variables:_
~~~php
$count_install_month
['month']['placed']['number_of_installs'] => $count_install_month
$value_install_month
['month']['placed']['value_of_orders_placed'] => number_format($value_install_month,2)
~~~

__$ value / # of installs:__

Simply divides "Value of installs" / "Count of installs"

_Variables:_
~~~php
['month']['placed']['value_number_of_installs'] => ($count_install_month>0 ? number_format(($value_install_month/$count_install_month),2) : "0.00")
~~~

### Month (completed):

Counts orders between the month first date (e.g. 01-09-19 00:00:00) and the beginning of tomorrow (e.g. 27-09-19 00:00:00). 

__# of Installs:__
__$ value of orders completed this month:__

Both values are received from 1 SQL-statement.

Where order_type_id = '1' and order_status_id = '3'
Sums the column 'order_total'

_SQL example:_
~~~sql
select count(o.order_id) as count, sum(order_total) as value from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.address_id = a.address_id and o.order_type_id = '1' and o.order_status_id = '3' and o.date_completed > 0 and o.date_completed >= '1567310400' and o.date_completed < '1569556800'
~~~

_Variables:_
~~~php
$this_month_complete_count, $this_month_complete_value
['month']['completed']['number_of_installs'] => $this_month_complete_count
['month']['completed']['value_of_orders_placed'] => number_format($this_month_complete_value,2)
~~~

__$ value / # of installs:__

_Variables:_
~~~php
['month']['completed']['value_number_of_installs'] => ($this_month_complete_count>0 ? number_format(($this_month_complete_value/$this_month_complete_count),2) : "0.00")
~~~

__% of CC Orders:__

Following SQLs are pretty much the same as the one above but for CC (Credit Cards)
[sum(order_total) in SQL statement is not used!](). So, I think we can remove it from SQL query.

_SQL example:_
~~~sql
select count(o.order_id) as count, sum(order_total) as value from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id != '4' and o.order_status_id = '3' and o.address_id = a.address_id and o.order_type_id = '1' and o.billing_method_id = '1' and o.date_added > 0 and o.date_added >= '1567310400' and o.date_added < '1569556800'
~~~

_Variables:_
~~~php
$count_all_stuff = $countIO_month+$countCC_month;
$countCC_percentage_month = $count_all_stuff
  ? ($countCC_month * 100) / $count_all_stuff
  : 0;
['month']['completed']['amount_of_cc_orders'] => number_format($countCC_percentage_month,2)
~~~

__% of Invoice Orders:__

_SQL example:_
~~~sql
select count(o.order_id) as count, sum(order_total) as value from orders o, addresses a , order_types ot, orders_statuses os, users u where o.order_type_id = ot.order_type_id and o.user_id = u.user_id and o.order_status_id = os.order_status_id and o.order_status_id != '4' and o.order_status_id = '3' and o.address_id = a.address_id and o.order_type_id = '1' and o.billing_method_id IN (2,3) and o.date_added > 0 and o.date_added >= '1567310400' and o.date_added < '1569556800'
~~~

_Variables:_
~~~php
($countIO_month==0) ? $countIO_percentage_month = 0 : $countIO_percentage_month = 100-$countCC_percentage_month;
['month']['completed']['value_of_invoice_orders'] => number_format($countIO_percentage_month,2)
~~~

### YTD (completed):	
__# of Installs:__
__$ value of orders completed from Jan 1:__
__$ value / # of installs:__
__% of CC Orders:__
__% of Invoice Orders:__

### Previous Month (completed):	
__# of Installs:__
__$ value of orders completed previous month:__
__$ value / # of installs:__
__% of CC Orders:__
__% of Invoice Orders:__

Previous Year (Completed):
==========================

### Month:	
__# of Installs:__
__$ value of orders placed this month:__
__$ value / # of installs:__
__% of CC Orders__
__% of Invoice Orders:__

### YTD:	
__# of Installs:__
__$ value of orders placed from Jan 1:__
__$ value / # of installs:__
__% of CC Orders:__
__% of Invoice Orders:__

### Full Year:	
__# of Installs:__
__$ value of orders placed from Jan 1:__
__$ value / # of installs:__
__% of CC Orders:__
__% of Invoice Orders:__

Installer Information:
======================

### Installer Name
### Last Login
### Overdue Pendings
### Overdue Scheduled



































<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzNDQ4MDU4NjMsLTEyNTAzMzA4ODZdfQ
==
-->