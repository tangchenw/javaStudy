Spring 通常用来开发 Web 应用。因此，在第 2 部分中，将会看到如何使用 Spring 的 MVC 框架为应用程序添加 Web 前端。

在 **第 5 章 构建 Spring Web 应用** 中，你将会学习到 Spring MVC 的基本用法，它是构建在 Spring 理念之上的一个 Web 框架。我们将会看到如何编写处理 Web 请求的控制器以及如何透明地绑定请求参数和负载到业务对象上，同时它还提供了数据检验和错误处理的功能。

在 **第 6 章 渲染 Web 视图** 中，将会基于第 5 章的内容继续讲解，展现了如何得到 Spring MVC 控制器所生成的模型数据，并将其渲染为用户浏览器中的 HTML。这一章的讨论包括 JavaServer Pages（JSP）、Apache Tiles 和 Thymeleaf 模板。

在 **第 7 章 Spring MVC 的高级技术** 中，将会学习到构建 Web 应用时的一些高级技术，包括自定义 Spring MVC 配置、处理 multipart 文件上传、处理异常以及使用 flash 属性跨请求传递数据。

**第 8 章 使用 Spring Web Flow** 将会为你展示如何使用 Spring Web Flow 来构建会话式、基于流程的 Web 应用程序。

鉴于安全是很多应用程序的重要关注点，因此 **第 9 章 保护 Web 应用** 将会为你介绍如何使用 Spring Security 来为 Web 应用程序提供安全性，保护应用中的信息。

# **构建 Spring Web 应用程序**

本章内容：

- 映射请求到 Spring 控制器
- 透明地绑定表单参数
- 校验表单提交

作为企业级 Java 开发者，你可能开发过一些基于 Web 的应用程序。对于很多 Java 开发人员来说，基于 Web 的应用程序是他们主要的关注点。如果你有这方面经验的话，你会意识到这种系统所面临的挑战。具体来讲，状态管理、工作流以及验证都是需要解决的重要特性。HTTP 协议的无状态性决定了这些问题都不那么容易解决。

Spring 的 Web 框架就是为了帮你解决这些关注点而设计的。Spring MVC 基于模型-视图-控制器（Model-View-Controller，MVC）模式实现，它能够帮你构建像 Spring 框架那样灵活和松耦合的 Web 应用程序。

在本章中，我们将会介绍 Spring MVC Web 框架，并使用新的 Spring MVC 注解来构建处理各种 Web 请求、参数和表单输入的控制器。在深入介绍 Spring MVC 之前，让我们先总体上介绍一下 Spring MVC，并建立起 Spring MVC 运行的基本配置。

## **Spring MVC 起步**

你见到过孩子们的捕鼠器游戏吗？这真是一个疯狂的游戏，它的目标是发送一个小钢球，让它经过一系列稀奇古怪的装置，最后触发捕鼠器。小钢球穿过各种复杂的配件，从一个斜坡上滚下来，被跷跷板弹起，绕过一个微型摩天轮，然后被橡胶靴从桶中踢出去。经过这些后，小钢球会对那只可怜又无辜的橡胶老鼠进行捕获。

乍看上去，你会认为 Spring MVC 框架与捕鼠器有些类似。Spring 将请求在调度 Servlet、处理器映射（handler mapping）、控制器以及视图解析器（view resolver）之间移动，而捕鼠器中的钢球则会在各种斜坡、跷跷板以及摩天轮之间滚动。但是，不要将 Spring MVC 与 Rube Goldberg-esque 捕鼠器游戏做过多比较。每一个 Spring MVC 中的组件都有特定的目的，并且它也没有那么复杂。

让我们看一下请求是如何从客户端发起，经过 Spring MVC 中的组件，最终再返回到客户端的。

### **跟踪 Spring MVC 的请求**

每当用户在 Web 浏览器中点击链接或提交表单的时候，请求就开始工作了。对请求的工作描述就像是快递投送员。与邮局投递员或 FedEx 投送员一样，请求会将信息从一个地方带到另一个地方。

请求是一个十分繁忙的家伙。从离开浏览器开始到获取响应返回，它会经历好多站，在每站都会留下一些信息同时也会带上其他信息。图 5.1 展示了请求使用 Spring MVC 所经历的所有站点。

![5.1 spring mvc 请求](C:\Users\汤琛\Desktop\学习资料\Spring详解\images\5.1 spring mvc 请求.jpg)

在请求离开浏览器时，会带有用户所请求内容的信息，至少会包含请求的 URL。但是还可能带有其他的信息，例如用户提交的表单信息。

请求旅程的第一站是 Spring 的 DispatcherServlet。与大多数基于 Java 的 Web 框架一样，Spring MVC 所有的请求都会通过一个前端控制器（front controller）Servlet。前端控制器是常用的 Web 应用程序模式，在这里一个单实例的 Servlet 将请求委托给应用程序的其他组件来执行实际的处理。在 Spring MVC 中，DispatcherServlet 就是前端控制器。

DispatcherServlet 的任务是将请求发送给 Spring MVC 控制器 （controller）。控制器是一个用于处理请求的 Spring 组件。在典型的应用程序中可能会有多个控制器，DispatcherServlet 需要知道应该将请求发送给哪个控制器。所以 DispatcherServlet 以会查询一个或多个处理器映射（handler mapping） 来确定请求的下一站在哪里。处理器映射会根据请求所携带的 URL 信息来进行决策。

一旦选择了合适的控制器，DispatcherServlet 会将请求发送给选中的控制器。到了控制器，请求会卸下其负载（用户提交的信息）并耐心等待控制器处理这些信息。（实际上，设计良好的控制器本身只处理很少甚至不处理工作，而是将业务逻辑委托给一个或多个服务对象进行处理。）

控制器在完成逻辑处理后，通常会产生一些信息，这些信息需要返回给用户并在浏览器上显示。这些信息被称为模型（model）。不过仅仅给用户返回原始的信息是不够的 —— 这些信息需要以用户友好的方式进行格式化，一般会是 HTML。所以，信息需要发送给一个视图 （view），通常会是 JSP。

控制器所做的最后一件事就是将模型数据打包，并且标示出用于渲染输出的视图名。它接下来会将请求连同模型和视图名发送回 DispatcherServlet 。

这样，控制器就不会与特定的视图相耦合，传递给 DispatcherServlet 的视图名并不直接表示某个特定的 JSP。实际上，它甚至并不能确定视图就是 JSP。相反，它仅仅传递了一个逻辑名称，这个名字将会用来查找产生结果的真正视图。DispatcherServlet 将会使用视图解析器（view resolver） 来将逻辑视图名匹配为一个特定的视图实现，它可能是也可能不是 JSP。

既然 DispatcherServlet 已经知道由哪个视图渲染结果，那请求的任务基本上也就完成了。它的最后一站是视图的实现（可能是 JSP），在这里它交付模型数据。请求的任务就完成了。视图将使用模型数据渲染输出，这个输出会通过响应对象传递给客户端（不会像听上去那样硬编码） 。

可以看到，请求要经过很多的步骤，最终才能形成返回给客户端的响应。大多数的步骤都是在 Spring 框架内部完成的，也就是图 5.1 所示的组件中。尽管本章的主要内容都关注于如何编写控制器，但在此之前我们首先看一下如何搭建 Spring MVC 的基础组件。

### **搭建 Spring MVC**

基于图 5.1，看上去我们需要配置很多的组成部分。幸好，借助于最近几个 Spring 新版本的功能增强，开始使用 Spring MVC 变得非常简单了。现在，我们要使用最简单的方式来配置 Spring MVC：所要实现的功能仅限于运行我们所创建的控制器。在第 7 章中，我们会看一些其他的配置选项。

**配置 DispatcherServlet**

DispatcherServlet 是 Spring MVC 的核心。在这里请求会第一次接触到框架，它要负责将请求路由到其他的组件之中。

按照传统的方式，像 DispatcherServlet 这样的 Servlet 会配置在 web.xml 文件中，这个文件会放到应用的 WAR 包里面。当然，这是配置 DispatcherServlet 的方法之一。但是，借助于 Servlet 3 规范和 Spring 3.1 的功能增强，这种方式已经不是唯一的方案了，这也不是我们本章所使用的配置方法。

我们会使用 Java 将 DispatcherServlet 配置在 Servlet 容器中，而不会再使用 web.xml 文件。如下的程序清单展示了所需的 Java 类。

```java
package spittr.config;

import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

public class SpitterWebInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

  @Override
  protected String[] getServletMappings() {
    return new String[] { "/" };
  }
  
  @Override
  protected Class<?>[] getRootConfigClasses() {
    return new Class<?>[] { RootConfig.class };
  }

  @Override
  protected Class<?>[] getServletConfigClasses() {
    return new Class<?>[] { WebConfig.class };
  }

}
```

在我们深入介绍程序清单 5.1 之前，你可能想知道 spittr 到底是什么意思。这个类的名字是 SpittrWebAppInitializer，它位于名为 spittr.config 的包中。我稍后会对其进行介绍（在 5.1.3 小节中），但现在，你只需要知道我们所要创建的应用名为 Spittr。

要理解程序清单 5.1 是如何工作的，我们可能只需要知道扩展 AbstractAnnotationConfigDispatcherServletInitializer 的任意类都会自动地配置 DispatcherServlet 和 Spring 应用上下文，Spring 的应用上下文会位于应用程序的 Servlet 上下文之中。

**AbstractAnnotationConfigDispatcherServletInitializer 剖析**

如果你坚持要了解更多细节的话，那就看这里吧。在 Servlet 3.0 环境中，容器会在类路径中查找实现 javax.servlet.ServletContainerInitializer 接口的类， 如果能发现的话，就会用它来配置 Servlet 容器。

Spring 提供了这个接口的实现，名为 SpringServletContainerInitializer，这个类反过来又会查找实现 WebApplicationInitializer 的类并将配置的任务交给它们来完成。Spring 3.2 引入了一个便利的 WebApplicationInitializer 基础实现，也就 是 AbstractAnnotationConfigDispatcherServletInitializer 因为我们的 SpittrWebAppInitializer 扩展了 AbstractAnnotationConfigDispatcherServletInitializer（同时也就实现了 WebApplicationInitializer），因此当部署到 Servlet 3.0 容器中的时候，容器会自动发现它，并用它来配置 Servlet 上下文。

尽管它的名字很长，但是 AbstractAnnotationConfigDispatcherServletInitializer 使用起来很简便。在程序清单 5.1 中，SpittrWebAppInitializer 重写了三个方法。

第一个方法是 getServletMappings()，它会将一个或多个路径映射到Dispatcher-Servlet上。在本例中，它映射的是 "/"，这表示它会是应用的默认 Servlet。它会处理进入应用的所有请求。

为了理解其他的两个方法，我们首先要理解 DispatcherServlet 和一个 Servlet 监听器（也就是 ContextLoaderListener）的关系。

**两个应用上下文之间的故事**

当 DispatcherServlet 启动的时候，它会创建 Spring 应用上下文，并加载配置文件或配置类中所声明的 bean。在以上程序中的 getServletConfigClasses() 方法中，我们要求 DispatcherServlet 加载应用上下文时，使用定义在 WebConfig 配置类（使用 Java 配置）中的 bean。

但是在 Spring Web 应用中，通常还会有另外一个应用上下文。另外的这个应用上下文是由 ContextLoaderListener 创建的。

我们希望 DispatcherServlet 加载包含 Web 组件的 bean，如控制器、视图解析器以及处理器映射，而 ContextLoaderListener 要加载应用中的其他 bean。这些 bean 通常是驱动应用后端的中间层和数据层组件。

实际上，AbstractAnnotationConfigDispatcherServletInitializer 会同时创建 DispatcherServlet 和 ContextLoaderListener。GetServletConfigClasses() 方法返回的带有 @Configuration 注解的类将会用来定义 DispatcherServlet 应用上下文中的 bean。getRootConfigClasses() 方法返回的带有 @Configuration 注解的类将会用来配置 ContextLoaderListener 创建的应用上下文中的 bean。

在本例中，根配置定义在 RootConfig 中，DispatcherServlet 的配置声明在 WebConfig 中。稍后我们将会看到这两个类的内容。

需要注意的是，通过 AbstractAnnotationConfigDispatcherServletInitializer 来配置 DispatcherServlet 是传统 web.xml 方式的替代方案。如果你愿意的话，可以同时包含 web.xml 和 AbstractAnnotationConfigDispatcherServletInitializer，但这其实并没有必要。

如果按照这种方式配置 DispatcherServlet，而不是使用 web.xml 的话，那唯一问题在于它只能部署到支持 Servlet 3.0 的服务器中才能正常工作，如 Tomcat 7 或更高版本。Servlet 3.0 规范在 2009 年 12 月份就发布了，因此很有可能你会将应用部署到支持 Servlet 3.0 的 Servlet 容器之中。如果你还没有使用支持 Servlet 3.0 的服务器，那么在 AbstractAnnotationConfigDispatcherServletInitializer 子类中配置 DispatcherServlet 的方法就不适合你了。你别无选择，只能使用 web.xml 了。我们将会在第 7 章学习 web.xml 和其他配置选项。但现在，我们先看一下以上程序中所引用的 WebConfig 和 RootConfig，了解一下如何启用 Spring MVC。

**启用 Spring MVC**

我们有多种方式来配置 DispatcherServlet，与之类似，启用 Spring MVC 组件的方法也不仅一种。以前，Spring 是使用 XML 进行配置的，你可以使用 <mvc:annotation-driven> 启用注解驱动的 Spring MVC。

我们会在第 7 章讨论 Spring MVC 配置可选项的时候，再讨论 <mvc:annotation>。不过，现在我们会让 Spring MVC 的搭建过程尽可能简单并基于 Java 进行配置。

我们所能创建的最简单的 Spring MVC 配置就是一个带有 @EnableWebMvc 注解的类：

```java
package spittr.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

@Configuration
@EnableWebMvc
public class WebConfig {
}
```

这可以运行起来，它的确能够启用 Spring MVC，但还有不少问题要解决：

- 没有配置视图解析器。如果这样的话，Spring 默认会使用 BeanNameViewResolver，这个视图解析器会查找 ID 与视图名称匹配的 bean，并且查找的 bean 要实现 View 接口，它以这样的方式来解析视图。
- 没有启用组件扫描。这样的结果就是，Spring 只能找到显式声明在配置类中的控制器。
- 这样配置的话，DispatcherServlet 会映射为应用的默认 Servlet，所以它会处理所有的请求，包括对静态资源的请求，如图片和样式表（在大多数情况下，这可能并不是你想要的效果）。

因此，我们需要在 WebConfig 这个最小的 Spring MVC 配置上再加一些内容，从而让它变得真正有用。如下程序清单中的 WebConfig 解决了上面所述的问题。

```java
package spittr.web;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

@Configuration
@EnableWebMvc
@ComponentScan("spittr.web")
public class WebConfig extends WebMvcConfigurerAdapter {

  @Bean
  public ViewResolver viewResolver() {
    InternalResourceViewResolver resolver = new InternalResourceViewResolver();
    resolver.setPrefix("/WEB-INF/views/");
    resolver.setSuffix(".jsp");
    return resolver;
  }
  
  @Override
  public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
    configurer.enable();
  }
}
```

在程序清单 5.2 中第一件需要注意的事情是 WebConfig 现在添加了 @ComponentScan 注解，因此将会扫描 spitter.web 包来查找组件。稍后你就会看到，我们所编写的控制器将会带有 @Controller 注解，这会使其成为组件扫描时的候选 bean。因此，我们不需要在配置类中显式声明任何的控制器。

接下来，我们添加了一个 ViewResolver bean。更具体来讲，是 InternalResourceViewResolver。我们将会在第 6 章更为详细地讨论视图解析器。我们只需要知道它会查找 JSP 文件，在查找的时候，它会在视图名称上加一个特定的前缀和后缀（例如，名为 home 的视图将会解析为 /WEB-INF/views/home.jsp）。

最后，新的 WebConfig 类还扩展了 WebMvcConfigurerAdapter 并重写了其 configureDefaultServletHandling() 方法。通过调用 DefaultServletHandlerConfigurer 的 enable() 方法，我们要求 DispatcherServlet 将对静态资源的请求转发到 Servlet 容器中默认的 Servlet 上，而不是使用 DispatcherServlet 本身来处理此类请求。

WebConfig 已经就绪，那 RootConfig 呢？因为本章聚焦于 Web 开发，而 Web 相关的配置通过 DispatcherServlet 创建的应用上下文都已经配置好了，因此现在的 RootConfig 相对很简单：

```java
package spittr.config;

import java.util.regex.Pattern;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ComponentScan.Filter;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import org.springframework.web.servlet.config.annotation.EnableMvc;

@Configuration
@ComponentScan(basePackages={"spittr"}, 
    excludeFilters={
        @Filter(type=FilterType.ANNOTATION, value=EnableWebMvc.class)
    })
public class RootConfig {
}
```

唯一需要注意的是 RootConfig 使用了 @ComponentScan 注解。这样的话，在本书中，我们就有很多机会用非 Web 的组件来充实完善 RootConfig。

现在，我们基本上已经可以开始使用 Spring MVC 构建 Web 应用了。此时，最大的问题在于，我们要构建的应用到底是什么。

## **Spittr 应用简介**

为了实现在线社交的功能，我们将要构建一个简单的微博（microblogging）应用。在很多方面，我们所构建的应用与最早的微博应用 Twitter 很类似。在这个过程中，我们会添加一些小的变化。当然，我们要使用 Spring 技术来构建这个应用。

因为从 Twitter 借鉴了灵感并且通过 Spring 来进行实现，所以它就有了一个名字：Spitter。再进一步，应用网站命名中流行的模式，如 Flickr，我们去掉字母 e，这样的话，我们就将这个应用称为 Spittr。这个名称也有助于区分应用名称和领域类型，因为我们将会创建一个名为 Spitter 的领域类。

Spittr 应用有两个基本的领域概念：Spitter（应用的用户）和 Spittle（用户发布的简短状态更新）。当我们在书中完善 Spittr 应用的功能时，将会介绍这两个领域概念。在本章中，我们会构建应用的 Web 层，创建展现 Spittle 的控制器以及处理用户注册成为 Spitter 的表单。

舞台已经搭建完成了。我们已经配置了 DispatcherServlet，启用了基本的 Spring MVC 组件并确定了目标应用。让我们进入本章的核心内容：使用 Spring MVC 控制器处理 Web 请求。

## **编写基本的控制器**

在 Spring MVC 中，控制器只是方法上添加了 @RequestMapping 注解的类，这个注解声明了它们所要处理的请求。

开始的时候，我们尽可能简单，假设控制器类要处理对 `/` 的请求， 并渲染应用的首页。程序清单 5.3 所示的 HomeController 可能是最简单的 Spring MVC 控制器类了。 

```java
package spittr.web;

import static org.springframework.web.bind.annotation.RequestMethod.*;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class HomeController {

  @RequestMapping(value="/", method = GET)
  public String home(Model model) {
    return "home";
  }

}
```

你可能注意到的第一件事情就是 HomeController 带有 @Controller 注解。很显然这个注解是用来声明控制器的，但实际上这个注解对 Spring MVC 本身的影响并不大。

HomeController 是一个构造型（stereotype）的注解，它基于 @Component 注解。在这里，它的目的就是辅助实现组件扫描。因为 HomeController 带有 @Controller 注解，因此组件扫描器会自动找到 HomeController，并将其声明为 Spring 应用上下文中的一个 bean。

其实，你也可以让 HomeController 带有 @Component 注解，它所实现的效果是一样的，但是在表意性上可能会差一些，无法确定 HomeController 是什么组件类型。

HomeController 唯一的一个方法，也就是 home() 方法，带有 @Request-Mapping 注解。它的 value 属性指定了这个方法所要处理的请求路径，method 属性细化了它所处理的 HTTP 方法。在本例中，当收到对 `/` 的 HTTP GET 请求时，就会调用 home() 方法。

你可以看到，home() 方法其实并没有做太多的事情：它返回了一个 String 类型的 `home` 。这个 String 将会被 Spring MVC 解读为要渲染的视图名称。DispatcherServlet 会要求视图解析器将这个逻辑名称解析为实际的视图。

鉴于我们配置 InternalResourceViewResolver 的方式，视图名 `home` 将会解析为 `/WEB-INF/views/home.jsp` 路径的 JSP。现在，我们会让 Spittr 应用的首页相当简单，如下所示。

```jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ page session="false" %>
<html>
  <head>
    <title>Spitter</title>
    <link rel="stylesheet" 
          type="text/css" 
          href="<c:url value="/resources/style.css" />" >
  </head>
  <body>
    <h1>Welcome to Spitter</h1>

    <a href="<c:url value="/spittles" />">Spittles</a> | 
    <a href="<c:url value="/spitter/register" />">Register</a>
  </body>
</html>
```

这个 JSP 并没有太多需要注意的地方。它只是欢迎应用的用户，并提供了两个链接：一个是查看 Spittle 列表，另一个是在应用中进行注册。图 5.2 展现了此时的首页是什么样子的。

在本章完成之前，我们将会实现处理这些请求的控制器方法。但现在，让我们对这个控制器发起一些请求，看一下它是否能够正常工作。测试控制器最直接的办法可能就是构建并部署应用，然后通过浏览器对其进行访问，但是自动化测试可能会给你更快的反馈和更一致的独立结果。所以，让我们编写一个针对 HomeController 的测试。

![5.2 Spittr 首页](C:\Users\汤琛\Desktop\学习资料\Spring详解\images\5.2 Spittr 首页.jpg)

### **测试控制器**

让我们再审视一下 HomeController。如果你眼神不太好的话，你甚至可能注意不到这些注解，所看到的仅仅是一个简单的 POJO。我们都知道测试 POJO 是很容易的。因此，我们可以编写一个简单的类来测试 HomeController，如下所示：

```java
package spittr.web;

import static org.junit.Assert.assertEquals;
import org.junit.Test;
import spittr.web.HomeController;

public class HomeControllerTest {

  @Test
  public void testHomePage() throws Exception {
    HomeController controller = new HomeController();
    assertEuqals("home", controller.home());
  }

}
```

程序清单 5.5 中的测试很简单，但它只测试了 home() 方法中会发生什 么。在测试中会直接调用 home() 方法，并断言返回包含 `home` 值的 String。它完全没有站在 Spring MVC 控制器的视角进行测试。这个测试没有断言当接收到针对 `/` 的 GET 请求时会调用 home() 方法。因为它返回的值就是 `home`，所以也没有真正判断 home 是视图的名称。

不过从 Spring 3.2 开始，我们可以按照控制器的方式来测试 Spring MVC 中的控制器了，而不仅仅是作为 POJO 进行测试。Spring 现在包含了一种 mock Spring MVC 并针对控制器执行 HTTP 请求的机制。这样的话，在测试控制器的时候，就没有必要再启动 Web 服务器和 Web 浏览器 了。

为了阐述如何测试 Spring MVC 的控制器，我们重写 HomeControllerTest 并使用 Spring MVC 中新的测试特性。程序清单 5.6 展现了新的HomeController-Test。

```java
package spittr.web;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.springframework.test.web.servlet.setup.MockMvcBuilders.*;

import org.junit.Test;
import org.springframework.test.web.servlet.MockMvc;

import spittr.web.HomeController;

public class HomeControllerTest {

  @Test
  public void testHomePage() throws Exception {
    HomeController controller = new HomeController();
    MockMvc mockMvc = standaloneSetup(controller).build();
    mockMvc.perform(get("/"))
           .andExpect(view().name("home"));
  }

}
```

尽管新版本的测试只比之前版本多了几行代码，但是它更加完整地测试了 HomeController。这次我们不是直接调用 home() 方法并测试它的返回值，而是发起了对 `/` 的 GET 请求，并断言结果视图的名称为 home。它首先传递一个 HomeController 实例到 MockMvcBuilders.standaloneSetup() 并调用 build() 来构建 MockMvc 实例。然后它使用 MockMvc 实例来执行针对 `/` 的 GET 请求并设置期望得到的视图名称。

### **定义类级别的请求处理**

现在，已经为 HomeController 编写了测试，那么我们可以做一些重构，并通过测试来保证不会对功能造成什么破坏。我们可以做的一件事就是拆分 @Request-Mapping，并将其路径映射部分放到类级别上。程序清单 5.7 展示了这个过程。

```java
package spittr.web;

import static org.springframework.web.bind.annotation.RequestMethod.*;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/")
public class HomeController {

  @RequestMapping(method = GET)
  public String home(Model model) {
    return "home";
  }

}
```

在这个新版本的 HomeController 中，路径现在被转移到类级别的  @Request-Mapping 上，而 HTTP 方法依然映射在方法级别上。当控制器在类级别上添加@RequestMapping 注解时，这个注解会应用到控制器的所有处理器方法上。处理器方法上的 @RequestMapping 注解会对类级别上的 @RequestMapping 的声明进行补充。

就 HomeController 而言，这里只有一个控制器方法。与类级别的 @Request-Mapping 合并之后，这个方法的 @RequestMapping 表明 home() 将会处理对 `/` 路径的 GET 请求。

换言之，我们其实没有改变任何功能，只是将一些代码换了个地方，但是 HomeController 所做的事情和以前是一样的。因为我们现在有了测试，所以可以确保在这个过程中，没有对原有的功能造成破坏。

当我们在修改 @RequestMapping 时，还可以对 HomeController 做另外一个变更。@RequestMapping 的 value 属性能够接受一个 String 类型的数组。到目前为止，我们给它设置的都是一个 String 类型的 `/`。但是，我们还可以将它映射到对 `/homepage` 的请求，只需将类级别的 @RequestMapping 改为如下所示：

```java
@Controller
@RequesiMapping({"/", "/homepage"})
public class HomeController {
}
```

现在，HomeController 的 home() 方法能够映射到对 `/` 和 `/homepage` 的 GET 请求。

### **传递模型数据到视图中**

到现在为止，就编写超级简单的控制器来说，HomeController 已经是一个不错的样例了。但是大多数的控制器并不是这么简单。在 Spittr 应用中，我们需要有一个页面展现最近提交的 Spittle 列表。因此，我们需要一个新的方法来处理这个页面。

首先，需要定义一个数据访问的 Repository。为了实现解耦以及避免陷入数据库访问的细节之中，我们将 Repository 定义为一个接口，并在稍后实现它（第 10 章中）。此时，我们只需要一个能够获取 Spittle 列表的 Repository，如下所示的 SpittleRepository 功能已经足够了：

```java
package spittr.data;

import java.util.List;
import spittr.Spittle;

public interface SpittleRepository {

  List<Spittle> findSpittles(long max, int count);

}
```

现在，我们让 Spittle 类尽可能的简单，如下面的程序清单 5.8 所示。它的属性包括消息内容、时间戳以及 Spittle 发布时对应的经纬度。

```java
package spittr;

import java.util.Date;

import org.apache.commons.lang3.builder.EqualsBuilder;
import org.apache.commons.lang3.builder.HashCodeBuilder;

public class Spittle {

  private final Long id;
  private final String message;
  private final Date time;
  private Double latitude;
  private Double longitude;

  public Spittle(String message, Date time) {
    this(null, message, time, null, null);
  }
  
  public Spittle(Long id, String message, Date time, Double longitude, Double latitude) {
    this.id = id;
    this.message = message;
    this.time = time;
    this.longitude = longitude;
    this.latitude = latitude;
  }

  public long getId() {
    return id;
  }

  public String getMessage() {
    return message;
  }

  public Date getTime() {
    return time;
  }
  
  public Double getLongitude() {
    return longitude;
  }
  
  public Double getLatitude() {
    return latitude;
  }
  
  @Override
  public boolean equals(Object that) {
    return EqualsBuilder.reflectionEquals(this, that, "id", "time");
  }
  
  @Override
  public int hashCode() {
    return HashCodeBuilder.reflectionHashCode(this, "id", "time");
  }
  
}
```

就大部分内容来看，Spittle 就是一个基本的 POJO 数据对象 —— 没有什么复杂的。唯一要注意的是，我们使用 Apache Common Lang 包来实现 equals() 和 hashCode() 方法。这些方法除了常规的作用以外，当我们为控制器的处理器方法编写测试时，它们也是有用的。

既然我们说到了测试，那么我们继续讨论这个话题并为新的控制器方法编写测试。如下的程序清单使用 Spring 的 MockMvc 来断言新的处理器方法中你所期望的行为。

```java
@Test
public void shouldShowPagedSpittles() throws Exception {
  List<Spittle> expectedSpittles = createSpittleList(50);
  SpittleRepository mockRepository = mock(SpittleRepository.class);
  when(mockRepository.findSpittles(238900, 50))
      .thenReturn(expectedSpittles);
    
  SpittleController controller = new SpittleController(mockRepository);
  MockMvc mockMvc = standaloneSetup(controller)
      .setSingleView(new InternalResourceView("/WEB-INF/views/spittles.jsp"))
      .build();

  mockMvc.perform(get("/spittles?max=238900&count=50"))
    .andExpect(view().name("spittles"))
    .andExpect(model().attributeExists("spittleList"))
    .andExpect(model().attribute("spittleList", hasItems(expectedSpittles.toArray())));
}

...
  
private List<Spittle> createSpittleList(int count) {
  List<Spittle> spittles = new ArrayList<Spittle>();
  for (int i=0; i < count; i++) {
    spittles.add(new Spittle("Spittle " + i, new Date()));
  }
  return spittles;
}
```

这个测试首先会创建 SpittleRepository 接口的 mock 实现，这个实现会从它的 findSpittles() 方法中返回 20 个 Spittle 对象。然后，它将这个 Repository 注入到一个新的 SpittleController 实例中，然后创建 MockMvc 并使用这个控制器。

需要注意的是，与 HomeController 不同，这个测试在 MockMvc 构造器上调用了 setSingleView()。这样的话，mock 框架就不用解析控制器中的视图名了。在很多场景中，其实没有必要这样做。但是对于这个控制器方法，视图名与请求路径是非常相似的，这样按照默认的视图解析规则时，MockMvc 就会发生失败，因为无法区分视图路径和控制器的路径。在这个测试中，构建 Internal-ResourceView 时所设置的实际路径是无关紧要的，但我们将其设置为与InternalResourceViewResolver 配置一致。

这个测试对 `/spittles` 发起 GET 请求，然后断言视图的名称为 spittles 并且模型中包含名为 spittleList 的属性，在 spittleList 中包含预期的内容。 当然，如果此时运行测试的话，它将会失败。它不是运行失败，而是在编译的时候就会失败。这是因为我们还没有编写 SpittleController。现在，我们创建Spittle-Controller，让它满足程序清单 5.9 的预期。如下的 SpittleController 实现将会满足以上测试的要求。

```java
package spittr.web;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import spittr.Spittle;
import spittr.data.SpittleRepository;

@Controller
@RequestMapping("/spittles")
public class SpittleController {
  
  private SpittleRepository spittleRepository;

  @Autowired
  public SpittleController(SpittleRepository spittleRepository) {
    this.spittleRepository = spittleRepository;
  }

  @RequestMapping(method=RequestMethod.GET)
  public String spittles(Model model) {
    model.addAttribute(
      spittleRepository.findSpittles(Long.MAX_VALUE, 20)
    );
    return "spittles"
  }
  
}
```

我们可以看到 SpittleController 有一个构造器，这个构造器使用了 @Autowired 注解，用来注入 SpittleRepository。这个 SpittleRepository 随后又用在 spittles() 方法中，用来获取最新的 spittle 列表。

需要注意的是，我们在 spittles() 方法中给定了一个 Model 作为参 数。这样，spittles() 方法就能将 Repository 中获取到的 Spittle 列表填充到模型中。Model 实际上就是一个 Map（也就是 key-value 对的集合），它会传递给视图，这样数据就能渲染到客户端了。当调用 addAttribute() 方法并且不指定 key 的时候，那么 key 会根据值的对象类型推断确定。在本例中，因为它是一个 List<Spittle>，因此，键将会推断为 spittleList。

spittles() 方法所做的最后一件事是返回 spittles 作为视图的名字，这个视图会渲染模型。

如果你希望显式声明模型的 key 的话，那也尽可以进行指定。例如，下面这个版本的 spittles() 方法与程序清单5.10中的方法作用是一样的：

```java
@RequestMapping(method=RequestMethod.GET)
public String spittles(Model model) {
  model.addAttribute("splittleList",
    spittleRepository.findSpittles(Long.MAX_VALUE, 20)
  );
  return "spittles"
}
```

既然我们现在提到了各种可替代的方案，那下面还有另外一种方式来编写spittles() 方法：

```java
@RequestMapping(method=RequestMethod.GET)
public List<Spittle> spittles() {
  return spittleRepository.findSpittles(Long.MAX_VALUE, 20);
}
```

这个版本与其他的版本有些差别。它并没有返回视图名称，也没有显式地设定模型，这个方法返回的是 Spittle 列表。当处理器方法像这样返回对象或集合时，这个值会放到模型中，模型的 key 会根据其类型推断得出（在本例中，也就是 spittleList）。

而逻辑视图的名称将会根据请求路径推断得出。因为这个方法处理针对 `/spittles` 的 GET 请求，因此视图的名称将会是 spittles（去掉开头的斜线）。

不管你选择哪种方式来编写 spittles() 方法，所达成的结果都是相同的。模型中会存储一个 Spittle 列表，key 为 spittleList，然后这个列表会发送到名为 spittles 的视图中。按照我们配置 InternalResourceViewResolver 的方式，视图的 JSP 将会是 `/WEB-INF/views/spittles.jsp`。

现在，数据已经放到了模型中，在 JSP 中该如何访问它呢？实际上，当视图是 JSP 的时候，模型数据会作为请求属性放到请求（request）之中。因此，在 spittles.jsp 文件中可以使用 JSTL（JavaServer Pages Standard Tag Library）的 `<c:forEach>` 标签渲染 spittle 列表：

```jsp
<c:forEach items="${spittleList}" var="spittle" >
  <li id="spittle_<c:out value="spittle.id"/>">
    <div class="spittleMessage">
      <c:out value="${spittle.message}" />
    </div>
    <div>
      <span class="spittleTime">
        <c:out value="${spittle.time}" />
      </span>
      <span class="spittleLocation">(
        <c:out value="${spittle.latitude}" />, 
        <c:out value="${spittle.longitude}" />)
      </span>
    </div>
  </li>
</c:forEach>
```

图 5.3 为显示效果，能够让你对它在 Web 浏览器中是什么样子有个可视化的印象。

尽管 SpittleController 很简单，但是它依然比 HomeController 更进一步了。不过，SpittleController 和 HomeController 都没有处理任何形式的输入。现在，让我们扩展 SpittleController，让它从客户端接受一些输入。

![5.3 spittle jsp](C:\Users\汤琛\Desktop\学习资料\Spring详解\images\5.3 spittle jsp.jpg)

## **接受请求的输入**

有些 Web 应用是只读的。人们只能通过浏览器在站点上闲逛，阅读服务器发送到浏览器中的内容。

不过，这并不是一成不变的。众多的 Web 应用允许用户参与进去，将数据发送回服务器。如果没有这项能力的话，那 Web 将完全是另一番景象。

Spring MVC 允许以多种方式将客户端中的数据传送到控制器的处理器方法中，包括：

- 查询参数（Query Parameter）。
- 表单参数（Form Parameter）。
- 路径变量（Path Variable）。

你将会看到如何编写控制器处理这些不同机制的输入。作为开始，我们先看一下如何处理带有查询参数的请求，这也是客户端往服务器端发送数据时，最简单和最直接的方式。

### **处理查询参数**

在 Spittr 应用中，我们可能需要处理的一件事就是展现分页的 Spittle 列表。在现在的 SpittleController 中，它只能展现最新的 Spittle，并没有办法向前翻页查看以前编写的 Spittle 历史记录。如果你想让用户每次都能查看某一页的 Spittle 历史，那么就需要提供一种方式让用户传递参数进来，进而确定要展现哪些 Spittle 集合。

在确定该如何实现时，假设我们要查看某一页 Spittle 列表，这个列表会按照最新的 Spittle 在前的方式进行排序。因此，下一页中第一条的 ID 肯定会早于当前页最后一条的 ID。所以，为了显示下一页的 Spittle，我们需要将一个 Spittle 的 ID 传入进来，这个 ID 要恰好小于当 前页最后一条 Spittle 的 ID。另外，你还可以传入一个参数来确定要展现的 Spittle 数量。

为了实现这个分页的功能，我们所编写的处理器方法要接受如下的参 数：

- before 参数（表明结果中所有 Spittle 的 ID 均应该在这个值之前）。
- count 参数（表明在结果中要包含的 Spittle 数量）。

为了实现这个功能，我们将程序清单 5.10 中的 spittles() 方法替换为使用 before 和 count 参数的新 spittles() 方法。我们首先添加一个测试，这个测试反映了新 spittles() 方法的功能。

```java
@Test
public void shouldShowRecentSpittles() throws Exception {
  List<Spittle> expectedSpittles = createSpittleList(50);
  SpittleRepository mockRepository = mock(SpittleRepository.class);
  when(mockRepository.findSpittles(238900, 50))
    .thenReturn(expectedSpittles);

  SpittleController controller = new SpittleController(mockRepository);
  MockMvc mockMvc = standaloneSetup(controller)
      .setSingleView(new InternalResourceView("/WEB-INF/views/spittles.jsp"))
      .build();

  mockMvc.perform(get("/spittles?max=238900&count=50"))
    .andExpect(view().name("spittles"))
    .andExpect(model().attributeExists("spittleList"))
    .andExpect(model().attribute("spittleList", hasItems(expectedSpittles.toArray())));
}
```

这个测试方法与程序清单 5.9 中的测试方法关键区别在于它针对 `/spittles` 发送 GET 请求，同时还传入了 max 和 count 参数。它测试了这些参数存在时的处理器方法，而另一个测试方法则测试了没有这些参数时的情景。这两个测试就绪后，我们就能确保不管控制器发生什么样的变化，它都能够处理这两种类型的请求：

```java
@RequestMapping(method=RequestMethod.GET)
public List<Spittle> spittles(
  @RequestParam("max") long max,
  @RequestParam("count") int count) {
    return spittleRepository.findSpittles(max, count);
}
```

SpittleController 中的处理器方法要同时处理有参数和没有参数的场景，那我们需要对其进行修改，让它能接受参数，同时，如果这些参数在请求中不存在的话，就使用默认值 Long.MAX_VALUE 和 20。@RequestParam 注解的default-Value 属性可以完成这项任务：

```java
@RequestMapping(method=RequestMethod.GET)
public List<Spittle> spittles(
  @RequestParam(value="max", defaultValue=MAX_LONG_AS_STRING) long max,
  @RequestParam(value="count", defaultValue="20") int count) {
    return spittleRepository.findSpittles(max, count);
}
```

现在，如果 max 参数没有指定的话，它将会是 Long 类型的最大值。因为查询参数都是 String 类型的，因此 defaultValue 属性需要 String 类型的值。因此，使用 Long.MAX_VALUE 是不行的。我们可以将 Long.MAX_VALUE 转换为名为 MAX_LONG_AS_STRING 的 String 类型常量：

```java
private static final String MAX_LONG_AS_STRING = Long.toString(Long.MAX_VALUE);
```

尽管 defaultValue 属性给定的是 String 类型的值，但是当绑定到方法的 max 参数时，它会转换为 Long 类型。

如果请求中没有 count 参数的话，count 参数的默认值将会设置为 20。

请求中的查询参数是往控制器中传递信息的常用手段。另外一种方式也很流行，尤其是在构建面向资源的控制器时，这种方式就是将传递参数作为请求路径的一部分。让我们看一下如何将路径变量作为请求路径的一部分，从而实现信息的输入。

### **通过路径参数接受输入**

假设我们的应用程序需要根据给定的 ID 来展现某一个 Spittle 记录。其中一种方案就是编写处理器方法，通过使用 @RequestParam 注解，让它接受 ID 作为查询参数：

```java
@RequestMapping(value="/show", method=RequestMethod.GET)
public String showSpittles(
    @RequestParam("spittle_id") long spittleId, 
    Model model) {
  model.addAttribute(spittleRepository.findOne(spittleId));
  return "spittle";
}
```

这个处理器方法将会处理形如 `/spittles/show?spittle_id=12345` 这样的请求。尽管这也可以正常工作，但是从面向资源的角度来看这并不理想。在理想情况下，要识别的资源（Spittle）应该通过 URL 路径进行标示，而不是通过查询参数。对 `/spittles/12345` 发起 GET 请求要优于对 `/spittles/show?spittle_id=12345` 发起请求。前者能够识别出要查询的资源，而后者描述的是带有参数的一个操作 —— 本质上是通过 HTTP 发起的 RPC。

既然已经以面向资源的控制器作为目标，那我们将这个需求转换为一 测试。程序清单 5.12 展现了一个新的测试方法，它会断言 SpittleController中 对面向资源请求的处理。

```java
@Test
public void testSpittle() throws Exception {
  Spittle expectedSpittle = new Spittle("Hello", new Date());
  SpittleRepository mockRepository = mock(SpittleRepository.class);
  when(mockRepository.findOne(12345)).thenReturn(expectedSpittle);
    
  SpittleController controller = new SpittleController(mockRepository);
  MockMvc mockMvc = standaloneSetup(controller).build();

  mockMvc.perform(get("/spittles/12345"))
    .andExpect(view().name("spittle"))
    .andExpect(model().attributeExists("spittle"))
    .andExpect(model().attribute("spittle", expectedSpittle));
}
```

到目前为止，在我们编写的控制器中，所有的方法都映射到了（通过 @RequestMapping）静态定义好的路径上。但是，如果想让这个测试通过的话，我们编写的 @RequestMapping 要包含变量部分，这部分代表了 Spittle ID。

为了实现这种路径变量，Spring MVC 允许我们在 @RequestMapping 路径中添加占位符。占位符的名称要用大括号（`{` 和 `}`）括起来。路径中的其他部分要与所处理的请求完全匹配，但是占位符部分可以是任意的值。

下面的处理器方法使用了占位符，将 Spittle ID 作为路径的一部 分：

```java
@RequestMapping(value="/{spittleId}", method=RequestMethod.GET)
public String spittle(
    @PathVariable("spittleId") long spittleId, 
    Model model) {
  model.addAttribute(spittleRepository.findOne(spittleId));
  return "spittle";
}
```

例如，它就能够处理针对 `/spittles/12345` 的请求，也就是程序清单 5.12 中的路径我们可以看到，spittle() 方法的 spittleId 参数上添加了 @Path-Variable("spittleId") 注解，这表明在请求路径中，不管占位符部分的值是什么都会传递到处理器方法的 spittleId 参数中。如果对 `/spittles/54321` 发送 GET 请求，那么将会把 `54321` 传递进来，作为 spittleId 的值。

需要注意的是：在样例中 spittleId 这个词出现了好几次：先是在@Request-Mapping 的路径中，然后作为 @PathVariable 属性的值，最后又作为方法的参数名称。因为方法的参数名碰巧与占位符的名称相同，因此我们可以去掉 @PathVariable 中的 value 属性：

```java
@RequestMapping(value="/{spittleId}", method=RequestMethod.GET)
public String spittle(@PathVariable long spittleId, Model model) {
  model.addAttribute(spittleRepository.findOne(spittleId));
  return "spittle";
}
```

如果 @PathVariable 中没有 value 属性的话，它会假设占位符的名称与方法的参数名相同。这能够让代码稍微简洁一些，因为不必重复写占位符的名称了。但需要注意的是，如果你想要重命名参数时，必须要同时修改占位符的名称，使其互相匹配。

spittle() 方法会将参数传递到 SpittleRepository 的 findOne() 方法中，用来获取某个 Spittle 对象，然后将 Spittle 对象添加到模型中。模型的 key 将会是 spittle，这是根据传递到 addAttribute() 方法中的类型推断得到的。

这样 Spittle 对象中的数据就可以渲染到视图中了，此时需要引用请求中 key 为 spittle 的属性（与模型的 key 一致）。如下为渲染 Spittle 的 JSP 视图片段：

```jsp
<div class="spittleViwe">
  <div class="spittleMessage">
    <c:out value="${spittle.message}" />
  </div>
  <div>
    <span class="spittleTime">
      <c:out value="${spittle.time}" />
    </span>
  </div>
</div>
```

如果传递请求中少量的数据，那查询参数和路径变量是很合适的。但通常我们还需要传递很多的数据（也许是表单提交的数据），那查询参数显得有些笨拙和受限了。下面让我们来看一下如何编写控制器方法来处理表单提交。

## **处理表单**

Web 应用的功能通常并不局限于为用户推送内容。大多数的应用允许用户填充表单并将数据提交回应用中，通过这种方式实现与用户的交互。像提供内容一样，Spring MVC 的控制器也为表单处理提供了良好的支持。

使用表单分为两个方面：展现表单以及处理用户通过表单提交的数据。在 Spittr 应用中，我们需要有个表单让新用户进行注册。SpitterController 是一个新的控制器，目前只有一个请求处理的方法来展现注册表单。

```java
package spittr.web;

import static org.springframework.web.bind.annotation.RequestMethod.*;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.Errors;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;

import spittr.Spitter;
import spittr.data.SpitterRepository;

@Controller
@RequestMapping("/spitter")
public class SpitterController {
  
  @RequestMapping(value="/register", method=GET)
  public String showRegistrationForm() {
    return "registerForm";
  }

}
```

showRegistrationForm() 方法的 @RequestMapping 注解以及类级别上的 @RequestMapping 注解组合起来，声明了这个方法要处理的是针对 `/spitter/register` 的 GET 请求。这是一个简单的方法，没有任何输入并且只是返回名为 registerForm 的逻辑视图。按照我们配置InternalResource-ViewResolver的方式，这意味着将会使用 `/WEB-INF/views/registerForm.jsp` 这个 JSP 来渲染注册表单。

```java
@Test
public void shouldShowRegistration() throws Exception {
  SpitterController controller = new SpitterController();
  MockMvc mockMvc = standaloneSetup(controller).build();
  
  mockMvc.perform(get("/spitter/register")).andExpect(view().name("registerForm"));
}
```

这个测试方法与首页控制器的测试非常类似。它对 `/spitter/register` 发送 GET 请求，然后断言结果的视图名为 registerForm。

现在，让我们回到视图上。因为视图的名称为 registerForm，所以 JSP 的名称需要是 registerForm.jsp。这个 JSP 必须要包含一个 HTML `<form>` 标签，在这个标签中用户输入注册应用的信息。如下就是我们现在所要使用的 JSP。

```jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ page session="false" %>
<html>
  <head>
    <title>Spitter</title>
    <link rel="stylesheet" type="text/css" 
          href="<c:url value="/resources/style.css" />" >
  </head>
  <body>
    <h1>Register</h1>

    <form method="POST">
      First Name: <input type="text" name="firstName" /><br/>
      Last Name: <input type="text" name="lastName" /><br/>
      Email: <input type="email" name="email" /><br/>
      Username: <input type="text" name="username" /><br/>
      Password: <input type="password" name="password" /><br/>
      <input type="submit" value="Register" />
    </form>
  </body>
</html>
```

可以看到，这个 JSP 非常基础。它的 HTML 表单域中记录用户的名字、姓氏、用户名以及密码，然后还包含一个提交表单的按钮。在浏览器渲染之后，它的样子大致如图 5.5 所示。

需要注意的是：这里的标签中并没有设置 action 属性。在这种情况下，当表单提交时，它会提交到与展现时相同的 URL 路径上。也就是说，它会提交到 `/spitter/register` 上。

这就意味着需要在服务器端处理该 HTTP POST 请求。现在，我们在 SpitterController 中再添加一个方法来处理这个表单提交。

### **编写处理表单的控制器**

当处理注册表单的 POST 请求时，控制器需要接受表单数据并将表单数据保存为 Spitter 对象。最后，为了防止重复提交（用户点击浏览器的刷新按钮有可能会发生这种情况），应该将浏览器重定向到新创建用户的基本信息页面。这些行为通过下面的 shouldProcessRegistration() 进行了测试。

```java
@Test
public void shouldProcessRegistration() throws Exception {
  SpitterRepository mockRepository = mock(SpitterRepository.class);
  Spitter unsaved = new Spitter("jbauer", "24hours", "Jack", "Bauer", "jbauer@ctu.gov");
  Spitter saved = new Spitter(24L, "jbauer", "24hours", "Jack", "Bauer", "jbauer@ctu.gov");
  when(mockRepository.save(unsaved)).thenReturn(saved);
    
  SpitterController controller = new SpitterController(mockRepository);
  MockMvc mockMvc = standaloneSetup(controller).build();

  mockMvc.perform(post("/spitter/register")
         .param("firstName", "Jack")
         .param("lastName", "Bauer")
         .param("username", "jbauer")
         .param("password", "24hours")
         .param("email", "jbauer@ctu.gov"))
         .andExpect(redirectedUrl("/spitter/jbauer"));
    
  verify(mockRepository, atLeastOnce()).save(unsaved);
}
```

显然，这个测试比展现注册表单的测试复杂得多。在构建完 SpitterRepository 的 mock 实现以及所要执行的控制器和 MockMvc 之后，shouldProcess-Registration() 对 `/spitter/register` 发起了一个 POST 请求。作为请求的一部分，用户信息以参数的形式放到 request 中，从而模拟提交的表单。

在处理 POST 类型的请求时，在请求处理完成后，最好进行一下重定向，这样浏览器的刷新就不会重复提交表单了。在这个测试中，预期请求会重定向到`/spitter/jbauer`，也就是新建用户的基本信息页面。

最后，测试会校验 SpitterRepository 的 mock 实现最终会真正用来保存表单上传入的数据。

现在，我们来实现处理表单提交的控制器方法。通过 shouldProcess-Registration() 方法，我们可能认为要满足这个需求需要做很多的工作。但是，在如下的程序清单中，我们可以看到新的 SpitterController 并没有做太多的事情。

```java
package spittr.web;

import static org.springframework.web.bind.annotation.RequestMethod.*;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.Errors;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;

import spittr.Spitter;
import spittr.data.SpitterRepository;

@Controller
@RequestMapping("/spitter")
public class SpitterController {

  private SpitterRepository spitterRepository;

  @Autowired
  public SpitterController(SpitterRepository spitterRepository) {
    this.spitterRepository = spitterRepository;
  }
  
  @RequestMapping(value="/register", method=GET)
  public String showRegistrationForm() {
    return "registerForm";
  }
  
  @RequestMapping(value="/register", method=POST)
  public String processRegistration(Spitter spitter) {    
    spitterRepository.save(spitter);
    return "redirect:/spitter/" + spitter.getUsername();
  }
  
}
```

我们之前创建的 showRegistrationForm() 方法依然还在，不过请注意新创建的 processRegistration() 方法，它接受一个 Spitter 对象作为参数。这个对象有firstName、lastName、username 和 password 属性，这些属性将会使用请求中同名的参数进行填充。

当使用 Spitter 对象调用 processRegistration() 方法时，它会进而调用 Spitter-Repository 的 save() 方法，SpitterRepository 是在 SpitterController 的构造器中注入进来的。

processRegistration() 方法做的最后一件事就是返回一个 String 类型，用来指定视图。但是这个视图格式和以前我们所看到的视图有所不同。这里不仅返回了视图的名称供视图解析器查找目标视图，而且返回的值还带有重定向的格式。

当 InternalResourceViewResolver 看到视图格式中的 `redirect:` 前缀时，它就知道要将其解析为重定向的规则，而不是视图的名称。在本例中，它将会重定向到用户基本信息的页面。例如，如果 Spitter.username 属性的值为`jbauer`，那么视图将会重定向到 `/spitter/jbauer`。

需要注意的是，除 了 `redirect:`，InternalResourceViewResolver 还能识别`forward:` 前缀。当它发现视图格式中以 `forward:` 作为前缀时，请求将会前往（forward）指定的 URL 路径，而不再是重定向。

万事俱备！现在，程序清单 5.16 中的测试应该能够通过了。但是，我们的任务还没有完成，因为我们重定向到了用户基本信息页面，那么我们应该往 Spitter-Controller 中添加一个处理器方法，用来处理对基本信息页面的请求。如下的 showSpitterProfile() 将会完成这项任务：

```java
@RequestMapping(value="/{username}", method=GET)
public String showSpitterProfile(@PathVariable String username, Model model) {
  Spitter spitter = spitterRepository.findByUsername(username);
  model.addAttribute(spitter);
  return "profile";
}
```

SpitterRepository 通过用户名获取一个 Spitter 对象，showSpitterProfile() 得到这个对象并将其添加到模型中，然后返回 profile，也就是基本信息页面的逻辑视图名。像本章展现的其他视图一样，现在的基本信息视图非常简单：

```java
<h1>Your Profile</h1>
<c:out value="${spitter.username}" /><br/>
<c:out value="${spitter.firstName}" />
<c:out value="${spitter.lastName}" />
```

图 5.6 展现了在 Web 浏览器中渲染的基本信息页面。

如果表单中没有发送 username 或 password 的话，会发生什么情况呢？或者说，如果 firstName 或 lastName 的值为空或太长的话，又会怎么样呢？接下来，让我们看一下如何为表单提交添加校验，从而避免数据呈现的不一致性。

### 校验表单

如果用户在提交表单的时候，username 或 password 文本域为空的话，那么将会导致在新建 Spitter 对象中，username 或 password 是空的 String。至少这是一种怪异的行为。如果这种现象不处理的话，这将会出现安全问题，因为不管是谁只要提交一个空的表单就能登录应用。

同时，我们还应该阻止用户提交空的 firstName 和 / 或 lastName，使应用仅在一定程度上保持匿名性。有个好的办法就是限制这些输入域值的长度，保持它们的值在一个合理的长度范围，避免这些输入域的误用。

有种处理校验的方式非常初级，那就是在 processRegistration() 方法中添加代码来检查值的合法性，如果值不合法的话，就将注册表单重新显示给用户。这是一个很简短的方法，因此，添加一些额外的 if 语句也不是什么大问题，对吧？

与其让校验逻辑弄乱我们的处理器方法，还不如使用 Spring 对 Java 校验 API（Java Validation API，又称 JSR-303）的支持。从 Spring 3.0 开 始，在 Spring MVC 中提供了对 Java 校验 API 的支持。在 Spring MVC 中要使用 Java 校验 API 的话，并不需要什么额外的配置。只要保证在类路径下包含这个 Java API 的实现即可，比如 Hibernate Validator。

Java 校验 API 定义了多个注解，这些注解可以放到属性上，从而限制这些属性的值。所有的注解都位于 javax.validation.constraints 包中。表 5.1 列出了这些校验注解。

|     注解     |                             描述                             |
| :----------: | :----------------------------------------------------------: |
| @AssertFalse |       所注解的元素必须是 Boolean 类型，并且值为 false        |
| @AssertTrue  |        所注解的元素必须是 Boolean 类型，并且值为 true        |
| @DecimalMax  | 所注解的元素必须是数字，并且它的值要小于或等于给定的  BigDecimalString 值 |
| @DecimalMin  | 所注解的元素必须是数字，并且它的值要大于或等于给定的  BigDecimalString 值 |
|   @Digits    |      所注解的元素必须是数字，并且它的值必须有指定的位数      |
|   @Future    |             所注解的元素的值必须是一个将来的日期             |
|     @Max     |    所注解的元素必须是数字，并且它的值要小于或等于给定的值    |
|     @Min     |    所注解的元素必须是数字，并且它的值要大于或等于给定的值    |
|   @NotNull   |                所注解元素的值必须不能为 null                 |
|    @Null     |                  所注解元素的值必须为 null                   |
|    @Past     |            所注解的元素的值必须是一个已过去的日期            |
|   @Pattern   |           所注解的元素的值必须匹配给定的正则表达式           |
|    @Size     | 所注解的元素的值必须是 String、集合或数组，并且它的长度要符合给定的范围 |

除了表 5.1中的注解，Java 校验 API 的实现可能还会提供额外的校验注解。同时，也可以定义自己的限制条件。但就我们来讲，将会关注于上表中的两个核心限制条件。

请考虑要添加到 Spitter 域上的限制条件，似乎需要使用 @NotNull 和 @Size 注解。我们所要做的事情就是将这些注解添加到 Spitter 的属性上。如下的程序清单展现了 Spitter 类，它的属性已经添加了校验注解。

```java
package spittr;

import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;

import org.apache.commons.lang3.builder.EqualsBuilder;
import org.apache.commons.lang3.builder.HashCodeBuilder;

public class Spitter {

  private Long id;
  
  @NotNull
  @Size(min=5, max=16)
  private String username;

  @NotNull
  @Size(min=5, max=25)
  private String password;
  
  @NotNull
  @Size(min=2, max=30)
  private String firstName;

  @NotNull
  @Size(min=2, max=30)
  private String lastName;
  
  ...
}
```

现在，Spitter 的所有属性都添加了 @NotNull 注解，以确保它们的值不为 null。类似地，属性上也添加了 @Size 注解以限制它们的长度在最大值和最小值之间。对 Spittr 应用来说，这意味着用户必须要填完注册表单，并且值的长度要在给定的范围内。

我们已经为 Spitter 添加了校验注解，接下来需要修改 processRegistration() 方法来应用校验功能。启用校验功能的 processRegistration() 如下所示：

```java
@RequestMapping(value="/register", method=POST)
public String processRegistration(
    @Valid Spitter spitter, 
    Errors errors) {
  if (errors.hasErrors()) {
    return "registerForm";
  }
    
  spitterRepository.save(spitter);
  return "redirect:/spitter/" + spitter.getUsername();
}
```

与程序清单5.17 中最初的 processRegistration() 方法相比，这里有了很大的变化。Spitter 参数添加了 @Valid 注解，这会告知 Spring，需要确保这个对象满足校验限制。

在 Spitter 属性上添加校验限制并不能阻止表单提交。即便用户没有填写某个域或者某个域所给定的值超出了最大长度，processRegistration() 方法依然会被调用。这样，我们就需要处理校验的错误，就像在 processRegistration() 方法中所看到的那样。

如果有校验出现错误的话，那么这些错误可以通过 Errors 对象进行访问，现在这个对象已作为 processRegistration() 方法的参数。（很重要一点需要注意，Errors 参数要紧跟在带有 @Valid 注解的参数后面，@Valid 注解所标注的就是要检验的参数。）processRegistration() 方法所做的第一件事就是调用Errors.hasErrors() 来检查是否有错误。

如果有错误的话，Errors.hasErrors() 将会返回到registerForm，也就是注册表单的视图。这能够让用户的浏览器重新回到注册表单页面，所以他们能够修正错误，然后重新尝试提交。现在，会显示空的表单，但是在下一章中，我们将在表单中显示最初提交的值并将校验错误反馈给用户。

如果没有错误的话，Spitter 对象将会通过 Repository 进行保存，控制器会像之前那样重定向到基本信息页面。 

## **小结**

在本章中，我们为编写应用程序的 Web 部分开了一个好头。可以看到，Spring 有一个强大灵活的 Web 框架。借助于注解，Spring MVC 提供了近似于 POJO 的开发模式，这使得开发处理请求的控制器变得非常简单，同时也易于测试。

当编写控制器的处理器方法时，Spring MVC 极其灵活。概括来讲，如果你的处理器方法需要内容的话，只需将对应的对象作为参数，而它不需要的内容，则没有必要出现在参数列表中。这样，就为请求处理带来了无限的可能性，同时还能保持一种简单的编程模型。

尽管本章中的很多内容都是关于控制器的请求处理的，但是渲染响应同样也是很重要的。我们通过使用 JSP 的方式，简单了解了如何为控制器编写视图。但是就 Spring MVC 的视图来说，它并不限于本章所看到的简单 JSP。

在接下来的第 6 章中，我们将会更深入地学习 Spring 视图，包括如何在 JSP 中使用 Spring 标签库。我们还会学习如何借助 Apache Tiles 为视图添加一致的布局结构。同时，还会了解 Thymeleaf，这是一个很有意思的 JSP 替代方案，Spring 为其提供了内置的支持。