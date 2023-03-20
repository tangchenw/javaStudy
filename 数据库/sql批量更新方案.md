# SQL批量更新方案

## replace into 批量更新

```sql
$sql = replace into test_tbl (id,dr) values (1,'2'),(2,'3'),...(x,'y');
　#　test_tbl 为表名
```

id对应的为主键或者判断的唯一值，最好是主键，这样数据库检索较快。

dr 对应字段，有多个字段可以在后面追加。

## insert into ...on duplicate key update批量更新

```sql
$sql = insert into test_tbl (id,dr) values (1,'2'),(2,'3'),...(x,'y') on duplicate key update dr=values(dr);
```

该方法只更新重复记录，不会改变其它字段。

## 创建临时表，先更新临时表，然后从临时表中update

```sql
create temporary table tmp(id int(4) primary key,dr varchar(50));
　　insert into tmp values  (0,'gone'), (1,'xx'),...(m,'yy');
　　update test_tbl, tmp set test_tbl.dr=tmp.dr where test_tbl.id=tmp.id;
```

> 这种方法需要用户有temporary表的创建权限。

这方法涉及两个表的操作，也可能造成大压力。

## 使用mysql 自带的语句构建批量更新

```mysql
UPDATE yoiurtable

    　　SET dingdan = CASE id 
        　　WHEN 1 THEN 3 
       　　 WHEN 2 THEN 4 
       　　 WHEN 3 THEN 5 
   　　 END
　　WHERE id IN (1,2,3)
```

这句sql 的意思是，更新dingdan 字段，如果id=1 则dingdan 的值为3，如果id=2 则dingdan 的值为4…… where部分不影响代码的执行，但是会提高sql执行的效率。确保sql语句仅执行需要修改的行数，这里只有3条数据进行更新，而where子句确保只有3行数据执行。

如果更新多个值的话，只需要稍加修改

```mysql
UPDATE categories 
   SET dingdan = CASE id 
        WHEN 1 THEN 3 
        WHEN 2 THEN 4 
        WHEN 3 THEN 5 
    END, 
    title = CASE id 
        WHEN 1 THEN 'New Title 1'
        WHEN 2 THEN 'New Title 2'
        WHEN 3 THEN 'New Title 3'
    END
WHERE id IN (1,2,3)
```

代码示例：

```mysql
$display_order = array( 
    1 => 4, 
    2 => 1, 
    3 => 2, 
    4 => 3, 
    5 => 9, 
    6 => 5, 
    7 => 8, 
   8 => 9 ); 

$ids = implode(',', array_keys($display_order)); 
$sql = "UPDATE categories SET display_order = CASE id "; 
foreach ($display_order as $id => $ordinal) { 
    $sql .= sprintf("WHEN %d THEN %d ", $id, $ordinal); 
} 
$sql .= "END WHERE id IN ($ids)"; 
echo $sql;
```

```
该方法就是在数据进行处理时适合使用，能够让人清晰处理过程
```

对于数据库操作来说，最好的还是原生的sql语句执行效果最好，并且没有多次连接数据库的操作才好，所以优先的推荐还是方法2，insert into ...on duplicate key update