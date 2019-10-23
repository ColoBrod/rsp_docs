# Table of contents
[Instructions for updating production site](#instructions-for-updating-production-site)  
[Issues](#class-issues)  
[Miss Utility](#class-missutility)  
[ORDERED TODAY](#class-orderedtoday)  
[RESCHEDULED TODAY](#class-rescheduledtoday)  
[Post Total Change for Yesterday](#class-posttotalchange)  
[Post Total Change for Last Week](#class-posttotalchange)  
[Overdue Orders](#class-overdueorders)  
[Current Active Orders](#class-currentactiveorders)  
[Completed Orders](#class-completedorders)  
[Future Orders](#class-futureorders)  
[Agents to be made Inactive](#class-inventorysummary)  
[Agencies to be made Inactive](#class-inventorysummary)  
[Posts in the Field](#class-inventorysummary)  
[Total Operational Posts (Installed + Pending + Scheduled + Available)](#class-inventorysummary)  
[Pending/Scheduled Removals](#class-inventorysummary)  
[New/Active Agencies](#class-inventorysummary)  
[MONEY STATISTICS](#class-moneystatistics)  
[Current Year](#class-currentyear)    
[Previous Year (Completed)](#class-previousyear)    


# Instructions for updating production site

### `public_html/includes/configure.php`
 1. Change the path to folder `public_html`:
`define('DIW_FS', '/home/realtysp/web/realtysignpost.com/public_html/');`
2. Change the `BASE_URL`:
~~~php
if (isset($_SERVER['HTTP_HOST'])) {
	define('BASE_URL', "https://"  . (substr_count($_SERVER['HTTP_HOST'], 'www.') ? 'www.' : '') .  "realtysignpost.com");
} 
else {
	// There is no HTTP_HOST on CLI
	define('BASE_URL', 'https://www.realtysignpost.com');
}
~~~
3. Change the timezone for PHP:
~~~php
// Sets the EDT (Eastern daylight Time)
date_default_timezone_set("America/New_York");
~~~
4. Change MySQL settings:
~~~php
define('DB_SERVER', 'SECRET');
define('DB_SERVER_USERNAME', 'SECRET');
define('DB_SERVER_PASSWORD', 'SECRET');
define('DB_DATABASE', 'SECRET');
~~~
:question: `ini_set('display_errors', false);`  
This line of code makes all errors on site invisible. So it makes a better look for clients, but we also can't see any problems, debug and fix them.
### Removing banner "TEST SITE" from the top
1. `includes/twig_templates/layout.html.twig`. Comment **\<span\>** tag at the top.
2. `includes/template/template.tpl`
3. `includes/template/index.tpl`  

### Disabling cron-emails
Set `$live` variable to true at following files:
1. `cron_email_removal_stat.php`
2. `cron_email_removal_stat.php5` 
:question: Seems these 2 files above do pretty much the same job and .php5 is obsolete. Can we remove it?
3. `cron_email_removal_stat`
4. `cron_emails_debug.php`
5. `cron_emails_resend.php`
6. `cron_emails_monthly_stat.php`
7.  `cron_emails.php`  
8. 
### .htaccess
1. Remove this line:
`SetEnv SERVER_MODE TEST`
### .htpasswd
Remove this file  
  
  
:question: I think, it would be better to add a variable to `configure.php` - `CFG::$productionMode` or something of this sort. Which can be either `true` of `false`. Banners, emails, displaying errors, etc will be enabled/disabled automatically, depending on it's state. This way we can simplify updating process and prevent some errors.

#  class Issues
:exclamation: Original functions of `Issues::$onHold` and `Issues::$redFlag` were located in `account_overview.php` or in `admin_service_stats` (were 2 pretty much same pieces of code).

#### Issues::$unassigned
:memo: Selects all orders from tables 'orders' and 'installers_to_orders', where order_status_id is 1,2 or 5 (i.e. Pending, Scheduled, On Hold)
There is no 'count' in SQL-query. 
~~~ sql
SELECT o.order_id FROM orders o
LEFT JOIN ".TABLE_INSTALLERS_TO_ORDERS." ito ON (ito.order_id = o.order_id)
WHERE
o.date_schedualed > ".Stats::$relevantDate." and
o.order_status_id > 0 AND
o.order_status_id != '3' AND
o.order_status_id != '4' AND
ito.installer_id IS NULL
~~~
After previous SQL-query we have a table (list of orders). We iterate over each row in a table and filter through function `Issues::fetchAssignedOrderInstaller($order_id)`. The algorithm of this function is a little bit complex, in a nutshell it checks if the order (with `$order_id`) has an `assigned_installer` or `default_installer`. If it hasn't, returns false and we increase `Issues::$unassigned` by 1.  
:exclamation: the primordial `function tep_count_unassigned_orders()` is located in `general.php`. 
#### Issues::$redFlag
:memo: Counts orders, which have `order_issue = '1'` (table 'orders').

~~~sql
SELECT count(o.order_id) as count FROM ".TABLE_ORDERS." o
WHERE o.order_issue = '1'
~~~

#### Issues::$onHold
:memo: Counts orders, which have `order_issue = '1'` (table 'orders').
where order_status_id = 5 (table 'orders')
~~~sql
SELECT count(o.order_id) as count FROM ".TABLE_ORDERS." o
WHERE o.order_status_id = '5'
~~~

# class MissUtility
~~~php
// Following values received via pretty much the same SQL-query
// The difference is in variable $condition

// o.order_status_id < 3 AND omu.contacted = 0
MissUtility::$open			
// o.order_status_id < 3 AND omu.contacted = 1
MissUtility::$called		
// o.order_status_id = 3 AND omu.contacted = 1
MissUtility::$completed		
~~~
:memo: Counts orders from tables `orders` and `orders_miss_utility` where `agent_requested` is not 0 and both  `has_gas_lamp` or `has_lamp` are not 0 (table `orders_miss_utility`)
~~~sql
SELECT count(o.order_id) AS count
FROM ".TABLE_ORDERS." o
LEFT JOIN ".TABLE_ORDERS_MISS_UTILITY." omu ON (o.order_id = omu.order_id)
WHERE $condition
	AND NOT (
		omu.agent_requested = 0 
		AND (omu.has_gas_lamp = 0 OR omu.has_lamp = 0)
	)
~~~

#### MissUtility::$open
:memo: Selects all orders, where orders_status_id < 3 and orders_miss_utility.contacted = 0 and not (orders_miss_utility.agent_requested = 0 (orders_miss_utility.has_gas_lamp = 0 or orders_miss_utility.has_lamp = 0))
~~~ sql
SELECT count(o.order_id) AS count
FROM ".TABLE_ORDERS." o
LEFT JOIN ".TABLE_ORDERS_MISS_UTILITY." omu ON (o.order_id = omu.order_id)
WHERE o.order_status_id < 3 AND omu.contacted = 1
	AND NOT (omu.agent_requested = 0 AND (omu.has_gas_lamp = 0 OR omu.has_lamp = 0))
~~~

#### MissUtility::$called
:memo: Selects orders, where order_status_id < 3 and orders_miss_utility.contacted = 1
and not (orders_miss_utility.agent_requested = 0 and (orders_miss_utility.has_gas_lamp = 0 or orders_miss_utility.has_lamp = 0) )
~~~ sql
SELECT count(o.order_id) AS count
FROM ".TABLE_ORDERS." o
LEFT JOIN ".TABLE_ORDERS_MISS_UTILITY." omu ON (o.order_id = omu.order_id)
WHERE o.order_status_id < 3 AND omu.contacted = 1
	AND NOT (omu.agent_requested = 0 AND (omu.has_gas_lamp = 0 OR omu.has_lamp = 0))
~~~

#### MissUtility::$completed
:memo: Selects orders, where orders.order_status_id = 3 and orders_miss_utility = 1 and not (orders_miss_utility.agent_requested = 0 and (orders_miss_utility.has_gas_lamp = 0  or orders_miss_utility.has_lamp = 0) )
~~~ sql
SELECT count(o.order_id) AS count
FROM ".TABLE_ORDERS." o
LEFT JOIN ".TABLE_ORDERS_MISS_UTILITY." omu ON (o.order_id = omu.order_id)
WHERE o.order_status_id = 3 AND omu.contacted = 1
	AND NOT (omu.agent_requested = 0 AND (omu.has_gas_lamp = 0 OR omu.has_lamp = 0))
~~~

#### MissUtility::$percentage
:memo: This property is counted in the private method: `MissUtility::calcPercentage()`.   
First, counts all orders from the table `orders_miss_utility` and stores to `$allBgdn`. Than, counts all orders from table `orders` and stores to `$ordersAllBgdn`. The percentage is `($allBgdn*100)/$ordersAllBgdn`
~~~php
private  static  function  calcPercentage() {
	$result = self::query("SELECT  count(order_miss_utility_id) AS  count  FROM  ".TABLE_ORDERS_MISS_UTILITY);
	$allBgdn = ($result['count'] > 0) ? $result['count'] : 0;
	$result = self::query("SELECT  count(order_id) AS  count  FROM  ". TABLE_ORDERS ." WHERE order_id > 109892");
	$ordersAllBgdn = ($result['count'] > 0) ? $result['count'] : 0;
	return ($allBgdn*100)/$ordersAllBgdn;
}
~~~
In following SQL-query the condition `order_id > 109892` is used just in order to avoid all old orders - that were added before 2012 approximately. (I use pretty the much same condition in many cases: `o.date_added > '".Stats::\$relevantDate."'` where `Stats::$relevantDate` is a timestamp, that represents January 1 2012)
~~~ sql
SELECT count(order_id) AS count FROM ".TABLE_ORDERS." WHERE order_id > 109892
~~~
:question: What does **bgdn** stand for? I saved original variable names (converted to camel case), but I don't understand this abbreviation.

# class OrderedToday:
All these values - `OrderedToday::$installs`,  `OrderedToday::$removals`, `OrderedToday::$serviceCalls` - are derived from pretty much the same SQL-query:
~~~sql
SELECT count(o.order_id) AS count FROM ".TABLE_ORDERS." o
WHERE o.order_type_id = '$ordered_type_id' AND o.date_added >= '".self::$today."'
~~~
The difference is only in `$order_type_id`. The SQL above counts all orders from table `orders`, that were added since the beginning of today where `order_type_id = 1 or 2 or 3`

#### OrderedToday::$installs
:memo: `order_type_id = '1'` (table 'orders')
~~~ sql
SELECT count(o.order_id) AS count FROM ".TABLE_ORDERS." o
WHERE o.order_type_id = '1' AND o.date_added >= '".self::$today."'
~~~

#### OrderedToday::$removals
:memo: `order_type_id = '3'` (table 'orders')
~~~ sql
SELECT count(o.order_id) AS count FROM ".TABLE_ORDERS." o
WHERE o.order_type_id = '3' AND o.date_added >= '".self::$today."'
~~~

#### OrderedToday::$serviceCalls
:memo: `order_type_id = '2'` (table 'orders')
~~~ sql
SELECT count(o.order_id) AS count FROM ".TABLE_ORDERS." o
WHERE o.order_type_id = '2' AND o.date_added >= '".self::$today."'
~~~

# class  RescheduledToday
:memo: First of all, we receive list of orders via following query:
~~~ sql
SELECT
	o.order_id,
	rh.new_scheduled_date,
	rh.old_scheduled_date
	FROM ".TABLE_RESCHEDULE_HISTORY." rh
	JOIN ".TABLE_ORDERS." o ON (o.order_id = rh.order_id)
WHERE
	rh.rescheduled_date >= ".self::$midnight." AND
	o.order_type_id =  3
ORDER BY
	o.order_id,
	rh.rescheduled_date
~~~
We select from 2 tables: `reschedule_history` and `orders`, only **completed orders** with `rescheduled_date` starting from the midnight.
#### RescheduledToday::$removalsRescheduled
Groups all rows in an SQL-result with same `$order_id`. Than Counts number of rows.
#### RescheduledToday::$pushedBack
Counts results where `old_scheduled_date < new_scheduled_date`
#### RescheduledToday::$movedUp
Counts results where `old_scheduled_date > new_scheduled_date`

# class PostTotalChange:
:memo: Algorithm is same for both values except the time period. I am describing only for Yesterday:

### PostTotalChange::forYesterday
:memo: It takes in account orders from Yesterday between 00:00:01 and 23:59:59.  
First we count `$completeInstall`, `order_status_id = 3` (i.e. 'Completed') and `order_type_id = 1`:
~~~sql
SELECT count(o.order_id) AS count FROM ".TABLE_ORDERS." o
WHERE o.order_status_id = '3'
AND o.order_type_id = '1'
AND o.date_completed >= '".self::$startYesterday."'
AND o.date_completed < '".self::$endYesterday."'
~~~
Second `$completeRemove`, `order_status_id = 3` (i.e. 'Completed') and `order_type_id = 3`:
~~~sql
SELECT count(o.order_id) AS count FROM ".TABLE_ORDERS." o
WHERE o.order_status_id = '3'
AND o.order_type_id = '3'
AND o.date_completed >= '".self::$startYesterday."'
AND o.date_completed < '".self::$endYesterday."'
~~~
And finally, the result is the subtraction: `$completeInstall - $completeRemove`


### PostTotalChange::forLastWeek
:heavy_exclamation_mark: IMPORTANT: It counts orders for last 7 days, not for last week.
~~~php
self::$startLastWeek = strtotime('midnight - 6days 00:00:01');
self::$endLastWeek = strtotime('today 23:59:59');
~~~
:memo: The algorithm is absolutely the same as in an example above.

# class OverdueOrders: 
:exclamation: I think there previously was an Issue in both `$pending` and `$scheduled`. because of a typo (coder used function `time(...)` instead of `mktime()`), This SQL was counting all orders until this moment, not until the moment that was 2 days ago at 12:00.
:memo: SQL-query takes the time between the `Stats::$relevantDate` (i.e. January 1 2012) and `Stats::$datePendingOverdue` (i.e. 2 days ago at 12:00).   
The only difference between 2 SQL-queries is in `order_status_id`. For `$pending` it is `order_status_id = '1'` and for `$scheduled` it is `order_status_id = '2'`.

#### OverdueOrders::$pending
~~~sql
SELECT count(o.order_id) AS count FROM ".TABLE_ORDERS." o
WHERE
	o.order_status_id = '1' AND
	o.date_schedualed > '".self::$relevantDate."' AND
	o.date_schedualed < '".self::$datePendingOverdue."'
~~~
#### OverdueOrders::$scheduled
:memo: This SQL-query is absolutely the same as previous one, except `orders.order_status_id = 2`
~~~ sql
SELECT count(o.order_id) AS count FROM ".TABLE_ORDERS." o
WHERE
	o.order_status_id = '2' and
	o.date_schedualed > '".self::$relevantDate."' and
	o.date_schedualed < '".self::$datePendingOverdue."'
~~~

# class CurrentActiveOrders:

### CurrentActiveOrders::$pending
:memo: Includes orders with `orders_status_id = '1'` (Which is 'Pending') between 1 Jan 2012 and tomorrow midnight.
#### CurrentActiveOrders::$pending->installs
:memo: Counts orders where orders.order_type_id = '1'.
Only orders until tomorrow midnight
~~~ sql
select count(o.order_id) as count from ".TABLE_ORDERS." o
where
	o.order_status_id = '1' and
	o.date_schedualed > '".self::$relevantDate."' and
	o.date_schedualed < '".self::$datePendingCurrentActivePending."' and
	o.order_type_id = '1'
~~~

#### CurrentActiveOrders::$pending->removals
:memo: Counts orders where orders.order_type_id = '3'
~~~ sql
select count(o.order_id) as count from ".TABLE_ORDERS." o
where
	o.order_status_id = '1' and
	o.date_schedualed > '".self::$relevantDate."' and
	o.date_schedualed < '".self::$datePendingCurrentActivePending."' and
	o.order_type_id = '3'
~~~
#### CurrentActiveOrders::$pending->serviceCalls
:memo: Counts orders where orders.order_type_id = '2'
~~~ sql
select count(o.order_id) as count from ".TABLE_ORDERS." o
where
	o.order_status_id = '1' and
	o.date_schedualed > '".self::$relevantDate."' and
	o.date_schedualed < '".self::$datePendingCurrentActivePending."' and
	o.order_type_id = '2'
~~~
#### CurrentActiveOrders::$pending->total
:memo: Just sums together 3 previous values (Installs, Removals, Service Calls).

### CurrentActiveOrders::$schedule
:memo: Includes orders with `orders_status_id = '2'` (Which is 'Scheduled') between 1 Jan 2012 and the current moment.

#### CurrentActiveOrders::$schedule->installs
:memo: Counts all orders before now where orders.order_status_id = '2' and orders.order_type_id = '1'
~~~sql
select count(o.order_id) as count from ".TABLE_ORDERS." o
where
	o.order_status_id = '2' and
	o.date_schedualed > '".self::$relevantDate."' and
	o.date_schedualed < '".self::$now."' and
	o.order_type_id = '1'
~~~

#### CurrentActiveOrders::$schedule->removals
:memo: Counts all orders before now where orders.order_status_id = '2' and orders.order_type_id = '3'
~~~sql
select count(o.order_id) as count from "  . TABLE_ORDERS .  " o
where
	o.order_status_id = '2' and
	o.date_schedualed > '".self::$relevantDate."' and
	o.date_schedualed < '".self::$now."' and
	o.order_type_id = '3'
~~~

#### CurrentActiveOrders::$schedule->serviceCalls
:memo: Counts all orders before now where orders.order_status_id = '2' and orders.order_type_id = '2'
~~~sql
select count(o.order_id) as count from ".TABLE_ORDERS." o
where
	o.order_status_id = '2' and
	o.date_schedualed > '".self::$relevantDate."' and
	o.date_schedualed < '".self::$now."' and
	o.order_type_id = '2'
~~~

#### CurrentActiveOrders::$schedule->total

:memo: Just sums Install, Removals and Service Calls

# class CompletedOrders:

### CompletedOrders::$completedYesterday

:memo: Includes orders with `date_completed` between yesterday midnight and today's midnight. `order_status_id = 3` ('Completed').  
  
~~~sql 
select count(o.order_id) as count from ".TABLE_ORDERS." o
where
	o.order_status_id = '3' and
	o.date_completed >= '".$start."' and
	o.date_completed < '".$end."' and
	o.order_type_id = '$order_type_id'
~~~

#### CompletedOrders::$completedYesterday->installs
- installs: `orderd_type_id = '1'`
#### CompletedOrders::$completedYesterday->removals
- removals: `orderd_type_id = '3'`
#### CompletedOrders::$completedYesterday->serviceCalls
- serviceCalls: `orderd_type_id = '2'`
#### CompletedOrders::$completedYesterday->total
Sums Installs, Removals and Service Calls

### CompletedOrders::$completedToday

:memo: Includes orders with `date_completed` between today's midnight and now. `order_status_id = 3` ('Completed').  

~~~sql 
select count(o.order_id) as count from ".TABLE_ORDERS." o
where
	o.order_status_id = '3' and
	o.date_completed >= '".$start."' and
	o.date_completed < '".$end."' and
	o.order_type_id = '$order_type_id'
~~~
#### CompletedOrders::$completedToday->installs
- installs: `orderd_type_id = '1'`
#### CompletedOrders::$completedToday->removals
- removals: `orderd_type_id = '3'`
#### CompletedOrders::$completedToday->serviceCalls
- serviceCalls: `orderd_type_id = '2'`
#### CompletedOrders::$completedToday->total
Sums Installs, Removals and Service Calls

# class FutureOrders:
### FutureOrders::$pending->installs
### FutureOrders::$pending->removals
### FutureOrders::$pending->serviceCalls
### FutureOrders::$pending->total
:memo: Counts all orders where `order_status_id = 1` (i.e. 'Pending'), `date_scheduled` is more than the day after tomorrow midnight (current midnight + 2 days) and `order_type_id`:
~~~php
FutureOrders::$pending->installs;		// order_type_id = 1
FutureOrders::$pending->removals;		// order_type_id = 3
FutureOrders::$pending->serviceCalls;	// order_type_id = 2
~~~
:memo: `total` just sums these 3 values
### FutureOrders::$schedule->installs
### FutureOrders::$schedule->removals
### FutureOrders::$schedule->serviceCalls
### FutureOrders::$schedule->total
:memo: Counts all orders where `order_status_id = 2` (i.e. 'Scheduled'), `date_scheduled` is more than tomorrow midnight (current midnight + 1 day) and `order_type_id`:
~~~php
FutureOrders::$schedule->installs;		// order_type_id = 1
FutureOrders::$schedule->removals;		// order_type_id = 3
FutureOrders::$schedule->serviceCalls;	// order_type_id = 2
~~~
:memo: `total` just sums these 3 values
# class InventorySummary
### InventorySummary::$agentsToBeMadeInactive
### InventorySummary::$agenciesToBeMadeInactive
### InventorySummary::$postsInTheField->ffx
:memo: FFX and 4 next values are received from `lib/inventory/inventory.json.php5`. Param: `?summary=1`
~~~php
->ffx_posts_installed	=> $fairfax_posts_installed
->md_posts_installed	=> $md_posts_installed
->pa_posts_installed	=> $pa_posts_installed
->posts_avail 			=> $fairfax_posts_avail
->posts_total			=> $posts_total
~~~
:exclamation: I didn't test these 5 values. Moreover, I always had zeros. I'm just describing what the code seems to do.
:memo: First of all, we count total number of equipment items:
~~~sql
SELECT 
	e.equipment_id, 
	e.equipment_type_id, 
	e.name, 
	e.inventory_ruleset_id, 
	count( ei.equipment_item_id ) AS  count  
FROM  ".TABLE_EQUIPMENT_ITEMS. " ei 
JOIN "  . TABLE_EQUIPMENT .  " e ON ( e.equipment_id = ei.equipment_id ) 
WHERE e.equipment_type_id = 1 
GROUP BY 
	e.equipment_id, 
	e.equipment_type_id, e.name
~~~
:memo: Than, this SQL-query counts available equipment items. 
~~~sql
SELECT 
	e.equipment_id, count( ei.equipment_item_id ) AS  count  
FROM ".TABLE_EQUIPMENT_ITEMS." ei 
JOIN ".TABLE_EQUIPMENT." e ON ( e.equipment_id = ei.equipment_id ) 
WHERE 
	ei.equipment_status_id = 0 AND 
	e.equipment_type_id = 1 
GROUP BY 
	e.equipment_id
~~~
:memo: Latest activity date:
~~~sql
SELECT 
	ei.equipment_id,
	DATE_FORMAT(DATE(FROM_UNIXTIME(MAX(eih.date_added) -  18000)),'%c/%e/%Y') AS last_activity_date 
FROM  ".TABLE_EQUIPMENT_ITEMS_HISTORY." eih 
JOIN ".TABLE_EQUIPMENT_ITEMS." ei ON ( ei.equipment_item_id = eih.equipment_item_id ) 
JOIN ".TABLE_EQUIPMENT." e ON (e.equipment_id = ei.equipment_id) WHERE e.equipment_type_id = 1 
GROUP BY e.equipment_id
~~~
:memo: All the staff above is stored into array:
~~~php
$agent_equip[$equipment_id] = [
	'available' => ... ,
	'last_activity_date' => ... 
];
~~~
:memo: Than we find warehouse locations and status:
~~~sql
SELECT 
	e.equipment_id, 
	e.equipment_type_id, 
	es.equipment_status_name, 
	wd.name, 
	COUNT( ei.equipment_item_id ) AS  count  
FROM ".TABLE_EQUIPMENT_ITEMS." ei 
JOIN ".TABLE_EQUIPMENT." e ON (e.equipment_id = ei.equipment_id) 
JOIN ".TABLE_EQUIPMENT_STATUSES." es ON (ei.equipment_status_id = es.equipment_status_id) 
JOIN ".TABLE_WAREHOUSES_DESCRIPTION." wd ON (wd.warehouse_id = ei.warehouse_id) 
WHERE e.equipment_type_id = 1  
GROUP BY 
	e.equipment_id, 
	e.equipment_type_id, 
	es.equipment_status_name, 
	wd.name
~~~
:memo: Finally we iterate over each row (looks like: `['equipment_id', 'equipment_type_id', 'equipment_status_name', 'name']` and if `'name'` is `"Fairfax Warehouse"`, we sort followig way:
~~~php
// 'equipment_status_name' == "Available"
Increase by 1: avail, total
// 'equipment_status_name' == "Pending Install"
Increase by 1: total
// 'equipment_status_name' == "Installed"
Increase by 1: ffx, total
~~~
If `'name'` is `"MD Warehouse"`, we sort followig way:
~~~php
// 'equipment_status_name' == "Available"
// 'equipment_status_name' == "Pending Install"
Increase by 1: total
// 'equipment_status_name' == "Installed"
Increase by 1: total, md
~~~
If `"PA Warehouse"`, we sort followig way:
~~~php
// 'equipment_status_name' == "Available"
// 'equipment_status_name' == "Pending Install"
Increase by 1: total
// 'equipment_status_name' == "Installed"
Increase by 1: total, pa
~~~
### InventorySummary::$postsInTheField->md
### InventorySummary::$postsInTheField->pa
### InventorySummary::$postsInTheField->total
### InventorySummary::$postsAvail
~~~php
// For all this values:
InventorySummary::$postsInTheField->md
InventorySummary::$postsInTheField->pa
InventorySummary::$postsInTheField->total
InventorySummary::$postsAvail

// check InventorySummary::$postsInTheField->ffx
~~~
### InventorySummary::$totalOperationalPosts
### InventorySummary::$removals->pending
### InventorySummary::$removals->scheduled
### InventorySummary::$agencies->new
### InventorySummary::$agencies->active

# MONEY STATISTICS:


# class CurrentYear

:exclamation: There was an error because of a typo (coder used function `time` instead of function `mktime`). Previously it took in account orders with `order_added` or `order_completed` less than current moment (not less than tomorrow).


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

# class PreviousYear

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

































<!--stackedit_data:
eyJoaXN0b3J5IjpbLTc5NDkyNTY0Miw2MTIxMjY0NTQsLTE4OD
E1MTEzNzAsNTE0MjQxMTMsMjA5MzQzMTc5NCwtOTQyNzM0Nzks
LTkwNzU5MDE1NiwyMDEwNzk3MDcwLC03MzAyNTU3OTUsMjAzND
MyNjAyOSwtNzY1MzM4MjM0LDk1MjAzMTc3NiwyMDQ5MTY1NTIw
LC0xMDcxNjUwMjI4LC01MTg1Mzk2OTMsMTY4NzU3Mjc3NiwtMT
kyNTg1NDY3MSwxNDc5NTkyNzM5LDM5NTk2MTUyOCwtMTI1MTky
NDE3OF19
-->