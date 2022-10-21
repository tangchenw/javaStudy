# **渲染 Web 视图**

本章内容：

- 将模型数据渲染为 HTML
- 使用 JSP 视图
- 通过 tiles 定义视图布局
- 使用 Thymeleaf 视图

上一章主要关注于如何编写处理 Web 请求的控制器。我们也创建了一些简单的视图，用来渲染控制器产生的模型数据，但我们并没有花太多时间讨论视图，也没有讨论控制器完成请求到结果渲染到用户的浏览器中的这段时间内到底发生了什么，而这正是本章的主要内容。

## **理解视图解析**

在第 5 章中，我们所编写的控制器方法都没有直接产生浏览器中渲染所需的 HTML。这些方法只是将一些数据填充到模型中，然后将模型传递给一个用来渲染的视图。这些方法会返回一个 String 类型的值，这个值是视图的逻辑名称，不会直接引用具体的视图实现。尽管我们也编写了几个简单的 JavaServer Page（JSP）视图，但是控制器并不关心这些。

将控制器中请求处理的逻辑和视图中的渲染实现解耦是 Spring MVC 的一个重要特性。如果控制器中的方法直接负责产生 HTML 的话，就很难在不影响请求处理逻辑的前提下，维护和更新视图。控制器方法和视图的实现会在模型内容上达成一致，这是两者的最大关联，除此之外，两者应该保持足够的距离。

但是，如果控制器只通过逻辑视图名来了解视图的话，那 Spring 该如何确定使用哪一个视图实现来渲染模型呢？这就是 Spring 视图解析器的任务了。

在第 5 章中，我们使用名为 InternalResourceViewResolver 的视图解析器。在它的配置中，为了得到视图的名字，会使用 `/WEBINF/views/` 前缀和 `.jsp` 后缀，从而确定来渲染模型的 JSP 文件的物理位置。现在，我们回过头来看一下视图解析的基础知识以及 Spring 提供的其他视图解析器。

Spring MVC 定义了一个名为 ViewResolver 的接口，它大致如下所示：

```java
public interface ViewResolver {
  View resolverViewName(String viewName, Locale locale) throws Exception;
}
```

当给 resolveViewName() 方法传入一个视图名和 Locale 对象时，它会返回一个 View 实例。View 是另外一个接口，如下所示：

```java
public interface View {

  String getContentType();
  
  void render(Map<String, ?> model, HttpServletRequest request, HttpServlectResponse response) throws Exception;
}
```

View 接口的任务就是接受模型以及 Servlet 的 request 和 response 对象，并将输出结果渲染到 response 中。

这看起来非常简单。我们所需要做的就是编写 ViewResolver 和 View 的实现，将要渲染的内容放到 response 中，进而展现到用户的浏览器中。对吧？

实际上，我们并不需要这么麻烦。尽管我们可以编写 ViewResolver 和 View 的实现，在有些特定的场景下，这样做也是有必要的，但是一般来讲，我们并不需要关心这些接口。我在这里提及这些接口只是为了让你对视图解析内部如何工作有所了解。Spring 提供了多个内置的实现，如表 6.1 所示，它们能够适应大多数的场景。

|           视图解析器           |                             描述                             |
| :----------------------------: | :----------------------------------------------------------: |
|      BeanNameViewResolver      | 将视图解析为 Spring 应用上下文中的 bean，其中 bean 的 ID 与视图的名字相同 |
| ContentNegotiatingViewResolver | 通过考虑客户端需要的内容类型来解析视图， 委托给另外一个能够产生对应内容类型的视图 解析器 |
|     FreeMarkerViewResolver     |                 将视图解析为 FreeMarker 模板                 |
|  InternalResourceViewResolver  |        将视图解析为 Web 应用的内部资源（一般为 JSP）         |
|   JasperReportsViewResolver    |               将视图解析为 JasperReports 定义                |
|   ResourceBundleViewResolver   |          将视图解析为资源 bundle（一般为属性文件）           |
|       TilesViewResolver        | 将视图解析为 Apache Tile 定义，其中 tile ID 与视图名称相同。注意有两个不同的 TilesViewResolver 实现，分别对应于 Tiles 2.0 和 Tiles 3.0 |
|      UrlBasedViewResolver      | 直接根据视图的名称解析视图，视图的名称会匹配一个物理视图的定义 |
|   VelocityLayoutViewResolver   | 将视图解析为 Velocity 布局，从不同的Velocity 模板中组合页面  |
|      VelocityViewResolver      |                  将视图解析为 Velocity 模板                  |
|        XmlViewResolver         | 将视图解析为特定 XML 文件中的 bean 定义。类似于 BeanNameViewResolver |
|        XsltViewResolver        |                将视图解析为 XSLT 转换后的结果                |

Spring 4 和 Spring 3.2 支持表 6.1 中的所有视图解析器。Spring 3.1 支持除  Tiles 3 TilesViewResolver 之外的所有视图解析器。我们没有足够的篇幅介绍Spring 所提供的 13 种视图解析器。这其实也没什么，因为在大多数应用中，我们只会用到其中很少的一部分。

对于表 6.1 中的大部分视图解析器来讲，每一项都对应 Java Web 应用中特定的某种视图技术。InternalResourceViewResolver 一般会用于 JSP，TilesView-Resolver 用于 Apache Tiles 视图，而 FreeMarkerViewResolver 和 Velocity-ViewResolver 分别对应 FreeMarker 和 Velocity 模板视图。

在本章中，我们将会关注与大多数 Java 开发人员最息息相关的视图技术。因为大多数 Java Web 应用都会用到 JSP，我们首先将会介绍  InternalResource-ViewResolver，这个视图解析器一般会用来解析 JSP 视图。接下来，我们将会介绍 TilesViewResolver，控制 JSP 页面的布局。

在本章的最后，我们将会看一个没有列在表 6.1 中的视图解析器。Thymeleaf 是一种用来替代 JSP 的新兴技术，Spring 提供了与 Thymeleaf 的原生模板（natural template）协作的视图解析器，这种模板之所以得到这样的称呼是因为它更像是最终产生的 HTML，而不是驱动它们的 Java 代码。Thymeleaf 是一种非常令人兴奋的视图方案，所以你尽可以先往后翻几页，去 6.4 节看一下在 Spring 中是如何使用它的。

如果你依然停留在本页的话，那么你可能知道 JSP 曾经是，而且现在依然还是 Java 领域占主导地位的视图技术。在以前的项目中，也许你使用过 JSP，将来有可能还会继续使用这项技术，所以接下来让我们看一下如何在 Spring MVC 中使用 JSP 视图。

## **创建 JSP 视图**

不管你是否相信，JavaServer Pages 作为 Java Web 应用程序的视图技术已经超过 15 年了。尽管开始的时候它很丑陋，只是类似模板技术（如 Microsoft 的 Active Server Pages）的 Java 版本，但 JSP 这些年在不断进化，包含了对表达式语言和自定义标签库的支持。

Spring 提供了两种支持 JSP 视图的方式：

- InternalResourceViewResolver 会将视图名解析为 JSP 文件。另外，如果在你的 JSP 页面中使用了 JSP 标准标签库 （JavaServer Pages Standard Tag Library，JSTL）的话，InternalResourceViewResolver 能够将视图名解析为  JstlView 形式的 JSP 文件，从而将 JSTL 本地化和资源 bundle 变量暴露给JSTL的格式化（formatting）和信息（message）标签。
- Spring 提供了两个 JSP 标签库，一个用于表单到模型的绑定，另一个提供了通用的工具类特性。

不管你使用 JSTL，还是准备使用 Spring 的 JSP 标签库，配置解析 JSP 的视图解析器都是非常重要的。尽管 Spring 还有其他的几个视图解析器都能将视图名映射为 JSP 文件，但就这项任务来讲，InternalResourceViewResolver 是最简单和最常用的视图解析器。我们在第 5 章已经接触到了如何配置InternalResource-ViewResolver。但是在那里，我们只是匆忙体验了一下，以便于查看控制器在浏览器中的效果。接下来，我们将会更加仔细地了解 InternalResourceView-Resolver，看看如何让它完全听命于我们。

### **配置适用于 JSP 的视图解析器**

有一些视图解析器，如 ResourceBundleViewResolver 会直接将逻辑视图名映射为特定的 View 接口实现， 而 InternalResourceViewResolver 所采取的方式并不那么直接。它遵循一种约定，会在视图名上添加前缀和后缀，进而确定一个 Web 应用中视图资源的物理路径。

作为样例，考虑一个简单的场景，假设逻辑视图名为 home。通用的实践是将 JSP 文件放到 Web 应用的 WEB-INF 目录下，防止对它的直接访问。如果我们将所有的 JSP 文件都放在 `/WEB-INF/views/` 目录下， 并且 home 页的 JSP 名为`home.jsp`，那么我们可以确定物理视图的路径就是逻辑视图名 home 再加上`/WEB-INF/views/` 前缀和 `.jsp` 后缀。如图 6.1 所示。

![6.1 jsp 解析器](C:\Users\汤琛\Desktop\学习资料\Spring详解\images\6.1 jsp 解析器.jpg)

当使用 @Bean 注解的时候，我们可以按照如下的方式配置 InternalResource-ViewResolver，使其在解析视图时，遵循上述的约定。

```java
@Bean
public ViewResolver viewResolver() {
  InternalResourceViewResolver resolver = new InternalResourceViewResolver();
  resolver.setPrefix("/WEB-INF/views");
  resolver.setSuffix(".jsp");
  return resolver;
}
```

作为替代方案，如果你更喜欢使用基于 XML 的 Spring 配置，那么可以按照如下的方式配置 InternalResourceViewResolver：

```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver"
      p:prefix="/WEB-INF/views"
      p:suffix=".jsp" />
```

InternalResourceViewResolver 配置就绪之后，它就会将逻辑视图名解析为 JSP 文件，如下所示：

- home 将会解析为 `/WEB-INF/views/home.jsp`
- productList 将会解析为 `/WEB-INF/views/productList.jsp`
- books/detail 将会解析为 `/WEB-INF/views/books/detail.jsp`

让我们重点看一下最后一个样例。当逻辑视图名中包含斜线时，这个斜线也会带到资源的路径名中。因此，它会对应到 prefix 属性所引用目录的子目录下的 JSP 文件。这样的话，我们就可以很方便地将视图模板组织为层级目录结构，而不是将它们都放到同一个目录之中。

**解析JSTL视图**

到目前为止，我们对 InternalResourceViewResolver 的配置都很基础和简单。它最终会将逻辑视图名解析为 InternalResourceView 实例，这个实例会引用 JSP 文件。但是如果这些 JSP 使用 JSTL 标签来处理格式化和信息的话，那么我们会希望 InternalResourceViewResolver 将视图解析为 JstlView。

JSTL 的格式化标签需要一个 Locale 对象，以便于恰当地格式化地域相关的值，如日期和货币。信息标签可以借助 Spring 的信息资源和 Locale，从而选择适当的信息渲染到 HTML 之中。通过解析 JstlView，JSTL 能够获得 Locale 对象以及 Spring 中配置的信息资源。

如果想让 InternalResourceViewResolver 将视图解析为 JstlView，而不是 InternalResourceView 的话，那么我们只需设置它的 viewClass 属性即可：

```java
@Bean
public ViewResolver viewResolver() {
  InternalResourceViewResolver resolver = new InternalResourceViewResolver();
  resolver.setPrefix("/WEB-INF/views");
  resolver.setSuffix(".jsp");
  resolver.setViewClass(org.springframework.web.service.view.JstlView.class);
  return resolver;
}
```

同样，我们也可以使用 XML 完成这一任务：

```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver"
      p:prefix="/WEB-INF/views"
      p:suffix=".jsp"
      p:viewClass="org.springframework.web.service.view.JstlView" />
```

不管使用 Java 配置还是使用 XML，都能确保 JSTL 的格式化和信息标签能够获得 Locale 对象以及 Spring 中配置的信息资源。

### **使用 Spring 的 JSP 库**

当为 JSP 添加功能时，标签库是一种很强大的方式，能够避免在脚本块中直接编写 Java 代码。Spring 提供了两个 JSP 标签库，用来帮助定义 Spring MVC Web 的视图。其中一个标签库会用来渲染 HTML 表单标 签，这些标签可以绑定 model 中的某个属性。另外一个标签库包含了一些工具类标签，我们随时都可以非常便利地使用它们。

在这两个标签库中，你可能会发现表单绑定的标签库更加有用。所以，我们就从这个标签库开始学习 Spring 的 JSP 标签。我们将会看到如何将 Spittr 应用的注册表单绑定到模型上，这样表单就可以预先填充值，并且在表单提交失败后，能够展现校验错误。

**将表单绑定到模型上**

Spring 的表单绑定 JSP 标签库包含了 14 个标签，它们中的大多数都用来渲染 HTML 中的表单标签。但是，它们与原生 HTML 标签的区别在于它们会绑定模型中的一个对象，能够根据模型中对象的属性填充值。标签库中还包含了一个为用户展现错误的标签，它会将错误信息渲染到最终的 HTML 之中。

为了使用表单绑定库，需要在 JSP 页面中对其进行声明：

```jsp
<%@ taglib uri="http://www.springframework.org/tags/form" prefix="sf" %>
```

需要注意，我将前缀指定为 `sf`，但通常也可能使用 `form` 前缀。你可以选择任意喜欢的前缀，我之所以选择 `sf` 是因为它很简洁、易于输入，并且还是 Spring form 的简写形式。在本书中，当使用表单绑定库的时候，我会一直使用 `sf` 前缀。

在声明完表单绑定标签库之后，你就可以使用 14 个相关的标签了。如表 6.2 所示，借助 Spring 表单绑定标签库中所包含的标签，我们能够将模型对象绑定到渲染后的 HTML 表单中：

|     JSP 标签      |                             描述                             |
| :---------------: | :----------------------------------------------------------: |
|   <sf:checkbox>   |       渲染成一个HTML 标签，其中type属性设置 为checkbox       |
|  <sf:checkboxes>  | 渲染成多个 HTML <input> 标签，其中 type 属性设置为 checkbox  |
|    <sf:errors>    |            在一个 HTML <span> 中渲染输入域的错误             |
|     <sf:form>     | 渲染成一个 HTML <form> 标签，并为其内部标签暴露绑定路径，用于数据绑定 |
|    <sf:hidden>    |  渲染成一个 HTML <input> 标签，其中 type 属性设置为 hidden   |
|    <sf:input>     |   渲染成一个 HTML <input> 标签，其中 type 属性设置为 text    |
|    <sf:label>     |                 渲染成一个 HTML <label> 标签                 |
|    <sf:option>    | 渲染成一个 HTML 标签，其 selected 属性根据所绑定的值进行设置 |
|   <sf:options>    |    按照绑定的集合、数组或 Map，渲染成一个 HTML 标签的列表    |
|   <sf:password>   |       渲染成一个HTML 标签，其中type属性设置 为password       |
| <sf:radiobutton>  |   渲染成一个 HTML <input> 标签，其中 type 属性设置为 radio   |
| <sf:radiobuttons> |   渲染成多个 HTML <input> 标签，其中 type 属性设置为 radio   |
|    <sf:select>    |                渲染为一个 HTML <select> 标签                 |
|   <sf:textarea>   |               渲染为一个 HTML <textarea> 标签                |

要在一个样例中介绍所有的这些标签是很困难的，如果一定要这样做的话，肯定也会非常牵强。就 Spittr 样例来说，我们只会用到适合于 Spittr 应用中注册表单的标签。

具体来讲，也就是 `<sf:form>`、`<sf:input>` 和 `<sf:password>`。在注册 JSP 中使用这些标签后，所得到的程序如下所示：

```jsp
<sf:form method="POST" commandName="spitter" >
  First Name: <sf:input path="firstName" /><br/>
  Last Name: <sf:input path="lastName" /><br/>
  Email: <sf:input path="email" /><br/>
  Username: <sf:input path="username" /><br/>
  Password: <sf:password path="password" /><br/>
  <input type="submit" value="Register" />
</sf:form>
```

`<sf:form>` 会渲染会一个 HTML <form> 标签，但它也会通过 command-Name 属性构建针对某个模型对象的上下文信息。在其他的表单绑定标签中，会引用这个模型对象的属性。

在之前的代码中，我们将 commandName 属性设置为 spitter。因此，在模型中必须要有一个 key 为 spitter 的对象，否则的话，表单不能正常渲染（会出现 JSP 错误）。这意味着我们需要修改一 下 SpitterController，以确保模型中存在以 spitter 为 key 的 Spitter 对象：

```java
@RequestMapping(value="/register", method=GET)
public String showRegistrationForm(Model model) {
  model.addAttribute(new Spitter());
  return "registerForm";
}
```

修改后的 showRegistrationForm() 方法中，新增了一个 Spitter 实例到模型中。模型中的 key 是根据对象类型推断得到的，也就是 spitter，与我们所需要的完全一致。

回到这个表单中，前四个输入域将 HTML <input> 标签改成了`<sf:input>`。这个标签会渲染成一个 HTML <input> 标签，并且 type 属性将会设置为 text。我们在这里设置了 path 属性，<input> 标签的 value 属性值将会设置为模型对象中 path 属性所对应的值。例如，如果在模型中 Spitter 对象的 firstName 属性值为 Jack，那么 `<sf:input path="firstName">` 所渲染的 <input> 标签中，会存在 value="Jack"。

对于 password 输入域，我们使用 `<sf:password>` 来代替 `<sf:input>` 。`<sf:password>` 与 `<sf:input>` 类似，但是它所渲染的 HTML <input> 标签中，会将 type 属性设置为 password，这样当输入的时候，它的值不会直接明文显示。

为了帮助读者了解最终的 HTML 看起来是什么样子的，假设有个用户已经提交了表单，但值都是不合法的。校验失败后，用户会被重定向到注册表单，最终的 HTML 元素如下所示：

```jsp
<form id="spitter" action="/spitter/spitter/register" method="POST" >
  First Name: <input id="firstName" name="firstName" type="text" value="J" /><br/>
  Last Name: <input id="lastName" name="lastName" type="text" value="B" /><br/>
  Email: <input id="email" name="email" type="text" value="jack" /><br/>
  Username: <input id="username" name="username" type="text" value="jack" /><br/>
  Password: <input id="password" name="password" type="password" value="" /><br/>
  <input type="submit" value="Register" />
</form>
```

值得注意的是，从 Spring 3.1 开始，`<sf:input>` 标签能够允许我们指定 type 属性，这样的话，除了其他可选的类型外，还能指定 HTML 5 特定类型的文本域，如 date、range 和 email。例如，我们可以按照如下的方式指定 email 域：

```jsp
Email: <sf:input path="email" type="email" /><br/>
```

这样所渲染得到的 HTML 如下所示：

```jsp
Email: <input id="email" name="email" type="email" value="jack" /><br/>
```

相对于标准的 HTML 标签，使用 Spring 的表单绑定标签能够带来一定的功能提升，在校验失败后，表单中会预先填充之前输入的值。但是，这依然没有告诉用户错在什么地方。为了指导用户矫正错误，我 们需要使用 `<sf:errors>`。

**展现错误**

如果存在校验错误的话，请求中会包含错误的详细信息，这些信息是与模型数据放到一起的。我们所需要做的就是到模型中将这些数据抽取出来，并展现给用户。`<sf:errors>`能够让这项任务变得很简单。

例如，让我们看一下 `<sf:errors>` 将用到 registerForm.jsp 中的代码片段：

```jsp
<sf:form method="POST" commandName="spitter">
  First Name: <sf:input path="fisrtName" />
             <sf:errors path="firstName" /><br/>
  ...
</sf:form>
```

尽管我只展现了将 `<sf:errors>` 用到 First Name 输入域的场景，但是它可以按照同样简单的方式用到注册表单的其他输入域中。在这里，它的 path 属性设置成了 firstName，也就是指定了要显示 Spitter 模型对象中哪个属性的错误。如果 firstName 属性没有错误的话，那么 `<sf:errors>` 不会渲染任何内容。但如果有校验错误的话，那么它将会在一个 HTML <span> 标签中显示错误信息。

例如，如果用户提交字母 `J` 作为名字的话，那么如下的 HTML 片段就是针对 First Name 输入域所显示的内容：

```jsp
First Name: <input id="firstName" name="firstName" type="text" value="J" />
<span id="firstName.errors">size must be between 2 and 30</span>
```

现在，我们已经可以为用户展现错误信息，这样他们就能修正这些错误了。我们可以更进一步，修改错误的样式，使其更加突出显示。为了做到这一点，可以设置 cssClass 属性：

```jsp
<sf:form method="POST" commandName="spitter" >
  First Name: <sf:input path="fisrtName" />
    <sf:errors path="fisrtName" cssClass="error" /><br/>
...
</sf:form>
```

同样，简单起见，我只会展现如何为 firstName 输入域的 `<sf:errors>` 设置 cssClass 属性。你可以将其用到其他的输入域上。

现在 errors 的 `<span>` 会有一个值为 error 的 class 属性。剩下需要做的就是为这个类定义 CSS 样式。如下就是一个简单的 CSS 样式， 它会将错误信息渲染为红色：

```css
span.error {
  color: red;
}
```

在输入域的旁边展现错误信息是一种很好的方式，这样能够引起用户的关注，提醒他们修正错误。但这样也会带来布局的问题。另外一种处理校验错误方式就是将所有的错误信息在同一个地方进行显示。为了做到这一点，我们可以移除每个输入域上的 `<sf:errors>` 元素， 并将其放到表单的顶部，如下所示：

```jsp
<sf:form method="POST" commandName="spitter" >
  <sf:errors path="*" element="div" cssClass="errors" />
...
</sf:form>
```

这个 `<sf:erros>` 与之前相比，值得注意的不同之处在于它的 path 被设置成了 `*`。这是一个通配符选择器，会告诉 `<sf:errors>` 展现所有属性的所有错误。

同样需要注意的是，我们将 element 属性设置成了 div。默认情况下，错误都会渲染在一个 HTML <span> 标签中，如果只显示一个错误的话，这是不错的选择。但是，如果要渲染所有输入域的错误的话，很可能要展现不止一个错误，这时候使用 <span> 标签（行内元素）就不合适了。像 <div> 这样的块级元素会更为合适。因此，我们可以将 element 属性设置为 div，这样的话，错误就会渲染在一个 <div> 标签中。

像之前一样，cssClass 属性被设置 errors，这样我们就能为 <div> 设置样式。如下为 <div> 的 CSS 样式，它具有红色的边框和浅红色的背景：

```css
div.errors {
  background-color: #ffcccc;
  border: 2px solid red;
}
```

现在，我们在表单的上方显示所有的错误，这样页面布局可能会更加容易一些。但是，我们还没有着重显示需要修正的输入域。通过为每个输入域设置 cssErrorClass 属性，这个问题很容易解决。我们也可以将每个 label 都替换为 `<sf:label>`，并设置它的 cssErrorClass 属性。如下就是做完必要修改后的 First Name 输入域：

```jsp
<sf:form method="POST" commandName="spitter" >
  <sf:label path="firstName" cssErrorClass="error">First Name</sf:label>:
  <sf:input path="firstName" cssErrorClass="error" /><br/>
...
</sf:form>
```

`<sf:label>` 标签像其他的表单绑定标签一样，使用 path 来指定它属于模型对象中的哪个属性。在本例中，我们将其设置为 firstName，因此它会绑定 Spitter 对象的 firstName 属性。假设没有校验错误的话，它将会渲染为如下的 HTML <label>元素：

```jsp
<label for="fisrtName">First Name</label>
```

就其自身来说，设置 `<sf:label>` 的 path 属性并没有完成太多的功能。但是，我们还同时设置了 cssErrorClass 属性。如果它所绑定的属性有任何错误的话，在渲染得到的 <label> 元素中，class 属性将会被设置为 error，如下所示：

```jsp
<label for="fisrtName" class="error">First Name</label>
```

与之类似，`<sf:input>` 标签的 cssErrorClass 属性被设置为 error。如果有任何校验错误的话，在渲染得到的 <input> 标签中，class 属性将会被设置为 error。现在我们已经为文本标记和输入域设置了样式，这样当出现错误的时候，会将用户的注意力转移到此处。例如，如下的 CSS 会将文本标记渲染为红色，并将输入域设置为浅红色背景：

```css
label.error {
  color: red;
}
input.error {
  background-color: #ffcccc;
}
```

现在，我们有了很好的方式为用户展现错误信息。不过，我们还可以做另外一件事情，能够让这些错误信息更加易读。重新看一 下 Spitter 类，我们可以在校验注解上设置 message 属性，使其引用对用户更为友好的信息，而这些信息可以定义在属性文件中：

```java
@NotNull
@Size(min=5, max=16, message="{username.size}")
private String username;

@NotNull
@Size(min=5, max=25, message="{password.size}")
private String password;
  
@NotNull
@Size(min=2, max=30, message="{firstName.size}")
private String firstName;

@NotNull
@Size(min=2, max=30, message="{lastName.size}")
private String lastName;
  
@NotNull
@Email(message="{email.valid}")
private String email;
```

对于上面每个域，我们都将其 @Size 注解的 message 设置为一个字符串，这个字符串是用大括号括起来的。如果没有大括号的话，message 中的值将会作为展现给用户的错误信息。但是使用了大括号之后，我们使用的就是属性文件中的某一个属性，该属性包含了实际的信息。

接下来需要做的就是创建一个名为 ValidationMessages.properties 的文件，并将其放在根类路径之下：

```properties
firstName.size=First name must be between {min} and {max} characters long.
lastName.size=Last name must be between {min} and {max} characters long.
username.size=Username must be between {min} and {max} characters long.
password.size=Password must be between {min} and {max} characters long.
email.valid=The email address must be valid.
```

ValidationMessages.properties 文件中每条信息的 key 值对应于注解中 message 属性占位符的值。同时，最小和最大长度没有硬编码在 ValidationMessages.properties 文件中，在这个用户友好的信息中也有自己的占位符 —— {min} 和 {max} —— 它们会引用 @Size 注解上所设置的 min 和 max 属性。

**Spring 通用的标签库**

除了表单绑定标签库之外，Spring 还提供了更为通用的 JSP 标签库。实际上，这个标签库是 Spring 中最早的标签库。这么多年来，它有所变化，但是在最早版本的 Spring 中，它就已经存在了。

要使用 Spring 通用的标签库，我们必须要在页面上对其进行声明：

```jsp
<%@ taglib uri="http://www.springframework.org/tags" prefix="s" %>
```

与其他 JSP 标签库一样，prefix 可以是任意你所喜欢的值。在这里，通用的做法是将这个标签库的前缀设置为 spring。但是，我将其设置为 `s`，因为它更加简洁，更易于阅读和输入。

标签库声明之后，我们就可以使用表 6.3 中的十个 JSP 标签了。

|     JSP 标签      |                             描述                             |
| :---------------: | :----------------------------------------------------------: |
|     <s:bind>      | 将绑定属性的状态导出到一个名为 status 的页面作用域属性 中，与 <s:path> 组合使用获取绑定属性的值 |
|  <s:escapeBody>   |      将标签体中的内容进行 HTML 和 / 或 JavaScript 转义       |
| <s:hasBindErrors> | 根据指定模型对象（在请求属性中）是否有绑定错误，有条 件地渲染内容 |
|  <s:htmlEscape>   |               为当前页面设置默认的 HTML 转义值               |
|    <s:message>    | 根据给定的编码获取信息，然后要么进行渲染（默认行为），要么将其设置为页面作用域、请求作用域、会话作用 域或应用作用域的变量（通过使用 var 和 scope 属性实现） |
|  <s:nestedPath>   |            设置嵌入式的 path，用于 <s:bind> 之中             |
|     <s:theme>     | 根据给定的编码获取主题信息，然后要么进行渲染（默认行 为），要么将其设置为页面作用域、请求作用域、会话作用 域或应用作用域的变量（通过使用 var 和 scope 属性实现） |
|   <s:transform>   |      使用命令对象的属性编辑器转换命令对象中不包含的属性      |
|      <s:url>      | 创建相对于上下文的 URL，支持 URI 模板变量以及  HTML/XML/JavaScript 转义。可以渲染 URL（默认行为），也可以将其设置为页面作用域、请求作用域、会话作用域或应用作用域的变量（通过使用 var 和 scope 属性实现） |
|     <s:eval>      | 计算符合 Spring 表达式语言（Spring Expression Language， SpEL）语法的某个表达式的值，然后要么进行渲染（默认行为），要么将其设置为页面作用域、请求作用域、会话作用域或应用作用域的变量（通过使用 var 和 scope 属性实现） |

表 6.3 中的一些标签已经被 Spring 表单绑定标签库淘汰了。例如，`<bind>` 标签就是 Spring 最初所提供的表单绑定标签，它比我们在前面所介绍的标签复杂得多。

因为这些标签库的行为比表单绑定标签少得多，所以我不会详细介绍每个标签，而是快速介绍几个最为有用的标签，其余的留给读者自行去学习和探索。（即便你们会用到它们，很可能也不会那么频繁。）

**展现国际化信息**

到现在为止，我们的 JSP 模板包含了很多硬编码的文本。这其实也算不上什么大问题，但是如果你要修改这些文本的话，就不那么容易了。而且，没有办法根据用户的语言设置国际化这些文本。

例如，考虑首页中的欢迎信息：

```html
<h1>Welcome to Spitter</h1>
```

修改这个信息的唯一办法是打开 home.jsp，然后对其进行变更。我觉得，这算不上什么大事。但是，应用中的文本散布到多个模板中，如果要大规模修改应用的信息时，你需要修改大量的 JSP 文件。

另外一个更为重要的问题在于，不管你选择什么样的欢迎信息，所有的用户都会看到同样的信息。Web 是全球性的网络，你所构建的应用很可能会有全球化用户。因此，最好能够使用用户的语言与其进行交流，而不是只使用某一种语言。

对于渲染文本来说，是很好的方案，文本能够位于一个或多个属性文件中。借助 `<s:message>`，我们可以将硬编码的欢迎信息替换为如下的形式：

```html
<h1><s:message code="spitter.welcome" /></h1>
```

按照这里的方式，`<s:message>` 将会根据 key 为 spittr.welcome 的信息源来渲染文本。因此，如果我们希望 `<s:message>` 能够正常完成任务的话，就需要配置一个这样的信息源。

Spring 有多个信息源的类，它们都实现了 MessageSource 接口。在这些类中，更为常见和有用的是 ResourceBundleMessageSource。它会从一个属性文件中加载信息，这个属性文件的名称是根据基础名称（base name）衍生而来的。如下的 @Bean 方法配置了 ResourceBundleMessage-Source：

```java
@Bean
public MessageSource messageSource() {
  ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
  messageSource.setBasename("messages");
  return messageSource;
}
```

在这个 bean 声明中，核心在于设置 basename 属性。你可以将其设置为任意你喜欢的值，在这里，我将其设置为  message。将其设置为 message 后，ResourceBundleMessageSource 就会试图在根路径的属性文件中解析信息，这些属性文件的名称是根据这个基础名称衍生得到的。

另外的可选方案是使用 ReloadableResourceBundleMessageSource，它的工作方式与 ResourceBundleMessageSource 非常类似，但是它能够重新加载信息属性，而不必重新编译或重启应用。如下是配置Reloadable-ResourceBundleMessageSource 的样例：

```java
@Bean
public MessageSource messageSource() {
  ReloadableResourceBundleMessageSource messageSource = new ReloadableResourceBundleMessageSource();
  messageSource.setBasename("file:///etc/spitter/messages");
  messageSource.setCacheSeconds(10);
  return messageSource;
}
```

这里的关键区别在于 basename 属性设置为在应用的外部查找（而不是像 ResourceBundleMessageSource 那样在类路径下查找）。basename 属性可以设置为在类路径下（以 `classpath:` 作为前缀）、文件系统中（以 `file:` 作为前缀）或 Web 应用的根路径下（没有前缀）查找属性。在这里，我将其配置为在服务器文件系统的 `/etc/spittr` 目录下的属性文件中查找信息，并且基础的文件名为 `message`。

现在，我们来创建这些属性文件。首先，创建默认的属性文件，名为 messages.properties。它要么位于根类路径下（如果使用Resource-BundleMessageSource 的话），要么位于 pathname 属性指定的路径下（如果使用 ReloadableResourceBundleMessageSource 的话）。对 spittr.welcome 信息来讲，它需要如下的条目：

```properties
spitter.welcome=Welcome to Spittr!
```

如果你不再创建其他信息文件的话，那么我们所做的事情就是将 JSP 中硬编码的信息抽取到了属性文件中，依然作为硬编码的信息。它能够让我们一站式地修改应用中的所有信息，但是它所完成的任务并不限于此。

我们已经具备了对信息进行国际化的重要组成部分。例如，如果你想要为语言设置为西班牙语的用户展现西班牙语的欢迎信息，那么需要创建另外一个名为 messages_es.properties 的属性文件，并包含如下的条目：

```properties
spittr.welcome=Bienvenidos a Spittr!
```

现在，我们已经完成了一件了不起的事情。我们的应用目前只是多了几个`<s:messae>` 标签以及语言相关的属性文件，还没有完全实现国际化！我将应用其他部分的国际化留给读者去完成。

**创建 URL**

`<s:url>` 是一个很小的标签。它主要的任务就是创建 URL，然后将其赋值给一个变量或者渲染到响应中。它是 JSTL 中 `<c:url>` 标签的替代者，但是它具备几项特殊的技巧。

按照其最简单的形式，`<s:url>` 会接受一个相对于 Servlet 上下文的 URL，并在渲染的时候，预先添加上 Servlet 上下文路径。例如，考虑如下 `<s:url>` 的基本用法：

```html
<a href="<s:url href="/spitter/register" />">Register</a>
```

如果应用的 Servlet 上下文名为 spittr，那么在响应中将会渲染如下的 HTML：

```html
<a href="/spitter/register">Register</a>
```

这样，我们在创建 URL 的时候，就不必再担心 Servlet 上下文路径是什么了，将会负责这件事。

另外，我们还可以使用创建 URL，并将其赋值给一个变量供模板在稍后使用：

```jsp
<s:url href="/spitter/register" var="registerUrl" />
<a href="${registerUrl}">Register</a>
```

默认情况下，URL 是在页面作用域内创建的。但是通过设置 scope 属性，我们可以让 `<s:url>` 在应用作用域内、会话作用域内或请求作用域内创建 URL：

```jsp
<s:url href="/spittr/register" var="registerUrl" scope="request" />
```

如果希望在 URL 上添加参数的话，那么你可以使用 `<s:param>` 标签。比如，如下的 `<s:url>` 使用两个内嵌的 `<s:param>` 标签，来设置 `/spittles` 的 max 和 count 参数：

```jsp
<s:url href="/spittles" var="spittlesUrl">
  <s:param name="max" value="60" />
  <s:param name="count" value="20" />
</s:url>
```

到目前为止，我们还没有看到 `<s:url>` 能够实现，而 JSTL 的 `<c:url>` 无法实现的功能。但是，如果我们需要创建带有路径（path）参数的 URL 该怎么办呢？我们该如何设置 href 属性，使其具有路径变量的占位符呢？

例如，假设我们需要为特定用户的基本信息页面创建一个 URL。那没有问题，`<s:param>` 标签可以承担此任：

```jsp
<s:url href="/spitter/{username}" var="spitterUrl">
  <s:param name="username" value="jbauer" />
</s:url>
```

当 href 属性中的占位符匹配 `<s:param>` 中所指定的参数时，这个参数将会插入到占位符的位置中。如果 `<s:param>` 参数无法匹配 href 中的任何占位符，那么这个参数将会作为查询参数。

`<s:url>` 标签还可以解决 URL 的转义需求。例如，如果你希望将渲染得到的 URL 内容展现在 Web 页面上（而不是作为超链接），那么你应该要求`<s:url>` 进行 HTML 转义，这需要将 htmlEscape 属性设置为 true。

例如，如下的将会渲染 HTML 转义后的 URL：

```html
<s:url value="spittles" htmlEscape="true">
  <s:param name="max" value="60" />
  <s:param name="count" value="20" />
</s:url>
```

所渲染的 URL 结果如下所示：

```
/spitter/spittles?max=60&amp;count=20
```

另一方面，如果你希望在 JavaScript 代码中使用 URL 的话，那么应该将 javaScriptEscape 属性设置为 true：

```jsp
<s:url value="spittles" var="spittlesJSUrl" javascriptEscape="true">
  <s:param name="max" value="60" />
  <s:param name="count" value="20" />
</s:url>
<script>
  var spittleUrl="${spittlesJSUrl}"
</script>
```

这会渲染如下的结果到响应之中：

```jsp
<script>
  var spittleUrl="\/spitter\/spittles?max=60&count=20"
</script>
```

既然提到了转义，有一个标签专门用来转义内容，而不是转义标签。 接下来，让我们看一下。

**转义内容**

`<s:escapeBody>` 标签是一个通用的转义标签。它会渲染标签体中内嵌的内容，并且在必要的时候进行转义。

例如，假设你希望在页面上展现一个 HTML 代码片段。为了正确显示，我们需要将 `<` 和` >` 字符替换为 `<` 和 `>`，否则的话，浏览器将会像解析页面上其他 HTML 那样解析这段 HTML 内容。

当然，没有人禁止我们手动将其转义为 `<` 和 `>`，但是这很烦琐，并且代码难以阅读。我们可以使用，并让 Spring 完成这项任务：

```jsp
<s:escapeBody htmlEscape="true">
<h1>Hello</h1>
</s:escapeBody>
```

它将会在响应体中渲染成如下的内容：

```jsp
&lt;h1&gt;Hello&lt;/h1&gt;
```

虽然转义后的格式看起来很难读，但浏览器会很乐意将其转换为未转义的 HTML，也就是我们希望用户能够看到的样子。

通过设置 javaScriptEscape 属性，`<s:escapeBody>` 标签还支持 JavaScript 转义：

```jsp
<s:escapeBody javascriptEscape="true">
<h1>Hello</h1>
</s:escapeBody>
```

`<s:escapeBody>` 只完成一件事，并且完成得非常好。与 `<s:url>` 不同，它只会渲染内容，并不能将内容设置为变量。

现在，我们已经看到了如何使用 JSP 来定义 Spring 视图，现在让我们考虑一下如何使其在审美上更加有吸引力。我们可以在页面上增加一些通用的元素，比如添加包含站点 Logo 的头部、使用样式并在底部展现版权信息。我们不会在 Spittr 应用中的每个 JSP 都进行这样的修改，而是借助 Apache Tiles 来为模板实现一些通用且可重用的布局。

## 使用Thymeleaf

尽管 JSP 已经存在了很长的时间，并且在 Java Web 服务器中无处不在，但是它却存在一些缺陷。JSP 最明显的问题在于它看起来像 HTML 或 XML，但它其实上并不是。大多数的 JSP 模板都是采用 HTML 的形式，但是又掺杂上了各种 JSP 标签库的标签，使其变得很混乱。这些标签库能够以很便利的方式为 JSP 带来动态渲染的强大功能，但是它也摧毁了我们想维持一个格式良好的文档的可能性。作为一个极端的样例，如下的 JSP 标签甚至作为 HTML 参数的值：

```jsp
<input type="text" value="<c:out value="${thing.name}" />" />
```

标签库和 JSP 缺乏良好格式的一个副作用就是它很少能够与其产生的 HTML 类似。所以，在 Web 浏览器或 HTML 编辑器中查看未经渲染的 JSP 模板是非常令人困惑的，而且得到的结果看上去也非常丑陋。这个结果是不完整的 —— 在视觉上这简直就是一场灾难！因为 JSP 并不是真正的 HTML，很多浏览器和编辑器展现的效果都很难在审美上接近模板最终所渲染出来的效果。

同时，JSP 规范是与 Servlet 规范紧密耦合的。这意味着它只能用在基于 Servlet 的 Web 应用之中。JSP 模板不能作为通用的模板（如格式化  Email），也不能用于非 Servlet 的 Web 应用。

多年来，在 Java 应用中，有多个项目试图挑战 JSP 在视图领域的统治性地位。最新的挑战者是 Thymeleaf，它展现了一些切实的承诺，是一项很令人兴奋的可选方案。Thymeleaf 模板是原生的，不依赖于标签库。它能在接受原始 HTML 的地方进行编辑和渲染。因为它没有与 Servlet 规范耦合，因此 Thymeleaf 模板能够进入 JSP 所无法涉足的领域。现在，我们看一下如何在 Spring MVC 中使用 Thymeleaf。

### **配置 Thymeleaf 视图解析器**

为了要在 Spring 中使用 Thymeleaf，我们需要配置三个启用 Thymeleaf 与 Spring 集成的 bean：

- ThymeleafViewResolver：将逻辑视图名称解析为 Thymeleaf 模板视图；
- SpringTemplateEngine：处理模板并渲染结果；
- TemplateResolver：加载 Thymeleaf 模板。

如下为声明这些 bean 的 Java 配置。

```java
@Bean
public ViewResolver viewResolver(SpringTemplateEngine templateEngine) {
  ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
  viewResolver.setTemplateEngine(templateEngine);
  return viewResolver;
}

@Bean
public SpringTemplateEngine templateEngine(TemplateResolver templateResolver) {
  SpringTemplateEngine templateEngine = new SpringTemplateEngine();
  templateEngine.setTemplateResolver(templateResolver);
  return templateEngine;
}

@Bean
public TemplateResolver templateResolver() {
  TemplateResolver templateResolver = new ServletContextTemplateResolver();
  templateResolver.setPrefix("/WEB-INF/views/");
  templateResolver.setSuffix(".html");
  templateResolver.setTemplateMode("HTML5");
  return templateResolver;
} 
```

如果你更愿意使用 XML 来配置 bean，那么如下的声明能够完成该任务。

```xml
<bean id="viewResolver" class="org.thymeleaf.spring3.view.ThymeleafViewResolver"
      p:templateEngine-ref="templateEngine" />

<bean id="templateEngine" class="org.thymeleaf.spring3.SpringTemplateEngine"
      p:templateResolver-ref="templateResolver" />
      
      
<bean id="templateResolver" class="org.thymeleaf.templateResolver.ServletContextTemplateResolver"
      p:prefix="/WEB-INF/templates/"
      p:suffix=".html"
      p:templateMode="HTML5" />
```

不管使用哪种配置方式，Thymeleaf 都已经准备就绪了，它可以将响应中的模板渲染到 Spring MVC 控制器所处理的请求中。

ThymeleafViewResolver 是 Spring MVC 中 ViewResolver 的一个 实现类。像其他的视图解析器一样，它会接受一个逻辑视图名称，并将其解析为视图。不过在该场景下，视图会是一个 Thymeleaf 模板。

需要注意的是 ThymeleafViewResolver bean 中注入了一个对 Spring-TemplateEngine bean的引用。SpringTemplateEngine 会在 Spring 中启用 Thymeleaf 引擎，用来解析模板，并基于这些模板渲染结果。可以看到，我们为其注入了一个 TemplateResolver bean 的引用。

TemplateResolver 会最终定位和查找模板。与之前配置 InternalResource-ViewResolver 类似，它使用了 prefix 和 suffix 属性。前缀和后缀将会与逻辑视图名组合使用，进而定位 Thymeleaf 引擎。它的 templateMode 属性被设置成了 HTML 5，这表明我们预期要解析的模板会渲染成 HTML 5输出。

所有的 Thymeleaf bean 都已经配置完成了，那么接下来我们该创建几个视图了。

### **定义 Thymeleaf 模板**

Thymeleaf 在很大程度上就是 HTML 文件，与 JSP不同，它没有什么特 殊的标签或标签库。Thymeleaf 之所以能够发挥作用，是因为它通过自定义的命名空间，为标准的 HTML 标签集合添加 Thymeleaf 属性。如下的程序清单展现了 home.html，也就是使用 Thymeleaf 命名空间的首页模板。

```html
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org">
  <head>
    <title>Spitter</title>
    <link rel="stylesheet" 
          type="text/css" 
          th:href="@{/resources/style.css}"></link>
  </head>
  <body>
    <div id="content">
      <h1>Welcome to Spitter</h1>
  
      <a th:href="@{/spittles}">Spittles</a> | 
      <a th:href="@{/spitter/register}">Register</a>
  </body>
</html>
```

首页模板相对来讲很简单，只使用了 th:href 属性。这个属性与对应的原生 HTML 属性很类似，也就是 href 属性，并且可以按照相同的方式来使用。th:href 属性的特殊之处在于它的值中可以包含 Thymeleaf 表达式，用来计算动态的值。它会渲染成一个标准的 href 属性，其中会包含在渲染时动态创建得到的值。这是 Thymeleaf 命名空间中很多属性的运行方式：它们对应标准的 HTML 属性，并且具有相同的名称，但是会渲染一些计算后得到的值。在本例中，使用 th:href 属性的三个地方都用到了 `@{}` 表达式，用来计算相对于 URL 的路径（就像在 JSP 页面中，我们可能会使用的 JSTL `<c:url>` 标签或 Spring 标签类似）。

尽管 home.html 是一个相当简单的 Thymeleaf 模板，但是它依然很有价值，这在于它与纯 HTML 模板非常接近。唯一的区别之处在于 th:href 属性，否则的话，它就是基础且功能丰富的 HTML 文件。

这意味着 Thymeleaf 模板与 JSP 不同，它能够按照原始的方式进行编辑甚至渲染，而不必经过任何类型的处理器。当然，我们需要 Thymeleaf 来处理模板并渲染得到最终期望的输出。即便如此，如果没有任何特殊的处理，home.html 也能够加载到 Web 浏览器中，并且看上去与完整渲染的效果很类似。为了更加清晰地阐述这一点，图 6.6 对比了 home.jsp（上方）和 home.html（下方）在 Web 浏览器中的显示效果。

可以看到，在 Web 浏览器中，JSP 模板的渲染效果很糟糕。尽管我们可以看到一些熟悉的元素，但是 JSP 标签库的声明也显示了出来。在链接前出现了一些令人费解的未闭合标记，这是 Web 浏览器没有正常解析 `<s:url>` 标签的结果。

与之相反，Thymeleaf 模板的渲染效果基本上没有任何错误。稍微有点问题的是链接部分，Web 浏览器并不会像处理 href 属性那样处理 th:href，所以链接并没有渲染为链接的样子。除了这些细微的问题，模板的渲染效果与我们的预期完全符合。

像 home.jsp 这样的模板作为 Thymeleaf 入门是很合适的。但是 Spring 的 JSP 标签所擅长的是表单绑定。如果我们抛弃 JSP 的话，那是不是也要抛弃表单绑定呢？不必担心。Thymeleaf 提供了与之相匹敌的功能。

**借助 Thymeleaf 实现表单绑定**

表单绑定是 Spring MVC 的一项重要特性。它能够将表单提交的数据填充到命令对象中，并将其传递给控制器，而在展现表单的时候，表单中也会填充命令对象中的值。如果没有表单绑定功能的话，我们需要确保 HTML 表单域要映射后端命令对象中的属性，并且在校验失败后展现表单的时候，还要负责确保输入域中值要设置为命令对象的属性。

但是，如果有表单绑定的话，它就会负责这些事情了。为了复习一下表单绑定是如何运行的，下面展现了在 registration.jsp 中的 First Name 输入域：

```jsp
<sf:label path="firstName" cssErrorClass="error">First Name</sf:label>
<sf:input path="firstName" cassErrorClass="error" /><br/> 
```

在这里，调用了 Spring 表单绑定标签库的 `<sf:input>` 标签，它会渲染出一个 HTML input 标签，并且其 value 属性设置为后端对象 firstName 属性的值。它还使用了 Spring 的 `<sf:label>` 标签及其 cssErrorClass 属性，如果出现校验错误的话，会将文本标记渲染为红色。

但是，我们本节讨论的并不是 JSP，而是使用 Thymeleaf 替换 JSP。因此，我们不能使用 Spring 的 JSP 标签实现表单绑定，而是使用 Thymeleaf 的 Spring 方言。

作为阐述的样例，请参考如下的 Thymeleaf 模板片段，它会渲染 First Name 输入域：

```jsp
<label th:class="${#fields.hasErrors['firstName']}?'error'">First Name</label>
<input type="text" th:field="*{firstName}" th:class="${#fields.hasErrors['firstName']}?'error'" /><br/>
```

在这里，我们不再使用 Spring JSP 标签中的 cssClassName 属性，而是在标准的 HTML 标签上使用 th:class 属性。th:class 属性会渲染为一个 class 属性，它的值是根据给定的表达式计算得到的。在上面的这两个 th:class 属性中，它会直接检查 firstName 域有没有校验错误。如果有的话，class 属性在渲染时的值为 error。如果这个域没有错误的话，将不会渲染 class 属性。

<input> 标签使用了 th:field 属性，用来引用后端对象的 firstName 域。这可能与你的预期有点差别。在 Thymeleaf 模板中， 我们在很多情况下所使用的属性都对应于标准的 HTML 属性，因此貌似使用 th:value 属性来设置标签的 value 属性才是合理的。

其实不然，因为我们是在将这个输入域绑定到后端对象的 firstName 属性上，因此使用 th:field 属性引用 firstName 域。通过使用 th:field，我们将 value 属性设置为 firstName 的值， 同时还会将 name 属性设置为 firstName。

为了阐述 Thymeleaf 是如何实际运行的，如下的程序清单展示了完整的注册表单模板。

```jsp
<form method="POST" th:object="${spitter}">
  <div class="errors" th:if="${#fields.hasErrors('*')}">
    <ul>
      <li th:each="err : ${#fields.errors('*')}" 
          th:text="${err}">Input is incorrect
      </li>
    </ul>
  </div>
  <label th:class="${#fields.hasErrors('firstName')}? 'error'">First Name</label>: 
  <input type="text" th:field="*{firstName}"  th:class="${#fields.hasErrors('firstName')}? 'error'" /><br/>
  
  <label th:class="${#fields.hasErrors('lastName')}? 'error'">Last Name</label>: 
  <input type="text" th:field="*{lastName}" th:class="${#fields.hasErrors('lastName')}? 'error'" /><br/>
  
  <label th:class="${#fields.hasErrors('email')}? 'error'">Email</label>: 
  <input type="text" th:field="*{email}" th:class="${#fields.hasErrors('email')}? 'error'" /><br/>
  
  <label th:class="${#fields.hasErrors('username')}? 'error'">Username</label>: 
  <input type="text" th:field="*{username}" th:class="${#fields.hasErrors('username')}? 'error'" /><br/>
  
  <label th:class="${#fields.hasErrors('password')}? 'error'">Password</label>: 
  <input type="password" th:field="*{password}" th:class="${#fields.hasErrors('password')}? 'error'" /><br/>

  <input type="submit" value="Register" />
</form>
```

程序清单 6.7 使用了相同的 Thymeleaf 属性和 `*{}` 表达式，为所有的表单域绑定后端对象。这其实重复了我们在 First Name 域中所做的事情。

但是，需要注意我们在表单的顶部了也使用了 Thymeleaf，它会用来渲染所有的错误。元素使用 th:if 属性来检查是否有校验错误。如果有的话，会渲染，否则的话，它将不会渲染。

在 <div> 中，会使用一个无顺序的列表来展现每项错误。<li> 标签上的 th:each 属性将会通知 Thymeleaf 为每项错误都渲染一个 <li>，在每次迭代中会将当前错误设置到一个名为 err 的变量中。

<li> 标签还有一个 th:text 属性。这个命令会通知 Thymeleaf 计算某一个表达式（在本例中，也就是 err 变量）并将它的值渲染为 <li> 标签的内容体。实际上的效果就是每项错误对应一个 <li> 元素，并展现错误的文本。

你可能会想知道 `${}` 和 `*{}` 括起来的表达式到底有什么区别。`${}` 表达式（如 `${spitter}`）是变量表达式（variable expression）。一般来讲，它们会是[对象图导航语言（Object-Graph Navigation Language，OGNL）表达式](http://commons.apache.org/proper/commons-ognl/) 。但在使用 Spring 的时候，它们是 SpEL 表达式。在 `${spitter}` 这个例子中，它会解析为 key 为 spitter 的 model 属性。

而对于 `*{}` 表达式，它们是选择表达式（selection expression）。变量表达式是基于整个 SpEL 上下文计算的，而选择表达式是基于某一个选中对象计算的。在本例的表单中，选中对象就是 <form> 标签中 th:object 属性所设置的对象：模型中的 Spitter 对象。因此，`*{firstName}` 表达式就会计算为 Spitter 对象的 firstName 属性。

## 小结

处理请求只是 Spring MVC 功能的一部分。如果控制器所产生的结果想要让人看到，那么它们产生的模型数据就要渲染到视图中，并展现到用户的 Web 浏览器中。Spring 的视图渲染是很灵活的，并提供了多个内置的可选方案，包括传统的 JavaServer Pages 以及流行的 Apache Tiles 布局引擎。

在本章中，我们首先快速了解了一下 Spring 所提供的视图和视图解析可选方案。我们还深入学习了如何在 Spring MVC 中使用 JSP 和 Apache Tiles。

我们还看到了如何使用 Thymeleaf 作为 Spring MVC 应用的视图层，它被视为 JSP 的替代方案。Thymeleaf 是一项很有吸引力的技术，因为它能创建原始的模板，这些模板是纯 HTML，能像静态 HTML 那样以原始的方式编写和预览，并且能够在运行时渲染动态模型数据。除此之外，Thymeleaf 是与 Servlet 没有耦合关系的，这样它就能够用在 JSP 所不能使用的领域中。

Spittr 应用的视图定义完成之后，我们已经具有了一个虽然微小但是可部署且具有一定功能的 Spring MVC Web 应用。还有一些其他的特性需要更新进来，如数据持久化和安全性，我们会在合适的时候关注这些特性。但现在，这个应用开始变得有模有样了。

在深入学习应用的技术栈之前，在下一章我们将会继续讨论 Spring MVC，学习这个框架中一些更为有用和高级的功能。
