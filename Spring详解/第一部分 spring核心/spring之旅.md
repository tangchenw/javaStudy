## 基于POJO的轻量级和最小入侵性编程

### 依赖注入

```java
package sia.knights;

public class DamselRescuingKnight implements Knight {

  private RescueDamselQuest quest;

  public DamselRescuingKnight() {
    this.quest = new RescueDamselQuest();
  }

  public void embarkOnQuest() {
    quest.embark();
  }
  
}
```

在以上代码中可以看到，DamselRescuingKnight 在它的构造函数中自行创建了 RescueDamselQuest。这使得 DamselRescuingKnight 紧密地和 RescueDamselQuest 耦合到了一起，因此极大地限制了这个骑士执行探险的能力。如果一个少女需要救援，这个骑士能够召之即来。但是如果一条恶龙需要杀掉，或者一个圆桌……额……需要滚起来，那么这个骑士就爱莫能助了。

 耦合具有两面性（two-headed beast）。一方面，紧密耦合的代码难以测试、难以复用、难以理解，并且典型地表现出 “打地鼠” 式的 bug 特性（修复一个 bug，将会出现一个或者更多新的 bug）。另一方面，一定程度的耦合又是必须的 —— 完全没有耦合的代码什么也做不了。为了完成有实际意义的功能，不同的类必须以适当的方式进行交互。总而言之，耦合是必须的，但应当被小心谨慎地管理。

------

**对此依赖注入的构造器注入正是解决办法之一**

```java
package sia.knights;
  
public class BraveKnight implements Knight {

  private Quest quest;

  public BraveKnight(Quest quest) {
    this.quest = quest;
  }

  public void embarkOnQuest() {
    quest.embark();
  }

}
```

我们可以看到，不同于之前的 DamselRescuingKnight，BraveKnight 没有自行创建探险任务，而是在构造的时候把探险任务作为构造器参数传入。这是依赖注入的方式之一，即构造器注入（constructor injection）。

更重要的是，传入的探险类型是 Quest，也就是所有探险任务都必须实现的一个接口。所以，BraveKnight 能够响应 RescueDamselQuest、SlayDragonQuest、MakeRoundTableRounderQuest 等任意的 Quest 实现。

这里的要点是 BraveKnight 没有与任何特定的 Quest 实现发生耦合。对它来说，被要求挑战的探险任务只要实现了 Quest 接口，那么具体是哪种类型的探险就无关紧要了。这就是 DI 所带来的最大收益 —— 松耦合。如果一个对象只通过接口（而不是具体实现或初始化过程）来表明依赖关系，那么这种依赖就能够在对象本身毫不知情的情况下，用不同的具体实现进行替换。

```java
package sia.knights;

import java.io.PrintStream;

public class SlayDragonQuest implements Quest {

  private PrintStream stream;

  public SlayDragonQuest(PrintStream stream) {
    this.stream = stream;
  }

  public void embark() {
    stream.println("Embarking on quest to slay the dragon!");
  }

}
```

我们可以看到，SlayDragonQuest 实现了 Quest 接口，这样它就适合注入到 BraveKnight 中去了。与其他的Java入门样例有所不同，SlayDragonQuest 没有使用 System.out.println()，而是在构造方法中请求一个更为通用的 PrintStream。这里最大的问题在于，我们该如何将 SlayDragonQuest 交给 BraveKnight 呢？又如何将 PrintStream 交给 SlayDragonQuest 呢？

**基于XML的bean装配方式**

创建应用组件之间协作的行为通常称为装配（wiring）。Spring 有多种装配 bean 的方式，采用 XML 是很常见的一种装配方式。以下是一个简单的 Spring 配置文件：knights.xml，该配置文件将 BraveKnight、SlayDragonQuest 和 PrintStream 装配到了 一起。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="
    http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd">
	<!-- bean的声明方式 -->
  <bean id="knight" class="sia.knights.BraveKnight"> 
     <!-- 构造器注入 ref对象-->
    <constructor-arg ref="quest" />
  </bean>

  <bean id="quest" class="sia.knights.SlayDragonQuest">
    <constructor-arg value="#{T(System).out}" />
  </bean>

</beans>
```

在这里，BraveKnight 和 SlayDragonQuest 被声明为 Spring 中的 bean。就 BraveKnight bean 来讲，它在构造时传入了对 SlayDragonQuest bean 的引用，将其作为构造器参数。同时， SlayDragonQuest bean 的声明使用了 Spring 表达式语言（Spring Expression Language），将 System.out（这是一个 PrintStream）传入到了 SlayDragonQuest 的构造器中。

**基于注解的bean装配方式**

```java
package sia.knights.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import sia.knights.BraveKnight;
import sia.knights.Knight;
import sia.knights.Quest;
import sia.knights.SlayDragonQuest;

@Configuration
public class KnightConfig {

  @Bean
  public Knight knight() {
    return new BraveKnight(quest());
  }
  
  @Bean
  public Quest quest() {
    return new SlayDragonQuest(System.out);
  }

}
```

不管你使用的是基于 XML 的配置还是基于 Java 的配置，DI 所带来的收益都是相同的。尽管 BraveKnight 依赖于 Quest，但是它并不知道传递给它的是什么类型的 Quest，也不知道这个 Quest 来自哪里。与之类似，SlayDragonQuest 依赖于 PrintStream，但是在编码时它并不需要知道这个 PrintStream 是什么样子的。只有 Spring 通过它的配置，能够了解这些组成部分是如何装配起来的。这样的话，就可以在不改变所依赖的类的情况下，修改依赖关系。

现在已经声明了 BraveKnight 和 Quest 的关系，接下来我们只需要装载 XML 配置文件，并把应用启动起来。

**观察它如何工作**

Spring 通过应用上下文（Application Context）装载 bean 的定义并把它们组装起来。Spring 应用上下文全权负责对象的创建和组装。Spring 自带了多种应用上下文的实现，它们之间主要的区别仅仅在于如何加载配置。 因为 knights.xml 中的 bean 是使用 XML 文件进行配置的，所以选择  ClassPathXmlApplicationContext 作为应用上下文相对是比较合适的。该类加载位于应用程序类路径下的一个或多个 XML 配置文件。KnightMain.java 加载包含 Knight 的 Spring 上下文，main() 方法调用 ClassPathXmlApplicationContext 加载 knights.xml，并获得 Knight 对象的引用。

```java
package sia.knights;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class KnightMain {

  public static void main(String[] args) throws Exception {
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("META-INF/spring/knight.xml");
    Knight knight = context.getBean(Knight.class);
    knight.embarkOnQuest();
    context.close();
  }

}
```

这里的 main() 方法基于 knights.xml 文件创建了 Spring 应用上下文。随后它调用该应用上下文获取一个 ID 为 knight 的 bean。得到 Knight 对象的引用后，只需简单调用 embarkOnQuest() 方法就可以执行所赋予的探险任务了。注意这个类完全不知道我们的英雄骑士接受哪种探险任务，而且完全没有意识到这是由 BraveKnight 来执行的。只有 knights.xml 文件知道哪个骑士执行哪种探险任务。

## 应用切面

DI 能够让相互协作的软件组件保持松散耦合，而面向切面编程（aspect-oriented programming，AOP）允许你把遍布应用各处的功能分离出来形成可重用的组件。

面向切面编程往往被定义为促使软件系统实现关注点的分离一项技术。系统由许多不同的组件组成，每一个组件各负责一块特定功能。除了实现自身核心的功能之外，这些组件还经常承担着额外的职责。诸如日志、事务管理和安全这样的系统服务经常融入到自身具有核心业务逻辑的组件中去，这些系统服务通常被称为横切关注点，因为它们会跨越系统的多个组件。

如果将这些关注点分散到多个组件中去，你的代码将会带来双重的复杂性。

- 实现系统关注点功能的代码将会重复出现在多个组件中。这意味着如果你要改变这些关注点的逻辑，必须修改各个模块中的相关实现。即使你把这些关注点抽象为一个独立的模块，其他模块只是调用它的方法，但方法的调用还是会重复出现在各个模块中。
- 组件会因为那些与自身核心业务无关的代码而变得混乱。一个向地址簿增加地址条目的方法应该只关注如何添加地址，而不应该关注它是不是安全的或者是否需要支持事务。

如下图所示，我们可以把切面想象为覆盖在很多组件之上的一个外壳。应用是由那些实现各自业务功能的模块组成的。借助 AOP，可以使用各种功能层去包裹核心业务层。这些层以声明的方式灵活地应用到系统中，你的核心应用甚至根本不知道它们的存在。这是一个非常强大的理念，可以将安全、事务和日志关注点与核心业务逻辑相分离。

![1.3 AOP](C:\Users\汤琛\Desktop\学习资料\Spring详解\images\1.3 AOP.jpg)

为了示范在 Spring 中如何应用切面，让我们重新回到骑士的例子，并为它添加一个切面。

### AOP应用

每一个人都熟知骑士所做的任何事情，这是因为吟游诗人用诗歌记载了骑士的事迹并将其进行传唱。假设我们需要使用吟游诗人这个服务类来记载骑士的所有事迹（相当于日志系统）。以下代码块展示了我们会使用的 Minstrel 类。

```java
package sia.knights;

import java.io.PrintStream;

public class Minstrel {

  private PrintStream stream;
  
  public Minstrel(PrintStream stream) {
    this.stream = stream;
  }

  public void singBeforeQuest() {
    stream.println("Fa la la, the knight is so brave!");
  }

  public void singAfterQuest() {
    stream.println("Tee hee hee, the brave knight " +
    		"did embark on a quest!");
  }

}
```

正如你所看到的那样，Minstrel 是只有两个方法的简单类。在骑士执行每一个探险任务之前，singBeforeQuest() 方法会被调用；在骑士完成探险任务之后，singAfterQuest() 方法会被调用。在这两种情况下，Minstrel 都会通过一个 PrintStream 类来歌颂骑士的事迹，这个类是通过构造器注入进来的。

把 Minstrel 加入你的代码中并使其运行起来，这对你来说是小事一桩。我们适当做一下调整从而让 BraveKnight 可以使用 Minstrel。以下代码块 展示了将 BraveKnight 和 Minstrel 组合起来的第一次尝试。

```java
package com.springinaction.knights;

public class BraveKnight implements Knight {

  private Quest quest;
  private Minstrel minstrel;
  
  public BraveKnight(Quest quest, Minstrel minstrel) {
    this.quest = quest;
    this.minstrel = minstrel;
  }
  
  public void embarkOnQuest() throws QuestException {
    minstrel.singBeforeQuest();
    quest.embark();
    minstrl.singAfterQuest();
  }
  
} 
```

这应该可以达到预期效果。现在，你所需要做的就是回到 Spring 配置中，声明 Minstrel bean 并将其注入到 BraveKnight 的构造器之中。但是，请稍等……

我们似乎感觉有些东西不太对。管理他的吟游诗人真的是骑士职责范围内的工作吗？在我看来，吟游诗人应该做他份内的事，根本不需要骑士命令他这么做。毕竟，用诗歌记载骑士的探险事迹，这是吟游诗人的职责。为什么骑士还需要提醒吟游诗人去做他份内的事情呢？

此外，因为骑士需要知道吟游诗人，所以就必须把吟游诗人注入到 BarveKnight 类中。这不仅使 BraveKnight 的代码复杂化了，而且还让我疑惑是否还需要一个不需要吟游诗人的骑士呢？如果 Minstrel 为 null 会发生什么呢？我是否应该引入一个空值校验逻辑来覆盖该场景？

简单的 BraveKnight 类开始变得复杂，如果你还需要应对没有吟游诗人时的场景，那代码会变得更复杂。但利用 AOP，你可以声明吟游诗人必须歌颂骑士的探险事迹，而骑士本身并不用直接访问 Minstrel 的方法。

要将 Minstrel 抽象为一个切面，你所需要做的事情就是在一个 Spring 配置文件中声明它。以下代码块 是更新后的 knights.xml 文件，Minstrel 被声明为一个切面。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:aop="http://www.springframework.org/schema/aop"
  xsi:schemaLocation="http://www.springframework.org/schema/aop 
  http://www.springframework.org/schema/aop/spring-aop.xsd
  http://www.springframework.org/schema/beans 
  http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="knight" class="sia.knights.BraveKnight">
    <constructor-arg ref="quest" />
  </bean>

  <bean id="quest" class="sia.knights.SlayDragonQuest">
    <constructor-arg value="#{T(System).out}" />
  </bean>

  <bean id="minstrel" class="sia.knights.Minstrel">
    <constructor-arg value="#{T(System).out}" />
  </bean>

  <aop:config>
    <aop:aspect ref="minstrel">
      <aop:pointcut id="embark"
          expression="execution(* *.embarkOnQuest(..))"/>
        
      <aop:before pointcut-ref="embark" 
          method="singBeforeQuest"/>

      <aop:after pointcut-ref="embark" 
          method="singAfterQuest"/>
    </aop:aspect>
  </aop:config>
  
</beans>
```

这里使用了 Spring 的 aop 配置命名空间把 Minstrel bean 声明为一个切面。首先，需要把 Minstrel 声明为一个 bean，然后在元素中引用该 bean。为了进一步定义切面，声明 （使用）在 embarkOnQuest() 方法执行前调用 Minstrel 的 singBeforeQuest() 方法。这种方式被称为前置通知（before advice）。同时声明（使用）在 embarkOnQuest() 方法执行后调用 singAfterQuest() 方 法。这种方式被称为后置通知（after advice）。

在这两种方式中，pointcut-ref 属性都引用了名字为 embark 的切入点。该切入点是在前边的元素中定义的，并配置 expression 属性来选择所应用的通知。表达式的语法采用的是 AspectJ 的切点表达式语言。

现在，你无需担心不了解 AspectJ 或编写 AspectJ 切点表达式的细节， 我们稍后会在第 4  章详细地探讨 Spring AOP 的内容。现在你已经知道，Spring 在骑士执行探险任务前后会调用 Minstrel 的  singBeforeQuest() 和 singAfterQuest() 方法，这就足够了

这就是我们需要做的所有的事情！通过少量的 XML 配置，就可以把 Minstrel 声明为一个 Spring 切面。如果你现在还没有完全理解，不必担心，在第 4 章你会看到更多的 Spring AOP 示例，那将会帮助你彻底弄清楚。现在我们可以从这个示例中获得两个重要的观点。

首先，Minstrel 仍然是一个 POJO，没有任何代码表明它要被作为一个切面使用。当我们按照上面那样进行配置后，在 Spring 的上下文中，Minstrel 实际上已经变成一个切面了。

其次，也是最重要的，Minstrel 可以被应用到 BraveKnight 中，而 BraveKnight 不需要显式地调用它。实际上，BraveKnight 完全不知道 Minstrel 的存在。

必须还要指出的是，尽管我们使用 Spring 魔法把 Minstrel 转变为一 个切面，但首先要把它声明为一个 Spring bean。能够为其他 Spring bean 做到的事情都可以同样应用到 Spring 切面中，例如为它们注入依赖。 应用切面来歌颂骑士可能只是有点好玩而已，但是 Spring AOP 可以做很多有实际意义的事情。在后续的各章中，你还会了解基于 Spring AOP 实现声明式事务和安全（第 9 章和第 14 章）。

### 使用模板消除样式代码

样板式代码的一个常见范例是使用 JDBC 访问数据库查询数据。举个例子，如果你曾经用过 JDBC，那么你或许会写出类似下面的代码。

```java
public Employee getEmployeeById(long id) {
  
  Connection conn = null;
  PreparedStatement stmt = null;
  Result rs = null;
  
  try {
    conn = dataSource.getConnection();
    stmt = conn.prepareStatment(
      "select id, firstname, lastname, salary from " +
      "employee where id=?");
    stmt.setLong(1, id);
    rs = stmt.executeQuery();
    Employee employee = null;
    if (rs.next()) {
      employee = new Employee();
      employee.setId(rs.getLong("id"));
      employee.setFirstName(rs.getString("firstname"));
      employee.setLastName(rs.getString("lastname"));
      employee.setSalary(rs.getBigDecimal("salary"));
    }
    return employee;
  } catch (SQLException e) {
  } finally {
    if (rs != null) {
      try {
        rs.close();
      } catch (SQLException e) {
      }
    }
    
    if (stmt != null) {
      try {
        stmt.close();
      } catch (SQLException e) {
      }
    }
    
    if (conn != null) {
      try {
        conn.close();
      } catch (SQLException e) {
      }
    }    
  }
  return null;
}
```

正如你所看到的，这段 JDBC 代码查询数据库获得员工姓名和薪水。我打赌你很难把上面的代码逐行看完，这是因为少量查询员工的代码淹没在一堆 JDBC 的样板式代码中。首先你需要创建一个数据库连接，然后再创建一个语句对象，最后你才能进行查询。为了平息 JDBC 可能会出现的怒火，你必须捕捉 SQLException，这是一个检查型异常，即使它抛出后你也做不了太多事情。

最后，毕竟该说的也说了，该做的也做了，你不得不清理战场，关闭数据库连接、语句和结果集。同样为了平息 JDBC 可能会出现的怒火，你依然要捕捉 SQLException。

程序清单 1.12 中的代码和你实现其他JDBC操作时所写的代码几乎是相同的。只有少量的代码与查询员工逻辑有关系，其他的代码都是 JDBC 的样板代码。

JDBC 不是产生样板式代码的唯一场景。在许多编程场景中往往都会导致类似的样板式代码，JMS、JNDI 和使用 REST 服务通常也涉及大量的重复代码。

Spring 旨在通过模板封装来消除样板式代码。Spring 的 JdbcTemplate 使得执行数据库操作时，避免传统的 JDBC 样板代码成为了可能。

举个例子，使用 Spring 的 JdbcTemplate（利用了 Java 5 特性的 JdbcTemplate 实现）重写的 getEmployeeById() 方法仅仅关注于获取员工数据的核心逻辑，而不需要迎合 JDBC API 的需求。程序清单 1.13 展示了修订后的 getEmployeeById() 方法。

```java
public Employee getEmployeeById(long id) {
  return jdbcTemplate.queryForObject(
    "select id, firstname, lastname, salary " +
    "from employee where id=?",
    new RowMapper<Employee>() {
      public Employee mapRow(ResultSet rs, int rowNum) throws SQLException {
        Employee employee = new Employee();
        employee.setId(rs.getLong("id"));
        employee.setFirstName(rs.getString("firstname"));
        employee.setLastName(rs.getString("lastname"));
        employee.setSalary(rs.getBigDecimal("salary"));
        return employee;
      }
    }, 
    id);
}
```

正如你所看到的，新版本的 getEmployeeById() 简单多了，而且仅仅关注于从数据库中查询员工。模板的 queryForObject() 方法需要一个 SQL 查询语句，一个 RowMapper 对象（把数据映射为一个域对象），零个或多个查询参数。GetEmployeeById() 方法再也看不到以前的 JDBC 样板式代码了，它们全部被封装到了模板中。

我已经向你展示了 Spring 通过面向 POJO 编程、DI、切面和模板技术来简化 Java 开发中的复杂性。在这个过程中，我展示了在基于 XML 的配置文件中如何配置 bean 和切面，但这些文件是如何加载的呢？它们被加载到哪里去了？让我们再了解下 Spring 容器，这是应用中的所有 bean 所驻留的地方。

## 容纳你的Bean

在基于 Spring 的应用中，你的应用对象生存于 Spring 容器（container） 中。如图所示，Spring 容器负责创建对象，装配它们，配置它们并管理它们的整个生命周期，从生存到死亡（在这里，可能就是 new 到 finalize()）。

![1.4 spring 容器](C:\Users\汤琛\Desktop\学习资料\Spring详解\images\1.4 spring 容器.jpg)

接下来，你将了解如何配置 Spring，从而让它知道该创建、配置和组装哪些对象。但首先，最重要的是了解容纳对象的容器。理解容器将有助于理解对象是如何被管理的。

容器是 Spring 框架的核心。Spring 容器使用 DI 管理构成应用的组件，它会创建相互协作的组件之间的关联。毫无疑问，这些对象更简单干净，更易于理解，更易于重用并且更易于进行单元测试。

Spring 容器并不是只有一个。Spring 自带了多个容器实现，可以归为两种不同的类型。**bean 工厂（由 org.springframework.beans.factory.BeanFactory 接口定义）**是最简单的容器，提供基本的 DI 支持。**应用上下文（由 org.springframework.context.ApplicationContext 接口定义）**基于 BeanFactory 构建，并提供应用框架级别的服务，例如从属性文件解析文本信息以及发布应用事件给感兴趣的事件监听者。

虽然我们可以在 bean 工厂和应用上下文之间任选一种，但 bean 工厂对 大多数应用来说往往太低级了，因此，应用上下文要比 bean 工厂更受欢迎。我们会把精力集中在应用上下文的使用上，不再浪费时间讨论 bean 工厂。

### **使用应用上下文**

Spring 自带了多种类型的应用上下文。下面罗列的几个是你最有可能遇到的。

- AnnotationConfigApplicationContext：从一个或多个基于 Java 的配置类中加载 Spring 应用上下文。
- AnnotationConfigWebApplicationContext：从一个或多个基于 Java 的配置类中加载 Spring Web 应用上下文。
- lassPathXmlApplicationContext：从类路径下的一个或多个 XML 配置文件中加载上下文定义，把应用上下文的定义文件作为类资源。
- FileSystemXmlapplicationcontext：从文件系统下的一 个或多个 XML 配置文件中加载上下文定义。
- XmlWebApplicationContext：从 Web 应用下的一个或多个 XML 配置文件中加载上下文定义。

当在第 8 章讨论基于 Web 的 Spring 应用时，我们会对 AnnotationConfigWebApplicationContext 和  XmlWebApplicationContext 进行更详细的讨论。现在我们先简单地使用FileSystemXml-ApplicationContext 从文件系统中加载应用上下文或者使用 ClassPathXmlApplicationContext 从类 路径中加载应用上下文。

无论是从文件系统中装载应用上下文还是从类路径下装载应用上下文，将 bean 加载到 bean 工厂的过程都是相似的。例如，如下代码展示 了如何加载一个 FileSystemXmlApplicationContext：

```java
ApplicationContext context = new FileSystemXmlApplicationContext("c:/knight.xml");
```

类似地，你可以使用 ClassPathXmlApplicationContext 从应用的类路径下加载应用上下文：

```java
ApplicationContext context = new ClassPathXmlApplicationContext("knight.xml");
```

使用 FileSystemXmlApplicationContext 和使用 ClassPathXmlApplicationContext 的区别在于：FileSystemXmlApplicationContext 在指定的文件系统路径下查找 knight.xml文件；而 ClassPathXmlApplicationContext 是在所有的类路径（包含 JAR 文件）下查找 knight.xml 文件。

如果你想从 Java 配置中加载应用上下文，那么可以使用 AnnotationConfigApplicationContext：

```java
ApplicationContext context = new AnnotationConfigApplicationContext(com.springinaction.knights.config.KnightConfig.class);
```

在这里没有指定加载 Spring 应用上下文所需的 XML 文 件，AnnotationConfigApplicationContext 通过一个配置类加载 bean。

应用上下文准备就绪之后，我们就可以调用上下文的 getBean() 方法从 Spring 容器中获取 bean。 现在你应该基本了解了如何创建Spring容器，让我们对容器中bean的 生命周期做更进一步的探究。

### bean的生命周期

在传统的 Java 应用中，bean 的生命周期很简单。使用 Java 关键字 new 进行 bean 实例化，然后该 bean 就可以使用了。一旦该 bean 不再被使用，则由 Java 自动进行垃圾回收。

相比之下，Spring 容器中的 bean 的生命周期就显得相对复杂多了。正确理解 Spring bean 的生命周期非常重要，因为你或许要利用 Spring 提供的扩展点来自定义 bean 的创建过程。下图展示了 bean 装载到 Spring 应用上下文中的一个典型的生命周期过程。

![1.5 spring bean生命周期](C:\Users\汤琛\Desktop\学习资料\Spring详解\images\1.5 spring bean生命周期.jpg)

正如你所见，在 bean 准备就绪之前，bean 工厂执行了若干启动步骤。

1. Spring 对 bean 进行实例化；
2. Spring 将值和 bean 的引用注入到 bean 对应的属性中；
3. 如果 bean 实现了 BeanNameAware 接口，Spring 将 bean 的 ID 传递给 setBeanName()方法；
4. 如果 bean 实现了 BeanFactoryAware 接口，Spring 将调用  setBeanFactory() 方法，将 BeanFactory 容器实例传入；
5. 如果 bean 实现了 ApplicationContextAware 接口，Spring 将调用 setApplicationContext() 方法，将 bean 所在的应用上下文的引用传入进来；
6. 如果 bean 实现了 BeanPostProcessor 接口，Spring 将调用它们的 postProcessBefore-Initialization() 方法；
7. 如果 bean 实现了 InitializingBean 接口，Spring 将调用它们的 afterPropertiesSet() 方法。类似地，如果 bean 使用 initmethod 声明了初始化方法，该方法也会被调用；
8. 如果 bean 实现了 BeanPostProcessor 接口，Spring 将调用它们的 postProcessAfter-Initialization() 方法；
9. 此时，bean 已经准备就绪，可以被应用程序使用了，它们将一直驻留在应用上下文中，直到该应用上下文被销毁；
10. 如果 bean 实现了 DisposableBean 接口，Spring 将调用它的 destroy() 接口方法。

同样，如果 bean 使用 destroy-method 声明了销毁方法，该方法也会被调用。现在你已经了解了如何创建和加载一个  Spring 容器。但是一个空的容器并没有太大的价值，在你把东西放进去之前，它里面什么都没有。为了从 Spring 的 DI 中受益，我们必须将应用对象装配进 Spring 容器中。我们将在第 2 章对 bean 装配进行更详细的探讨。

## **俯瞰 Spring 风景线**

正如你所看到的，Spring 框架关注于通过 DI、AOP 和消除样板式代码来简化企业级 Java 开发。即使这是 Spring 所能做的全部事情，那 Spring 也值得一用。但是，Spring 实际上的功能超乎你的想象。

在 Spring 框架的范畴内，你会发现 Spring 简化 Java 开发的多种方式。但在 Spring 框架之外还存在一个构建在核心框架之上的庞大生态圈，它将 Spring 扩展到不同的领域，例如 Web 服务、REST、移动开发以及 NoSQL。

### **Spring 模块**

**Spring 核心容器**

容器是 Spring 框架最核心的部分，它管理着 Spring 应用中 bean 的创建、配置和管理。在该模块中，包括了 Spring bean 工厂，它为 Spring 提供 了 DI 的功能。基于 bean 工厂，我们还会发现有多种 Spring 应用上下文的实现，每一种都提供了配置 Spring 的不同方式。

除了 bean 工厂和应用上下文，该模块也提供了许多企业服务，例如Email、JNDI 访问、EJB 集成和调度。

所有的 Spring 模块都构建于核心容器之上。当你配置应用时，其实你隐式地使用了这些类。贯穿本书，我们都会涉及到核心模块，在第 2 章中我们将会深入探讨 Spring 的 DI。

**Spring 的 AOP 模块**

在 AOP 模块中，Spring 对面向切面编程提供了丰富的支持。这个模块是 Spring 应用系统中开发切面的基础。与 DI 一样，AOP 可以帮助应用对象解耦。借助于 AOP，可以将遍布系统的关注点（例如事务和安全）从它们所应用的对象中解耦出来。

**数据访问与集成**

使用 JDBC 编写代码通常会导致大量的样板式代码，例如获得数据库连接、创建语句、处理结果集到最后关闭数据库连接。Spring 的 JDBC 和 DAO（Data Access Object）模块抽象了这些样板式代码，使我们的数据库代码变得简单明了，还可以避免因为关闭数据库资源失败而引发的问题。该模块在多种数据库服务的错误信息之上构建了一个语义丰富的异常层，以后我们再也不需要解释那些隐晦专有的 SQL 错误信息了！

对于那些更喜欢 ORM（Object-Relational Mapping）工具而不愿意直接使用 JDBC 的开发者，Spring 提供了 ORM 模块。Spring 的 ORM 模块建立在对 DAO 的支持之上，并为多个 ORM 框架提供了一种构建 DAO 的简便方式。Spring 没有尝试去创建自己的 ORM 解决方案，而是对许多流行的 ORM 框架进行了集成，包括 Hibernate、Java Persisternce API、Java Data Object 和 iBATIS SQL Maps。Spring 的事务管理支持所有的 ORM 框架以及 JDBC。

**Web与远程调用**

MVC（Model-View-Controller）模式是一种普遍被接受的构建 Web 应用的方法，它可以帮助用户将界面逻辑与应用逻辑分离。Java 从来不缺少 MVC 框架，Apache 的 Struts、JSF、WebWork 和 Tapestry 都是可选的最流行的MVC框架。

虽然 Spring 能够与多种流行的 MVC 框架进行集成，但它的 Web 和远程调用模块自带了一个强大的 MVC 框架，有助于在 Web 层提升应用的松耦合水平。在第 5 章到第 7 章中，我们将会学习 Spring 的 MVC 框架。

除了面向用户的 Web 应用，该模块还提供了多种构建与其他应用交互的远程调用方案。Spring 远程调用功能集成了  RMI（Remote Method Invocation）、Hessian、Burlap、JAX-WS，同时 Spring 还自带了一个 远程调用框架：HTTP invoker。Spring 还提供了暴露和使用 REST API 的良好支持。

**Instrumentation**

Spring 的 Instrumentation 模块提供了为 JVM 添加代理（agent）的功能。 具体来讲，它为 Tomcat 提供了一个织入代理，能够为 Tomcat 传递类文 件，就像这些文件是被类加载器加载的一样。

如果这听起来有点难以理解，不必对此过于担心。这个模块所提供的 Instrumentation 使用场景非常有限，在本书中，我们不会介绍该模块。

**测试**

鉴于开发者自测的重要性，Spring 提供了测试模块以致力于 Spring 应用的测试。

通过该模块，你会发现 Spring 为使用 JNDI、Servlet 和 Portlet 编写单元测试提供了一系列的 mock 对象实现。对于集成测试，该模块为加载 Spring 应用上下文中的 bean 集合以及与 Spring 上下文中的 bean 进行交互提供了支持。

在本书中，有很多的样例都是测试驱动的，将会使用到  Spring 所提供的测试功能。

## **Spring 的新功能**

### **Spring 3.1 新特性**

Spring 3.1 还提供了声明式缓存的支持以及众多针对 Spring MVC 的功能增强。下面的列表展现了 Spring 3.1 重要的功能升级：

- 为了解决各种环境下（如开发、测试和生产）选择不同配置的问题，Spring 3.1 引入了环境 profile 功能。借助于 profile，就能根据应用部署在什么环境之中选择不同的数据源 bean；
- 在 Spring 3.0 基于 Java 的配置之上，Spring 3.1 添加了多个 enable 注解，这样就能使用这个注解启用 Spring 的特定功能；
- 添加了 Spring 对声明式缓存的支持，能够使用简单的注解声明缓存边界和规则，这与你以前声明事务边界很类似；
- 新添加的用于构造器注入的 c 命名空间，它类似于 Spring 2.0 所提供的面向属性的 p 命名空间，p 命名空间用于属性注入，它们都是非常简洁易用的；
- **Spring 开始支持 Servlet 3.0**，包括在基于 Java 的配置中声明 Servlet 和 Filter，而不再借助于 web.xml；
- 改善 Spring 对 JPA 的支持，使得它能够在 Spring 中完整地配置 JPA，不必再使用persistence.xml 文件。

Spring 3.1 还包含了多项针对 Spring MVC 的功能增强：

- 自动绑定路径变量到模型属性中；
- 提供了 @RequestMapping produces 和 consumes 属性，用于匹配请求中的 Accept 和 Content-Type 头部信息；
- 提供了 **@RequestPart 注解**，用于将 multipart 请求中的某些部分绑定到处理器的方法参数中；
- 支持 Flash 属性（在 redirect 请求之后依然能够存活的属性）以及用于在请求间存放 flash 属性的 RedirectAttributes 类型。

除了 Spring 3.1 所提供的新功能以外，同等重要的是要注意 Spring 3.1 不再支持的功能。具体来讲，为了支持原生的 EntityManager，Spring 的 JpaTemplate 和 JpaDaoSupport 类被废弃掉了。尽管它们已经被废弃了，但直到 Spring 3.2 版本，它依然是可以使用的。但最好不要再使用它们了，因为它们不会进行更新以支持 JPA 2.0，并且已经在 Spring 4 中移除掉了。

### **Spring 3.2 新特性**

Spring 3.1 在很大程度上聚焦于配置改善以及其他的一些增强，包括 Spring MVC 的增强，而 Spring 3.2 是主要关注 Spring MVC 的一个发布版本。Spring MVC 3.2 带来了如下的功能提升：

- Spring 3.2 的控制器（Controller）可以**使用 Servlet 3.0 的异步请求**，允许在一个独立的线程中处理请求，从而将 Servlet 线程解放出来处理更多的请求；
- 尽管从 Spring 2.5 开始，Spring MVC 控制器就能以 POJO 的形式进行很便利地测试，但是 Spring 3.2 引入了 Spring MVC 测试框架，用于为控制器编写更为丰富的测试，断言它们作为控制器的行为行为是否正确，而且在使用的过程中并不需要 Servlet 容器；
- 除了提升控制器的测试功能，Spring 3.2 还包含了基于 RestTemplate 的客户端的测试支持，在测试的过程中，不需要往真正的 REST 端点上发送请求；
- @ControllerAdvice 注解能够将通用的 @ExceptionHandler、@InitBinder 和 @ModelAttributes 方法收集到一个类中，并应用到所有控制器上；
- 在 Spring 3.2 之前，只能通过 ContentNegotiatingViewResolve r使用完整的内容协商（full content negotiation）功能。但是在 Spring 3.2 中，完整的内容协商功能可以在整个 Spring MVC 中使用，即便是依赖于消息转换器（message converter）使用和产生内容的控制器方法也能使用该功能；
- Spring MVC 3.2 包含了一个新的 **@MatrixVariable 注解**，这个注解能够将请求中的矩阵变量（matrix variable）绑定到处理器的方法参数中；
- 基础的抽象类 AbstractDispatcherServletInitializer 能够非常便利地配置 DispatcherServlet，而不必再使用 web.xml。与之类似，当你希望通过基于Java 的方式来配置 Spring 的时候，可以使用 AbstractAnnotationConfigDispatcherServletInitializer 的子类；
- 新增了 ResponseEntityExceptionHandler，可以用来替代 DefaultHandlerExceptionResolver。ResponseEntityExceptionHandler 方法会返回 ResponseEntity，而不是 ModelAndView；
- RestTemplate 和 @RequestBody 的参数可以支持范型；
- RestTemplate 和 @RequestMapping 可以支持 HTTP PATCH 方 法；
- 在拦截器匹配时，支持使用 URL 模式将其排除在拦截器的处理功能之外。

虽然 Spring MVC 是 Spring 3.2 改善的核心内容，但是它依然还增加了多项非 MVC 的功能改善。下面列出了 Spring 3.2 中几项最为有意思的新特性：

- @Autowired、@Value 和 @Bean 注解能够作为元注解，用于创建自定义的注入和 bean 声明注解；
- **@DateTimeFormat 注解**不再强依赖 JodaTime。如果提供了 JodaTime，就会使用它，否则的话，会使用SimpleDateFormat；
- Spring 的声明式缓存提供了对 JCache 0.5 的支持；
- 支持定义全局的格式来解析和渲染日期与时间；
- 在集成测试中，能够配置和加载 WebApplicationContext；
- 在集成测试中，能够针对 request 和 session 作用域的 bean 进行测 试。

### **Spring 4.0 新特性**

当编写本书时，Spring 4.0 是最新的发布版本。在 Spring 4.0 中包含了很多令人兴奋的新特性，包括：

- Spring 提供了对 **WebSocket 编程**的支持，包括支持 JSR-356 —— Java API for WebSocket；
- 鉴于 WebSocket 仅仅提供了一种低层次的 API，急需高层次的抽象，因此 Spring 4.0 在 WebSocket 之上提供了一个高层次的面向消息的编程模型，该模型基于 SockJS，并且包含了对STOMP协议的支持；
- 新的消息（messaging）模块，很多的类型来源于 Spring Integration 项目。这个消息模块支持 Spring 的 SockJS/STOMP 功能，同时提供了基于模板的方式发布消息；
- Spring 是第一批（如果不说是第一个的话）支持 Java 8 特性的 Java 框架，比如它所**支持的 lambda 表达式**。别的暂且不说，这首先能够让使用特定的回调接口（如 RowMapper 和 JdbcTemplate） 更加简洁，代码更加易读；
- 与 Java 8 同时得到支持的是 JSR-310 —— Date 与 Time API，在处理日期和时间时，它为开发者提供了比 java.util.Date 或 java.util.Calendar 更丰富的 API；
- 添加了条件化创建 bean 的功能，在这里只有开发人员定义的条件满足时，才会创建所声明的 bean；
- Spring 4.0 包含了 Spring RestTemplate 的一个新的异步实现， 它会立即返回并且允许在操作完成后执行回调；
- 添加了对多项 JEE 规范的支持，包括 JMS 2.0、JTA 1.2、JPA 2.1 和 Bean Validation 1.1。

