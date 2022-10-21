# **保护 Web 应用**

本章内容：

- Spring Security 介绍
- 使用 Servlet 规范中的 Filter 保护 Web 应用
- 基于数据库和 LDAP 进行认证

有一点不知道你是否在意过，那就是在电视剧中大多数人从不锁门？这是司空见惯的现象。在《*Seinfeld*》中，Kramer 经常到 Jerry 的房间里并从他的冰箱里拿东西吃。在《*Friends*》中，很多剧中的角色经常不敲门就不假思索地进入别人的房间。有一次在伦敦，Ross 甚至闯入 Chandler 的旅馆房间，差一点就撞见 Chandler 和 Ross 妹妹的私情。

在《*Leave it to Beaver*》热播的时代，人们不锁门这事儿并不值得大惊小怪。但是在这个隐私和安全被看得极其重要的年代，看到电视剧中的角色允许别人大摇大摆地进入自己的寓所或家中，实在让人难以置信。

现在，信息可能是我们最有价值的东西，一些不怀好意的人想尽办法试图偷偷进入不安全的应用程序来窃取我们的数据和身份信息。作为软件开发人员，我们必须采取措施来保护应用程序中的信息。无论你是通过用户名/密码来保护电子邮件账号，还是基于交易 PIN 来保护经纪账户，安全性都是绝大多数应用系统中的一个重要切面 （aspect）。

我有意选择了“切面”这个词来描述应用系统的安全性。安全性是超越应用程序功能的一个关注点。应用系统的绝大部分内容都不应该参与到与自己相关的安全性处理中。尽管我们可以直接在应用程序中编写安全性功能相关的代码（这种情况并不少见），但更好的方式还是将安全性相关的关注点与应用程序本身的关注点进行分离。

如果你觉得安全性听上去好像是使用面向切面技术实现的，那你猜对了。在本章中，我们将使用切面技术来探索保护应用程序的方式。不过我们不必自己开发这些切面 —— 我们将介绍 Spring Security，这是一种基于 Spring AOP 和 Servlet 规范中的 Filter 实现的安全框架。

## **Spring Security 简介**

Spring Security 是为基于 Spring 的应用程序提供声明式安全保护的安全性框架。Spring Security 提供了完整的安全性解决方案，它能够在 Web 请求级别和方法调用级别处理身份认证和授权。因为基于 Spring 框架，所以 Spring Security 充分利用了依赖注入（dependency injection， DI）和面向切面的技术。

最初，Spring Security 被称为 Acegi Security。Acegi 是一个强大的安全框架，但是它存在一个严重的问题：那就是需要大量的 XML 配置。我不会向你介绍这种复杂配置的细节。总之一句话，典型的 Acegi 配置有几百行 XML 是很常见的。

到了 2.0 版本，Acegi Security 更名为 Spring Security。但是 2.0 发布版本所带来的不仅仅是表面上名字的变化。为了在 Spring 中配置安全性，Spring Security 引入了一个全新的、与安全性相关的 XML 命名空间。

这个新的命名空间连同注解和一些合理的默认设置，将典型的安全性配置从几百行 XML 减少到十几行。Spring Security 3.0 融入了 SpEL，这进一步简化了安全性的配置。

它的最新版本为 3.2，Spring Security 从两个角度来解决安全性问题。它使用 Servlet 规范中的 Filter 保护 Web 请求并限制 URL 级别的访问。Spring Security 还能够使用 Spring AOP 保护方法调用 —— 借助于对象代理和使用通知，能够确保只有具备适当权限的用户才能访问安全保护的方法。

在本章中，我们将会关注如何将 Spring Security 用于 Web 层的安全性之中。在稍后的第 14 章中，我们会重新学习 Spring Security，了解它如何保护方法的调用。

### **理解 Spring Security 的模块**

不管你想使用 Spring Security 保护哪种类型的应用程序，第一件需要做的事就是将 Spring Security 模块添加到应用程序的类路径下。Spring Security 3.2 分为 11 个模块，如表 9.1 所示。

|           模块           |                             描述                             |
| :----------------------: | :----------------------------------------------------------: |
|           ACL            | 支持通过访问控制列表（access control list，ACL）为域对象提供安全性 |
|     切面（Aspects）      | 一个很小的模块，当使用 Spring Security 注解时，会使用基于 AspectJ 的切面，而不是使用标准的 Spring AOP |
| CAS客户端 （CAS Client） | 提供与Jasig 的中心认证服务（Central Authentication Service， CAS）进行集成的功能 |
|  配置 （Configuration）  | 包含通过 XML 和 Java 配置 Spring Security 的功能支持核心（Core） 提供 Spring Security基本库 |
|  加密 （Cryptography）   |                  提供了加密和密码编码的功能                  |
|           LDAP           |                    支持基于 LDAP 进行认证                    |
|          OpenID          |                支持使用 OpenID 进行集中式认证                |
|         Remoting         |               提供了对 Spring Remoting 的支持                |
|  标签库（Tag Library）   |                Spring Security 的 JSP 标签库                 |
|           Web            |     提供了 Spring Security 基于 Filter 的 Web 安全性支持     |

应用程序的类路径下至少要包含 Core 和 Configuration 这两个模块。Spring Security 经常被用于保护 Web 应用，这显然也是 Spittr 应用的场景，所以我们还需要添加 Web 模块。同时我们还会用到 Spring Security 的 JSP 标签库，所以我们需要将这个模块也添加进来。

现在，我们已经为在 Spring Security 中进行安全性配置做好了准备。让我们看看如何使用 Spring Security 的 XML 命名空间。

### **过滤 Web 请求**

Spring Security 借助一系列 Servlet Filter 来提供各种安全性功能。你可能会想，这是否意味着我们需要在 web.xml 或 WebApplicationInitializer 中配置多个 Filter 呢？实际上，借助于 Spring 的小技巧，我们只需配置一个 Filter 就可以了。

DelegatingFilterProxy 是一个特殊的 Servlet Filter，它本身所做的工作并不多。只是将工作委托给一个 javax.servlet.Filter 实现类，这个实现类作为一个注册在 Spring 应用的上下文中， 如图 9.1 所示。

![9.1 filter bean](C:\Users\汤琛\Desktop\学习资料\Spring详解\images\9.1 filter bean.jpg)

如果你喜欢在传统的 web.xml 中配置 Servlet 和 Filter 的话，可以使用 `<filter>` 元素，如下所示：

```xml
<filter>
  <filter-name>springSecurityFilterChain</filter-name>
  <filter-class>
    org.springframework.web.filter.DelegatingFilteProxy
  </filter-class>
</filter>
```

在这里，最重要的是 `<filter-name>` 设置成了 springSecurityFilterChain。这是因为我们马上就会将 Spring Security 配置在 Web 安全性之中，这里会有一个名为 springSecurityFilterChain 的 Filter bean，DelegatingFilterProxy 会将过滤逻辑委托给它。

如果你希望借助 WebApplicationInitializer 以 Java 的方式来配置 Delegating-FilterProxy 的话，那么我们所需要做的就是创建一个扩展的新类：

```java
package spitter.config;
import org.springframwork.security.web.context.AbstractSecurityWebApplicationInitializer;

public class SecurityWebInitializer extends AbstractSecurityWebApplicationInitializer{
}
```

AbstractSecurityWebApplicationInitializer 实现了 WebApplicationInitializer，因此 Spring 会发现它，并用它在 Web 容器中注册 DelegatingFilterProxy。尽管我们可以重载它的 appendFilters() 或 insertFilters() 方法来注册自己选择的 Filter，但是要注册 DelegatingFilterProxy 的话，我们并不需要重载任何方法。

不管我们通过 web.xml 还是通过 AbstractSecurityWebApplicationInitializer 的子类来配置 DelegatingFilterProxy，它都会拦截发往应用中的请求，并将请求委托给 ID 为 springSecurityFilterChain bean。

springSecurityFilterChain 本身是另一个特殊的 Filter，它也被称为FilterChain-Proxy。它可以链接任意一个或多个其他的 Filter。Spring Security 依赖一系列 Servlet Filter 来提供不同的安全特性。但是，你几乎不需要知道这些细节，因为你不需要显式声明 springSecurityFilterChain 以及它所链接在一起的其他  Filter。当我们启用 Web 安全性的时候，会自动创建这些 Filter。

为了让 Web 安全性运行起来，我们创建一个最简单的安全性配置。

### **编写简单的安全性配置**

在 Spring Security 的早期版本中（在其还被称为 Acegi Security 之时），为了在 Web 应用中启用简单的安全功能，我们需要编写上百行的 XML 配置。Spring Security 2.0 提供了安全性相关的 XML 配置命名空间，让情况有了一些好转。

Spring 3.2 引入了新的 Java 配置方案，完全不再需要通过 XML 来配置安全性功能了。如下的程序清单展现了 Spring Security 最简单的 Java 配置。

```java
package spitter.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.annotation.web.servlet.configuration.EnableWebSecurity;

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
}
```

顾名思义，@EnableWebSecurity 注解将会启用 Web 安全功能。但它本身并没有什么用处，Spring Security 必须配置在一个实现了 WebSecurityConfigurer 的 bean 中，或者（简单起见）扩展 WebSecurityConfigurerAdapter。在 Spring 应用上下文中，任何实现了 WebSecurityConfigurer 的 bean 都可以用来配置 Spring Security，但是最为简单的方式还是像程序清单 9.1 那样扩展 WebSecurityConfigurerAdapter 类。

@EnableWebSecurity 可以启用任意 Web 应用的安全性功能，不过，如果你的应用碰巧是使用 Spring MVC 开发的，那么就应该考虑使用 @EnableWeb-MvcSecurity 替代它，如程序清单 9.2 所示。

```java
package spitter.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.annotation.web.servlet.configuration.EnableWebMvcSecurity;

@Configuration
@EnableWebMvcSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
}
```

除了其他的内容以外，@EnableWebMvcSecurity 注解还配置了一个 Spring MVC 参数解析解析器（argument resolver），这样的话处理器方法就能够通过带有 @AuthenticationPrincipal 注解的参数获得认证用户的 principal（或username）。它同时还配置了一个 bean， 在使用 Spring 表单绑定标签库来定义表单时，这个 bean 会自动添加一个隐藏的跨站请求伪造（cross-site request forgery，CSRF）token 输入域。

看起来似乎并没有做太多的事情，但程序清单 9.1 和 9.2 中的配置类会给应用产生很大的影响。其中任何一种配置都会将应用严格锁定，导致没有人能够进入该系统了！

尽管不是严格要求的，但我们可能希望指定 Web 安全的细节，这要通过重载 WebSecurityConfigurerAdapter 中的一个或多个方法来实现。我们可以通过重载 WebSecurityConfigurerAdapter 的三个 configure() 方法来配置 Web 安全性，这个过程中会使用传递进来的参数设置行为。表 9.2 描述了这三个方法。

|                  方法                   |                    描述                     |
| :-------------------------------------: | :-----------------------------------------: |
|         configure(WebSecurity)          | 通过重载，配置 Spring Security 的 Filter 链 |
|         configure(HttpSecurity)         |    通过重载，配置如何通过拦截器保护请求     |
| configure(AuthenticationManagerBuilder) |       通过重载，配置 user-detail 服务       |

让我们重新看一下程序清单 9.2，可以看到它没有重写上述三个 configure() 方法中的任何一个，这就说明了为什么应用现在是被锁定的。尽管对于我们的需求来讲默认的 Filter 链是不错的，但是默认的 configure(HttpSecurity) 实际上等同于如下所示：

```java
protected void configure(HttpSecurity http) throws Exception {
  http
    .authorizeRequests()
    .anyRequest().authorized()
    .and()
    .formLogin()
    .and()
    .httpBasic();
}
```

这个简单的默认配置指定了该如何保护 HTTP 请求，以及客户端认证用户的方案。通过调用 authorizeRequests() 和 anyRequest().authenticated() 就会要求所有进入应用的 HTTP 请求都要进行认证。它也配置 Spring Security 支持基于表单的登录以及 HTTP Basic 方式的认证。

同时，因为我们没有重载 configure(AuthenticationManagerBuilder) 方法，所以没有用户存储支撑认证过程。没有用户存储，实际上就等于没有用户。所以，在这里所有的请求都需要认证，但是没有人能够登录成功。

为了让 Spring Security 满足我们应用的需求，还需要再添加一点配置。具体来讲，我们需要：

- 配置用户存储；
- 指定哪些请求需要认证，哪些请求不需要认证，以及所需要的权限；
- 提供一个自定义的登录页面，替代原来简单的默认登录页。

除了 Spring Security 的这些功能，我们可能还希望基于安全限制，有选择性地在 Web 视图上显示特定的内容。

但首先，我们看一下如何在认证的过程中配置访问用户数据的服务。

## 选择查询用户详细信息的服务

假如你计划去一个独家经营的饭店享受一顿晚餐，当然，你会提前几周预订，保证到时候能有一个位置。当到达饭店的时候，你会告诉服务员你的名字。但令人遗憾的是，里面并没有你的预订记录。美好的夜晚眼看就要泡汤了。但是没有人会如此轻易地放弃，你会要求服务员再次确认预订名单。此时，事情变得有些怪异了。

服务员说没有预订名单。你的名字不在名单上 —— 名单上没有任何人 —— 因为根本就不存在这么个名单。这就解释了为什么位置是空的，但我们却进不去。几周后，我们也就明白这家饭店为何最终会关门大吉，被一家墨西哥美食店所代替。

这也是此时我们应用程序的现状。我们没有办法进入应用，即便用户认为他们应该能够登录进去，但实际上却没有允许他们访问应用的数据记录。因为缺少用户存储，现在的应用程序太封闭了，变得不可用。

我们所需要的是用户存储，也就是用户名、密码以及其他信息存储的地方，在进行认证决策的时候，会对其进行检索。

好消息是，Spring Security 非常灵活，能够基于各种数据存储来认证用 户。它内置了多种常见的用户存储场景，如内存、关系型数据库以及 LDAP。但我们也可以编写并插入自定义的用户存储实现。

借助 Spring Security 的 Java 配置，我们能够很容易地配置一个或多个数据存储方案。那我们就从最简单的开始：在内存中维护用户存储。

### **使用基于内存的用户存储**

因为我们的安全配置类扩展了 WebSecurityConfigurerAdapter，因此配置用户存储的最简单方式就是重载 configure() 方法，并以 AuthenticationManagerBuilder 作为传入参数。AuthenticationManagerBuilder 有多个方法可以用来配置 Spring Security 对认证的支持。通过 inMemoryAuthentication() 方法，我们可以启用、配置并任意填充基于内存的用户存储。

例如，在如程序清单 9.3 中，SecurityConfig 重载了 configure() 方法，并使用两个用户来配置内存用户存储。

```java
package spittr.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.annotation.web.servlet.configuration.EnableWebMvcSecurity;

@Configuration
@EnableWebMvcSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
  
  @Override
  protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth
      .inMemoryAuthentication()
      .withUser("user").password("password").roles("USER");
  }
}
```

我们可以看到，configure() 方法中的 AuthenticationManagerBuilder 使用构造者风格的接口来构建认证配置。通过简单地调用 inMemoryAuthentication() 就能启用内存用户存储。但是我们还需要有一些用户，否则的话，这和没有用户并没有什么区别。

因此，我们需要调用 withUser() 方法为内存用户存储添加新的用户，这个方法的参数是 username。withUser() 方法返回的是UserDetailsManager-Configurer.UserDetailsBuilder，这个对象提供了多个进一步配置用户的方法，包括设置用户密码的 password() 方法以及为给定用户授予一个或多个角色权限的 roles() 方法。

在程序清单 9.3 中，我们添加了两个用户，“user”和“admin”，密码均为“password”。“user”用户具有 USER 角色，而“admin”用户具有 ADMIN 和 USER 两个角色。我们可以看到，and() 方法能够将多个用户的配置连接起来。

除了 password()、roles() 和 and() 方法以外，还有其他的几个方法可以用来配置内存用户存储中的用户信息。表 9.3 描述了 UserDetailsManager-Configurer.UserDetailsBuilder 对象所有可用的方法。

需要注意的是，roles() 方法是 authorities() 方法的简写形式。roles() 方法所给定的值都会添加一个“ROLE_”前缀，并将其作为权限授予给用户。实际上，如下的用户配置与程序清单 9.3 是等价的：

```java
auth
  .inMemoryAuthentication()
  .withUser("user").password("password").authorities("ROLE_USER")
  .and()
  .withUser("admin").password("password").authorities("ROLE_USER", "ROLE_ADMIN");
```

|                     方法                      |            描述            |
| :-------------------------------------------: | :------------------------: |
|            accountExpired(boolean)            |    定义账号是否已经过期    |
|            accountLocked(boolean)             |    定义账号是否已经锁定    |
|                     and()                     |        用来连接配置        |
|       authorities(GrantedAuthority...)        | 授予某个用户一项或多项权限 |
| authorities(List<? extends GrantedAuthority>) | 授予某个用户一项或多项权限 |
|            authorities(String...)             | 授予某个用户一项或多项权限 |
|          credentialsExpired(boolean)          |    定义凭证是否已经过期    |
|               disabled(boolean)               |    定义账号是否已被禁用    |
|               password(String)                |       定义用户的密码       |
|               roles(String...)                | 授予某个用户一项或多项角色 |

对于调试和开发人员测试来讲，基于内存的用户存储是很有用的，但是对于生产级别的应用来讲，这就不是最理想的可选方案了。为了用于生产环境，通常最好将用户数据保存在某种类型的数据库之中。

### **基于数据库表进行认证**

用户数据通常会存储在关系型数据库中，并通过 JDBC 进行访问。为了配置 Spring Security 使用以 JDBC 为支撑的用户存储，我们可以使用 jdbc-Authentication() 方法，所需的最少配置如下所示：

```java
@Autowired
DataSource dataSource;
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth
    .jdbcAuthentication()
    .dataSource(dataSource);
}
```

我们必须要配置的只是一个 DataSource，这样的话，就能访问关系型数据库了。在这里，DataSource 是通过自动装配的技巧得到的。

**重写默认的用户查询功能**

尽管默认的最少配置能够让一切运转起来，但是它对我们的数据库模式有一些要求。它预期存在某些存储用户数据的表。更具体来说，下面的代码片段来源于 Spring Security 内部，这块代码展现了当查找用户信息时所执行的 SQL 查询语句：

```java
public static final String DEF_USERS_BY_USERNAME_QUERY =
  "select username, password, enabled " +
  "from users " +
  "where username = ?";
public static final String DEF_AUTHORITIES_BY_USERNAME_QUERY =
  "select username, authority " +
  "from authorities " +
  "where username = ?";
public static final String DEF_GROUP_AUTHORITIES_BY_USERNAME_QUERY =
  "select g.id, g.group_name, ga.authority " +
  "from groups g, group_members gm, group_authorities ga " +
  "where gm.username = ? " +
  "and g.id = ga.group_id " +
  "and g.id = gm.group_id";
```

在第一个查询中，我们获取了用户的用户名、密码以及是否启用的信息，这些信息会用来进行用户认证。接下来的查询查找了用户所授予的权限，用来进行鉴权，最后一个查询中，查找了用户作为群组的成员所授予的权限。

如果你能够在数据库中定义和填充满足这些查询的表，那么基本上就不需要你再做什么额外的事情了。但是，也有可能你的数据库与上面所述并不一致，那么你就会希望在查询上有更多的控制权。如果是这样的话，我们可以按照如下的方式配置自己的查询：

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth
    .jdbcAuthentication()
    .dataSource(dataSource)
    .usersByUsernameQuery(
      "select username, password, true " +
      "from Spitter where username=?")
    .authoritiesByUsernameQuery(
      "select username, 'ROLE_USER' from Spitter where username=?");
}
```

在本例中，我们只重写了认证和基本权限的查询语句，但是通过调用 group-AuthoritiesByUsername() 方法，我们也能够将群组权限重写为自定义的查询语句。

将默认的 SQL 查询替换为自定义的设计时，很重要的一点就是要遵循查询的基本协议。所有查询都将用户名作为唯一的参数。认证查询会选取用户名、密码以及启用状态信息。权限查询会选取零行或多行包含该用户名及其权限信息的数据。群组权限查询会选取零行或多行数据，每行数据中都会包含群组 ID、群组名称以及权限。

**使用转码后的密码**

看一下上面的认证查询，它会预期用户密码存储在了数据库之中。这里唯一的问题在于如果密码明文存储的话，会很容易受到黑客的窃取。但是，如果数据库中的密码进行了转码的话，那么认证就会失败，因为它与用户提交的明文密码并不匹配。

为了解决这个问题，我们需要借助 passwordEncoder() 方法指定一个密码转码器（encoder）：

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth
    .jdbcAuthentication()
    .dataSource(dataSource)
    .usersByUsernameQuery(
      "select username, password, true " +
      "from Spitter where username=?")
    .authoritiesByUsernameQuery(
      "select username, 'ROLE_USER' from Spitter where username=?")
    .passwordEncoder(new StandardPasswordEnconder("123456"));
}
```

passwordEncoder() 方法可以接受 Spring Security 中 PasswordEncoder 接口的任意实现。Spring Security 的加密模块包括 了三个这样的实现：BCryptPasswordEncoder、NoOpPasswordEncoder 和  StandardPasswordEncoder。

上述的代码中使用了 StandardPasswordEncoder，但是如果内置的实现无法满足需求时，你可以提供自定义的实现。PasswordEncoder 接口非常简单：

```java
public interface PasswordEncoder {
  String encode(CharSequence rawPassword);
  boolean matched(CharSequence rawPassword, String encodedPassword);
}
```

不管你使用哪一个密码转码器，都需要理解的一点是，数据库中的密码是永远不会解码的。所采取的策略与之相反，用户在登录时输入的密码会按照相同的算法进行转码，然后再与数据库中已经转码过的密码进行对比。这个对比是在 PasswordEncoder 的 matches() 方法中进行的。

### **基于 LDAP 进行认证**

为了让 Spring Security 使用基于 LDAP 的认证，我们可以使用 ldapAuthentication() 方法。这个方法在功能上类似于 jdbcAuthentication()，只不过是 LDAP 版本。如下的 configure() 方法展现了 LDAP 认证的简单配置：

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth
    .ldapAuthentication()
    .userSearchFilter("{uid={0}}")
    .groupSearchFilter("member={0}");
}
```

方法 userSearchFilter() 和 groupSearchFilter() 用来为基础 LDAP 查询提供过滤条件，它们分别用于搜索用户和组。默认情况下，对于用户和组的基础查询都是空的，也就是表明搜索会在 LDAP 层级结构的根开始。但是我们可以通过指定查询基础来改变这个默认行为：

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth
    .ldapAuthentication()
    .userSearchBase("ou=people");
    .userSearchFilter("{uid={0}}")
    .groupSearchBase("ou=groups");
    .groupSearchFilter("member={0}");
}
```

userSearchBase() 属性为查找用户提供了基础查询。同样，groupSearch-Base() 为查找组指定了基础查询。我们声明用户应该在名为 people 的组织单元下搜索而不是从根开始。而组应该在名为 groups 的组织单元下搜索。

**配置密码比对**

基于 LDAP 进行认证的默认策略是进行绑定操作，直接通过 LDAP 服务器认证用户。另一种可选的方式是进行比对操作。这涉及将输入的密码发送到 LDAP 目录上，并要求服务器将这个密码和用户的密码进行比对。因为比对是在 LDAP 服务器内完成的，实际的密码能保持私密。 如果你希望通过密码比对进行认证，可以通过声明 passwordCompare() 方法来实现：

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth
    .ldapAuthentication()
    .userSearchBase("ou=people");
    .userSearchFilter("{uid={0}}")
    .groupSearchBase("ou=groups");
    .groupSearchFilter("member={0}")
    .passwordCompare();
}
```

默认情况下，在登录表单中提供的密码将会与用户的 LDAP 条目中的 userPassword 属性进行比对。如果密码被保存在不同的属性中，可以通过 passwordAttribute() 方法来声明密码属性的名称：

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth
    .ldapAuthentication()
    .userSearchBase("ou=people");
    .userSearchFilter("{uid={0}}")
    .groupSearchBase("ou=groups");
    .groupSearchFilter("member={0}")
    .passwordCompare()
    .passwordEncoder(new Md5PasswordEncoder())
    .passwordAttribute("passcode");
}
```

在本例中，我们指定了要与给定密码进行比对的是“passcode”属性。另外，我们还可以指定密码转码器。在进行服务器端密码比对时，有一点非常好，那就是实际的密码在服务器端是私密的。但是进行尝试的密码还是需要通过线路传输到 LDAP 服务器上，这可能会被黑客所拦截。为了避免这一点，我们可以通过调用 passwordEncoder() 方法指定加密策略。

在本示例中，密码会进行 MD5 加密。这需要 LDAP 服务器上密码也使用 MD5 进行加密。

**引用远程的 LDAP 服务器**

到目前为止，我们忽略的一件事就是 LDAP 和实际的数据在哪里。我们很开心地配置 Spring 使用 LDAP 服务器进行认证，但是服务器在哪里呢？

默认情况下，Spring Security 的 LDAP 认证假设 LDAP 服务器监听本机的 33389 端口。但是，如果你的 LDAP 服务器在另一台机器上，那么可以使用 contextSource() 方法来配置这个地址：

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth
    .ldapAuthentication()
    .userSearchBase("ou=people");
    .userSearchFilter("{uid={0}}")
    .groupSearchBase("ou=groups");
    .groupSearchFilter("member={0}")
    .contextSource()
    .url("ldap://habuma.com:389/dc=habuma,dc=com");
}
```

contextSource() 方法会返回一个 ContextSourceBuilder 对象，这个对象除了其他功能以外，还提供了 url() 方法用来指定 LDAP 服务器的地址。

**配置嵌入式的 LDAP 服务器**

如果你没有现成的 LDAP 服务器供认证使用，Spring Security 还为我们提供了嵌入式的 LDAP 服务器。我们不再需要设置远程 LDAP 服务器的 URL，只需通过 root() 方法指定嵌入式服务器的根前缀就可以了：

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth
    .ldapAuthentication()
    .userSearchBase("ou=people");
    .userSearchFilter("{uid={0}}")
    .groupSearchBase("ou=groups");
    .groupSearchFilter("member={0}")
    .contextSource()
    .root("dc=habuma,dc=com");
}
```

当 LDAP 服务器启动时，它会尝试在类路径下寻找 LDIF 文件来加载数据。LDIF（LDAP Data Interchange Format，LDAP数据交换格式）是以文本文件展现 LDAP 数据的标准方式。每条记录可以有一行或多行，每项包含一个名值对。记录之间通过空行进行分割。

如果你不想让 Spring 从整个根路径下搜索 LDIF 文件的话，那么可以通过调用 ldif() 方法来明确指定加载哪个 LDIF 文件：

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth
    .ldapAuthentication()
    .userSearchBase("ou=people");
    .userSearchFilter("{uid={0}}")
    .groupSearchBase("ou=groups");
    .groupSearchFilter("member={0}")
    .contextSource()
    .root("dc=habuma,dc=com")
    .ldif("classpath:users.ldif");
}
```

在这里，我们明确要求 LDAP 服务器从类路径根目录下的 users.ldif 文件中加载内容。如果你比较好奇的话，如下就是一个包含用户数据 LDIF 文件，我们可以使用它来加载嵌入式 LDAP 服务器：

```properties
dn: ou=groups,dc=habuma,dc=com 
objectclass: top 
objectclass: organizationalUnit 
ou: groups
dn: ou=people,dc=habuma,dc=com 
objectclass: top 
objectclass: organizationalUnit 
ou: people
dn: uid=habuma, ou=people, dc=hab\oma, dc=com
objectclass: top
objectclass: person
objectclass: organizationalPerson
objectclass: inetOrgPerson
cn: Craig Walls
sn: Walls
uid: habuma
userPassword: password
dn: uid=jsmith,ou=people,dc=habuma,dc=com
objectclass: top
objectclass: person
objectclass: organizationalPerson
objectclass: inetOrgPerson
cn: John Smith
sn: Smith
uid: jsmith
userPassword: password
dn: cn=spittr,outgroups,dc=habuma,dc=com
objectclass: top
objectclass: groupOfNames
cn: spittr
member: uid=habuma,ou=people,dc=habuma,dc=com
```

Spring Security 内置的用户存储非常便利，并且涵盖了最为常用的用户场景。但是，如果你的认证需求不是那么通用的话，那么就需要创建并配置自定义的用户详细信息服务了。

### **配置自定义的用户服务**

假设我们需要认证的用户存储在非关系型数据库中，如 Mongo 或 Neo4j，在这种情况下，我们需要提供一个自定义的 UserDetailsService 接口实现。 UserDetailsService 接口非常简单：

```java
public interface UserDetailsService {
  UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

我们所需要做的就是实现 loadUserByUsername() 方法，根据给定的用户名来查找用户。loadUserByUsername() 方法会返回代表给定用户的 UserDetails 对象。如下的程序清单展现了一个 UserDetailsService 的实现，它会从给定的 SpitterRepository 实现中查找用户。

```java
package spittr.security;

import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;

import spittr.Spitter;
import spittr.data.SpitterRepository;

public class SpitterUserService(SpitterRepository spitterRepository) {
  
  private final SpitterRepository spitterRepository;
  
  public SpitterUserService(SpitterRepository spitterRepository){
    this.spitterRepository = spitterRepository;
  }
  
  @Override
  public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    Spitter spitter = spitterRepository.findByUsername(username);
    if (spitter != null) {
      List<GrantedAuthority> authorities = new ArrayList<>();
      authorities.add(new SimpleGrantedAuthority("ROLE_SPITTER"));
    }
    
    return new User(
      spitter.getUsername(),
      spitter.getPassword(),
      authorities);
  }
  throw new UsernameNotFoundException(
    "User '" + username + "' not found.");
}
```

SpitterUserService 有意思的地方在于它并不知道用户数据存储在什么地方。设置进来的 SpitterRepository 能够从关系型数据库、文档数据库或图数据中查找 Spitter 对象，甚至可以伪造一个。SpitterUserService 不知道也不会关心底层所使用的数据存储。它只是获得 Spitter 对象，并使用它来创建 User 对象。（User 是 UserDetails 的具体实现。）

为了使用 SpitterUserService 来认证用户，我们可以通过 userDetailsService() 方法将其设置到安全配置中：

```java
@Autowired
SpitterRepository spitterRepository;

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth
    .userDetailsService(new SpitterUserService(spitterRepository));
}
```

userDetailsService() 方法（类似于 jdbcAuthentication()、ldapAuthentication() 以及 inMemoryAuthentication()）会配置一个用户存储。不过，这里所使用的不是 Spring 所提供的用户存储，而是使用 UserDetailsService 的实现。

另外一种值得考虑的方案就是修改 Spitter，让其实现 UserDetails。这样的话，loadUserByUsername() 就能直接返回 Spitter 对象了，而不必再将它的值复制到 User 对象中。

## **拦截请求**

在前面的 9.1.3 小节中，我们看到一个特别简单的 Spring Security 配置，在这个默认的配置中，会要求所有请求都要经过认证。有些人可能会说，过多的安全性总比安全性太少要好。但也有一种说法就是要适量地应用安全性。

在任何应用中，并不是所有的请求都需要同等程度地保护。有些请求需要认证，而另一些可能并不需要。有些请求可能只有具备特定权限的用户才能访问，没有这些权限的用户会无法访问。

例如，考虑 Spittr 应用的请求。首页当然是公开的，不需要进行保护。类似地，因为所有的 Spittle 都是公开的，所以展现 Spittle 的页面不需要安全性。但是，创建 Spittle 的请求只有认证用户才能执行。同样，尽管用户基本信息页面是公开的，不需要认证，但是，如果要处理 `/spitters/me` 请求，并展现当前用户的基本信息时，那么就需要进行认证，从而确定要展现谁的信息。

对每个请求进行细粒度安全性控制的关键在于重载 configure(HttpSecurity) 方法。如下的代码片段展现了重载的 configure(HttpSecurity) 方法，它为不同的 URL 路径有选择地应用安全性：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http
    .authorizeRequests()
    .antMatchers("/spitters/me").authenticated()
    .antMatchers(HttpMethod.POST, "/spittles").authenticated()
    .anyRequest().permitAll();
}
```

configure() 方法中得到的 HttpSecurity 对象可以在多个方面配置 HTTP 的安全性。在这里，我们首先调用 authorizeRequests()，然后调用该方法所返回的对象的方法来配置请求级别的安全性细节。其中，第一次调用 antMatchers() 指定了对 `/spitters/me` 路径的请求需要进行认证。第二次调用 antMatchers() 更为具体，说明对 `/spittles` 路径的 HTTP POST 请求必须要经过认证。最后对 anyRequests() 的调用中，说明其他所有的请求都是允许的，不需要认证和任何的权限。

antMatchers() 方法中设定的路径支持 Ant 风格的通配符。在这里我们并没有这样使用，但是也可以使用通配符来指定路径，如下所示：

```java
.antMatchers("/spitter/**").authenticated();
```

我们也可以在一个对 antMatchers() 方法的调用中指定多个路径：

```java
.antMatchers("/spitter/**", "/spittles/mine").authenticated();
```

