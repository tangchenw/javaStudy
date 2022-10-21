

1.三层架构
	**界面层：**和用户打交道的，接收用户的请求参数，显示处理结果的。（jsp,html,servlet)
	**业务逻辑层：**接收了界面层传递的数据，计算逻辑，调用数据库，获取数据
	**数据访问层：**就是访问数据库，执行对数据的查询，修改，删除等得的。
	三层对应的包
	界面层： controlller包（servlet)
	页面逻辑层:servlet包（xxxService类）
	数据访问层：dao包（xxxDao类）
	三层中类的交互
	用户使用界面层——>业务逻辑层——>数据访问层（持久层）——>数据库（mysql)
	三层对应的处理框架
	界面层——servlet——springmvc（框架）
	业务逻辑层——service类——spring（框架）
	数据访问层——dao类——mybatis(框架）

Mybatis做数据库操作强，但是他不能做其它的。mybatis，一个框架，代码在github

1)sql mapper:sql映射
2）Data Access Objects(DAOs):数据访问

模糊查询中like #{name},（不推荐使用）实际开发中用户不会去输入%%，该查询是使用PrepatedStatement占位符的方式 ？like ‘%${value}%’ 使用的是Statement对象的字符串拼接符拼接SQL
主键回填中设置useGenerateKeys="true",keyProperty="主键字段"；

# Mybatis动态SQL

Mybatis的动态SQL在xml里面支持的标签

- if
- choose(when、oterwise)
- trim(where、set)
- foreach
- bind

## where、set、trim用法

### where

where标签的作用：如果该标签包含的元素中有返回值，就插入一个where;如果where后面的字符串是以AND和OR开头的，就将它们剔除。

### set用法

set标签的作用：如果该标签包含的元素中有返回值，就插入一个set；如果set后面字符串以逗号结尾，剔除这个逗号。

### trim用法

where和set标签的功能都可以用trim标签来实现，并且在底层就是通过TrimSqlNode实现的。

where标签的trim实现

```xml
<trim prefix="where" prefixOverrides="AND |OR ">
</trim>
```

set标签对应的trim实现如下：

```xml
<trim prefix="SET" suffixOverride=","></trim>
```

trim标签属性介绍：

- prefix:当trim元素内包含内容时，会给内容增加prefix指定的前缀。
- prefixOverrdies:当trim元素内包含内容时，会把内容中匹配的前缀字符串去掉。
- suffix:当trim元素内包含内容时，会给内容增加suffix指定的后缀。
- suffixOverrides:当trim元素内包含内容时，会把内容中匹配的后缀字符串去掉。



![image-20220809153607632](C:\Users\汤琛\AppData\Roaming\Typora\typora-user-images\image-20220809153607632.png)



**bind用法**

bind标签可以使用OGNL表达式创建一个变量并将其绑定到上下文当中。

例子：

```xml
<if test="userName != null and userName != ''">
            and user_name like concat('%',#{userName},'%')
</if>
```

使用concat函数连接字符串，在MySQL中，这个函数支持多个参数，但是在oracle只支持两个参数，由于数据库语法的差异，如果更换数据库，有些SQL语句可能需要重写。使用bind标签可以避免更换数据库带来的一些麻烦。

```xml
<if test="userName != null and userName != ''">
    <bind name="userNameLike" value="'%'+ userName + '%'"/>
            and user_name like #{userNameLike}
</if>
```

### 多数据库支持

bind标签并不能解决更换数据库带来的所有问题，需要通过if标签以及Mybatis提供的databaseIdProvider数据库厂商标识配置。

在mybatis-config.xml（mybatis配置文件）中添加

```xml
<databaseIdProvider tpye="DB_VENDOR"/>
```

这里的DB_VENDOR会通过DatabaseMetaData#getDatabaseProductName()返回的字符串进行设置。由于通常情况下这个字符串都非常长而且相同产品的不同版本会返回不同的值，所以通常会通过设置属性别名来使其变短，如下：

```xml
    <databaseIdProvider type="DB_VENDOR">
        <property name="SQL Server" value="sqlserver"/>
        <property name="DB2" value="db2"/>
        <property name="Oracle" value="oracle"/>
        <property name="MySQL" value="mysql"/>
        <property name="PostgreSQL" value="postgresql"/>
        <property name="Derby" value="derby"/>
        <property name="HSQL" value="hsqldb"/>
        <property name="H2" value="h2"/>
    </databaseIdProvider>
```

使用：

```xml
#mysql用法
<select id="selectByUser" databaseId="mysql" resultType="tk.mybatis.simple.model.SysUser">
        select *
        from sys_user
        where 1 = 1
        <if test="userName != null and userName != ''">
            and user_name like concat('%',#{userName},'%')
        </if>
</select>
#Oracle用法
<select id="selectByUser" databaseId="oracle" resultType="tk.mybatis.simple.model.SysUser">
        select *
        from sys_user
        where 1 = 1
        <if test="userName != null and userName != ''">
            and user_name like '%'||#{userName}||'%'
        </if>
</select>
#上面代码的重复性还是太高了可以使用if进行判断来减少重复代码。
<select id="selectByUser" databaseId="mysql" resultType="tk.mybatis.simple.model.SysUser">
        select *
        from sys_user
        where 1 = 1
        <if test="userName != null and userName != ''">
            <if test="_databaseId = 'mysql'">
               and user_name like concat('%',#{userName},'%') 
            </if>
            <if test="_databaseId = 'oracle'">
               and user_name like '%'||#{userName}||'%'
            </if>
        </if>
</select>
```



# OGNL用法

Mybatis常用的OGNL表达式如下：

1. e1 or e2
2. e1 and e2
3. e1 == e2 或  e1 eq  e2
4. e1 != e2  或  e1 neq e2
5. e1  lt  e2:小于
6. e1 lte e2:小于等于，其它表示为gt(大于)、gte(大于等于)
7. e1 + e2、e1 *  e2、e1/e2、e1  -  e2、e1%e2
8. !e   或  not e:非，取反
9. e.method(args):调用对象方法
10. e.property:对象属性值
11. e1 [e2]:按索引取值（List,数组和Map)
12. @class@method(args):调用类的静态方法
13. @class@field:调用类的静态字段值

# Mybatis代码生成器

核心配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<!-- 配置生成器 -->
<generatorConfiguration>
    <!-- 可以用于加载配置项或者配置文件，在整个配置文件中就可以使用${propertyKey}的方式来引用配置项
        resource：配置资源加载地址，使用resource，MBG从classpath开始找，比如com/myproject/generatorConfig.properties
        url：配置资源加载地质，使用URL的方式，比如file:///C:/myfolder/generatorConfig.properties.
        注意，两个属性只能选址一个;

        另外，如果使用了mybatis-generator-maven-plugin，那么在pom.xml中定义的properties都可以直接在generatorConfig.xml中使用
    <properties resource="" url="" />
     -->

    <!-- 在MBG工作的时候，需要额外加载的依赖包
       location属性指明加载jar/zip包的全路径
   <classPathEntry location="/Program Files/IBM/SQLLIB/java/db2java.zip" />
     -->

    <!--
        context:生成一组对象的环境
        id:必选，上下文id，用于在生成错误时提示
        defaultModelType:指定生成对象的样式
            1，conditional：类似hierarchical；
            2，flat：所有内容（主键，blob）等全部生成在一个对象中；
            3，hierarchical：主键生成一个XXKey对象(key class)，Blob等单独生成一个对象，其他简单属性在一个对象中(record class)
        targetRuntime:
            1，MyBatis3：默认的值，生成基于MyBatis3.x以上版本的内容，包括XXXBySample；
            2，MyBatis3Simple：类似MyBatis3，只是不生成XXXBySample；
        introspectedColumnImpl：类全限定名，用于扩展MBG
    -->
    <context id="mysql" defaultModelType="hierarchical" targetRuntime="MyBatis3Simple" >

        <!-- 自动识别数据库关键字，默认false，如果设置为true，根据SqlReservedWords中定义的关键字列表；
            一般保留默认值，遇到数据库关键字（Java关键字），使用columnOverride覆盖
         -->
        <property name="autoDelimitKeywords" value="false"/>
        <!-- 生成的Java文件的编码 -->
        <property name="javaFileEncoding" value="UTF-8"/>
        <!-- 格式化java代码 -->
        <property name="javaFormatter" value="org.mybatis.generator.api.dom.DefaultJavaFormatter"/>
        <!-- 格式化XML代码 -->
        <property name="xmlFormatter" value="org.mybatis.generator.api.dom.DefaultXmlFormatter"/>

        <!-- beginningDelimiter和endingDelimiter：指明数据库的用于标记数据库对象名的符号，比如ORACLE就是双引号，MYSQL默认是`反引号； -->
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>

        <!-- 必须要有的，使用这个配置链接数据库
            @TODO:是否可以扩展
         -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql:///pss" userId="root" password="admin">
            <!-- 这里面可以设置property属性，每一个property属性都设置到配置的Driver上 -->
        </jdbcConnection>

        <!-- java类型处理器
            用于处理DB中的类型到Java中的类型，默认使用JavaTypeResolverDefaultImpl；
            注意一点，默认会先尝试使用Integer，Long，Short等来对应DECIMAL和 NUMERIC数据类型；
        -->
        <javaTypeResolver type="org.mybatis.generator.internal.types.JavaTypeResolverDefaultImpl">
            <!--
                true：使用BigDecimal对应DECIMAL和 NUMERIC数据类型
                false：默认,
                    scale>0;length>18：使用BigDecimal;
                    scale=0;length[10,18]：使用Long；
                    scale=0;length[5,9]：使用Integer；
                    scale=0;length<5：使用Short；
             -->
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>


        <!-- java模型创建器，是必须要的元素
            负责：1，key类（见context的defaultModelType）；2，java类；3，查询类
            targetPackage：生成的类要放的包，真实的包受enableSubPackages属性控制；
            targetProject：目标项目，指定一个存在的目录下，生成的内容会放到指定目录中，如果目录不存在，MBG不会自动建目录
         -->
        <javaModelGenerator targetPackage="com._520it.mybatis.domain" targetProject="src/main/java">
            <!--  for MyBatis3/MyBatis3Simple
                自动为每一个生成的类创建一个构造方法，构造方法包含了所有的field；而不是使用setter；
             -->
            <property name="constructorBased" value="false"/>

            <!-- 在targetPackage的基础上，根据数据库的schema再生成一层package，最终生成的类放在这个package下，默认为false -->
            <property name="enableSubPackages" value="true"/>

            <!-- for MyBatis3 / MyBatis3Simple
                是否创建一个不可变的类，如果为true，
                那么MBG会创建一个没有setter方法的类，取而代之的是类似constructorBased的类
             -->
            <property name="immutable" value="false"/>

            <!-- 设置一个根对象，
                如果设置了这个根对象，那么生成的keyClass或者recordClass会继承这个类；在Table的rootClass属性中可以覆盖该选项
                注意：如果在key class或者record class中有root class相同的属性，MBG就不会重新生成这些属性了，包括：
                    1，属性名相同，类型相同，有相同的getter/setter方法；
             -->
            <property name="rootClass" value="com._520it.mybatis.domain.BaseDomain"/>

            <!-- 设置是否在getter方法中，对String类型字段调用trim()方法 -->
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>


        <!-- 生成SQL map的XML文件生成器，
            注意，在Mybatis3之后，我们可以使用mapper.xml文件+Mapper接口（或者不用mapper接口），
                或者只使用Mapper接口+Annotation，所以，如果 javaClientGenerator配置中配置了需要生成XML的话，这个元素就必须配置
            targetPackage/targetProject:同javaModelGenerator
         -->
        <sqlMapGenerator targetPackage="com._520it.mybatis.mapper" targetProject="src/main/resources">
            <!-- 在targetPackage的基础上，根据数据库的schema再生成一层package，最终生成的类放在这个package下，默认为false -->
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>


        <!-- 对于mybatis来说，即生成Mapper接口，注意，如果没有配置该元素，那么默认不会生成Mapper接口
            targetPackage/targetProject:同javaModelGenerator
            type：选择怎么生成mapper接口（在MyBatis3/MyBatis3Simple下）：
                1，ANNOTATEDMAPPER：会生成使用Mapper接口+Annotation的方式创建（SQL生成在annotation中），不会生成对应的XML；
                2，MIXEDMAPPER：使用混合配置，会生成Mapper接口，并适当添加合适的Annotation，但是XML会生成在XML中；
                3，XMLMAPPER：会生成Mapper接口，接口完全依赖XML；
            注意，如果context是MyBatis3Simple：只支持ANNOTATEDMAPPER和XMLMAPPER
        -->
        <javaClientGenerator targetPackage="com._520it.mybatis.mapper" type="ANNOTATEDMAPPER" targetProject="src/main/java">
            <!-- 在targetPackage的基础上，根据数据库的schema再生成一层package，最终生成的类放在这个package下，默认为false -->
            <property name="enableSubPackages" value="true"/>

            <!-- 可以为所有生成的接口添加一个父接口，但是MBG只负责生成，不负责检查
            <property name="rootInterface" value=""/>
             -->
        </javaClientGenerator>

        <!-- 选择一个table来生成相关文件，可以有一个或多个table，必须要有table元素
            选择的table会生成一下文件：
            1，SQL map文件
            2，生成一个主键类；
            3，除了BLOB和主键的其他字段的类；
            4，包含BLOB的类；
            5，一个用户生成动态查询的条件类（selectByExample, deleteByExample），可选；
            6，Mapper接口（可选）

            tableName（必要）：要生成对象的表名；
            注意：大小写敏感问题。正常情况下，MBG会自动的去识别数据库标识符的大小写敏感度，在一般情况下，MBG会
                根据设置的schema，catalog或tablename去查询数据表，按照下面的流程：
                1，如果schema，catalog或tablename中有空格，那么设置的是什么格式，就精确的使用指定的大小写格式去查询；
                2，否则，如果数据库的标识符使用大写的，那么MBG自动把表名变成大写再查找；
                3，否则，如果数据库的标识符使用小写的，那么MBG自动把表名变成小写再查找；
                4，否则，使用指定的大小写格式查询；
            另外的，如果在创建表的时候，使用的""把数据库对象规定大小写，就算数据库标识符是使用的大写，在这种情况下也会使用给定的大小写来创建表名；
            这个时候，请设置delimitIdentifiers="true"即可保留大小写格式；

            可选：
            1，schema：数据库的schema；
            2，catalog：数据库的catalog；
            3，alias：为数据表设置的别名，如果设置了alias，那么生成的所有的SELECT SQL语句中，列名会变成：alias_actualColumnName
            4，domainObjectName：生成的domain类的名字，如果不设置，直接使用表名作为domain类的名字；可以设置为somepck.domainName，那么会自动把domainName类再放到somepck包里面；
            5，enableInsert（默认true）：指定是否生成insert语句；
            6，enableSelectByPrimaryKey（默认true）：指定是否生成按照主键查询对象的语句（就是getById或get）；
            7，enableSelectByExample（默认true）：MyBatis3Simple为false，指定是否生成动态查询语句；
            8，enableUpdateByPrimaryKey（默认true）：指定是否生成按照主键修改对象的语句（即update)；
            9，enableDeleteByPrimaryKey（默认true）：指定是否生成按照主键删除对象的语句（即delete）；
            10，enableDeleteByExample（默认true）：MyBatis3Simple为false，指定是否生成动态删除语句；
            11，enableCountByExample（默认true）：MyBatis3Simple为false，指定是否生成动态查询总条数语句（用于分页的总条数查询）；
            12，enableUpdateByExample（默认true）：MyBatis3Simple为false，指定是否生成动态修改语句（只修改对象中不为空的属性）；
            13，modelType：参考context元素的defaultModelType，相当于覆盖；
            14，delimitIdentifiers：参考tableName的解释，注意，默认的delimitIdentifiers是双引号，如果类似MYSQL这样的数据库，使用的是`（反引号，那么还需要设置context的beginningDelimiter和endingDelimiter属性）
            15，delimitAllColumns：设置是否所有生成的SQL中的列名都使用标识符引起来。默认为false，delimitIdentifiers参考context的属性

            注意，table里面很多参数都是对javaModelGenerator，context等元素的默认属性的一个复写；
         -->
        <table tableName="userinfo" >

            <!-- 参考 javaModelGenerator 的 constructorBased属性-->
            <property name="constructorBased" value="false"/>

            <!-- 默认为false，如果设置为true，在生成的SQL中，table名字不会加上catalog或schema； -->
            <property name="ignoreQualifiersAtRuntime" value="false"/>

            <!-- 参考 javaModelGenerator 的 immutable 属性 -->
            <property name="immutable" value="false"/>

            <!-- 指定是否只生成domain类，如果设置为true，只生成domain类，如果还配置了sqlMapGenerator，那么在mapper XML文件中，只生成resultMap元素 -->
            <property name="modelOnly" value="false"/>

            <!-- 参考 javaModelGenerator 的 rootClass 属性
            <property name="rootClass" value=""/>
             -->

            <!-- 参考javaClientGenerator 的  rootInterface 属性
            <property name="rootInterface" value=""/>
            -->

            <!-- 如果设置了runtimeCatalog，那么在生成的SQL中，使用该指定的catalog，而不是table元素上的catalog
            <property name="runtimeCatalog" value=""/>
            -->

            <!-- 如果设置了runtimeSchema，那么在生成的SQL中，使用该指定的schema，而不是table元素上的schema
            <property name="runtimeSchema" value=""/>
            -->

            <!-- 如果设置了runtimeTableName，那么在生成的SQL中，使用该指定的tablename，而不是table元素上的tablename
            <property name="runtimeTableName" value=""/>
            -->

            <!-- 注意，该属性只针对MyBatis3Simple有用；
                如果选择的runtime是MyBatis3Simple，那么会生成一个SelectAll方法，如果指定了selectAllOrderByClause，那么会在该SQL中添加指定的这个order条件；
             -->
            <property name="selectAllOrderByClause" value="age desc,username asc"/>

            <!-- 如果设置为true，生成的model类会直接使用column本身的名字，而不会再使用驼峰命名方法，比如BORN_DATE，生成的属性名字就是BORN_DATE,而不会是bornDate -->
            <property name="useActualColumnNames" value="false"/>


            <!-- generatedKey用于生成生成主键的方法，
                如果设置了该元素，MBG会在生成的<insert>元素中生成一条正确的<selectKey>元素，该元素可选
                column:主键的列名；
                sqlStatement：要生成的selectKey语句，有以下可选项：
                    Cloudscape:相当于selectKey的SQL为： VALUES IDENTITY_VAL_LOCAL()
                    DB2       :相当于selectKey的SQL为： VALUES IDENTITY_VAL_LOCAL()
                    DB2_MF    :相当于selectKey的SQL为：SELECT IDENTITY_VAL_LOCAL() FROM SYSIBM.SYSDUMMY1
                    Derby     :相当于selectKey的SQL为：VALUES IDENTITY_VAL_LOCAL()
                    HSQLDB    :相当于selectKey的SQL为：CALL IDENTITY()
                    Informix  :相当于selectKey的SQL为：select dbinfo('sqlca.sqlerrd1') from systables where tabid=1
                    MySql     :相当于selectKey的SQL为：SELECT LAST_INSERT_ID()
                    SqlServer :相当于selectKey的SQL为：SELECT SCOPE_IDENTITY()
                    SYBASE    :相当于selectKey的SQL为：SELECT @@IDENTITY
                    JDBC      :相当于在生成的insert元素上添加useGeneratedKeys="true"和keyProperty属性
            <generatedKey column="" sqlStatement=""/>
             -->

            <!--
                该元素会在根据表中列名计算对象属性名之前先重命名列名，非常适合用于表中的列都有公用的前缀字符串的时候，
                比如列名为：CUST_ID,CUST_NAME,CUST_EMAIL,CUST_ADDRESS等；
                那么就可以设置searchString为"^CUST_"，并使用空白替换，那么生成的Customer对象中的属性名称就不是
                custId,custName等，而是先被替换为ID,NAME,EMAIL,然后变成属性：id，name，email；

                注意，MBG是使用java.util.regex.Matcher.replaceAll来替换searchString和replaceString的，
                如果使用了columnOverride元素，该属性无效；

            <columnRenamingRule searchString="" replaceString=""/>
             -->


            <!-- 用来修改表中某个列的属性，MBG会使用修改后的列来生成domain的属性；
               column:要重新设置的列名；
               注意，一个table元素中可以有多个columnOverride元素哈~
             -->
            <columnOverride column="username">
                <!-- 使用property属性来指定列要生成的属性名称 -->
                <property name="property" value="userName"/>

                <!-- javaType用于指定生成的domain的属性类型，使用类型的全限定名
                <property name="javaType" value=""/>
                 -->

                <!-- jdbcType用于指定该列的JDBC类型
                <property name="jdbcType" value=""/>
                 -->

                <!-- typeHandler 用于指定该列使用到的TypeHandler，如果要指定，配置类型处理器的全限定名
                    注意，mybatis中，不会生成到mybatis-config.xml中的typeHandler
                    只会生成类似：where id = #{id,jdbcType=BIGINT,typeHandler=com._520it.mybatis.MyTypeHandler}的参数描述
                <property name="jdbcType" value=""/>
                -->

                <!-- 参考table元素的delimitAllColumns配置，默认为false
                <property name="delimitedColumnName" value=""/>
                 -->
            </columnOverride>

            <!-- ignoreColumn设置一个MGB忽略的列，如果设置了改列，那么在生成的domain中，生成的SQL中，都不会有该列出现
               column:指定要忽略的列的名字；
               delimitedColumnName：参考table元素的delimitAllColumns配置，默认为false

               注意，一个table元素中可以有多个ignoreColumn元素
            <ignoreColumn column="deptId" delimitedColumnName=""/>
            -->
        </table>

    </context>

</generatorConfiguration>
```

# Mybatis高级查询

## 一对一映射

### 自动映射处理一对一关系

SysUser.class类中添加对应映射对象

```java
    private SysRole sysRole;

    public SysRole getSysRole() {
        return sysRole;
    }

    public void setSysRole(SysRole sysRole) {
        this.sysRole = sysRole;
    }	
```

UserMapper.xml中直接使用别名进行映射

```java
    <select id="selectUserAndRoleById" resultType="tk.mybatis.simple.model.SysUser">
        select u.id,u.user_name userName,u.user_password userPassword,u.user_email userEmail,
               u.user_info userInfo,u.head_img headImg,u.create_time createTime,r.id "sysRole.id",
               r.role_name "sysRole.roleName",r.enabled "sysRole.enabled",r.create_by "sysRole.createBy",
               r.create_time "sysRole.createTime"
        from sys_user u
        inner join sys_user_role ur on ur.user_id = u.id
        inner join sys_role r on r.id = ur.role_id
        where u.id = #{id}
    </select>
```

### ResultMap配置一对一映射

UserMapper.xml中配置resultMap映射关系集

```xml
 <resultMap id="userRoleMap" type="tk.mybatis.simple.model.SysUser">
        <id property="id" column="id"></id>
        <result property="userName" column="user_name"/>
        <result property="userPassword" column="user_password"/>
        <result property="userEmail" column="user_email"/>
        <result property="userInfo" column="user_info"/>
        <result property="headImg" column="head_img" jdbcType="BLOB"/>
        <result property="createTime" column="create_time" jdbcType="TIMESTAMP"/>

        <result property="sysRole.id"  column="role_id"/>
        <result property="sysRole.roleName" column="role_name"/>
        <result property="sysRole.enabled"  column="enabled"/>
        <result property="sysRole.createBy"    column="create_by"/>
        <result property="sysRole.createTime"  column="role_create_time" jdbcType="TIMESTAMP"/>
    </resultMap>
	<!--直接处理-->
<select id="selectUserAndRoleById2" resultMap="userRoleMap">
        select u.id,u.user_name,u.user_password ,u.user_email,
               u.user_info,u.head_img,u.create_time,r.id role_id,
               r.role_name,r.enabled,r.create_by,
               r.create_time role_create_time
        from sys_user u
                 inner join sys_user_role ur on ur.user_id = u.id
                 inner join sys_role r on r.id = ur.role_id
        where u.id = #{id}
    </select>
```

**优化结果集写法**

```xml
   <resultMap id="userMap" type="tk.mybatis.simple.model.SysUser">
        <id property="id" column="id"></id>
        <result property="userName" column="user_name"/>
        <result property="userPassword" column="user_password"/>
        <result property="userEmail" column="user_email"/>
        <result property="userInfo" column="user_info"/>
        <result property="headImg" column="head_img" jdbcType="BLOB"/>
        <result property="createTime" column="create_time" jdbcType="TIMESTAMP"/>
    </resultMap>
    <!--利用MBG逆向工程生成基础Map之后继承基础map再添加映射对象结果集-->
    <resultMap id="userRoleMap" type="tk.mybatis.simple.model.SysUser" extends="userMap">
        <result property="sysRole.id"  column="role_id"/>
        <result property="sysRole.roleName" column="role_name"/>
        <result property="sysRole.enabled"  column="enabled"/>
        <result property="sysRole.createBy"    column="create_by"/>
        <result property="sysRole.createTime"  column="role_create_time" jdbcType="TIMESTAMP"/>
    </resultMap>
```

**association标签优化**

```xml
   <resultMap id="userRoleMap" type="tk.mybatis.simple.model.SysUser" extends="userMap">
        <!--association标签作用：
        property:对应实体类的属性名，必填
        javaType:属性对应的java类型
        resultMap:直接使用现有的resultMap,此处不需要配置
        columnPrefix:查询列前缀，子标签的column属性配置时可以省略前缀，但是在写sql时必须都加上前缀-->
        <association property="sysRole" columnPrefix="role_"    javaType="tk.mybatis.simple.model.SysRole">
            <result property="sysRole.id"  column="role_id"/>
            <result property="sysRole.roleName" column="role_name"/>
            <result property="sysRole.enabled"  column="enabled"/>
            <result property="sysRole.createBy"    column="create_by"/>
            <result property="sysRole.createTime"  column="create_time" jdbcType="TIMESTAMP"/>
        </association>
    </resultMap>

 <select id="selectUserAndRoleById2" resultMap="userRoleMap">
        select u.id,u.user_name,u.user_password ,u.user_email,
               u.user_info,u.head_img,u.create_time,r.id role_id,
               r.role_name role_role_name,r.enabled role_eanbled,r.create_by role_create_by,
               r.create_time role_create_time
        from sys_user u
                 inner join sys_user_role ur on ur.user_id = u.id
                 inner join sys_role r on r.id = ur.role_id
        where u.id = #{id}
    </select>
```

**再次优化对象结果集**

```xml
    <!--直接提取映射对象-->
    <resultMap id="RoleMap" type="tk.mybatis.simple.model.SysRole">
        <result property="id"  column="id"/>
        <result property="roleName" column="role_name"/>
        <result property="enabled"  column="enabled"/>
        <result property="createBy"    column="create_by"/>
        <result property="createTime"  column="create_time" jdbcType="TIMESTAMP"/>
    </resultMap>

    <resultMap id="userRoleMap" type="tk.mybatis.simple.model.SysUser" extends="userMap">
        <association property="sysRole" columnPrefix="role_" resultMap="RoleMap" >
        </association>
    </resultMap>
```

### association标签的嵌套查询

association标签进行嵌套查询的常用标签如下：

- select:另一个映射查询的id，Mybatis会额外执行这个查询获取嵌套对象的结果。
- column:列名（或别名），将主查询中列的结果作为嵌套查询的参数，配置方式如cloumn={prop1=col1,prop2=col2},prop1和prop2将作为嵌套查询的参数。
- fetchType:数据加载方式，可选值为lazy和eager，分别为延迟加载和积极加载，这个配置会覆盖全局的lazyLoadingEnabled配置。

**UserMapper.xml**

```xml
 <!--嵌套查询,启用懒加载，只有在调用了sysRole对象的时候才会被加载-->
    <resultMap id="userRoleMapSelect" type="tk.mybatis.simple.model.SysUser" extends="userMap">
        <association property="sysRole" column="{id=role_id}"
                     select="tk.mybatis.simple.mapper.RoleMapper.selectRoleById" fetchType="lazy"/>
    </resultMap>
    
    <!--主查询，查询出role_id之后，将role_id的结果传递给嵌套查询selectRoleById通过{id=role_id}，将role_id赋值给嵌套查询的参数id-->
    <select id="selectUserAndRoleByIdSelect"    resultMap="userRoleMapSelect">
        select u.id,u.user_name,u.user_password,u.user_email,u.user_info,u.head_img,u.create_time,
               ur.role_id from sys_user u
        inner join sys_user_role ur on ur.user_id = u.id
        where u.id = #{id}
    </select>
```

**RoleMapper.xml**

```xml
  <select id="selectRoleById" resultType="tk.mybatis.simple.model.SysRole">
        select * from sys_role where id = #{id}
    </select>
```

## 一对多映射

### collection集合的嵌套结果映射

SysUser.class类中添加对应映射对象

```java
private List<SysRole> roleList;

    public List<SysRole> getRoleList() {
        return roleList;
    }

    public void setRoleList(List<SysRole> roleList) {
        this.roleList = roleList;
    }
```

UserMapper.xml中直接使用collection标签进行映射

```xml
   <resultMap id="userRoleListMap" type="tk.mybatis.simple.model.SysUser" extends="userMap">
        <id property="id" column="id"></id>
        <result property="userName" column="user_name"/>
        <result property="userPassword" column="user_password"/>
        <result property="userEmail" column="user_email"/>
        <result property="userInfo" column="user_info"/>
        <result property="headImg" column="head_img" jdbcType="BLOB"/>
        <result property="createTime" column="create_time" jdbcType="TIMESTAMP"/>
        <collection property="roleList" columnPrefix="role_" resultMap="tk.mybatis.simple.mapper.RoleMapper.RoleMap"    javaType="tk.mybatis.simple.model.SysRole">
        </collection>
    </resultMap>

 <!--一对多映射结果集查询-->
    <select id="selectAllUserAndRoles" resultMap="userRoleListMap">
        select u.id,u.user_name,u.user_password ,u.user_email,
               u.user_info,u.head_img,u.create_time,r.id role_id,
               r.role_name role_role_name,r.enabled role_eanbled,r.create_by role_create_by,
               r.create_time role_create_time
        from sys_user u
                 inner join sys_user_role ur on ur.user_id = u.id
                 inner join sys_role r on r.id = ur.role_id
    </select>
```

**一对多三表映射**

SysRole.java添加映射对象

```java
  private List<SysPrivilege> privilegeList;


    public List<SysPrivilege> getPrivilegeList() {
        return privilegeList;
    }

    public void setPrivilegeList(List<SysPrivilege> privilegeList) {
        this.privilegeList = privilegeList;
    }
```

PrivilegeMapper.xml添加映射结果集

```xml
<resultMap id="privilegeMap" type="tk.mybatis.simple.model.SysPrivilege">
        <id property="id"   column="id"/>
        <result property="privilegeName" column="privilege_name"/>
        <result property="privilegeUrl" column="privilege_url"/>
    </resultMap>
```

RoleMapper.xml添加映射结果集

```xml
<resultMap id="RoleMap" type="tk.mybatis.simple.model.SysRole">
        <result property="id"  column="id"/>
        <result property="roleName" column="role_name"/>
        <result property="enabled"  column="enabled"/>
        <result property="createBy"    column="create_by"/>
        <result property="createTime"  column="create_time" jdbcType="TIMESTAMP"/>
    </resultMap>
	 <!--设置引入privilegeMapper结果集-->
    <resultMap id="rolePrivilegeListMap" type="tk.mybatis.simple.model.SysRole" extends="RoleMap">
        <collection property="privilegeList"    columnPrefix="privilege_" resultMap="tk.mybatis.simple.mapper.PrivilegeMapper.privilegeMap"/>
    </resultMap>
```

UserMapper.xml更改映射结果集

```xml
    <resultMap id="userRoleListMap" type="tk.mybatis.simple.model.SysUser" extends="userMap">
        <collection property="roleList" columnPrefix="role_" resultMap="tk.mybatis.simple.mapper.RoleMapper.rolePrivilegeListMap">
        </collection>
    </resultMap>

<!--更改添加privielege一对多映射结果集查询-->
    <select id="selectAllUserAndRoles" resultMap="userRoleListMap">
        select u.id,u.user_name,u.user_password ,u.user_email,
               u.user_info,u.head_img,u.create_time,r.id role_id,
               r.role_name role_role_name,r.enabled role_eanbled,r.create_by role_create_by,
               r.create_time role_create_time,
               p.id role_privilege_id,p.privilege_name role_privilege_privilege_name,
               p.privilege_url role_privilege_privilege_rul
        from sys_user u
                 inner join sys_user_role ur on ur.user_id = u.id
                 inner join sys_role r on r.id = ur.role_id
                 inner join sys_role_privilege rp on rp.role_id = r.id
                 inner join sys_privilege p on p.id = rp.privilege_id
    </select>
```

**优化SysRole属性**

添加CreateInfo属性类

```java
public class CreateInfo {

    private String createBy;

    private Date createTime;

    public String getCreateBy() {
        return createBy;
    }

    public void setCreateBy(String createBy) {
        this.createBy = createBy;
    }

    public Date getCreateTime() {
        return createTime;
    }

    public void setCreateTime(Date createTime) {
        this.createTime = createTime;
    }
}
```

SysRole.java中引入

```java
  private CreateInfo createInfo;

    public CreateInfo getCreateInfo() {
        return createInfo;
    }

    public void setCreateInfo(CreateInfo createInfo) {
        this.createInfo = createInfo;
    }
```

更改RoleMapper.xml中基础结果映射集

```xml
 <resultMap id="RoleMap" type="tk.mybatis.simple.model.SysRole">
        <result property="id"  column="id"/>
        <result property="roleName" column="role_name"/>
        <result property="enabled"  column="enabled"/>
        <association property="createInfo">
            <result property="createBy"    column="create_by"/>
            <result property="createTime"  column="create_time" jdbcType="TIMESTAMP"/>
        </association>
    </resultMap>
```

**自下而上嵌套查询**

PrivilegeMapper.xml

```xml
<select id="selectPrivilegeByRoleId" resultMap="privilegeMap">
        select p.*
        from sys_privilege p
        inner join sys_role_privilege rp on rp.privilege_id = p.id
        where role_id = #{roleId}
    </select>
```

RoleMapper.xml

```xml
 <resultMap id="rolePrivilegeListMapSelect" extends="RoleMap"   type="tk.mybatis.simple.model.SysRole">
        <collection property="privilegeList" fetchType="lazy" column="{roleId=id}"
                    select="tk.mybatis.simple.mapper.PrivilegeMapper.selectPrivilegeByRoleId"/>
    </resultMap>

    <select id="selectRoleByUserId" resultMap="rolePrivilegeListMapSelect">
        select r.id,r.role_name,r.enabled,r.create_by,r.create_time
        from sys_role r
        inner join sys_user_role ur on ur.role_id = r.id
        where ur.user_id = #{userId}
    </select>
```

UserMapper.xml

```xml
   <resultMap id="userRoleListMapSelect" type="tk.mybatis.simple.model.SysUser" extends="userMap">
        <collection property="roleList" fetchType="lazy" select="tk.mybatis.simple.mapper.RoleMapper.selectRoleByUserId"
                    column="{userId=id}"/>
    </resultMap>
    
     <select id="selectAllUserAndRolesSelect" resultMap="userRoleListMapSelect">
        select u.id,u.user_name,u.user_password,u.user_email,u.user_info,u.head_img,
               u.create_time
        from sys_user u
        where u.id = #{id}
    </select>
```

最后在UserMapper接口中添加对应方法

```java
SysUser selectAllUserAndRolesSelect(Long id);
```

## 鉴别器映射

discriminator鉴别器标签，类似于java中的switch语句。

discriminator标签常用的属性：

- column:该属性用于设置要进行鉴别比较值的列。
- javaType:该属性用于指定列的类型，保证使用相同的java类型来比较值。

discriminator标签可以有1个或多个case标签，case标签包含以下三个属性。

- value:该值为discriminator指定的column用来匹配的值。
- resultMap:当column的值和value的值匹配时，可以配置使用resultMap指定的映射，resultMap优先级高于resultMap.
- resultType:当cloumn的值和value的值匹配时，用于配置使用resultType指定的映射。

**RoleMapper.xml中添加鉴别器映射**

```xml
 
<resultMap id="rolePrivilegeListMapSelect" extends="RoleMap"   type="tk.mybatis.simple.model.SysRole">
        <collection property="privilegeList" fetchType="lazy" column="{roleId=id}"
                    select="tk.mybatis.simple.mapper.PrivilegeMapper.selectPrivilegeByRoleId"/>
    </resultMap>
<resultMap id="RoleMap" type="tk.mybatis.simple.model.SysRole">
        <result property="id"  column="id"/>
        <result property="roleName" column="role_name"/>
        <result property="enabled"  column="enabled"/>
        <association property="createInfo">
            <result property="createBy"    column="create_by"/>
            <result property="createTime"  column="create_time" jdbcType="TIMESTAMP"/>
        </association>
    </resultMap>
    <!--说明，根据enabled属性，case 为判断enabled属性的值，为1则使用rolePrivilegeListMapSelect,为0禁用角色则启用RoleMap结果集-->
<resultMap id="rolePrivilegeListMapChoose" type="tk.mybatis.simple.model.SysRole">
        <discriminator column="enabled" javaType="int">
            <case value="1" resultMap="rolePrivilegeListMapSelect"></case>
            <case value="0" resultMap="RoleMap"></case>
        </discriminator>
    </resultMap>

 <select id="selectRoleByUserIdChoose" resultMap="rolePrivilegeListMapChoose">
        select r.id,r.role_name,r.enabled,r.create_by,r.create_time
        from sys_role r
        inner join sys_user_role ur.role_id = r.id
        where ur.user_id = #{userId}
    </select>
```

RoleMapper.xml中添加对应方法

```java
    List<SysRole> selectRoleByUserIdChoose(Long userId);
```



## 存储过程

**根据用户id查询其它数据**

```sql
DROP PROCEDURE IF EXISTS `select_user_by_id`;
#结束符修改为；；
DELIMITER ;;
#存储过程创建
CREATE PROCEDURE `select_user_by_id`(
    #入参
	IN userId BIGINT,
    #出参
	OUT userName VARCHAR(50),
	OUT userPassword VARCHAR(50),
	OUT userEmail VARCHAR(50),
	OUT userInfo TEXT,
	OUT headImg BLOB,
	OUT createTime DATETIME
)
#作用于
BEGIN
#根据id查询数据
SELECT user_name,user_password,user_email,user_info,head_img,create_time
INTO userName,userPassword,userEmail,userInfo,headImg,createTime
from sys_user
where id = userId;
#使用结束符结束存储过程;;
END
;;
#修改回原来的结束符
DELIMITER;
```

mapper使用：

```xml
    <!--存储过程,sql类型为callable,存储过程用不上mybatis二级缓存，防止某些错误故设置为false-->
    <select id="selectUserById" statementType="CALLABLE" useCache="false">
        {call select_user_by_id(
            #{id,mode=IN},
            #{userName,mode = OUT,jdbcType=VARCHAR},
            #{userPassword,mode=OUT,jdbcType=VARCHAR},
            #{userEmail,mode=OUT, jdbcType=VARCHAR},
            #{userInfo,mode=OUT,jdbcType=VARCHAR},
            #{headImg,mode=OUT,jdbcType=BLOB,javaType=_byte[]},
            #{createTime,mode=OUT,jdbcType=TIMESTAMP}
            )}
    </select>
```

接口添加：

```java
void selectUserById(SysUser user);
```

功能测试：

```java
@Test
public void testSelectUserById(){
    SqlSession sqlSession = getSqlSession();
    try {
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        SysUser sysUser = new SysUser();
        sysUser.setId(1L);
        mapper.selectUserById(sysUser);
        Assert.assertNotNull(sysUser.getUserName());
        System.out.println("用户名:"+sysUser.getUserName());
    } finally {
        sqlSession.close();
    }
}
```

**简单根据用户名和分页参数进行查询，返回总数和分页数据**

```sql
DROP PROCEDURE IF EXISTS `select_user_page`;
DELIMITER ;;
CREATE PROCEDURE `select_user_page`(
	IN userName VARCHAR(50),
	IN _offset BIGINT,
	IN _limit BIGINT,
	OUT total BIGINT
	)
BEGIN

SELECT count(*) INTO total
from sys_user
where user_name like concat('%',userName,'%');

SELECT * from sys_user
where user_name like concat('%',userName,'%')
limit _offset,_limit;
END
;;
DELIMITER ;
```

UserMapper.xml使用：

```xml
<!--调用存储过程，使用Map接受返回参数-->
<select id="selectUserPage" statementType="CALLABLE"    useCache="false" resultMap="userMap">
    {call select_user_page(
        #{userName,mode=IN},
        #{offset,mode=IN},
        #{limit,mode=IN},
        #{total,mode=OUT,jdbcType=BIGINT}
        )}
</select>
```

接口添加：

```xml
List<SysUser> selectUserPage(Map<String,Object> params);
```

功能测试：

```java
@Test
public void testSelectUserPage(){
    SqlSession sqlSession = getSqlSession();
    try {
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        Map<String,Object> params = new HashMap<>();
        params.put("userName","ad");
        params.put("offset",0);
        params.put("limit",10);
        List<SysUser> users = mapper.selectUserPage(params);
        //此处可以获取到total，且可以获取到最后的结果集。因为该方法通过total出参返回了查询的总数，resultMap设置了最后的结果集
        Long total = (Long) params.get("total");
        System.out.println("总数"+total);
        for (SysUser user : users) {
            System.out.println("用户名："+user.getUserName());
        }
    } finally {
        sqlSession.close();
    }
}
```

**插入和删除用户和用户角色关联数据**

```xml
<select id="insertUserAndRoles" statementType="CALLABLE">
    {call insert_user_and_roles(
        #{user.id,mode=OUT,jdbcType=BIGINT},
        #{user.userName,mode=IN},
        #{user.userPassword,mode=IN},
        #{user.userEmail,mode=IN},
        #{user.userInfo,mode=IN},
        #{user.headImg,mode=IN,jdbcType=BLOB},
        #{user.createTime,mode=OUT,jdbcType=TIMESTAMP},
        #{roleIds,mode=IN}
        )}
</select>

<delete id="deleteUserById" statementType="CALLABLE">
    {call delete_user_by_id(#{id,mode=IN})}
</delete>
```

接口添加

```java
void insertUserAndRoles(@Param("user")SysUser user,@Param("roleIds")String roleIds);

int deleteUserById(Long id);
```

功能测试

```java
@Test
public void testInsertAndDelete(){
    SqlSession sqlSession = getSqlSession();
    try {
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        SysUser user = new SysUser();
        user.setUserName("test1");
        user.setUserPassword("123456");
        user.setUserEmail("test@qq.com");
        user.setUserInfo("test info");
        user.setHeadImg(new byte[]{1,2,3});
        mapper.insertUserAndRoles(user,"1,2");
        Assert.assertNotNull(user.getId());
        Assert.assertNotNull(user.getCreateTime());
        mapper.deleteUserById(user.getId());
    } finally {
        sqlSession.close();
    }
}
```

## 使用枚举或其它对象

### 使用Mybatis提供的枚举处理器

新建对应枚举类

```java
public enum Enabled {
    //禁用
    disabled,
    //启用
    enabled;
}
```

修改实体类中属性的类型为枚举类型

```java
    private Enabled enabled;

    public Enabled getEnabled() {
        return enabled;
    }

    public void setEnabled(Enabled enabled) {
        this.enabled = enabled;
    }
```

功能测试

```java
@Test
public void testUpdateById(){
    SqlSession sqlSession = getSqlSession();
    try {
        RoleMapper mapper = sqlSession.getMapper(RoleMapper.class);
        SysRole sysRole = mapper.selectById(2L);
        Assert.assertEquals(Enabled.enabled,sysRole.getEnabled());
        sysRole.setEnabled(Enabled.disabled);
        mapper.updateRoleById(sysRole);
    } finally {
        sqlSession.rollback();
        sqlSession.close();
    }
}
```

会发现上面代码测试时会报错，这是因为mybatis默认是使用EnumTypeHandler这个类来处理枚举类型，这个处理器只会对枚举的字面量进行处理（也就是disabled枚举值会当作是"disabled"字符串来处理）而数据库中disabled为int类型，因此会报错。所以想要处理这种情况可以使用EnumOrdinalTypeHandler处理器，它会使用枚举类型的索引值来进行处理（默认是先后顺序从0开始）

因此在mybatis的配置文件中添加该处理器即可

```xml
    <typeHandlers>
        <typeHandler handler="org.apache.ibatis.type.EnumOrdinalTypeHandler" javaType="tk.mybatis.simple.type.Enabled"/>
    </typeHandlers>
```

当然还有时候枚举值既不想用字面值，又不想用索引值的情况，这个时候就需要自己实现类型处理器了。

### 自定义的类型处理器

自定义枚举类

```java
public enum Enabled {
    //启用
    enabled(1),
    //禁用
    disabled(0);

    private final int value;

    private Enabled(int value){
        this.value = value;
    }

    int getValue(){
        return value;
    }
}
```

实体类中引用枚举类

```java
private Enabled enabled;

public Enabled getEnabled() {
    return enabled;
}

public void setEnabled(Enabled enabled) {
    this.enabled = enabled;
}
```

使用枚举类对应的自定义类型处理器（新建EnabledTypeHandler类）

```java
public class EnabledTypeHandler implements TypeHandler<Enabled> {

    private final Map<Integer,Enabled> enabledMap = new HashMap<>();

    public EnabledTypeHandler() {
        for (Enabled value : Enabled.values()) {
            enabledMap.put(value.getValue(),value);
        }
    }

    public EnabledTypeHandler(Class<?> type){
        this();
    }

    @Override
    public void setParameter(PreparedStatement ps, int i, Enabled parameter, JdbcType jdbcType) throws SQLException {
        ps.setInt(i,parameter.getValue());
    }

    @Override
    public Enabled getResult(ResultSet rs, String columnName) throws SQLException {
        Integer value = rs.getInt(columnName);
        return enabledMap.get(value);
    }

    @Override
    public Enabled getResult(ResultSet rs, int columnIndex) throws SQLException {
        Integer value = rs.getInt(columnIndex);
        return enabledMap.get(value);
    }

    @Override
    public Enabled getResult(CallableStatement cs, int columnIndex) throws SQLException {
        Integer value = cs.getInt(columnIndex);
        return enabledMap.get(value);
    }
}
```

修改mybatis配置文件中枚举处理类，使用自定义处理类

```xml
<typeHandlers>
    <typeHandler handler="tk.mybatis.simple.type.EnabledTypeHandler" javaType="tk.mybatis.simple.type.Enabled"/>
</typeHandlers>
```

# Mybatis缓存配置

## 一级缓存

一级缓存也叫本地缓存，mybatis默认开启并且不能控制，作用于sqlSession全生命周期

一级缓存测试

```java
@Test
public void testL1Cache(){
    SqlSession sqlSession = getSqlSession();
    SysUser sysUser = null;
    try {
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        //第一次查询id=1用户
        sysUser = mapper.selectById(1L);
        //对当前获取的对象进行赋值
        sysUser.setUserName("New Name");
        //第二次查询id=1的用户，发现没有执行查询语句
        SysUser sysUser1 = mapper.selectById(1L);
        //第二次查询得到的对象的用户名和第一次获取的对象的用户名相同
        Assert.assertEquals("New Name",sysUser1.getUserName());
        //而且对象也相同，这就是mybatis的一级缓存在起作用
        Assert.assertEquals(sysUser,sysUser1);
    } finally {
        sqlSession.close();
    }
    //开启新的sql
    System.out.println("开启新的sqlSession");
    sqlSession = getSqlSession();
    try {
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        //同样查询，执行了sql语句
        SysUser sysUser1 = mapper.selectById(1L);
        //判断查询出来的用户名是不是New Name，发现不是，这就证明mybatis缓存只能作用于sqlSession的生命周期
        Assert.assertNotEquals("New Name",sysUser1.getUserName());
        //同样，对象也不相同
        Assert.assertNotEquals(sysUser,sysUser1);
        //执行删除操作，mybatis执行insert,update,delete操作都会清除缓存
        mapper.deleteById(2L);
        //再次执行同样的查询语句，发现执行了sql，证明确实会清除缓存
        SysUser sysUser2 = mapper.selectById(1L);
        Assert.assertNotEquals(sysUser1,sysUser2);
    } finally {
        sqlSession.close();
    }
}
```

想要去除缓存带来的影响可以对使用的查询用户添加flushCache标签,将刷新缓存设置为true即可

```xml
<select id="selectById" flushCache="true" resultMap="userMap">
    select * from sys_user where id = #{id}
</select>
```

## 二级缓存

Mybatis的二级缓存可以理解为存在于sqlSessionFactory的生命周期中，当存在多个SqlSessionFactory时，它们的缓存都是绑定在各自对象上面的，缓存数据在一般情况下是不相通的，只有在使用Redis这样的缓存数据库时，才可以共享缓存。

### 配置二级缓存

**Mapper.xml中配置二级缓存**

在命名空间中添加<cache/>标签即可开启二级缓存

```xml
<mapper namespace="tk.mybatis.simple.mapper.RoleMapper">
    <cache/>
    </mapper>
```

默认的二级缓存有以下效果

- 映射语句文件中所有select语句将会被缓存。
- 映射语句文件中所有的insert、update、delete语句会刷新缓存
- 缓存会使用least recently used(LRU算法，最近最少使用的)来回收
- 根据时间表（如No flush interval，没有刷新间隔），缓存不会以任何时间顺序来刷新。
- 缓存会存储集合或对象（无论查询方法返回什么类型的值）的1024个引用。
- 缓存会被视为read/write（可读/可写）的，意味着对象检索不是共享的，而且可以安全地被调用者修改，而不干扰其它调用者或线程所做的潜在修改。

所有这些属性都可以通过缓存标签的属性来修改，如下：

```xml
<cache eviction="FIFO" flushInterval="60000" size="512" readOnly="true"/>
```

创建了一个先进先出的缓存，并且每隔60s刷新一次，存储集合或对象的512个引用，而且返回的对象被认为是可读的，因此在不同线程中的调用者之间修改它们会导致冲突。cahce可配置的属性如下：

- eviction（回收策略）
  1. LRU（最近最少使用的):移除最长时间不被使用的对象，这是默认值。
  2. FIFO（先进先出）：按对象进入缓存的顺序来移除它们
  3. SOFT(软引用)：移除基于垃圾回收器状态和软引用规则的对象。
  4. WEAK(弱引用)：更积极地移除基于垃圾收集器状态和弱引用规则的对象。

- flushInterval(刷新间隔)：可以被设置为任意的正整数，而且它们代表一个合理的毫秒形式的时间段。默认情况不设置，即没有刷新时间，缓存仅仅在调用语句时刷新。
- size(引用数目)：可以被设置为任意的正整数，要记住缓存的对象数目和运行环境的可用内存资源数目，默认是1024.
- readOnly(只读)：属性可以被设置为true或者false。只读的缓存会给所有调用者返回缓存对象的相同实例，因此这些对象不能被修改，这提供了很重要的性能优势。可读写的缓存会通过序列化返回缓存对象的数据拷贝，这种方式会慢一些，但是安全，因此默认是flase.

**Mapper接口中配置二级缓存**

添加CacheNamespace注解即可，该注解同样可以配置多个属性

```java
@CacheNamespace(
        eviction = FifoCache.class,
        flushInterval = 60000,
        size = 512,
        readWrite = true
)
public interface RoleMapper {}
```


```xml
<!--参照缓存配置-->
<cache-ref namespace="tk.mybatis.simple.mapper.RoleMapper"/>
```

功能测试

```java
@Test
public void testL2Cache(){
    SqlSession sqlSession = getSqlSession();
    SysRole role1 = null;
    try {
        RoleMapper mapper = sqlSession.getMapper(RoleMapper.class);
        role1 = mapper.selectById(1L);
        role1.setRoleName("New Name");
        SysRole role2 = mapper.selectById(1L);
        //相等，用上了一级缓存
        Assert.assertEquals("New Name",role2.getRoleName());
        //相等，一级缓存
        Assert.assertEquals(role1,role2);
    } finally {
        sqlSession.close();
    }
    System.out.println("开启新的SqlSession");
    sqlSession = getSqlSession();
    try {
        RoleMapper mapper = sqlSession.getMapper(RoleMapper.class);
        SysRole role2 = mapper.selectById(1L);
        //相等，二级缓存
        Assert.assertEquals("New Name",role2.getRoleName());
        //不相等，二级缓存反序列化出来的对象是不相等的，虽然它们里面的属性值相等（可以理解为克隆）
        Assert.assertNotEquals(role1,role2);
        SysRole role3 = mapper.selectById(1L);
        //同样不相等，反序列化出来的对象
        Assert.assertNotEquals(role2,role3);
    } finally {
        sqlSession.close();
    }

}
```

## 集成Redis缓存

添加mybatis集成redis二级缓存依赖

```xml
<dependency>
    <groupId>org.mybatis.caches</groupId>
    <artifactId>mybatis-redis</artifactId>
    <version>1.0.0-beta2</version>
</dependency>
```

添加redis配置文件redis.properties

```properties
host=localhost
port=6379
connectionTimeout=5000
soTimeout=5000
password=
database=0
clientName=
```

在使用redis作为mybatis二级缓存的xml配置文件中添加缓存标签配置

```xml
<!--使用type设置使用缓存的类型之后，其它属性都不会起作用-->
<cache type="org.mybatis.caches.redis.RedisCache"/>
```

## 脏数据的产生和避免

Mybatis的二级缓存是和命名空间绑定的，所以通常情况下每一个Mapper映射文件都拥有自己的二级缓存，不同Mapper的二级缓存互不影响。在常见的数据库操作中，多表联合查询非常常见，由于关系型数据库的设计，使得很多时候需要关联多个表才能获得想要的数据。在关联多表查询时肯定会将该查询放到某个命名空间下的映射文件中，这样一个多表的查询就会缓存在该命名空间的二级缓存中。涉及这些表的增删改操作通常不在一个映射中，它们的命名空间不同，因此当有数据变化时，多表查询的缓存未必会被清空，这种情况下会产生脏数据。

```java
@Test
public void testDirtyData(){
    SqlSession sqlSession = getSqlSession();
    try {
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        SysUser user = mapper.selectUserAndRoleById(1001L);
        Assert.assertEquals("普通用户",user.getSysRole().getRoleName());
        System.out.println("角色名："+user.getSysRole().getRoleName());
    } finally {
        sqlSession.close();
    }
    sqlSession = getSqlSession();
    try {
        RoleMapper mapper = sqlSession.getMapper(RoleMapper.class);
        SysRole role = mapper.selectById(2L);
        role.setRoleName("脏数据");
        mapper.updateRoleById(role);
        sqlSession.commit();
    } finally {
        sqlSession.close();
    }
    System.out.println("开启新的sqlSession");
    sqlSession = getSqlSession();
    try {
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        RoleMapper roleMapper = sqlSession.getMapper(RoleMapper.class);
        SysUser user = userMapper.selectUserAndRoleById(1001L);
        SysRole role = roleMapper.selectById(2L);
        //由于读取的是缓存中的数据，所以查询的时候是普通用户
        Assert.assertEquals("普通用户",user.getSysRole().getRoleName());
        //缓存未命中，读取的是修改后的数据库中的数据，也就是正确的修改之后的数据
        Assert.assertEquals("脏数据",role.getRoleName());
        System.out.println("角色名："+user.getSysRole().getRoleName());
        //还原数据
        role.setRoleName("普通用户");
        roleMapper.updateRoleById(role);
        sqlSession.commit();
    } finally {
        sqlSession.close();
    }
}
```

# Mybatis插件开发

Mybatis允许在已映射语句执行过程中的某一点进行拦截调用。默认情况下，Mybatis允许使用插件来拦截的接口和方法包括以下几个：

- Executor(update、query、flushStatments、commit、rollback、getTransaction、close、isClosed)
- ParameterHandler(getParameterObject、setParameters)
- ResultSetHandler(handlerResultSets、handlerCursorResultSets、handlerOutputParameters)
- StatementHandler(prepare、parameterize、batch、update、query)

这4个接口及其包含的方法的细节可以通过查看每个方法的定义来了解。如果不仅仅是想调用监控方法，那么应该很好地了解正在重写的方法的行为。因为试图修改或者重写已有方法行为的时候，很可能会破坏mybatis的核心模块。这些都底层的类和方法，所以使用插件的时候要特别当心。下面将对拦截器的各个细节进行详细介绍。

## 拦截器接口介绍

Mybatis插件可以用来实现拦截器接口Interceptor（org.apache.ibatis.plugin.Interceptor）,在实现类中对拦截器对象和方法进行处理。

先来看看Intercepotr接口

```java
/**
 * @author Clinton Begin
 */
public interface Interceptor {

  Object intercept(Invocation invocation) throws Throwable;

  default Object plugin(Object target) {
    return Plugin.wrap(target, this);
  }

  default void setProperties(Properties properties) {
    // NOP
  }

}
```

先从最简单的拦截器接口讲起，首先是setProperties方法，这个方法用来传递插件的参数，可以通过参数改变插件的行为。参数值是如何传递进来的呢？可以看一下拦截器的配置方法。在mybatis配置文件中，一般情况下，拦截器配置如下：

```xml
 <!--interceptor属性为拦截器的全限定类名，如果需要参数，通过property标签进行配置，配置后的参数会通过setProperties方法传递给拦截器-->
    <plugins>
        <plugin interceptor="tk.mybatis.simple.plugin.XXXInterceptor">
            <property name="prop1" value="value1"/>
            <property name="prop2" value="value2"/>
        </plugin>
    </plugins>
```

再看plugin方法,

```java
//该方法的实现通常如下，target参数就是拦截器需要拦截的对象，该方法会在创建被拦截的接口实现类时被调用。该方法的实现很简单，只需要调用mybatis提供的plugin类的wrap静态方法就可以通过java的动态代理拦截目标对象。
public Object plugin(Object target) {
  return Plugin.wrap(target, this);
}
```

Plugin类中plugin方法实现如下：

```java
/**
 *    Copyright 2009-2021 the original author or authors.
 *
 *    Licensed under the Apache License, Version 2.0 (the "License");
 *    you may not use this file except in compliance with the License.
 *    You may obtain a copy of the License at
 *
 *       http://www.apache.org/licenses/LICENSE-2.0
 *
 *    Unless required by applicable law or agreed to in writing, software
 *    distributed under the License is distributed on an "AS IS" BASIS,
 *    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 *    See the License for the specific language governing permissions and
 *    limitations under the License.
 */
package org.apache.ibatis.plugin;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;
import java.util.Set;

import org.apache.ibatis.reflection.ExceptionUtil;
import org.apache.ibatis.util.MapUtil;

/**
 * @author Clinton Begin
 */
public class Plugin implements InvocationHandler {

  private final Object target;
  private final Interceptor interceptor;
  private final Map<Class<?>, Set<Method>> signatureMap;

  private Plugin(Object target, Interceptor interceptor, Map<Class<?>, Set<Method>> signatureMap) {
    this.target = target;
    this.interceptor = interceptor;
    this.signatureMap = signatureMap;
  }

  public static Object wrap(Object target, Interceptor interceptor) {
    Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);
    Class<?> type = target.getClass();
    Class<?>[] interfaces = getAllInterfaces(type, signatureMap);
    if (interfaces.length > 0) {
      return Proxy.newProxyInstance(
          type.getClassLoader(),
          interfaces,
          new Plugin(target, interceptor, signatureMap));
    }
    return target;
  }

  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
      Set<Method> methods = signatureMap.get(method.getDeclaringClass());
      if (methods != null && methods.contains(method)) {
        return interceptor.intercept(new Invocation(target, method, args));
      }
      return method.invoke(target, args);
    } catch (Exception e) {
      throw ExceptionUtil.unwrapThrowable(e);
    }
  }

  private static Map<Class<?>, Set<Method>> getSignatureMap(Interceptor interceptor) {
    Intercepts interceptsAnnotation = interceptor.getClass().getAnnotation(Intercepts.class);
    // issue #251
    if (interceptsAnnotation == null) {
      throw new PluginException("No @Intercepts annotation was found in interceptor " + interceptor.getClass().getName());
    }
    Signature[] sigs = interceptsAnnotation.value();
    Map<Class<?>, Set<Method>> signatureMap = new HashMap<>();
    for (Signature sig : sigs) {
      Set<Method> methods = MapUtil.computeIfAbsent(signatureMap, sig.type(), k -> new HashSet<>());
      try {
        Method method = sig.type().getMethod(sig.method(), sig.args());
        methods.add(method);
      } catch (NoSuchMethodException e) {
        throw new PluginException("Could not find method on " + sig.type() + " named " + sig.method() + ". Cause: " + e, e);
      }
    }
    return signatureMap;
  }

  private static Class<?>[] getAllInterfaces(Class<?> type, Map<Class<?>, Set<Method>> signatureMap) {
    Set<Class<?>> interfaces = new HashSet<>();
    while (type != null) {
      for (Class<?> c : type.getInterfaces()) {
        if (signatureMap.containsKey(c)) {
          interfaces.add(c);
        }
      }
      type = type.getSuperclass();
    }
    return interfaces.toArray(new Class<?>[0]);
  }

}
```

最后一个intercept方法是Mybatis运行时要执行的拦截方法。通过该方法的参数invocation可以得到很多有用的信息。该参数的常用方法如下：

```java
@Override
public Object intercept(Invocation invocation) throws Throwable {
    //获取当前被拦截的对象
    Object target = invocation.getTarget();
    //获取当前被拦截的对象的方法
    Method method = invocation.getMethod();
    //获取k可以返回被拦截方法中的参数
    Object[] args = invocation.getArgs();
    //实际上执行了method.invoke(target,args)方法，上面代码没有做任何特殊处理，直接返回了执行的结果
    Object result = invocation.proceed();
    return result;
}
```

当配置了多个拦截器时，mybatis会遍历所有的拦截器，按顺序执行拦截器的plugin方法，被拦截的对象会被层层代理。在执行拦截对象的方法时，会一层层地调用拦截器，拦截器通过Invocation.proceed()调用下一层的方法，直到真正的方法被执行。方法执行的结果会从最里层开始向外一层层返回，所以如果存在按顺序配置的A、B、C三个签名相同的拦截器，Mybatis会按照C>B>A>target.proceed()>A>B>C的顺序执行。如果A、B、C签名不同，就会按照mybatis拦截对象的逻辑执行。

## 拦截器签名介绍

除了需要实现拦截器接口外，还需要给实现类配置以下拦截器注解。

@Intercepts和@Signature，这两个注解用来配置拦截器要拦截的接口的方法。

@Intercepts注解中的属性是一个@Signature（签名）数组，可以在同一个拦截器中同时拦截不同的接口和方法

```java
@Intercepts({
        @Signature(
            //设置拦截的接口，就是最开始提到的四个接口
                type = ResultSetHandler.class,
            //设置拦截接口的方法名，可选值是前面4个接口对应的方法，需要和接口匹配。
                method = "handlerResultSets",
            //设置拦截方法的参数类型数组，通过方法和参数类型可以确定唯一一个方法
                args = {Statement.class}
        )
})
public class DemoInterceptor implements Interceptor {}
```

由于Mybatis代码具体实现的原因，可以被拦截的四个接口中的方法并不是都可以被拦截的。下面将针对这4种接口，将可以被拦截的方法以及方法被调用的位置和对应的拦截器签名依次列举出来，大家可以根据方法调用的位置和方法提供的参数来选择想要拦截的方法。

### Executor接口

Executor接口包括以下几个方法

-   int update(MappedStatement ms, Object parameter) throws SQLException;

> 该方法会在所有的INSERT、UPDATE、DELETE执行时被调用，因此想要拦截这三类操作，可以拦截该方法。接口方法对应的签名如下：

```java
@Intercepts({
        @Signature(
                type = Executor.class,
                method = "update",
                args = {MappedStatement.class,Object.class}
        )
})
```

-   <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException;

> 该方法会在所有SELECT查询方法执行时被调用。通过这个接口参数可以获取很多有用的信息，因此这是最常被拦截的一个方法。使用该方法需要注意的是：虽然接口中有一个同名的参数更多的接口，但是这个参数多的接口不能被拦截。接口对应的签名如下：

```java
@Intercepts({
        @Signature(
                type = Executor.class,
                method = "query",
                args = {MappedStatement.class,Object.class, RowBounds.class, ResultHandler.class}
        )
})
```

-------------------

-   <E> Cursor<E> queryCursor(MappedStatement ms, Object parameter, RowBounds rowBounds) throws SQLException;

> 该方法只有在查询的返回类型为Cursor时被调用。接口对应的签名如下：

```java
@Intercepts({
        @Signature(
                type = Executor.class,
                method = "queryCursor",
                args = {MappedStatement.class,Object.class, RowBounds.class}
        )
})
```

------

-   List<BatchResult> flushStatements() throws SQLException;

> 该方法只在通过SqlSession方法调用flushStatements方法或者执行的接口方法中带有@Flush注解时才被调用，接口方法对应的签名如下。

```java
@Intercepts({
        @Signature(
                type = Executor.class,
                method = "flushStatements",
                args = {}
        )
})
```

---------

-   void commit(boolean required) throws SQLException;

> 该方法只在通过SqlSession方法调用commit方法的时候调用，接口方法对应的签名如下。

```java
@Intercepts({
        @Signature(
                type = Executor.class,
                method = "commit",
                args = {boolean.class}
        )
})
```

-------

-   void rollback(boolean required) throws SQLException;

> 该方法只在通过SqlSession方法调用rollback方法的时候调用，接口方法对应的签名如下。

```java
@Intercepts({
        @Signature(
                type = Executor.class,
                method = "rollback",
                args = {boolean.class}
        )
})
```

-------

-   Transaction getTransaction();

> 该方法只在通过SqlSession方法获取数据库连接时才被调用，接口方法对应的签名如下。

```java
@Intercepts({
        @Signature(
                type = Executor.class,
                method = "getTransaction",
                args = {}
        )
})
```

--------

-   void close(boolean forceRollback);

> 该方法只在延迟加载获取新的Executor后才会被执行，接口方法对应的签名如下：

```java
@Intercepts({
        @Signature(
                type = Executor.class,
                method = "close",
                args = {boolean.class}
        )
})
```

-----------

-   boolean isClosed();

> 该方法只在延迟加载执行查询方法前被执行，接口方法对应的签名如下：

```java
@Intercepts({
        @Signature(
                type = Executor.class,
                method = "isClosed",
                args = {}
        )
})
```

<font color=red>**还有一些接口中的方法没有列出，都是不能够被拦截的方法，具体方法可以自己查看源码**</font>

### ParameterHandler接口

该接口只包含两个方法

```java
public interface ParameterHandler {
  //该方法只在执行存储过程处理出参的时候被调用，接口方法对应的签名使用如Executor接口签名使用一致。
  Object getParameterObject();
  //该方法在所有数据库方法设置SQL参数时被调用
  void setParameters(PreparedStatement ps) throws SQLException;

}
```

### ResultSetHandler

该接口只包含三个方法

```java
public interface ResultSetHandler {
	//该方法会在除了存储过程以及返回值类型为Cursor<T>以外的查询方法中被调用
  <E> List<E> handleResultSets(Statement stmt) throws SQLException;
	//该方法在mybatis3.4.0版本之后添加，只会在返回值类型为cursor<T>的查询方法中被调用
  <E> Cursor<E> handleCursorResultSets(Statement stmt) throws SQLException;
	//该方法只在使用存储过程处理出参时被调用
  void handleOutputParameters(CallableStatement cs) throws SQLException;

}
```

> ResultSetHandler接口的第一个方法对于拦截处理Mybatis的查询结果非常有用，并且由于这个接口被调用的位置在处理二级缓存之前，因此通过这种方式处理的结果可以执行二级缓存。在后面会提到就该方法提供的一个针对Map类型结果处理Key值的插件。

### StatementHandler

该接口内包含的方法如下所示：

```java
public interface StatementHandler {
	//该方法会在数据库执行前被调用，优先于当前接口中的其它方法而被执行。
  Statement prepare(Connection connection, Integer transactionTimeout)
      throws SQLException;
	//该方法prepare方法之后执行，用于处理参数信息。
  void parameterize(Statement statement)
      throws SQLException;
	//在全局设置配置defaultExecutorType="BATCH"时，执行数据操作才会调用该方法。
  void batch(Statement statement)
      throws SQLException;

  int update(Statement statement)
      throws SQLException;
	//执行select方法时被调用
  <E> List<E> query(Statement statement, ResultHandler resultHandler)
      throws SQLException;
	//3.4.0之后添加的方法，只会在返回值类型为Cursor<T>的查询中被调用。
  <E> Cursor<E> queryCursor(Statement statement)
      throws SQLException;

  BoundSql getBoundSql();

  ParameterHandler getParameterHandler();

}
```

### 下划线键值转小写驼峰插件开发

介绍完拦截器接口以及其签名的使用，存在着这样一个场景，使用mybatis时，为了方便扩展而使用了Map类型的返回值。使用map作为返回值时，Map中的键值就是查询结果中的列名，而数据库的列名一般都是由大小写字母或者下划线形式组成，和java使用的驼峰形式不一致。而且由于不同数据库查询结果列的大小写也并不一致，因此为了保证使用map时属性一致，可以对Map类型的结果进行特殊处理，即将不同格式的列名转换为java中的驼峰形式。这种情况下，可以使用拦截器，通过拦截ResultSetHandler接口中的handlerResultSets方法去处理Map类型的结果。拦截器实现代码如下：

```java
package tk.mybatis.simple.plugin;


import org.apache.ibatis.executor.resultset.ResultSetHandler;
import org.apache.ibatis.plugin.*;

import java.sql.Statement;
import java.util.*;

/**
 * @author 汤琛
 * @PROJECT_NAME: MybatisStudy
 * @DESCRIPTION: Mybatis Map类型下划线key转小写驼峰形式
 * @DATE: 2022/8/23 17:24
 */
@Intercepts({
        @Signature(
                type = ResultSetHandler.class,
                method = "handleResultSets",
                args = {Statement.class}
        )
})
public class CameHumpInterceptor implements Interceptor {
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        List<Object> list = (List<Object>) invocation.proceed();
        for (Object o : list) {
            if ( o instanceof Map){
                processMap((Map) o);
            }else{
                break;
            }
        }
        return list;
    }

    /**
     * @description 处理map集合
     * @date 2022/8/23 17:29
     * @param map
     * @throws
     */
    public void processMap(Map<String,Object> map){
        Set<String> keySet = new HashSet<>(map.keySet());
        for (String s : keySet) {
            //将以大写开头的字符串转换为小写，如果包含下划线也会处理为驼峰
            //此处只通过这两个简单的标识来判断是否进行转换
            if ((s.charAt(0) >='A' && s.charAt(0) <='Z') || s.indexOf("_") >=0 ){
                Object value = map.get(s);
                //此处删除的不是set集合的数据，因此不存在安全性问题，不允许在foreach中删除正在遍历的集合元素，可能会出现异常情况。
                map.remove(s);
                map.put(underlineToCamehump(s),value);
            }
        }
    }

    /**
     * @description 忽略key中的_将_后面的首字符转变为大写，同时将首字母大写的字符串转换为小写
     * @date 2022/8/23 17:38
     * @param key
     * @throws
     * @return java.lang.String
     */
    public static String underlineToCamehump(String key){
        //创建字符缓存流，非线程安全
        StringBuilder sb = new StringBuilder();

        boolean nextUpperCase = false;
        //遍历字符串
        for (int i = 0; i < key.length(); i++) {
            char c = key.charAt(i);
            //如果当前字符是_，而且不是首个字符，设置下一个字符大写标记为真
            if (c == '_'){
                if (sb.length() > 0){
                    nextUpperCase = true;
                }
            }else{
                //如果不是_，而且需要大写，则将当前字符转换为大写存入字符缓存流中，设置下一个字符大写标记为假。
                if (nextUpperCase){
                    sb.append(Character.toUpperCase(c));
                    nextUpperCase =false;
                }else{
                    //不需要大写，但是当前字符可能是大写状态需要转换为小写。
                    sb.append(Character.toLowerCase(c));
                }
            }
        }
        return sb.toString();
    }


    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target,this);
    }
	
    //此处拦截器不需要传递参数，故不需要设置属性值。
    @Override
    public void setProperties(Properties properties) {
//        Interceptor.super.setProperties(properties);
    }
}

```

mybatis配置文件中配置插件

```xml
    <!--interceptor属性为拦截器的全限定类名，如果需要参数，通过property标签进行配置，配置后的参数会通过setProperties方法传递给拦截器-->
    <plugins>
        <plugin interceptor="tk.mybatis.simple.plugin.CameHumpInterceptor"></plugin>
<!--        <plugin interceptor="tk.mybatis.simple.plugin.XXXInterceptor">-->
<!--            <property name="prop1" value="value1"/>-->
<!--            <property name="prop2" value="value2"/>-->
<!--        </plugin>-->
    </plugins>
```

xml中sql测试

```xml
<select id="selectMapById" parameterType="Long" resultType="java.util.Map">
    select   user_name,
             user_password,
             user_email,
             user_info,
             head_img,
             create_time from sys_user where id = #{id}
</select>
```

mapper接口中方法声明

```java
Map<String,Object> selectMapById(Long id);
```

功能测试

```java
@Test
public void testSelectMapById(){
    SqlSession sqlSession = getSqlSession();
    try {
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        Map<String, Object> map = mapper.selectMapById(1L);
        map.forEach((k,v)->{
            System.out.println(k+"--------->"+v);
        });
    } finally {
        sqlSession.close();
    }
}
```

### 分页插件开发

首先创建一个分页插件拦截器类，通过对Executor拦截器接口中的query方法进行拦截，从而实现对所有查询语句都实现分页的功能；类如下：

```java
@Intercepts({
        @Signature(
                type = Executor.class,
                method = "query",
                args = {MappedStatement.class,Object.class, RowBounds.class, ResultHandler.class}
        )
})
public class PageInterceptor implements Interceptor {

    private static final List<ResultMapping> EMPTY_RESULTMAPPING = new ArrayList<ResultMapping>(0);
    private Dialect dialect;
    private Field additionalParametersField;
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        //获取拦截方法的参数
        Object[] args = invocation.getArgs();
        MappedStatement ms = (MappedStatement) args[0];
        Object parameterObject = args[1];
        RowBounds rowBounds = (RowBounds) args[2];
        //调用方法判断是否需要分页，如果不需要，直接返回结果。
        if (!dialect.skip(ms.getId(),parameterObject,rowBounds)){
            ResultHandler resultHandler = (ResultHandler) args[3];
            //获取当前目标对象
            Executor executor = (Executor) invocation.getTarget();
            BoundSql boundSql = ms.getBoundSql(parameterObject);
            //反射获取动态参数
            Map<String,Object> additionalParameters = (Map<String, Object>) additionalParametersField.get(boundSql);
            //判断是否需要进行count查询
            if (dialect.beforeCount(ms.getId(), parameterObject,rowBounds)){
                //根据当前的ms创建一个返回值为long类型的ms
                MappedStatement countMs = newMappedStatement(ms, Long.class);
                //创建count查询的缓存key
                CacheKey countKey = executor.createCacheKey(countMs,parameterObject,RowBounds.DEFAULT,boundSql);
                //调用方言获取count sql
                String countsql = dialect.getCountSql(boundSql,parameterObject,rowBounds,countKey);
                BoundSql countBoundSql = new BoundSql(ms.getConfiguration(),countsql,boundSql.getParameterMappings(),parameterObject);
                //当使用动态SQL时，可能会产生临时的参数
                //这些参数需要手动设置到新的BoundSql中
                additionalParameters.forEach((k,v)->{
                    countBoundSql.setAdditionalParameter(k,v);
                });
                //执行count查询
                Object countResultList = executor.query(countMs,parameterObject,RowBounds.DEFAULT,resultHandler,countKey,countBoundSql);
                Long count = (Long) ((List) countResultList).get(0);
                //处理查询总数
                dialect.afterCount(count,parameterObject,rowBounds);
                if (count == 0L){
                    //当查询总数为0时，返回空的结果
                    return dialect.afterPage(new ArrayList(),parameterObject,rowBounds);
                }
            }
            //判断是否需要进行分页查询
            if (dialect.beforePage(ms.getId(),parameterObject,rowBounds)){
                //生成分页的缓存key
                CacheKey pageKey = executor.createCacheKey(ms,parameterObject,rowBounds,boundSql);
                //调用方言获取分页sql
                String pageSql = dialect.getPageSql(boundSql,parameterObject,rowBounds,pageKey);
                BoundSql pageBoundSql = new BoundSql(ms.getConfiguration(),pageSql,boundSql.getParameterMappings(),parameterObject);
                additionalParameters.forEach((k,v)->{
                    pageBoundSql.setAdditionalParameter(k,v);
                });
                //执行分页查询
                List resultList =executor.query(ms,parameterObject,RowBounds.DEFAULT,resultHandler,pageKey,pageBoundSql);
                return dialect.afterPage(resultList,parameterObject,rowBounds);
            }
        }
        //返回默认的查询
        return invocation.proceed();
    }
    /**
     * @description 根据现有的ms创建一个新的返回值类型，使用新的返回值类型
     * @date 2022/8/24 16:43
     * @param ms
     * @param resultType
     * @throws
     * @return org.apache.ibatis.mapping.MappedStatement
     */
    public MappedStatement newMappedStatement(MappedStatement ms,Class<?> resultType){
        MappedStatement.Builder builder = new MappedStatement.Builder(
                ms.getConfiguration(),
                ms.getId()+"_count",
                ms.getSqlSource(),
                ms.getSqlCommandType());
        builder.resource(ms.getResource());
        builder.fetchSize(ms.getFetchSize());
        builder.statementType(ms.getStatementType());
        builder.keyGenerator(ms.getKeyGenerator());
        if (ms.getKeyProperties() != null && ms.getKeyProperties().length != 0){
            StringBuilder keyProperties = new StringBuilder();
            for (String keyProperty : ms.getKeyProperties()) {
                keyProperties.append(keyProperties).append(",");
            }
            keyProperties.delete(keyProperties.length()-1,keyProperties.length());
            builder.keyProperty(keyProperties.toString());
        }
        builder.timeout(ms.getTimeout());
        builder.parameterMap(ms.getParameterMap());
        //count查询返回值
        List<ResultMap> resultMaps = new ArrayList<ResultMap>();
        ResultMap resultMap = new ResultMap.Builder(
                ms.getConfiguration(),
                ms.getId(),
                resultType,
                EMPTY_RESULTMAPPING).build();
        resultMaps.add(resultMap);
        builder.resultMaps(resultMaps);
        builder.resultSetType(ms.getResultSetType());
        builder.cache(ms.getCache());
        builder.flushCacheRequired(ms.isFlushCacheRequired());
        builder.useCache(ms.isUseCache());
        return builder.build();
    }

    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target,this);
    }

    @Override
    public void setProperties(Properties properties) {
        String dialectClass = properties.getProperty("dialect");
        try {
            dialect = (Dialect) Class.forName(dialectClass).newInstance();
        } catch (Exception e) {
            throw new RuntimeException("使用PageInterceptor分页插件时，必须设置dialect属性");
        }
        dialect.setProperties(properties);
        try {
            //设置属性可见
            additionalParametersField = BoundSql.class.getDeclaredField("additionalParameters");
            additionalParametersField.setAccessible(true);
        } catch (NoSuchFieldException e) {
            throw new RuntimeException(e);
        }
    }
}
```

添加一个Dialect接口，用于实现分页功能；不同的数据库需要分别实现Dialect接口中的方法，对自身的数据库分页功能完成实现

```java
public interface Dialect {
    /**
     * @description 跳过count和分页查询
     * @date 2022/8/24 15:30
     * @param msId  执行的Mybatis方法全名
     * @param parameterObject 方法参数
     * @param rowBounds  分页参数
     * @throws
     * @return boolean  true跳过，返回默认查询结果;false则执行分页查询
     */
    boolean skip(String msId, Object parameterObject, RowBounds rowBounds);

    /**
     * @description 执行分页前，返回true会进行count查询，返回false会继续下面的beforePage判断
     * @date 2022/8/24 15:32
     * @param msId  执行的Mybatis方法全名
     * @param parameterObject   方法参数
     * @param rowBounds 分页参数
     * @throws
     * @return boolean
     */
    boolean beforeCount(String msId,Object parameterObject, RowBounds rowBounds);

    /**
     * @description 生成count查询Sql
     * @date 2022/8/24 15:41
     * @param boundSql boundSql绑定的SQL对象
     * @param parameterObject 方法参数
     * @param rowBounds  分页参数
     * @param cacheKey  count缓存key
     * @throws
     * @return java.lang.String
     */
    String getCountSql(BoundSql boundSql, Object parameterObject, RowBounds rowBounds, CacheKey cacheKey);

    /**
     * @description 执行完count查询后
     * @date 2022/8/24 15:46
     * @param count 查询结果总数
     * @param parameterObject 接口参数
     * @param rowBounds 分页查询
     * @throws
     */
    void afterCount(long count, Object parameterObject, RowBounds rowBounds);

    /**
     * @description 执行分页前，返回true会进行分页查询，返回false会返回默认的查询结果
     * @date 2022/8/24 15:47
     * @param msId  执行的Mybatis方法全名
     * @param parameterObject   方法参数
     * @param rowBounds 分页查询
     * @throws
     * @return boolean
     */
    boolean beforePage(String msId, Object parameterObject, RowBounds rowBounds);

    /**
     * @description 生成分页查询sql
     * @date 2022/8/24 15:49
     * @param boundSql  绑定sql对象
     * @param parameterObject   方法参数
     * @param rowBounds 分页参数
     * @param cacheKey  分页缓存key
     * @throws
     * @return java.lang.String
     */
    String getPageSql(BoundSql boundSql, Object parameterObject,RowBounds rowBounds,CacheKey cacheKey);

    /**
     * @description 分页查询后，处理分页结果，拦截器中直接return该方法的返回值
     * @date 2022/8/24 15:50
     * @param pageList  分页查询结果
     * @param parameterObject   方法参数
     * @param rowBounds 分页参数
     * @throws
     * @return java.lang.Object
     */
    Object afterPage(List pageList, Object parameterObject, RowBounds rowBounds);

    /**
     * @description 设置参数
     * @date 2022/8/24 15:51
     * @param properties  插件属性
     * @throws
     */
    void setProperties(Properties properties);
}
```

MySql的分页实现如下：

```java
public class MySqlDialect implements Dialect {
    @Override
    public boolean skip(String msId, Object parameterObject, RowBounds rowBounds) {
        //这里使用RowBounds分页，没有RowBounds参数的时候会使用RowBounds.DEFAULT作为默认值
        return rowBounds == RowBounds.DEFAULT;
    }

    @Override
    public boolean beforeCount(String msId, Object parameterObject, RowBounds rowBounds) {
        //只有使用了PageRowBounds才能记录总数，否则查询了总数也没用
        return rowBounds instanceof PageRowBounds;
    }

    @Override
    public String getCountSql(BoundSql boundSql, Object parameterObject, RowBounds rowBounds, CacheKey cacheKey) {
        //简单嵌套实现了MySQL count查询
        return "select count(*) from ("+ boundSql.getSql() + ") temp";
    }

    @Override
    public void afterCount(long count, Object parameterObject, RowBounds rowBounds) {
        //记录总数，按照beforeCount逻辑，只有PageRowBounds才会查询count，所以这里强制转换
        ((PageRowBounds) rowBounds).setTotal(count);
    }

    @Override
    public boolean beforePage(String msId, Object parameterObject, RowBounds rowBounds) {
        return rowBounds != RowBounds.DEFAULT;
    }

    @Override
    public String getPageSql(BoundSql boundSql, Object parameterObject, RowBounds rowBounds, CacheKey cacheKey) {
        //cacheKey会影响缓存，通过固定的RowBounds可以保证二级缓存有效
        cacheKey.update("RowBounds");
        return boundSql.getSql()+" limit " + rowBounds.getOffset() + "," + rowBounds.getLimit();
    }

    @Override
    public Object afterPage(List pageList, Object parameterObject, RowBounds rowBounds) {
        //直接返回结果
        return pageList;
    }

    @Override
    public void setProperties(Properties properties) {
        //没有其它参数，不做处理
    }
}
```

由于mybatis自带的rowBounds内存分页不存在total总记录数，故创建一个可以实现返回总记录数的PageRowBounds类

```java
public class PageRowBounds extends RowBounds {
    private long total;

    public PageRowBounds() {
        super();
    }

    public PageRowBounds(int offset, int limit) {
        super(offset, limit);
    }

    public long getTotal() {
        return total;
    }

    public void setTotal(long total) {
        this.total = total;
    }
}
```

有了以上代码之后，需要使用分页插件只需要在mybatis配置文件中添加plugin配置即可如下：

```xml
    <!--interceptor属性为拦截器的全限定类名，如果需要参数，通过property标签进行配置，配置后的参数会通过setProperties方法传递给拦截器-->
    <plugins>
        <plugin interceptor="tk.mybatis.simple.plugin.CameHumpInterceptor"></plugin>
<!--        <plugin interceptor="tk.mybatis.simple.plugin.XXXInterceptor">-->
<!--            <property name="prop1" value="value1"/>-->
<!--            <property name="prop2" value="value2"/>-->
<!--        </plugin>-->
        <plugin interceptor="tk.mybatis.simple.plugin.PageInterceptor">
            <property name="dialect" value="tk.mybatis.simple.plugin.MySqlDialect"/>
        </plugin>
    </plugins>
```

配置好之后，功能测试如下：

mapper接口中添加对映方法

```java
/**
 * @description 分页查询所有
 * @date 2022/8/25 9:58
 * @param rowBounds 分页参数
 * @throws
 * @return java.util.List<tk.mybatis.simple.model.SysRole>
 */
List<SysRole> selectAll(RowBounds rowBounds);
```

测试方法

```java
@Test
public void testSelectAllByRowBounds(){
    SqlSession sqlSession = getSqlSession();
    try {
        RoleMapper roleMapper = sqlSession.getMapper(RoleMapper.class);
        //查询第一个，使用RowBounds类型时不会查询总数
        RowBounds rowBounds = new RowBounds(0,1);
        List<SysRole> roles = roleMapper.selectAll(rowBounds);
        for (SysRole role : roles) {
            System.out.println("角色名："+role.getRoleName());
        }
        //使用PageRowBounds时，会查询总数
        PageRowBounds pageRowBounds = new PageRowBounds(0,1);
        List<SysRole> roles1 = roleMapper.selectAll(pageRowBounds);
        //获取总数
        System.out.println("查询总数："+pageRowBounds.getTotal());
        for (SysRole role : roles1) {
            System.out.println("角色名："+role.getRoleName());
        }
        //再次查询第二个角色
        pageRowBounds = new PageRowBounds(1,1);
        List<SysRole> roles2 = roleMapper.selectAll(pageRowBounds);
        System.out.println("查询总数："+pageRowBounds.getTotal());
        for (SysRole role : roles2) {
            System.out.println("角色名："+role.getRoleName());
        }
    } finally {
        sqlSession.close();
    }
}
```
