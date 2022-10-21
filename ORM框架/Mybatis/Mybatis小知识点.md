BLOB对应的类型是ByteArrayInputStream,就是二进制数据流。

------------------------

Oracle数据库不支持主键自增，因此不支持使用useGeneratedKeys来获取自增的主键返回，只能通过selectKey标签来获取，且order属性要设置未BEFORE，因为oracle需要先从序列中获取值，再把这个值当成主键插入数据库中。

各个数据库在selectKey标签中用于获取主键的SQL语句

- Oracle   SELECT SEQ_ID.nextval from dual
- MySQL   SELECT LAST_INSERT_ID()
- DB2   VALUES  IDENTITY_VAL_LOCAL().
- SQLSERVER   SELECT SCOPE_IDENTITY().
- CLOUDSCAPE    VALUES IDENTITY_VAL_LOCAL().
- DERBY     VALUES IDENTITY_VAL_LOCAL().
- HSQLDB    CALL   IDENTITY().
- SYBASE   SELECT @@IDENTITY.
- DB2_MF    SELECT IDENTITY_VAL_LOCAL() FROM SYSIBM.SYSDUMMY1.
- INFORMIX   select dbinfo('sqlca.sqlerrd1')  from systables where  tabid=1.

------------------

Mybatis使用注解的优缺点：

优点：对于需求比较简单的系统，效率比较高。

缺点：当SQL有变化时都需要重新编译代码，一般情况下不建议使用注解方式。



-------------

对于低版本MBG代码生成器，如果model生成代码注释设置了为不生成注释，则生成mapper.xml文件时是追加内容而不是覆盖原文件内容。

解决方案：pom中引入高版本mbg

```xml
<!-- MyBatis 生成器 -->
<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-core</artifactId>
    <version>1.3.7</version>
</dependency>
```

mbg配置文件中添加插件

```xml
<!--生成mapper.xml时覆盖原文件-->
<plugin type="org.mybatis.generator.plugins.UnmergeableXmlMappersPlugin" />
```

---------

mbg生成的代码默认不添加@Mapper注解，但是在service中使用@Autowired自动注入的时候会报错。

解决方案：自定义一个配置类用于扫描mapper包下面的接口

```java
@Configuration
@MapperScan("com.example.mall.learning.mbg.mapper")
public class MyBatisConfig {
}
```
