# MySQL语法用法总结

**DESCRIBE语句** MySQL支持用DESCRIBE作为SHOW COLUMNS FROM的一种快捷方式。换句话说， DESCRIBE customers;是SHOW COLUMNS FROM customers;的一种快捷方式。  

所支持的其他SHOW语句还有：  

- SHOW STATUS，用于显示广泛的服务器状态信息；  
- SHOW CREATE DATABASE和SHOW CREATE TABLE，分别用来显示创建特定数据库或表的MySQL语句；  
- SHOW GRANTS，用来显示授予用户（所有用户或特定用户）的安全权限；  
- SHOW ERRORS和SHOW WARNINGS， 用来显示服务器错误或警告消息。  

> 请在mysql命令行实用程序中，执行命令HELP SHOW;显示允许的SHOW语句。  

## 拼接字段

### Concat()函数

> 多数DBMS使用+或||来实现拼接，MySQL则使用Concat()函数来实现。当把SQL语句转换成MySQL语句时一定要把这个区别铭记在心。  

![image-20220802143523485](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220802143523485.png)

## 笛卡尔积

**笛卡儿积（cartesian product）** 由没有联结条件的表关系返回的结果为笛卡儿积。检索出的行的数目将是第一个表中的行数乘
以第二个表中的行数。  

> 应该保证所有联结都有WHERE子句，否则MySQL将返回比想要的数据多得多的数据。同理，应该保证WHERE子句的正确性。不正确的过滤条件将导致MySQL返回不正确的数据。  

## 用自联结而不用子查询

> 用自联结而不用子查询 自联结通常作为外部语句用来替代从相同表中检索数据时使用的子查询语句。虽然最终的结果是相同的，但有时候处理联结远比处理子查询快得多。应该试一下两种方法，以确定哪一种的性能更好。  

## 组合查询

多数SQL查询都只包含从一个或多个表中返回数据的单条SELECT语句。 MySQL也允许执行多个查询（多条SELECT语句），并将结果作为单个查询结果集返回。这些组合查询通常称为并（ union） 或复合查询（compound query）。  

> 多数情况下，组合相同表的两个查询完成的工作与具有多个WHERE子句条件的单条查询完成的工作相同。换句话说，任何具有多个WHERE子句的SELECT语句都可以作为一个组合查询给出，在以下段落中可以看到这一点。这两种技术在不同的查询中性能也不同。因此，应该试一下这两种技术，以确定对特定的查询哪一种性能更好。  

### UNION操作符

UNION的使用很简单。所需做的只是给出每条SELECT语句，在各条语句之间放上关键字UNION。  

![image-20220802145204412](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220802145204412.png)

使用规则如下：

- UNION必须由两条或两条以上的SELECT语句组成，语句之间用关键字UNION分隔（因此，如果组合4条SELECT语句，将要使用3个
  UNION关键字）。  
- UNION中的每个查询必须包含相同的列、表达式或聚集函数（不过各个列不需要以相同的次序列出）。  
- 列数据类型必须兼容：类型不必完全相同，但必须是DBMS可以隐含地转换的类型（例如，不同的数值类型或不同的日期类型）。  

### UNION ALL

UNION从查询结果集中自动去除了重复的行（换句话说，它的行为与单条SELECT语句中使用多个WHERE子句条件一样）。因为供应商1002生产的一种物品的价格也低于5，所以两条SELECT语句都返回该行。在使用UNION时，重复的行被自动取消。  

这是UNION的默认行为，但是如果需要，可以改变它。事实上，如果想返回所有匹配行，可使用UNION ALL而不是UNION。  

## 全文本搜索

MySQL支持几种基本的数据库引擎。并非所有的引擎都支持全文本搜索。两个最常使用的引擎为MyISAM和InnoDB，前者支持全文本搜索，而后者不支持。

### 使用规则

为了进行全文本搜索，必须索引被搜索的列，而且要随着数据的改变不断地重新索引。在对表列进行适当设计后， MySQL会自动进行所有的索引和重新索引。 

在索引之后， SELECT可与Match()和Against()一起使用以实际执行搜索。

一般在创建表时启用全文本搜索。 CREATE TABLE语句接受FULLTEXT子句，它给出被索引列的一个逗号分隔的列表。  

![QQ截图20220802145824](C:\Users\汤琛\Desktop\学习资料\数据库\Mysql\images\QQ截图20220802145824.png)

> 可以在创建表时指定FULLTEXT，或者在稍后指定（在这种情况下所有已有数据必须立即索引）。  

### 进行全文本检索

在索引之后，使用两个函数Match()和Against()执行全文本搜索，其中Match()指定被搜索的列， Against()指定要使用的搜索表达式。  

![image-20220802150050017](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220802150050017.png)

![image-20220802150057139](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220802150057139.png)

此SELECT语句检索单个列note_text。由于WHERE子句，一个全文本搜索被执行。 Match(note_text)指示MySQL针对指定的
列进行搜索， Against('rabbit')指定词rabbit作为搜索文本。由于有两行包含词rabbit，这两个行被返回。  

> 除非使用BINARY方式，否则全文本搜索不区分大小写。  

## LOW_PRIORITY  

数据库经常被多个客户访问，对处理什么请求以及用什么次序处理进行管理是MySQL的任务。 INSERT操作可能很耗时（特别是有很多索引需要更新时），而且它可能降低等待处理的SELECT语句的性能。
如果数据检索是最重要的（通常是这样），则你可以通过在INSERT和INTO之间添加关键字LOW_PRIORITY，指示MySQL降低INSERT语句的优先级，如下所示：

`INSERT LOW_PRIORITY INTO`

顺便说一下，这也适用于UPDATE和DELETE语句。  

## 提升INSERT的性能

![image-20220802152449904](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220802152449904.png)

![image-20220802152531877](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220802152531877.png)

> 其中单条INSERT语句有多组值，每组值用一对圆括号括起来，用逗号分隔。  此技术可以提高数据库处理的性能，因为MySQL用单条INSERT语句处理多个插入比使用多条INSERT语句快。  

## 插入检索出来的数据

INSERT一般用来给表插入一个指定列值的行。但是， INSERT还存在另一种形式，可以利用它将一条SELECT语句的结果插入表中。这就是所谓的INSERT SELECT，顾名思义，它是由一条INSERT语句和一条SELECT语句组成的。  

假如你想从另一表中合并客户列表到你的customers表。 不需要每次读取一行，然后再将它用INSERT插入，可以如下进行：  

![image-20220802153258564](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220802153258564.png)

![image-20220802153305748](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220802153305748.png)

## DELETE语句

**删除表内容而不是表**：DELETE语句从表中删除行，甚至是删除表中所有行。但是， DELETE不删除表本身。  

**更快删除**：如果想从表中删除所有行，不要使用DELETE。可使用**TRUNCATE TABLE**语句，它完成相同的工作，但速度更快（ TRUNCATE实际是删除原来的表并重新创建一个表，而不是逐行删除表中的数据）。  

## MySQL数据引擎

以下是几个需要知道的引擎：  

- InnoDB是一个可靠的事务处理引擎，它不支持全文本搜索；  
- MEMORY在功能等同于MyISAM， 但由于数据存储在内存（不是磁盘）中，速度很快（特别适合于临时表）；  
- MyISAM是一个性能极高的引擎，它支持全文本搜索，但不支持事务处理。  

引擎类型可以混用。除productnotes表使用MyISAM外，本书中的样例表都使用InnoDB。 原因是作者希望支持事务处理（因此，使用InnoDB），但也需要在productnotes中支持全文本搜索（因此，使用MyISAM）。  

## 视图

> 需要MySQL 5 MySQL 5添加了对视图的支持。因此，本章内容适用于MySQL 5及以后的版本。  

视图是虚拟的表。与包含数据的表不一样，视图只包含使用时动态检索数据的查询。  

理解视图的最好方法是看一个例子。第15章中用下面的SELECT语句从3个表中检索数据：  

![image-20220802163407869](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220802163407869.png)

此查询用来检索订购了某个特定产品的客户。任何需要这个数据的人都必须理解相关表的结构，并且知道如何创建查询和对表进行联结。
为了检索其他产品（或多个产品）的相同数据，必须修改最后的WHERE子句。  

现在，假如可以把整个查询包装成一个名为productcustomers的虚拟表，则可以如下轻松地检索出相同的数据：  

![image-20220802163556341](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220802163556341.png)

这就是视图的作用。 productcustomers是一个视图，作为视图，它不包含表中应该有的任何列或数据，它包含的是一个SQL查询（与上面用以正确联结表的相同的查询）。  

### 为什么使用视图  

我们已经看到了视图应用的一个例子。下面是视图的一些常见应用。  

- 重用SQL语句。  
- 简化复杂的SQL操作。在编写查询后，可以方便地重用它而不必知道它的基本查询细节。
- 使用表的组成部分而不是整个表。  
- 保护数据。可以给用户授予表的特定部分的访问权限而不是整个表的访问权限。   
- 更改数据格式和表示。视图可返回与底层表的表示和格式不同的数据。  

在视图创建之后，可以用与表基本相同的方式利用它们。可以对视图执行SELECT操作，过滤和排序数据，将视图联结到其他视图或表，甚至能添加和更新数据（添加和更新数据存在某些限制。关于这个内容稍后还要做进一步的介绍）。  

重要的是知道视图仅仅是用来查看存储在别处的数据的一种设施。视图本身不包含数据，因此它们返回的数据是从其他表中检索出来的。
在添加或更改这些表中的数据时，视图将返回改变过的数据。  

> 性能问题：因为视图不包含数据，所以每次使用视图时，都必须处理查询执行时所需的任一个检索。如果你用多个联结和过滤创建了复杂的视图或者嵌套了视图，可能会发现性能下降得很厉害。因此，在部署使用了大量视图的应用前，应该进行测试。  

### 视图的规则和限制  

下面是关于视图创建和使用的一些最常见的规则和限制 。

- 与表一样，视图必须唯一命名（不能给视图取与别的视图或表相同的名字）。  
- 对于可以创建的视图数目没有限制。  
- 为了创建视图，必须具有足够的访问权限。这些限制通常由数据库管理人员授予。  
- 视图可以嵌套，即可以利用从其他视图中检索数据的查询来构造一个视图。  
- ORDER BY可以用在视图中，但如果从该视图检索数据SELECT中也含有ORDER BY，那么该视图中的ORDER BY将被覆盖。  
- 视图不能索引，也不能有关联的触发器或默认值。  
- 视图可以和表一起使用。例如，编写一条联结表和视图的SELECT语句。  

### 使用视图  

在理解什么是视图（以及管理它们的规则及约束）后，我们来看一下视图的创建。  

- 视图用CREATE VIEW语句来创建。  
- 使用SHOW CREATE VIEW viewname；来查看创建视图的语句。  
- 用DROP删除视图，其语法为DROP VIEW viewname;  
- 更新视图时，可以先用DROP再用CREATE，也可以直接用CREATE OR REPLACE VIEW。如果要更新的视图不存在，则第2条更新语句会创建一个视图；如果要更新的视图存在，则第2条更新语句会替换原有视图。  

## 存储过程

存储过程简单来说，就是为以后的使用而保存的一条或多条MySQL语句的集合。可将其视为批文件，虽然它们的作用不仅限于批处理。  

### 为什么要使用存储过程  

既然我们知道了什么是存储过程，那么为什么要使用它们呢？有许多理由，下面列出一些主要的理由。  

- 通过把处理封装在容易使用的单元中，简化复杂的操作。 
- 由于不要求反复建立一系列处理步骤，这保证了数据的完整性。如果所有开发人员和应用程序都使用同一（试验和测试）存储过程，则所使用的代码都是相同的。这一点的延伸就是防止错误。需要执行的步骤越多，出错的可能性就越大。防止错误保证了数据的一致性。  
- 简化对变动的管理。如果表名、列名或业务逻辑（或别的内容）有变化，只需要更改存储过程的代码。使用它的人员甚至不需要知道这些变化。  
- 提高性能。因为使用存储过程比使用单独的SQL语句要快。  
- 存在一些只能用在单个请求中的MySQL元素和特性，存储过程可以使用它们来编写功能更强更灵活的代码。  

### 使用存储过程  

使用存储过程需要知道如何执行（运行）它们。存储过程的执行远比其定义更经常遇到，因此，我们将从执行存储过程开始介绍。然后再
介绍创建和使用存储过程。  

#### 执行存储过程  

MySQL称存储过程的执行为调用，因此MySQL执行存储过程的语句为CALL。 CALL接受存储过程的名字以及需要传递给它的任意参数。请看以下例子：  

![image-20220804151150722](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804151150722.png)

存储过程可以显示结果，也可以不显示结果 。

#### 创建存储过程  

正如所述，编写存储过程并不是微不足道的事情。为让你了解这个过程，请看一个例子——一个返回产品平均价格的存储过程。以下是其
代码：  

![image-20220804154116166](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804154116166.png)

我们稍后介绍第一条和最后一条语句。此存储过程名为productpricing，用CREATE PROCEDURE productpricing()语句定义。如果存储过程接受参数，它们将在()中列举出来。此存储过程没有参数，但后跟的()仍然需要。 BEGIN和END语句用来限定存储过程体，过程体本身仅是一个简单的SELECT语句。  

在MySQL处理这段代码时，它创建一个新的存储过程productpricing。没有返回数据，因为这段代码并未调用存储过程，这里只是为
以后使用而创建它。  

那么，如何使用这个存储过程？  

![image-20220804160152844](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804160152844.png)

CALL productpricing();执行刚创建的存储过程并显示返回的结果。因为存储过程实际上是一种函数，所以存储过程名后需要有()符号（即使不传递参数也需要）。  

#### 删除存储过程  

存储过程在创建之后，被保存在服务器上以供使用，直至被删除。删除命令从服务器中删除存储过程。  

![image-20220804160406874](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804160406874.png)

> 如果指定的过程不存在，则DROP PROCEDURE将产生一个错误。当过程存在想删除它时（如果过程不存在也不产生错误）可使用DROP PROCEDURE IF EXISTS。  

#### 使用参数  

**变量（variable）** 内存中一个特定的位置，用来临时存储数据。  

![image-20220804160814251](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804160814251.png)

此存储过程接受3个参数： pl存储产品最低价格， ph存储产品最高价格， pa存储产品平均价格。每个参数必须具有指定的类型，这里使用十进制值。关键字OUT指出相应的参数用来从存储过程传出一个值（返回给调用者）。 MySQL支持IN（传递给存储过程）、 OUT（从存储过程传出，如这里所用）和INOUT（对存储过程传入和传出）类型的参数。存储过程的代码位于BEGIN和END语句内，如前所见，它们是一系列SELECT语句，用来检索值，然后保存到相应的变量（通过指定INTO关键字）。  

为调用此修改过的存储过程，必须指定3个变量名.

![image-20220804161603967](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804161603967.png)

由于此存储过程要求3个参数，因此必须正好传递3个参数，不多也不少。所以，这条CALL语句给出3个参数。它们是存储过程将保存结果的3个变量的名字。  

> 所有MySQL变量都必须以@开始。  

**ordertotal接受订单号并返回该订单的合计** : 

![image-20220804162959241](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804162959241.png)

![image-20220804163009939](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804163009939.png)

onumber定义为IN，因为订单号被传入存储过程。 ototal定义为OUT，因为要从存储过程返回合计。 SELECT语句使用这两个参数， WHERE子句使用onumber选择正确的行， INTO使用ototal存储计算出来的合计。  

为调用这个新存储过程，可使用以下语句：  

![image-20220804163118605](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804163118605.png)

必须给ordertotal传递两个参数；第一个参数为订单号，第二个参数为包含计算出来的合计的变量名  

为了显示此合计，可如下进行：  

![image-20220804163212888](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804163212888.png)

@total已由ordertotal的CALL语句填写， SELECT显示它包含的值。  

为了得到另一个订单的合计显示，需要再次调用存储过程，然后重新显示变量：  

![image-20220804163257777](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804163257777.png)

#### 建立智能存储过程  

迄今为止使用的所有存储过程基本上都是封装MySQL简单的SELECT语句。虽然它们全都是有效的存储过程例子，但它们所能完成的工作你直接用这些被封装的语句就能完成（如果说它们还能带来更多的东西，  那就是使事情更复杂）。只有在存储过程内包含业务规则和智能处理时，它们的威力才真正显现出来  

考虑这个场景。你需要获得与以前一样的订单合计，但需要对合计增加营业税，不过只针对某些顾客（或许是你所在州中那些顾客）。那么，你需要做下面几件事情：  

- 获得合计（与以前一样）；  
- 把营业税有条件地添加到合计；  
- 返回合计（带或不带税）。  

存储过程的完整工作如下：  

![image-20220804163626961](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804163626961.png)

![image-20220804163635790](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804163635790.png)

此存储过程有很大的变动。首先，增加了注释（前面放置--）。在存储过程复杂性增加时，这样做特别重要。添加了另外一个参数taxable，它是一个布尔值（如果要增加税则为真，否则为假）。在存储过程体中，用DECLARE语句定义了两个局部变量。 DECLARE要求指定变量名和数据类型，它也支持可选的默认值（这个例子中的taxrate的默认被设置为6%）。 SELECT语句已经改变，因此其结果存储到total（局部变量）而不是ototal。 IF语句检查taxable是否为真，如果为真，则用另一SELECT语句增加营业税到局部变量total。最后，用另一SELECT语句将total（它增加或许不增加营业税）保存到ototal。  

> 本例子中的存储过程在CREATE PROCEDURE语句中包含了一个COMMENT值。它不是必需的，但如果给出，将在SHOW PROCEDURE STATUS的结果中显示。  

#### 检查存储过程  

为显示用来创建一个存储过程的CREATE语句，使用SHOW CREATEPROCEDURE语句：  

![image-20220804165854971](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804165854971.png)

为了获得包括何时、由谁创建等详细信息的存储过程列表， 使用SHOW PROCEDURE STATUS。  

> SHOW PROCEDURE STATUS列出所有存储过程。为限制其输出，可使用LIKE指定一个过滤模式，例如：  ![image-20220804165938817](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804165938817.png)

## 使用游标

有时，需要在检索出来的行中前进或后退一行或多行。这就是使用游标的原因。 游标（ cursor） 是一个存储在MySQL服务器上的数据库查询，它不是一条SELECT语句，而是被该语句检索出来的结果集。在存储了游标之后，应用程序可以根据需要滚动或浏览其中的数据。  

游标主要用于交互式应用，其中用户需要滚动屏幕上的数据，并对数据进行浏览或做出更改。  

> 不像多数DBMS， MySQL游标只能用于存储过程（和函数）。  

#### 使用游标  

使用游标涉及几个明确的步骤。  

- 在能够使用游标前，必须声明（定义）它。这个过程实际上没有检索数据，它只是定义要使用的SELECT语句。  
- 一旦声明后，必须打开游标以供使用。这个过程用前面定义的SELECT语句把数据实际检索出来 。
- 对于填有数据的游标，根据需要取出（检索）各行。  
- 在结束游标使用时，必须关闭游标。  

在声明游标后，可根据需要频繁地打开和关闭游标。在游标打开后，可根据需要频繁地执行取操作。  

#### 创建游标  

游标用DECLARE语句创建。 DECLARE命名游标，并定义相应的SELECT语句，根据需要带WHERE和其他子句。例如，下面的语句定义了名为ordernumbers的游标， 使用了可以检索所有订单的SELECT语句。  

![image-20220804171603985](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804171603985.png)

这个存储过程并没有做很多事情， DECLARE语句用来定义和命名游标，这里为ordernumbers。 存储过程处理完成后，游标就消失（因为它局限于存储过程）  .

在定义游标之后，可以打开它。  

#### 打开和关闭游标  

游标用OPEN CURSOR语句来打开：  

![image-20220804171755978](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804171755978.png)

在处理OPEN语句时执行查询，存储检索出的数据以供浏览和滚动。  

游标处理完成后，应当使用如下语句关闭游标：  ![image-20220804171824111](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804171824111.png)

CLOSE释放游标使用的所有内部内存和资源，因此在每个游标不再需要时都应该关闭。  

在一个游标关闭后，如果没有重新打开，则不能使用它。但是，使用声明过的游标不需要再次声明，用OPEN语句打开它就可以了。  

> 如果你不明确关闭游标， MySQL将会在到达END语句时自动关闭它。  

![image-20220804172019639](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804172019639.png)

这个存储过程声明、打开和关闭一个游标。但对检索出的数据什么也没做。  

#### 使用游标数据  

在一个游标被打开后，可以使用FETCH语句分别访问它的每一行。FETCH指定检索什么数据（所需的列），检索出来的数据存储在什么地方。它还向前移动游标中的内部行指针，使下一条FETCH语句检索下一行（不重复读取同一行）。  

第一个例子从游标中检索单个行（第一行）：  

![image-20220804172321222](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804172321222.png)

其中FETCH用来检索当前行的order_num列（将自动从第一行开始）到一个名为o的局部声明的变量中。对检索出的数据不做任何处理。  

在下一个例子中，循环检索数据，从第一行到最后一行  :

![image-20220804172411738](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804172411738.png)

![image-20220804172421001](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804172421001.png)

与前一个例子一样，这个例子使用FETCH检索当前order_num到声明的名为o的变量中。但与前一个例子不一样的是，这个例子中的FETCH是在REPEAT内，因此它反复执行直到done为真（由UNTILdone END REPEAT;规定）。为使它起作用，用一个DEFAULT 0（假，不结束）定义变量done。那么， done怎样才能在结束时被设置为真呢？答案是用以下语句：  

![image-20220804172812747](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220804172812747.png)

这条语句定义了一个CONTINUE HANDLER，它是在条件出现时被执行的代码。 这里， 它指出当SQLSTATE '02000'出现时， SET done=1。 SQLSTATE'02000'是一个未找到条件， 当REPEAT由于没有更多的行供循环而不能继续时，出现这个条件。  

> DECLARE语句的发布存在特定的次序。用DECLARE语句定义的局部变量必须在定义任意游标或句柄之前定义，而句柄必须在游标之后定义。不遵守此顺序将产生错误消息  

如 果 调 用 这 个 存 储 过 程 ， 它 将 定 义 几 个 变 量 和 一 个 CONTINUE HANDLER，定义并打开一个游标，重复读取所有行，然后关闭游标。  

如果一切正常，你可以在循环内放入任意需要的处理（在FETCH语句之后，循环结束之前）。  

> 除这里使用的REPEAT语句外， MySQL还支持循环语句，它可用来重复执行代码，直到使用LEAVE语句手动退出为止。通常REPEAT语句的语法使它更适合于对游标进行循环  

![image-20220805091557619](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220805091557619.png)

![image-20220805091612760](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220805091612760.png)

在这个例子中，我们增加了另一个名为t的变量（存储每个订单的合计）。 此存储过程还在运行中创建了一个新表（如果它不存在话）， 名为ordertotals。 这个表将保存存储过程生成的结果。 FETCH像以前一样取每个order_num，然后用CALL执行另一个存储过程（我们在
前一章中创建）来计算每个订单的带税的合计（结果存储到t）。最后，用INSERT保存每个订单的订单号和合计。  

此存储过程不返回数据，但它能够创建和填充另一个表，可以用一条简单的SELECT语句查看该表：  

![image-20220805091708353](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220805091708353.png)

这样，我们就得到了存储过程、游标、逐行处理以及存储过程调用其他存储过程的一个完整的工作样例。  

## 触发器

MySQL语句在需要时被执行，存储过程也是如此。但是，如果你想要某条语句（或某些语句）在事件发生时自动执行，怎么办呢？例如：  

- 每当增加一个顾客到某个数据库表时，都检查其电话号码格式是否正确，州的缩写是否为大写；
- 每当订购一个产品时，都从库存数量中减去订购的数量；  
- 无论何时删除一行，都在某个存档表中保留一个副本。

所有这些例子的共同之处是它们都需要在某个表发生更改时自动处理。这确切地说就是触发器。 触发器是MySQL响应以下任意语句而
自动执行的一条MySQL语句（或位于BEGIN和END语句之间的一组语句）：  

- DELETE;
- INSERT;
- UPDATE;

其它MySQL语句不支持触发器

### 创建触发器  

在创建触发器时，需要给出4条信息：  

- 唯一的触发器名
- 触发器关联的表
- 触发器应该响应的活动（DELETE;UPDATE;INSERT)
- 触发器何时执行（处理之前或者之后）

> 在MySQL 5中，触发器名必须在每个表中唯一，但不是在每个数据库中唯一。这表示同一数据库中的两个表可具有相同名字的触发器。这在其他每个数据库触发器名必须唯一的DBMS中是不允许的，而且以后的MySQL版本很可能会使命名规则更为严格。因此，现在最好是在数据库范围内使用唯一的触发器名。  

触发器用CREATE TRIGGER语句创建。下面是一个简单的例子：  

![image-20220805093125650](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220805093125650.png)

CREATE TRIGGER用来创建名为newproduct的新触发器。触发器可在一个操作发生之前或之后执行，这里给出了AFTER INSERT，所以此触发器将在INSERT语句成功执行后执行。这个触发器还指定FOREACH ROW，因此代码对每个插入行执行。在这个例子中，文本Product
added将对每个插入的行显示一次。  

为了测试这个触发器，使用INSERT语句添加一行或多行到products中，你将看到对每个成功的插入，显示Product added消息。  

> 只有表才支持触发器，视图不支持（临时表也不支持）。

触发器按每个表每个事件每次地定义，每个表每个事件每次只允许一个触发器。因此，每个表最多支持6个触发器（每条INSERT、 UPDATE和DELETE的之前和之后）。单一触发器不能与多个事件或多个表关联，所以，如果你需要一个对INSERT和UPDATE操作执行的触发器，则应该定义两个触发器  。

> 如果BEFORE触发器失败，则MySQL将不执行请求的操作。此外，如果BEFORE触发器或语句本身失败， MySQL将不执行AFTER触发器（如果有的话）。  

### 删除触发器  

现在，删除触发器的语法应该很明显了。为了删除一个触发器，可使用DROP TRIGGER语句，如下所示：  

```sql
DROP TRIGGER newproduct;
```

触发器不能更新或覆盖。为了修改一个触发器，必须先删除它，然后再重新创建。  

### 使用触发器  

在有了前面的基础知识后，我们现在来看所支持的每种触发器类型以及它们的差别  

#### Insert触发器

INSERT触发器在INSERT语句执行之前或之后执行。需要知道以下几点：  

- 在INSERT触发器代码内，可引用一个名为NEW的虚拟表，访问被插入的行；  
- 在BEFORE INSERT触发器中， NEW中的值也可以被更新（允许更改被插入的值）；  
- 对于AUTO_INCREMENT列， NEW在INSERT执行之前包含0，在INSERT执行之后包含新的自动生成值。  

下面举一个例子（一个实际有用的例子）。 AUTO_INCREMENT列具有MySQL自动赋予的值。第21章建议了几种确定新生成值的方法，但下面是一种更好的方法：  

![image-20220805094012924](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220805094012924.png)

此代码创建一个名为neworder的触发器，它按照AFTER INSERT ON orders执行。在插入一个新订单到orders表时， MySQL生成一个新订单号并保存到order_num中。触发器从NEW. order_num取得这个值并返回它。此触发器必须按照AFTER INSERT执行，因为在BEFORE  INSERT语句执行之前，新order_num还没有生成。对于orders的每次插入使用这个触发器将总是返回新的订单号。  

为测试这个触发器，试着插入一下新行，如下所示：  

![image-20220805094209458](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220805094209458.png)

orders 包 含 3 个 列 。 order_date 和 cust_id 必 须 给 出 ，order_num由MySQL自动生成，而现在order_num还自动被返回  

> 通常，将BEFORE用于数据验证和净化（目的是保证插入表中的数据确实是需要的数据）。本提示也适用于UPDATE触发器。  

#### DELETE触发器  

DELETE触发器在DELETE语句执行之前或之后执行。需要知道以下两点：  

- 在DELETE触发器代码内，你可以引用一个名为OLD的虚拟表，访问被删除的行  
- OLD中的值全都是只读的，不能更新。  

下面的例子演示使用OLD保存将要被删除的行到一个存档表中：  

![image-20220805095228648](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220805095228648.png)

在任意订单被删除前将执行此触发器。它使用一条INSERT语句将OLD中的值（要被删除的订单）保存到一个名为archive_
orders的存档表中（为实际使用这个例子，你需要用与orders相同的列创建一个名为archive_orders的表）。  

使用BEFORE DELETE触发器的优点（相对于AFTER DELETE触发器来说）为，如果由于某种原因，订单不能存档， DELETE本身将被放弃  

> 正如所见，触发器deleteorder使用BEGIN和END语句标记触发器体。这在此例子中并不是必需的，不过也没有害处。使用BEGIN END块的好处是触发器能容纳多条SQL语句（在BEGIN END块中一条挨着一条）。  

### UPDATE触发器  

UPDATE触发器在UPDATE语句执行之前或之后执行。需要知道以下几点：  

- 在UPDATE触发器代码中，你可以引用一个名为OLD的虚拟表访问以前（ UPDATE语句前）的值，引用一个名为NEW的虚拟表访问新更新的值；  
- 在BEFORE UPDATE触发器中， NEW中的值可能也被更新（允许更改将要用于UPDATE语句中的值）；  
- OLD中的值全都是只读的，不能更新。  

下面的例子保证州名缩写总是大写（不管UPDATE语句中给出的是大写还是小写）：  

![image-20220805095934448](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220805095934448.png)

显然， 任何数据净化都需要在UPDATE语句之前进行，就像这个 例 子 中一样。每次更新一个行时， NEW.vend_state中的值（将用来更新表行的值)都用Upper(NEW.vend_state)替换。  

### 关于触发器的进一步介绍  

在结束本章之前，我们再介绍一些使用触发器时需要记住的重点。  

- 与其他DBMS相比， MySQL 5中支持的触发器相当初级。未来的MySQL版本中有一些改进和增强触发器支持的计划。
- 创建触发器可能需要特殊的安全访问权限，但是，触发器的执行是自动的。如果INSERT、 UPDATE或DELETE语句能够执行，则相关
  的触发器也能执行。  
- 应该用触发器来保证数据的一致性（大小写、格式等）。在触发器中执行这种类型的处理的优点是它总是进行这种处理，而且是透
  明地进行，与客户机应用无关。  
- 触发器的一种非常有意义的使用是创建审计跟踪。使用触发器，把更改（如果需要，甚至还有之前和之后的状态）记录到另一个表非常容易。  
- 遗憾的是， MySQL触发器中不支持CALL语句。这表示不能从触发器内调用存储过程。所需的存储过程代码需要复制到触发器内。  

## 事务

关于事务处理需要知道的几个术语  ：

- 事务（transaction）指一组SQL语句；  
- 回退（rollback）指撤销指定SQL语句的过程；  
- 提交（commit）指将未存储的SQL语句结果写入数据库表；  
- 保留点（ savepoint）指事务处理中设置的临时占位符（ placeholder），你可以对它发布回退（与回退整个事务处理不同）。  

> 事务处理用来管理INSERT、 UPDATE和DELETE语句。你不能回退SELECT语句。（这样做也没有什么意义。）你不能回退CREATE或DROP操作。事务处理块中可以使用这两条语句，但如果你执行回退，它们不会被撤销。  

### 使用保留点  

简单的ROLLBACK和COMMIT语句就可以写入或撤销整个事务处理。但是，只是对简单的事务处理才能这样做，更复杂的事务处理可能需要部分提交或回退 .

这些占位符称为保留点。为了创建占位符，可如下使用SAVEPOINT语句： 

  

```sql
savepoint delete1
```

每个保留点都取标识它的唯一名字，以便在回退时， MySQL知道要回退到何处。为了回退到本例给出的保留点，可如下进行：  

```sql
rollback to delete1
```

> 可以在MySQL代码中设置任意多的保留点，越多越好。为什么呢？因为保留点越多，你就越能按自己的意愿灵活地进行回退  

> 保留点在事务处理完成（执行一条ROLLBACK或COMMIT）后自动释放。自MySQL 5以来，也可以用RELEASESAVEPOINT明确地释放保留点。