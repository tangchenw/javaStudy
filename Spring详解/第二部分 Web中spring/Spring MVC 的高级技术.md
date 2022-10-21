# **Spring MVC 的高级技术**

本章内容：

- Spring MVC 配置的替代方案
- 处理文件上传
- 在控制器中处理异常
- 使用flash属性

稍等，还没有结束！ 

如果你在电视购物节目上看过一些小发明或产品广告的话，你可能听过类似这样的话。在广告描述完产品并宣称它能够做什么之后，我们可能会听到“稍等，还没有结束！”，然后广告会继续告诉我们产品还有什么令人激动的特性。

在很多方面，Spring MVC（其实，整个 Spring 也是如此）也有“还没有 结束！”这样的感觉。就在我们觉得已经掌握了 Spring MVC 能够做什么之后，我们会发现它所能做的还不止如此。

在第 5 章中，我们学习了 Spring MVC 的基础知识，以及如何编写控制器来处理各种请求。基于这些知识，我们在第 6 章学习了如何创建 JSP 和 Thymeleaf 视图，这些视图会将模型数据展现给用户。你可能认为我们已经掌握了 Spring MVC 的全部知识。但是稍等！还没有结束！

在本章中，我们会继续 Spring MVC 的话题，本章所介绍的特性已经超出了第 5 章和第 6 章基础知识的范畴。我们将会看到如何编写控制器来处理文件上传、如何处理控制器所抛出的异常，以及如何在模型中传递数据，使其能够在重定向（redirect）之后依然存活。

但首先，我要兑现一个承诺。在第 5 章中，我快速展现了如何通过 Abstract-AnnotationConfigDispatcherServletInitializer 搭建 Spring MVC，当时我承诺会为读者展现其他的配置方案。所以，在介绍文件上传和异常处理之前，我们先花一点时间探讨一下如何用其他的方式来搭建 DispatcherServlet 和  Context-LoaderListener。

## **Spring MVC 配置的替代方案**

在第 5 章中，我们通过扩展 AbstractAnnotationConfigDispatcherServletInitializer 快速搭建了 Spring MVC 环境。在这个便利的基础类中，假设我们需要基本的 DispatcherServlet 和 ContextLoaderListener 环境，并且 Spring 配置是使用 Java 的，而不是 XML。

尽管对很多 Spring 应用来说，这是一种安全的假设，但是并不一定总能满足我们的要求。除了 DispatcherServlet 以外，我们可能还需要额外的 Servlet 和 Filter；我们可能还需要对 DispatcherServlet 本身做一些额外的配置；或者，如果我们需要将应用部署到 Servlet 3.0 之前的容器中，那么还需要将 DispatcherServlet 配置到传统的 web.xml 中。

### **自定义 DispatcherServlet 配置**

虽然从程序清单 7.1 的外观上不一定能够看得出来，但是 AbstractAnnotation-ConfigDispatcherServletInitializer 所完成的事情其实比看上去要多。在 SpittrWeb-AppInitializer 中我们所编写的三个方法仅仅是必须要重载的 abstract 方法。但实际上还有更多的方法可以进行重载，从而实现额外的配置。

此类的方法之一就是 customizeRegistration()。在 AbstractAnnotationConfig-DispatcherServletInitializer 将 DispatcherServlet 注册到 Servlet 容器中之后，就会调用 customizeRegistration()，并将 Servlet 注册后得到的 Registration.Dynamic 传递进来。通过重载 customizeRegistration() 方法，我们可以对 Dispatcher-Servlet 进行额外的配置。

例如，在本章稍后的内容中（7.2 节），我们将会看到如何在 Spring MVC 中处理 multipart 请求和文件上传。如果计划使用 Servlet 3.0 对 multipart 配置的支持，那么需要使用 DispatcherServlet 的 registration 来启用 multipart 请求。我们可以重 载 customizeRegistration() 方法来设置 MultipartConfigElement，如下所示：

```java
@Override
protected void customizeRegistration(Dynamic registration) {
  registration.setMultipartConfig(
    new MultipartConfigElement("/tmp/spittr/uploads");
  );
}
```

借助 customizeRegistration() 方法中的 ServletRegistration.Dynamic，我们能够完成多项任务，包括通过调用 setLoadOnStartup() 设置 load-on-startup 优先级，通过 setInitParameter() 设置初始化参数，通过调用 setMultipartConfig() 配置 Servlet 3.0 对 multipart 的支持。在前面的样例中，我们设置了对 multipart 的支持，将上传文件的临时存储目录设置在 `/tmp/spittr/uploads` 中。

### **添加其他的 Servlet 和 Filter**

按照 AbstractAnnotationConfigDispatcherServletInitializer 的定义，它会创建 DispatcherServlet 和 ContextLoaderListener。但是，如果你想注册其他的 Servlet、 Filter 或 Listener 的话，那该怎么办呢？

基于 Java 的初始化器（initializer）的一个好处就在于我们可以定义任意数量的初始化器类。因此，如果我们想往 Web 容器中注册其他组件的话，只需创建一个新的初始化器就可以了。最简单的方式就是实现 Spring 的 WebApplicationInitializer 接口。

例如，如下的程序清单展现了如何创建 WebApplicationInitializer 实现并注册一个 Servlet。

```java
package com.myapp.config;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.ServletRegistration.Dynamic;
import org.springframework.web.WebApplicationInitializer;
import com.myapp.MyServlet;

public class MyServletInitializer extends WebApplicationInitializer {
  
  @Override
  public void onStartup(ServletContext servletContext) throws ServletException {
    Dynamic myServlet = servlectContext.addServlet("myServlet", MyServlet.class);
    
    myServlect.addMapping("/custom/**");
  }

}
```

程序清单 7.1 是相当基础的 Servlet 注册初始化器类。它注册了一个 Servlet 并将其映射到一个路径上。我们也可以通过这种方式来手动注册 Dispatcher-Servlet。（但这并没有必要，因为AbstractAnnotationConfigDispatcher-ServletInitializer 没用太多代码就将这项任务完成得很漂亮。）

类似地，我们还可以创建新的 WebApplicationInitializer 实现来注册 Listener 和 Filter。例如，如下的程序清单展现了如何注册 Filter。

```java
@Override
public void onStartup(ServlectContext servletContext) throws ServletException {

  javax.servlet.FilterRegistration.Dynamic filter = servletContext.addFilter("myFilter", MyFilter.class);
  
  filter.addMappingForUrlPatterns(null, false, "/custom/**");
} 
```

如果要将应用部署到支持 Servlet 3.0 的容器中，那么 WebApplicationInitializer 提供了一种通用的方式，实现在 Java 中注册 Servlet、Filter 和 Listener。不过，如果你只是注册 Filter， 并且该 Filter 只会映射到 DispatcherServlet 上的话，那么在 AbstractAnnotationConfigDispatcherServletInitializer 中还有一种快捷方式。

为了注册 Filter 并将其映射到 DispatcherServlet，所需要做的仅仅是重载 AbstractAnnotationConfigDispatcherServletInitializer 的 getServletFilters() 方法。例如，在如下的代码中，重载了 AbstractAnnotationConfigDispatcher-ServletInitializer 的 getServletFilters() 方法以注册 Filter：

```java
@Override
protected Filter() getServletFilters() {
  return new Filter[] { new MyFilter() };
}
```

我们可以看到，这个方法返回的是一个 javax.servlet.Filter 的数组。在这里它只返回了一个 Filter，但它实际上可以返回任意数量的 Filter。在这里没有必要声明它的映射路径，getServletFilters() 方法返回的所有 Filter 都会映射到 DispatcherServlet 上。

如果要将应用部署到 Servlet 3.0 容器中，那么 Spring 提供了多种方式来注册 Servlet（包括 DispatcherServlet）、Filter 和 Listener，而不必创建 web.xml 文件。但是，如果你不想采取以上所述方案的话，也是可以的。假设你需要将应用部署到不支持 Servlet 3.0 的容器中（或者你只是希望使用 web.xml 文件），那么我们完全可以按照传统的方式，通过 web.xml 配置 Spring MVC。让我们看一下该怎么做。

### 在 web.xml 中声明 DispatcherServlet

在典型的 Spring MVC 应用中，我们会需要 DispatcherServlet 和 ContextLoader-Listener。AbstractAnnotationConfigDispatcherServletInitializer 会自动注册它们，但是如果需要在 web.xml 中注册的话，那就需要我们自己来完成这项任务了。 如下是一个基本的 web.xml 文件，它按照传统的方式搭建了 Dispatcher-Servlet 和 ContextLoaderListener。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns="http://java.sun.com/xml/ns/javaee"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
      http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" >

  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/spring/root-context.xml</param-value>
  </context-param>
  
  <listener>
    <listener-class>
      org.springframework.web.context.ContextLoaderListener
    </listener-class>
  </listener>
  
  <servlet>
    <servlet-name>appServlet</servlet-name>
    <servlet-class>
      org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    <load-on-startup>1</load-on-startup>
  </servlet>
  
  <servlet-mapping>
    <servlet-name>appServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

就像我在第 5 章曾经介绍过的，ContextLoaderListener 和 DispatcherServlet 各自都会加载一个 Spring 应用上下文。上下文参数 contextConfigLocation 指定了一个 XML 文件的地址，这个文件定义了根应用上下文，它会被 ContextLoader-Listener 加载。如程序清单 7.3 所示，根上下文会从 `/WEB-INF/spring/rootcontext.xml` 中加载 bean 定义。

DispatcherServlet 会根据 Servlet 的名字找到一个文件，并基于该文件加载应用上下文。在程序清单 7.3 中，Servlet 的名字是 appServlet，因此 Dispatcher-Servlet 会从 `/WEBINF/appServlet-context.xml` 文件中加载其应用上下文。

如果你希望指定 DispatcherServlet 配置文件的位置的话，那么可以在 Servlet 上指定一个 contextConfigLocation 初始化参数。例如，如下的配置中，DispatcherServlet 会从 `/WEB-INF/spring/appServlet/servlet-context.xml` 加载它的 bean：

```xml
<servlet>
  <servlet-name>appServlet</servlet-name>
  <servlet-class>
    org.springframework.web.servlet.DispatcherServlet
  </servlet-class>
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>
      /WEB-INF/spring/appServlet/servlet-context.xml
    </param-value>
  </init-parma>
  <load-on-startup>1</load-on-startup>
</servlet>
```

当然，上面阐述的都是如何让 DispatcherServlet 和 ContextLoaderListener 从 XML 中加载各自的应用上下文。但是，在本书中的大部分内容中，我们都更倾向于使用 Java 配置而不是 XML 配置。因此，我们需要让 Spring MVC 在启动的时候，从带有 @Configuration 注解的类上加载配置。

要在 Spring MVC 中使用基于 Java 的配置，我们需要告诉 DispatcherServlet 和 ContextLoaderListener 使用 AnnotationConfigWebApplicationContext，这是一 个 WebApplicationContext 的实现类，它会加载 Java 配置类，而不是使用 XML。要实现这种配置，我们可以设置 contextClass 上下文参数以及 DispatcherServlet 的初始化参数。如下的程序清单展现 了一个新的 web.xml，在这个文件中，它所搭建的 Spring MVC 使用基于 Java 的 Spring 配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns="http://java.sun.com/xml/ns/javaee"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
      http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" >

  <context-param>
    <param-name>contextClass</param-name>
    <param-value>
      org.springframework.web.context.support.AnnotationConfigWebApplicationContext
    </param-value>
  </context-param>
  
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>
      com.habuma.spitter.config.RootConfig
    </param-value>
  </context-param>
  
  <listener>
    <listener-class>
      org.springframework.web.context.ContextLoaderListener
    </listener-class>
  </listener>
  
  <servlet>
    <servlet-name>appServlet</servlet-name>
    <servlet-class>
      org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    <init-param>
      <param-name>contextClass</param-name>
      <param-value>
        org.springframework.web.context.support.AnnotationConfigWebApplicationContext
      </param-value>
    </init-param>
    <context-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>
        com.habuma.spitter.config.WebConfigConfig
      </param-value>
    </context-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  
  <servlet-mapping>
    <servlet-name>appServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

现在我们已经看到了如何以多种不同的方式来搭建 Spring MVC，那么接下来我们会看一下如何使用 Spring MVC 来处理文件上传。

## **处理 multipart 形式的数据**

在 Web 应用中，允许用户上传内容是很常见的需求。在 Facebook 和 Flickr 这样的网站中，用户通常会上传照片和视频，并与家人和朋友分享。还有一些服务允许用户上传照片，然后按照传统方式将其打印在纸上，或者用在 T 恤衫和咖啡杯上。

Spittr 应用在两个地方需要文件上传。当新用户注册应用的时候，我们希望他们能够上传一张图片，从而与他们的个人信息相关联。当用户提交新的 Spittle 时，除了文本消息以外，他们可能还会上传一 张照片。

一般表单提交所形成的请求结果是很简单的，就是以 `&` 符分割的多个 name-value 对。例如，当在 Spittr 应用中提交注册表单时，请求会如下所示：

```html
firstName=Charless&lastName=Xavier&email=professor%40xmen.org&username=professorx&password=letmein01
```

尽管这种编码形式很简单，并且对于典型的基于文本的表单提交也足够满足要求，但是对于传送二进制数据，如上传图片，就显得力不从心了。与之不同的是，multipart 格式的数据会将一个表单拆分为多个部分（part），每个部分对应一个输入域。在一般的表单输入域中，它所对应的部分中会放置文本型数据，但是如果上传文件的话，它所对应的部分可以是二进制，下面展现了 multipart 的请求体：

```html
-----------WebKitFormBoundaryqgkaBn8lHJCuNmiW
Content-Disposition: form-data; name="firstName"

Charles
-----------WebKitFormBoundaryqgkaBn8lHJCuNmiW
Content-Disposition: form-data; name="lastName"

Xavier
-----------WebKitFormBoundaryqgkaBn8lHJCuNmiW
Content-Disposition: form-data; name="email"

charles@xmen.com
-----------WebKitFormBoundaryqgkaBn8lHJCuNmiW
"Content-Disposition: form-data; name="username"

professorx
-----------WebKitFormBoundaryqgkaBn8lHJCuNmiW
Content-Disposition: form-data; name="password"

letmeinO1
-----------WebKitFormBoundaryqgkaBn8lHJCuNmiW
Content-Disposition: form-data; name="profilePicture"; filename="me.jpg"

Content-Type: image/jpeg

  [[ Binary image data goes here ]]
-----------WebKitFormBoundaryqgkaBn8lHJCuNmiW--
```

在这个 multipart 的请求中，我们可以看到 profilePicture 部分与其他部分明显不同。除了其他内容以外，它还有自己的 Content-Type 头，表明它是一个 JPEG 图片。尽管不一定那么明显，但 profilePicture 部分的请求体是二进制数据，而不是简单的文本。

尽管 multipart 请求看起来很复杂，但在 Spring MVC 中处理它们却很容易。在编写控制器方法处理文件上传之前，我们必须要配置一个 multipart 解析器，通过它来告诉 DispatcherServlet 该如何读取 multipart 请求。

DispatcherServlet 并没有实现任何解析 multipart 请求数据的功能。它将该任务委托给了 Spring 中 MultipartResolver 策略接口的实现，通过这个实现类来解析 multipart 请求中的内容。从 Spring 3.1 开 始，Spring 内置了两个 MultipartResolver 的实现供我们选择：

- CommonsMultipartResolver：使用 Jakarta Commons FileUpload 解析 multipart 请求；
- StandardServletMultipartResolver：依赖于 Servlet 3.0 对 multipart 请求的支持（始于 Spring 3.1）。

一般来讲，在这两者之 间，StandardServletMultipartResolver 可能会是优选的方案。它使用 Servlet 所提供的功能支持，并不需要依赖任何其他的项目。如果我们需要将应用部署到 Servlet 3.0 之前的容器中，或者还没有使用 Spring 3.1 或更高版本，那么可能就需要 CommonsMultipartResolver 了。

**使用 Servlet 3.0 解析 multipart 请求**

兼容 Servlet 3.0 的 StandardServletMultipartResolver 没有构造器参数，也没有要设置的属性。这样，在 Spring 应用上下文中，将其声明为 bean 就会非常简单，如下所示：

```java
@Bean
public MultipartResolver multipartResolver() throws IOException {
  return new StandardServletMultipartResolver();
}
```

既然这个 @Bean 方法如此简单，你可能就会怀疑我们到底该如何限制 StandardServletMultipartResolver 的工作方式呢。如果我们想要限制用户上传文件的大小，该怎么实现？如果我们想要指定文件在上传时，临时写入目录在什么位置的话，该如何实现？因为没有属性和构造器参数，StandardServletMultipartResolver 的功能看起来似乎有些受限。

其实并不是这样，我们是有办法配置 StandardServletMultipartResolver 的限制条件的。只不过不是在 Spring 中配置StandardServletMultipart-Resolver， 而是要在 Servlet 中指定 multipart 的配置。至少，我们必须要指定在文件上传的过程中，所写入的临时文件路径。如果不设定这个最基本配置的话，StandardServletMultipartResolver 就无法正常工作。具体来讲，我们必须要在 web.xml 或 Servlet 初始化类中，将 multipart 的具体细节作为 DispatcherServlet 配置的一部分。

如果我们采用 Servlet 初始化类的方式来配置 DispatcherServlet 的话，这个初始化类应该已经实现了 WebApplicationInitializer，那我们可以在 Servlet registration 上调用 setMultipartConfig() 方法，传入一个 MultipartConfig-Element 实例。如下是最基本的 DispatcherServlet multipart 配置，它将临时路径设置为 `/tmp/spittr/uploads`：

```java
DispatcherServlet ds = new DispatchServlet();
Dynamic registration = context.addServlet("appServlet", ds);
registration.addMapping("/");
registration.setMultipartConfig(new MultipartConfigElement("/tmp/spittr/uploads"));
```

如果我们配置 DispatcherServlet 的 Servlet 初始化类继承了 Abstract-AnnotationConfigDispatcherServletInitializer 或 AbstractDispatcher-ServletInitializer 的话，那么我们不会直接创建 DispatcherServlet  实例并将其注册到 Servlet 上下文中。这样的话，将不会有对 Dynamic Servlet registration 的引用供我们使用了。但是，我们可以通过重载customize-Registration() 方法（它会得到一个 Dynamic 作为参数）来配置 multipart 的具体细节：

```java
@Override
protected void customizeRegistration(Dynamic registration) {
  registration.setMultipartConfig(
    new MultipartConfigElement("/tmp/spittr/uploads");
  );
}
```

到目前为止，我们所使用是只有一个参数的 MultipartConfigElement 构造器，这个参数指定的是文件系统中的一个绝对目录，上传文件将会临时写入该目录中。但是，我们还可以通过其他的构造器来限制上传文件的大小。除了临时路径的位 置，其他的构造器所能接受的参数如下：

- 上传文件的最大容量（以字节为单位）。默认是没有限制的。
- 整个 multipart 请求的最大容量（以字节为单位），不会关心有多少个 part 以及每个 part 的大小。默认是没有限制的。
- 在上传的过程中，如果文件大小达到了一个指定最大容量（以字节为单位），将会写入到临时文件路径中。默认值为 0，也就是所有上传的文件都会写入到磁盘上。

例如，假设我们想限制文件的大小不超过 2MB，整个请求不超过 4MB，而且所有的文件都要写到磁盘中。下面的代码使用 MultipartConfigElement 设置了这些临界值：

```java
@Override
protected void customizeRegistration(Dynamic registration) {
  registration.setMultipartConfig(
    new MultipartConfigElement("/tmp/spittr/uploads",
      2097152, 4194304, 0);
  );
}
```

如果我们使用更为传统的 web.xml 来配置 MultipartConfigElement 的话，那么可以使用 <servlet> 中的 <multipart-config> 元素，如下所示：

```xml
<servlet>
  <servlet-name>appServlet</servlet-name>
  <servlet-class>
    org.springframework.web.servlet.DispatchServlet
  </servlet-class>
  <load-on-startup>1</load-on-startup>
  <multipart-config>
    <location>/tmp/spittr/upload</location>
    <max-file-size>2097152</max-file-size>
    <max-request-size>4194304</max-request-size>
  </multipart-config>
</servlet>
```

<multipart-config> 的默认值与 MultipartConfigElement 相同。与 MultipartConfigElement 一样，必须要配置的是 <location>。

**配置 Jakarta Commons FileUpload multipart 解析器**

通常来讲，StandardServletMultipartResolver 会是最佳的选择，但是如果我们需要将应用部署到非 Servlet 3.0 的容器中，那么就得需要替代的方案。如果喜欢的话，我们可以编写自己的 MultipartResolver 实现。不过，除非想要在处理 multipart 请求的时候执行特定的逻辑，否则的话，没有必要这样做。Spring 内置了 CommonsMultipartResolver，可以作为 StandardServletMultipartResolver 的替代方案。

将 CommonsMultipartResolver 声明为 Spring bean 的最简单方式如下：

```java
@Bean
public MultipartResolver multipartResolver() {
  return new MultipartResolver();
}
```

与 StandardServletMultipartResolver 有所不同，CommonsMultipart-Resolver 不会强制要求设置临时文件路径。默认情况下，这个路径就是 Servlet 容器的临时目录。不过，通过设置 uploadTempDir 属性，我们可以将其指定为一个不同的位置：

```java
@Bean
public MultipartResolver multipartResolver() throws IOException {
  CommonsMultipartResolver multipartResolver = new CommonsMultipartResolver();
  multipartResolver.setUploadTempDir(
    new FileSystemResource("/tmp/spittr/uploads");
  );
  return multipartResolver;
}
```

实际上，我们可以按照相同的方式指定其他的 multipart 上传细节，也就是设置 CommonsMultipartResolver 的属性。例如，如下的配置就等价于我们在前文通过 MultipartConfigElement 所配置的 StandardServlet-MultipartResolver：

```java
@Bean
public MultipartResolver multipartResolver() throws IOException {
  CommonsMultipartResolver multipartResolver = new CommonsMultipartResolver();
  multipartResolver.setUploadTempDir(
    new FileSystemResource("/tmp/spittr/uploads"));
  multipartResolver.setMaxUploadSize(2097152);
  multipartResolver.setMaxInMemorySize(0);
  return multipartResolver;
}
```

在这里，我们将最大的文件容量设置为 2MB，最大的内存大小设置为 0 字节。这两个属性直接对应于 MultipartConfigElement 的第二个和第四个构造器参数，表明不能上传超过 2MB 的文件，并且不管文件的大小如何，所有的文件都会写到磁盘中。但是与 MultipartConfigElement 有所不同，我们无法设定 multipart 请求整体的最大容量。

### **处理 multipart 请求**

现在已经在 Spring 中（或 Servlet 容器中）配置好了对 mutipart 请求的处理，那么接下来我们就可以编写控制器方法来接收上传的文件。要实现这一点，最常见的方式就是在某个控制器方法参数上添加 @RequestPart 注解。

假设我们允许用户在注册 Spittr 应用的时候上传一张图片，那么我们需要修改表单，以允许用户选择要上传的图片，同时还需要修改 SpitterController 中的 processRegistration() 方法来接收上传的图片。如下的代码片段来源于 Thymeleaf 注册表单视图（registrationForm.html），着重强调了表单所需的修改：

```jsp
<form method="POST" th:object="${spitter}" enctype="multipart/form-data">
  
  ...
  <label>Profile Picture</label>:
  <input type="file" name="profilePicture" accept="image/jpeg,image/png,image/gif" /><br/>
  ...
  
</form>
```

<form> 标签现在将 enctype 属性设置为 multipart/formdata，这会告诉浏览器以 multipart 数据的形式提交表单，而不是以表单数据的形式进行提交。在 multipart 中，每个输入域都会对应一个 part。

除了注册表单中已有的输入域，我们还添加了一个新的域，其 type 为 file。这能够让用户选择要上传的图片文件。accept 属性用来将文件类型限制为 JPEG、PNG 以及 GIF 图片。根据其 name 属性，图片数据将会发送到 multipart 请求中的 profilePicture part 之中。

现在，我们需要修改 processRegistration() 方法，使其能够接受上传的图片。其中一种方式是添加 byte 数组参数，并为其添加 @RequestPart 注解。如下为示例：

```java
@RequestMapping(value="/register", method=POST)
public String processRegistration(
    @RequestPart("profilePicture") byte[]  profilePicture,
    @Valid Spittr spittr,
    Errors errors) {
  ...
} 
```

当注册表单提交的时候，profilePicture 属性将会给定一个 byte 数组，这个数组中包含了请求中对应 part 的数据（通过 @RequestPart 指定）。如果用户提交表单的时候没有选择文件，那么这个数组会是空（而不是 null）。获取到图片数据后，processRegistration() 方法剩下的任务就是将文件保存到某个位置。

我们将会稍后讨论如何保存文件。但首先，想一下，对于提交的图片数据我们都了解哪些信息呢。或者，更为重要的是，我们还不知道些什么呢？尽管我们已经得到了 byte 数组形式的图片数据，并且根据它能够得到图片的大小，但是对于其他内容我们就一无所知了。我们不知道文件的类型是什么，甚至不知道原始的文件名是什么。你需要判断如何将 byte 数组转换为可存储的文件。

**接受 MultipartFile**

使用上传文件的原始 byte 比较简单但是功能有限。因此，Spring 还提供了 MultipartFile 接口，它为处理 multipart 数据提供了内容更为丰富的对象。如下的程序清单展现了 MultipartFile 接口的概况。

```java
package org.springframework.web.multipart

import java.io.File;
import java.io.IOException;
import java.io.InputStream;

public interface MultipartFile {
  String getName();
  String getOriginalFilename();
  String getContentType();
  boolean isEmpty();
  long getSize();
  byte[] getBytes() throws IOException;
  InputStream getInputStream() throws IOException;
  void transferTo(File dest) throws IOException;
}
```

我们可以看到，MultipartFile 提供了获取上传文件 byte 的方式，但是它所提供的功能并不仅限于此，还能获得原始的文件名、大小以及内容类型。它还提供了一个 InputStream，用来将文件数据以流的方式进行读取。

除此之外，MultipartFile 还提供了一个便利的 transferTo() 方 法，它能够帮助我们将上传的文件写入到文件系统中。作为样例，我们可以在 process-Registration() 方法中添加如下的几行代码，从而将上传的图片文件写入到文件系统中：

```java
profilePicture.tranferTo(new File("/data/spittr/" + profilePicture.getOriginalFilename()));
```

将文件保存到本地文件系统中是非常简单的，但是这需要我们对这些文件进行管理。我们需要确保有足够的空间，确保当出现硬件故障时，文件进行了备份，还需要在集群的多个服务器之间处理这些图片文件的同步。

**将文件保存到 Amazon S3 中**

另外一种方案就是让别人来负责处理这些事情。多加几行代码，我们就能将图片保存到云端。例如，如下的程序清单所展现的 saveImage() 方法能够将上传的文件保存到 Amazon S3 中，我们在 processRegistration() 中可以调用该方法。

```java
private void saveImage(MultipartFile image) throws ImageUploadException {
  try {
    AWSCredentials awsCredentials = new AWSCredentials(s3AccessKey, s2SecretKey);
    S3Service s3 = new ResetS3Service(awsCredentials);
    
    S3Bucket bucket = s3.getBucket("spittrImages");
    S3Object imageObject = new S3Object(image.getOriginalFilename);
    
    imageObject.setDataInputStream(image.getInputStream());
    imageObject.setContentLength(image.getSize());
    imageObject.setContentType(image.getContentType());
    
    AccessControlList acl = new AccessControlList();
    acl.setOwner(bucket.getOwner());
    acl.grantPermission(GroupGrants.ALL_USERS, Permission.PERMISSION_READ);
    imageObject.setAcl(acl);
    
    s3.putObject(bucket, imageObject);
  } catch (Exception e) {
    throw new ImageUploadException("Unable to save image", e);
  }
}
```

程序清单 7.6 将 MultipartFile 保存到 Amazon S3 中 saveImage() 方法所做的第一件事就是构建 Amazon Web Service（AWS）凭证。为了完成这一点，你需要有一个 S3 Access Key 和 S3 Secret Access Key。当注册 S3 服务的时候，Amazon 会将其提供给你。它们会通过值注入的方式提供给 SpitterController。

AWS 凭证准备好后，saveImage() 方法创建了一个 JetS3t 的 RestS3Service 实例，可以通过它来操作 S3 文件系统。它获取 spitterImages bucket 的引用并创建用来包含图片的 S3Object 对象，接下来将图片数据填充到 S3Object。

在调用 putObject() 方法将图片数据写到 S3 之前，saveImage() 方法设置了 S3Object 的权限，从而允许所有的用户查看它。这是很重要的 —— 如果没有它的话，这些图片对我们应用程序的用户就是不可见的。最后，如果出现任何问题的话，将会抛出 ImageUploadException 异常。

**以 Part 的形式接受上传的文件**

如果你需要将应用部署到 Servlet 3.0 的容器中，那么会有 MultipartFile 的一个替代方案。Spring MVC 也能接受 javax.servlet.http.Part 作为控制器方法的参数。如果使用 Part 来替换 MultipartFile 的话，那么 processRegistration() 的方法签名将会变成如下的形式：

```java
@RequestMapping(value="/register", method=POST)
public String processRegistration(
    @RequestPart("profilePicture") Part profilePicture,
    @Valid Spittr spittr,
    Errors errors) {
  ...
} 
```

就主体来言（不开玩笑地说），Part 接口与 MultipartFile 并没有太大的差别。在如下的程序清单中，我们可以看到 Part 接口的有一些方法其实是与 MultipartFile 相对应的。

```java
package javax.servlet.http;

import java.io.*;
import java.util.*;

public interface Part {
  InputStream getInputStream() throws IOException;
  String getContentType();
  String getName();
  String getSubmittedFileName();
  long getSize();
  void write(String fileName) throws IOException;
  void delete() throws IOException;
  String getHeader(String name);
  Collection<String> getHeaders(String name);
  Collection<String> getHeaderNames();
}
```

在很多情况下，Part 方法的名称与 MultipartFile 方法的名称是完全相同的。有一些比较类似，但是稍有差异，比如 getSubmittedFileName() 对应于 getOriginalFilename()。类似地，write() 对应于 transferTo()，借助该方法我们能够将上传的文件写入文件系统中：

```java
profilePicture.write("/data/spittr/" + profilePicture.getOriginalFilename());
```

值得一提的是，如果在编写控制器方法的时候，通过 Part 参数的形式接受文件上传，那么就没有必要配置 MultipartResolver 了。只有使用 MultipartFile 的时候，我们才需要 MultipartResolver。

## **处理异常**

到现在为止，在 Spittr 应用中，我们假设所有的功能都正常运行。但是如果某个地方出错的话，该怎么办呢？当处理请求的时候，抛出异常该怎么处理呢？如果发生了这样的情况，该给客户端什么响应呢？

不管发生什么事情，不管是好的还是坏的，Servlet 请求的输出都是一个 Servlet 响应。如果在请求处理的时候，出现了异常，那它的输出依然会是 Servlet 响应。异常必须要以某种方式转换为响应。

Spring 提供了多种方式将异常转换为响应：

- 特定的 Spring 异常将会自动映射为指定的 HTTP 状态码；
- 异常上可以添加 @ResponseStatus 注解，从而将其映射为某一个 HTTP 状态码；
- 在方法上可以添加 @ExceptionHandler 注解，使其用来处理异常。

处理异常的最简单方式就是将其映射到 HTTP 状态码上，进而放到响应之中。接下来，我们看一下如何将异常映射为某一个 HTTP 状态码。

### 将异常映射为 HTTP 状态码

在默认情况下，Spring 会将自身的一些异常自动转换为合适的状态码。表 7.1 列出了这些映射关系。

|               Spring 异常               |         HTTP 状态码          |
| :-------------------------------------: | :--------------------------: |
|              BindException              |      400 - Bad Request       |
|     ConversionNotSupportedException     | 500 - Internal Server Error  |
|   HttpMediaTypeNotAcceptableException   |     406 - Not Acceptable     |
|   HttpMediaTypeNotSupportedException    | 415 - Unsupported Media Type |
|     HttpMessageNotReadableException     |      400 - Bad Request       |
|     HttpMessageNotWritableException     | 500 - Internal Server Error  |
| HttpRequestMethodNotSupportedException  |   405 - Method Not Allowed   |
|     MethodArgumentNotValidException     |      400 - Bad Request       |
| MissingServletRequestParameterException |      400 - Bad Request       |
|   MissingServletRequestPartException    |      400 - Bad Request       |
|  NoSuchRequestHandlingMethodException   |       404 - Not Found        |
|          TypeMismatchException          |      400 - Bad Request       |

表 7.1 中的异常一般会由 Spring 自身抛出，作为 DispatcherServlet 处理过程中或执行校验时出现问题的结果。例如，如果 DispatcherServlet 无法找到适合处理请求的控制器方法，那么将会抛出 NoSuchRequestHandlingMethod-Exception 异常，最终的结果就是产生 404 状态码的响应（Not Found）。

尽管这些内置的映射是很有用的，但是对于应用所抛出的异常它们就无能为力了。幸好，Spring 提供了一种机制，能够通过 @ResponseStatus 注解将异常映射为 HTTP 状态码。

为了阐述这项功能，请参考 SpittleController 中如下的请求处理方法，它可能会产生 HTTP 404 状态（但目前还没有实现）：

```java
@RequestMapping(value="/{spittleId}", method=RequestMethod.GET)
public String spittle(
    @PathVariable("spittleId") long spittleId, 
    Model model) {
  Spittle spittle = spittleRepository.findOne(spittleId);
  if (spittle == null) {
    throw new SpittleNotFoundException();
  }
  model.addAttribute(spittle);
  return "spittle";
}
```

在这里，会从 SpittleRepository 中，通过 ID 检索 Spittle 对象。如果 findOne() 方法能够返回 Spittle 对象的话，那么会将 Spittle 放到模型中，然后名为 spittle 的视图会负责将其渲染到响应之中。但是如果 findOne() 方法返回 null 的话，那么将会抛出 SpittleNotFoundException 异常。现在 SpittleNot-FoundException 就是一个简单的非检查型异常，如下所示： 

```java
package spittr.web;

public class SpittleNotFoundException extends RuntimeException {
}
```

如果调用 spittle() 方法来处理请求，并且给定 ID 获取到的结果为空，那么 SpittleNotFoundException（默认）将会产生 500 状态码（Internal Server Error）的响应。实际上，如果出现任何没有映射的异常，响应都会带有 500 状态码，但是，我们可以通过映射 SpittleNotFoundException 对这种默认行为进行变更。

当抛出 SpittleNotFoundException 异常时，这是一种请求资源没有找到的场景。如果资源没有找到的话，HTTP 状态码 404 是最为精确的响应状态码。所以，我们要使用 @ResponseStatus 注解将 SpittleNotFoundException 映射为 HTTP 状态码 404。

```java
package spittr.web;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

@ResponseStatus(value=HttpStatus.NOT_FOUND, reason="Spittle Not Found")
public class SpittleNotFoundException extends RuntimeException {
}
```

在引入 @ResponseStatus 注解之后，如果控制器方法抛出 SpittleNotFound-Exception 异常的话，响应将会具有 404 状态码，这是因为 Spittle Not Found。

### **编写异常处理的方法**

在很多的场景下，将异常映射为状态码是很简单的方案，并且就功能来说也足够了。但是如果我们想在响应中不仅要包括状态码，还要包含所产生的错误，那该怎么办呢？此时的话，我们就不能将异常视为 HTTP 错误了，而是要按照处理请求的方式来处理异常了。

作为样例，假设用户试图创建的 Spittle 与已创建的 Spittle 文本完全相同，那么 SpittleRepository 的 save() 方法将会抛出 DuplicateSpittle Exception 异常。这意味着 SpittleController 的 saveSpittle() 方法可能需要处理这个异常。如下面的程序清单所示，saveSpittle() 方法可以直接处理这个异常。

```java
@RequestMapping(method=RequestMethod.POST)
public String saveSpittle(SpittleForm form, Model model) {
  try {
    spittleRepository.save(new Spittle(null, form.getMessage(), new Date(), 
        form.getLongitude(), form.getLatitude()));
    return "redirect:/spittles";
  } catch (DuplicateSpittleException e) {
    return "error/duplicate";
  }
}
```

程序清单 7.9 中并没有特别之处，它只是在 Java 中处理异常的基本样例，除此之外，也就没什么了。

它运行起来没什么问题，但是这个方法有些复杂。该方法可以有两个路径，每个路径会有不同的输出。如果能让 saveSpittle() 方法只关注正确的路径，而让其他方法处理异常的话，那么它就能简单一些。

首先，让我们首先将 saveSpittle() 方法中的异常处理方法剥离掉：

```java
@RequestMapping(method=RequestMethod.POST)
public String saveSpittle(SpittleForm form, Model model) {
  spittleRepository.save(new Spittle(null, form.getMessage(), new Date(), 
    form.getLongitude(), form.getLatitude()));
  return "redirect:/spittles";
}
```

可以看到，saveSpittle() 方法简单了许多。因为它只关注成功保存 Spittle 的情况，所以只有一个执行路径，很容易理解（和测试）。

现在，我们为 SpittleController 添加一个新的方法，它会处理抛出 Duplicate-SpittleException 的情况：

```java
@ExceptionHandler(DuplicateSpittleException.class)
public String handleDuplicateSpittle() {
  return "error/duplicate";
}
```

handleDuplicateSpittle() 方法上添加了 @ExceptionHandler 注解，当抛出  DuplicateSpittleException 异常的时候，将会委托该方法来处理。它返回的是一个 String，这与处理请求的方法是一致的，指定了要渲染的逻辑视图名，它能够告诉用户他们正在试图创建一条重复的条目。

对于 @ExceptionHandler 注解标注的方法来说，比较有意思的一点在于它能处理同一个控制器中所有处理器方法所抛出的异常。所以，尽管我们从save-Spittle() 中抽取代码创建了 handleDuplicateSpittle() 方法，但是它能够处理 SpittleController 中所有方法所抛出的 DuplicateSpittleException 异常。我们不用在每一个可能抛出 DuplicateSpittleException 的方法中添加异常处理代码，这一个方法就涵盖了所有的功能。

既然 @ExceptionHandler 注解所标注的方法能够处理同一个控制器类中所有处理器方法的异常，那么你可能会问有没有一种方法能够处理所有控制器中处理器方法所抛出的异常呢。从 Spring 3.2 开始，这肯定是能够实现的，我们只需将其定义到控制器通知类中即可。

什么是控制器通知方法？很高兴你会问这样的问题，因为这就是我们下面要讲的内容。

##

## **为控制器添加通知**

如果控制器类的特定切面能够运用到整个应用程序的所有控制器中，那么这将会便利很多。举例来说，如果要在多个控制器中处理异常，那 @ExceptionHandler 注解所标注的方法是很有用的。不过，如果多个控制器类中都会抛出某个特定的异常，那么你可能会发现要在所有的控制器方法中重复相同的 @Exception-Handler 方法。或者，为了避免重复，我们会创建一个基础的控制器类，所有控制器类要扩展这个类，从而继承通用的 @ExceptionHandler 方法。

Spring 3.2 为这类问题引入了一个新的解决方案：控制器通知。控制器通知（controller advice）是任意带有 @ControllerAdvice 注解的类，这个类会包含一个或多个如下类型的方法：

- @ExceptionHandler 注解标注的方法；
- @InitBinder 注解标注的方法；
- @ModelAttribute 注解标注的方法。

在带有 @ControllerAdvice 注解的类中，以上所述的这些方法会运用到整个应用程序所有控制器中带有 @RequestMapping 注解的方法上。

@ControllerAdvice 注解本身已经使用了 @Component，因此@Controller-Advice 注解所标注的类将会自动被组件扫描获取到，就像带有 @Component 注解的类一样。

@ControllerAdvice 最为实用的一个场景就是将所有的 @ExceptionHandler 方法收集到一个类中，这样所有控制器的异常就能在一个地方进行一致的处理。例如，我们想将 DuplicateSpittleException 的处理方法用到整个应用程序的所有控制器上。如下的程序清单展现的 AppWideExceptionHandler 就能完成这一任务，这是一个带有 @ControllerAdvice 注解的类。

```java
package spittr.web;

import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

@ControllerAdvice
public class AppWideExceptionHandler {

  @ExceptionHandler(DuplicateSpittleException.class)
  public String handleNotFound() {
    return "error/duplicate";
  }

}
```

现在，如果任意的控制器方法抛出了 DuplicateSpittleException，不管这个方法位于哪个控制器中，都会调用这个 duplicateSpittleHandler() 方法来处理异 常。我们可以像编写 @RequestMapping 注解的方法那样来编写 @Exception-Handler 注解的方法。如程序清单 7.10 所示，它返回 `error/duplicate` 作为逻辑视图名，因此将会为用户展现一个友好的出错页面。

## **跨重定向请求传递数据**

在 5.4.1 小节中，在处理完 POST 请求后，通常来讲一个最佳实践就是执行一下重定向。除了其他的一些因素外，这样做能够防止用户点击浏览器的刷新按钮或后退箭头时，客户端重新执行危险的 POST 请求。

在第 5 章，在控制器方法返回的视图名称中，我们借助 了 `redirect:` 前缀的力量。当控制器方法返回的 String 值 以 `redirect:` 开头的话，那么这个 String 不是用来查找视图的，而是用来指导浏览器进行重定向的路径。我们可以回头看一下程序清单 5.17，可以看到 processRegistration() 方法返回的 `redirect:String` 如下所示：

```java
return "redirect:/spitter/" + spitter.getUsername();
```

`redirect: `前缀能够让重定向功能变得非常简单。你可能会想 Spring 很难再让重定向功能变得更简单了。但是，请稍等：Spring 为重定向功能还提供了一些其他的辅助功能。

具体来讲，正在发起重定向功能的方法该如何发送数据给重定向的目标方法呢？一般来讲，当一个处理器方法完成之后，该方法所指定的模型数据将会复制到请求中，并作为请求中的属性，请求会转发（forward）到视图上进行渲染。因为控制器方法和视图所处理的是同一个请求，所以在转发的过程中，请求属性能够得以保存。

但是，如图 7.1 所示，当控制器的结果是重定向的话，原始的请求就结束了，并且会发起一个新的 GET 请求。原始请求中所带有的模型数据也就随着请求一起消亡了。在新的请求属性中，没有任何的模型数据，这个请求必须要自己计算数据。

![7.1 重定向](C:\Users\汤琛\Desktop\学习资料\Spring详解\images\7.1 重定向.jpg)

显然，对于重定向来说，模型并不能用来传递数据。但是我们也有一些其他方案，能够从发起重定向的方法传递数据给处理重定向方法中：

- 使用 URL 模板以路径变量和/或查询参数的形式传递数据；
- 通过 flash 属性发送数据。

首先，我们看一下 Spring 如何帮助我们通过路径变量和 / 或查询参数的形式传递数据。

### **通过 URL 模板进行重定向**

通过路径变量和查询参数传递数据看起来非常简单。例如，在程序清单 5.19 中，我们以路径变量的形式传递了新创建 Spitter 的 username。但是按照现在的写法，username 的值是直接连接到重定向 String 上的。这能够正常运行，但是还远远不能说没有问题。当构建 URL 或 SQL 查询语句的时候，使用 String 连接是很危险的。

```java
return "redirect:/spitter/{username}";
```

除了连接 String 的方式来构建重定向 URL，Spring 还提供了使用模板的方式来定义重定向 URL。例如，在程序清单 5.19 中，processRegistration() 方法的最后一行可以改写为如下的形式：

```java
@RequestMapping(value="/register", method=POST);
public String processRegistration(
    Spitter spitter, Model model) {
  spitterRepository.save(spitter);
  
  model.addAttribute("username", spitter.getUsername());
  return "redirect:/spitter/{username}";
}
```

现在，username 作为占位符填充到了 URL 模板中，而不是直接连接到重定向 String 中，所以 username 中所有的不安全字符都会进行转义。这样会更加安全，这里允许用户输入任何想要的内容作为 username，并会将其附加到路径上。

除此之外，模型中所有其他的原始类型值都可以添加到 URL 中作为查询参数。作为样例，假设除了 username 以外，模型中还要包含新创建 Spitter 对象的 id 属性，那 processRegistration() 方法可以改写为如下的形式：

```java
@RequestMapping(value="/register", method=POST)
public String processRegistration(
    Spitter spitter, Model model) {
  spitterRepository.save(spitter);
  model.addAttribute("username", spitter.getUsername());
  model.addAttribute("spitterId", spitter.getId());
  return "redirect:/spitter/{username}";
}
```

所返回的重定向 String 并没有太大的变化。但是，因为模型中的 spitterId 属性没有匹配重定向 URL 中的任何占位符，所以它会自动以查询参数的形式附加到重定向 URL 上。

如果 username 属性的值是 habuma 并且 spitterId 属性的值是 42，那么结果得到的重定向 URL 路径将会是 `/spitter/habuma?spitterId=42`。

通过路径变量和查询参数的形式跨重定向传递数据是很简单直接的方式，但它也有一定的限制。它只能用来发送简单的值，如 String 和数字的值。在 URL 中，并没有办法发送更为复杂的值，但这正是 flash 属性能够提供帮助的领域。

### **使用 flash 属性**

假设我们不想在重定向中发送 username 或 ID 了，而是要发送实际的 Spitter 对象。如果我们只发送 ID 的话，那么处理重定向的方法还需要从数据库中查找才能得到 Spitter 对象。但是，在重定向之前，我们其实已经得到了 Spitter 对象。为什么不将其发送给处理重定向的方法，并将其展现出来呢？

Spitter 对象要比 String 和 int 更为复杂。因此，我们不能像路径变量或查询参数那么容易地发送 Spitter 对象。它只能设置为模型中的属性。

但是，正如我们前面所讨论的那样，模型数据最终是以请求参数的形式复制到请求中的，当重定向发生的时候，这些数据就会丢失。因此，我们需要将 Spitter 对象放到一个位置，使其能够在重定向的过程中存活下来。

有个方案是将 Spitter 放到会话中。会话能够长期存在，并且能够跨多个请求。所以我们可以在重定向发生之前将 Spitter 放到会话中，并在重定向后，从会话中将其取出。当然，我们还要负责在重定向后在会话中将其清理掉。

实际上，Spring 也认为将跨重定向存活的数据放到会话中是一个很不错的方式。但是，Spring 认为我们并不需要管理这些数据，相反，Spring 提供了将数据发送为 flash 属性（flash attribute）的功能。 按照定义，flash 属性会一直携带这些数据直到下一次请求，然后才会消失。

Spring 提供了通过 RedirectAttributes 设置 flash 属性的方法，这是 Spring 3.1 引入的 Model 的一个子接口。RedirectAttributes 提供了 Model 的所有功能，除此之外，还有几个方法是用来设置 flash 属性的。 具体来讲，Redirect-Attributes 提供了一组 addFlashAttribute() 方法来添加 flash 属性。重新看一 下 processRegistration() 方法，我们可以使用 addFlashAttribute() 将 Spitter 对象添加到模型中：

```java
@RequestMapping(value="/register", method=POST)
public String processRegistration(
    Spitter spitter, RedirectAttribute model) {
  spitterRespository.save(spitter);
  model.addAttribute("username", spitter.getUsername());
  model.addFlashAttribute("spitter", spitter);
  return "redirect:/spitter/{username}";
}
```

在这里，我们调用了 addFlashAttribute() 方法，并将 spitter 作为 key，Spitter 对象作为值。另外，我们还可以不设置 key 参数，让 key 根据值的类型自行推断得出：

```java
model.addFlashAttribute(spitter);
```

因为我们传递了一个 Spitter 对象给 addFlashAttribute() 方法，所以推断得到的 key 将会是 spitter。

在重定向执行之前，所有的 flash 属性都会复制到会话中。在重定向后，存在会话中的 flash 属性会被取出，并从会话转移到模型之中。处理重定向的方法就能从模型中访问 Spitter 对象了，就像获取其他的模型对象一样。图 7.2 阐述了它是如何运行的。

![7.2 flash属性](C:\Users\汤琛\Desktop\学习资料\Spring详解\images\7.2 flash属性.jpg)

为了完成 flash 属性的流程，如下展现了更新版本的 showSpitterProfile() 方法，在从数据库中查找之前，它会首先从模型中检查 Spitter 对象：

```java
@RequestMapping(value="/{username}", method=GET)
public String showSpitterProfile(
    @PathVariable String username, Model model) {
  if (!model.containsAttribute("spitter")) {
    model.addAttribute(spitterRepository.findByUsername(username));
  }
  return "profile";
}
```

可以看到，showSpitterProfile() 方法所做的第一件事就是检查是否存有 key 为 spitter 的 model 属性。如果模型中包含 spitter 属性，那就什么都不用做了。这里面包含的 Spitter 对象将会传递到视图中进行渲染。但是如果模型中不包含 spitter 属性的话，那么 showSpitterProfile() 将会从 Repository 中查找 Spitter，并将其存放到模型中。

## **小结**

在 Spring 中，总是会有“还没有结束”的感觉：更多的特性、更多的选择以及实现开发目标的更多方式。Spring MVC 有很多功能和技巧。

当然，Spring MVC 的环境搭建是有多种可选方案的一个领域。在本章中，我们首先看了一下搭建 Spring MVC 中 DispatcherServlet 和 ContextLoaderListener 的多种方式。我们还看到了如何调整 DispatcherServlet 的注册功能以及如何注册自定义的 Servlet 和 Filter。如果你需要将应用部署到更老的应用服务器上，我们还快速了解了如何使用 web.xml 声明 DispatcherServlet 和  ContextLoader-Listener。

然后，我们了解了如何处理 Spring MVC 控制器所抛出的异常。尽管带有 @RequestMapping 注解的方法可以在自身的代码中处理异常，但是如果我们将异常处理的代码抽取到单独的方法中，那么控制器的代码会整洁得多。

为了采用一致的方式处理通用的任务，包括在应用的所有控制器中处理异常，Spring 3.2 引入了 @ControllerAdvice，它所创建的类能够将控制器的通用行为抽取到同一个地方。

最后，我们看了一下如何跨重定向传递数据，包括 Spring 对 flash 属性的支持：类似于模型的属性，但是能在重定向后存活下来。这样的话，就能采用非常恰当的方式为 POST 请求执行一个重定向回应，而且能够将处理 POST 请求时的模型数据传递过来，然后在重定向后使用或展现这些模型数据。

如果你还有疑惑的话，那么可以告诉你，这就是我所说的“更多的功 能”！其实，我们并没有讨论到 Spring MVC 的每个方面。我们将会在第 16 章中重新讨论 Spring MVC，到时你会看到如何使用它来创建 REST API。

但现在，我们将会暂时放下 Spring MVC，看一下 Spring Web Flow，这是一个构建在 Spring MVC 之上的流程框架，它能够引导用户执行一系列向导步骤。

