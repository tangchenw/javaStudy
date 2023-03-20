## Day1

584.寻找用户推荐人 https://leetcode.cn/problems/find-customer-referee/

记录原因：sql里面的不等于（！=），不包含null

> = 或 ！= 只能判断基本数据类型 is 关键字只能判断null <=> 既能判断null 又能判断 基本数据类型

```SQL
select name from customer where referee_id != 2 or referee_id is NULL #解1：！=判断基本类型，is 判断NULL

select name from customer where referee_id <=> 2 #解2： <=>2


select name 
from customer
where id  not in 
(select id 
from customer where  referee_id =2)  #解3： 子查询来规避NULL


select name from customer 
where
ifnull(referee_id,0)!=2; #解4： 利用ifnull函数来转换Null为0与2进行比较从而顺带查询null值
```

## Day2

183.从不订购的客户 https://leetcode.cn/problems/customers-who-never-order/

记录原因：简单的子查询写了复杂的SQL语句。

```SQL
select Name as Customers from Customers  where Id not in 
(select O.CustomerId from Customers as C,Orders as O where C.Id = O.CustomerId)  #此处为最开始解题思路。

select Name 'Customers' from Customers where Id not in(select CustomerId from Order)
```

1873.计算特殊奖金 https://leetcode.cn/problems/calculate-special-bonus/

记录原因：无法用单条查询语句写出根据条件修改查询出来的结果集。

> if函数的使用，可用于字段修改字段值IF(condition, value_if_true, value_if_false) ，先判断，如果真则是第二个参数的值，如果是假则是第三个参数的值。

```SQL
update Employees set salary = 0 where MOD(employee_id,2)=0 or name like
'_M';
select employee_id,salary as bonus from Employees;  #改了值之后再查询，无法满足题解条件只编写一条SQL


#if函数解决，也可用MOD(employee_id,2)!=0 and name not like 'M%',salary,0);一个意思。
select employee_id,if(MOD(employee_id,2)=0 or name like 'M%',0,salary) as bonus from Employees


#组合查询UNION + NOT LIKE；找出符合的那一部分取原值同时UNION组合查询出不符合的那一部分修改原值为0.
SELECT employee_id ,salary AS bonus
FROM Employees
WHERE employee_id%2!=0 AND name NOT LIKE ('M%')
UNION 
SELECT employee_id ,salary*0 AS bonus
FROM Employees
WHERE employee_id%2=0 OR name LIKE ('M%')
ORDER BY employee_id;

#CASE+RIGHT +MOD，  case when then else end用法；不一定所有的条件都要用where来执行，当boolean条件的时候就可以用if或者case函数来解决。
SELECT employee_id,
(CASE WHEN MOD(employee_id,2)!=0 AND LEFT(name,1)!='M' THEN salary
     ELSE 0
END) bonus
FROM Employees
ORDER BY employee_id
```

627.变更性别 https://leetcode.cn/problems/swap-salary/

记录原因：太呆了，if函数用得不熟练

> set cloum1 = if()；更新语句中给字段的赋值可以使用if来进行选择赋值。

```SQL
update Salary set sex = if(sex = 'f','m','f'); #if条件判断单目运算符即可，与java不同
```



196.删除重复的邮箱 https://leetcode.cn/problems/delete-duplicate-emails/?envType=study-plan&id=sql-beginner&plan=sql&plan_progress=bqxlacm

记录原因：在MYSQL中，不能先Select一个表的记录，再按此条件Update和Delete同一个表的记录，否则会出错：You can't specify target table 'xxx' for update in FROM clause.

> 使用嵌套Select——将Select得到的查询结果作为中间表，再Select一遍中间表作为结果集，即可规避错误。

```sql
#MYSQL不支持一边查询一边删除，必须将查询结果作为中间表重新查询一遍才能作为结果集。
delete from Person where id in (select id from Person group by email having min(id))


#修正，此处的haing min(id)依然有问题，以分组完成之后再进行过滤操作
delete from Person where id in (select * from(select id from Person group by email having min(id)) as t)

#正确语法。
delete from Person where id not in 
(select * from
(select min(id) from Person group by email)
 as t)
```



1667.修复表的名字 https://leetcode.cn/problems/fix-names-in-a-table/?envType=study-plan&id=sql-beginner&plan=sql&plan_progress=bqxlacm

记录原因：对与concat()连接字符串函数，substring(字段，起始位置，长度)剪切字符串函数；upper（）和lower（）大小写函数的使用不熟悉。

```sql
#此处UPPER也可以使用UPPER(LEFT(name,1))；LEFT是返回最左侧几个字符，RIGHT是最右侧。
select user_id,CONCAT(UPPER(SUBSTRING(name,1,1)), LOWER(SUBSTRING(name,2)) )  as name from Users order by user_id asc;
```



1484.按日期分组销售产品  https://leetcode.cn/problems/group-sold-products-by-the-date/?envType=study-plan&id=sql-beginner&plan=sql&plan_progress=bqxlacm

记录原因：**group_concat**函数的不了解。

> group_concat([[DISTINCT](https://so.csdn.net/so/search?q=DISTINCT&spm=1001.2101.3001.7020)] 要连接的字段 [Order BY ASC/DESC 排序字段] [Separator '分隔符'])

```sql
#关键在第三行group_concat函数的使用，连接了相同的行，同时可以进行去重排序，自定义分隔符操作
select sell_date,
count(distinct product) as num_sold,
group_concat(distinct product order by product separator ',')   as products 
from Activities
group by sell_date
order by sell_date asc;
```



1965.丢失信息的雇员 https://leetcode.cn/problems/employees-with-missing-information/

记录原因：简单SQL复杂化



```sql
#此处的not in查询太过于复杂
select employee_id from Employees
where employee_id 
not in (select E.employee_id from Employees as E,Salaries as S
where E.employee_id = S.employee_id)
UNION
select employee_id from Salaries
where  employee_id
not in (select S.employee_id from Employees as E,Salaries as S
where E.employee_id = S.employee_id)
order by employee_id asc;

#可以直接查询另一张表的id然后找本表中不在另外一张表的id即可
select employee_id from Employees
where employee_id 
not in (select employee_id from Salaries)
UNION
select employee_id from Salaries
where  employee_id
not in (select employee_id from Employees)
order by employee_id asc;
```



1795.每个产品在不同商店的价格 https://leetcode.cn/problems/rearrange-products-table/

记录原因：行转换和列转换都没接触过，完全不知道怎么写。

> 行转列用groupby+sumif，列转行用union all

```sql
#列转行，'store1'表示查询的时候单纯将store1字符串作为参数传入store。
select product_id, 'store1' as store, store1 as price from Products where store1 is not null 
union all 
select product_id, 'store2' as store, store2 as price from Products where store2 is not null 
union all 
select product_id, 'store3' as store, store3 as price from Products where store3 is not null;
```



608.树节点 https://leetcode.cn/problems/tree-node/

记录原因：加深对 NULL值判断的印象，NULL只能用IS NULL或者IS NOT NULL来进行判断， NOT IN中包含null值时将全部返回null值，即为false。

```sql
#not in中要加 p_id is not null。
select id,
case when p_id is null then 'Root'
     when id not in (select p_id from tree where p_id is not null) then 'Leaf'
      else 'Inner' end as Type 
from tree;
```



197.上升的温度

记录原因：日期的比较计算函数的使用

```sql
#dateDiff计算日期的差值
select a.Id from  Weather as a inner join Weather as b on a.Temperature > b.Temperature and dateDiff(a.RecordDate,b.RecordDate) = 1
```





1158.市场分析I https://leetcode.cn/problems/market-analysis-i/

记录原因：count用法

> 1.count(1)与count(*)得到的结果一致,包含null值。
>
> 2.count(字段)不计算null值
>
> 3.count(null)结果恒为0

```SQL
#此处count(o.buyer_id)似乎是忽略了null值的列，导致没有统计到所在的行
select u.user_id as buyer_id,U.join_date,
if(year(O.order_date) = '2019',count(o.buyer_id),0) as orders_in_2019
from Users U left join Orders O
on U.user_id = o.buyer_id
group by u.user_id

#使用sum，此外select 副表的buyer_id会存在问题，似乎也是Null值导致的。
select user_id as buyer_id,u.join_date,
sum(if(year(O.order_date) = '2019',1,0)) as orders_in_2019
from Users U left join Orders O
on U.user_id = o.buyer_id
group by user_id;
```





