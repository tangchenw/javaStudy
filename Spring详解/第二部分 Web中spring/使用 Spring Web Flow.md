# **使用 Spring Web Flow**

本章内容：

- 创建会话式的 Web 应用程序
- 定义流程状态和行为
- 保护 Web 流程

关于互联网，很奇妙的一件事就是它很容易让你迷失。有如此之多的内容可以查看和阅读，而超链接是互联网强大魔力的核心。无怪乎将其称为网，正如蜘蛛织出的网，它会将经过的任何东西困住。我必须承认：之所以在编写此书时花费了如此多的时间，其中的一个原因就是我曾经迷失在维基百科无休无止的链接之中。

有时候，Web 应用程序需要控制网络冲浪者的方向，引导他们一步步地访问应用。比较典型的例子就是电子商务站点的结账流程，从购物车开始，应用程序会引导你依次经过派送详情、账单信息以及最终的订单确认流程。

Spring Web Flow 是一个 Web 框架，它适用于元素按规定流程运行的程序。在本章中，我们将会探索 Spring Web Flow 并了解它如何应用于 Spring Web 框架平台。

其实我们可以使用任何 Web 框架编写流程化的应用程序。我曾经看到过一个应用程序，在 Struts 中构建了特定的流程。但是这样就没有办法将流程与实现分开了，你会发现流程的定义分散在组成流程的各个元素中。没有地方能够完整地描述整个流程。

Spring Web Flow 是 Spring MVC 的扩展，它支持开发基于流程的应用程序。它将流程的定义与实现流程行为的类和视图分离开来。

在介绍 Spring Web Flow 的时候，我们将暂时放下 Spittr 样例并使用生成披萨订单的新 Web 应用程序。我们会使用 Spring Web Flow 来定义订单流程。

使用 Spring Web Flow 的第一步是在项目中安装它。让我们从这里开始吧。

## **在 Spring 中配置 Web Flow**

Spring Web Flow 是构建于 Spring MVC 基础之上的。这意味着所有的流程请求都需要首先经过 Spring MVC 的 DispatcherServlet。我们需要在 Spring 应用上下文中配置一些 bean 来处理流程请求并执行流程。

现在，还不支持在 Java 中配置 Spring Web Flow，所以我们别无选择，只能在 XML 中对其进行配置。有一些 bean 会使用 Spring Web Flow 的 Spring 配置文件命名空间来进行声明。因此，我们需要在上下文定义 XML 文件中添加这个命名空间声明：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xmlns:flow="http://www.springframework.org/schema/webflow-config"
 xsi:schemaLocation="http://www.springframework.org/schema/webflow-config 
   http://www.springframework.org/schema/webflow-config/spring-webflow-config-2.3.xsd
   http://www.springframework.org/schema/beans 
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

</beans>
```

在声明了命名空间之后，我们就为装配 Web Flow 的 bean 做好了准备，让我们从流程执行器（flow executor）开始吧。

### **装配流程执行器**

正如其名字所示，流程执行器（flow executor）驱动流程的执行。当用户进入一个流程时，流程执行器会为用户创建并启动一个流程执行实例。当流程暂停的时候（如为用户展示视图时），流程执行器会在用户执行操作后恢复流程。

在 Spring 中，`<flow:flow-executor>` 元素会创建一个流程执行器：

```xml
<flow:flow-executor id="flowExecutor" />
```

尽管流程执行器负责创建和执行流程，但它并不负责加载流程定义。这个责任落在了流程注册表（flow registry）身上，接下来我们会创建它。

### **配置流程注册表**

流程注册表（flow registry）的工作是加载流程定义并让流程执行器能够使用它们。我们可以在 Spring 中使用 `<flow:flow-registry>` 配置流程注册表，如下所示：

```xml
<flow:flow-registry id="flowRegistry" base-path="/WEB-INF/flows">
  <flow:flow-location-pattern value="*-flow.xml" />
</flow:flow-registry>
```

在这里的声明中，流程注册表会在 `/WEB-INF/flows` 目录下查找流程定义，这是通过 base-path 属性指明的。依据 `<flow:flow-location-pattern>` 元素的值，任何文件名以 `-flow.xml` 结尾的 XML 文件都将视为流程定义。

所有的流程都是通过其 ID 来进行引用的。这里我们使用了 `<flow:flow-location-pattern>` 元素，流程的 ID 就是相对于 base-path 的路径 —— 或者双星号所代表的路径。图 8.1 展示了示例中的流程 ID 是如何计算的。

![8.1 流程定位](C:\Users\汤琛\Desktop\学习资料\Spring详解\images\8.1 流程定位.jpg)

作为另一种方式，我们可以去除 base-path 属性，而显式声明流程定义文件的位置：

```xml
<flow:flow-registry id="flowRegistry">
  <flow:flow-location-pattern path="/WEB-INF/flows/springpizza.xml" />
</flow:flow-registry>
```

在这里，使用了而不是，path 属性直接指明了 `/WEB-INF/flows/springpizza.xml` 作为流程定义。当我们这样配置的话，流程的 ID 是从流程定义文件的文件名中获得的，在这里就是 springpizza。

如果你希望更显式地指定流程 ID，那你可以通过元素的 id 属性来进行设置。例如，要将 pizza 作为流程 ID，可以像这样配置：

```xml
<flow:flow-registry id="flowRegistry">
  <flow:flow-location id="pizza" path="/WEB-INF/flows/springpizza.xml" />
</flow:flow-registry>
```

### **处理流程请求**

我们在前一章曾经看到，DispatcherServlet 一般将请求分发给控制器。但是对于流程而言，我们需要一个 FlowHandlerMapping 来帮助 DispatcherServlet 将流程请求发送给 Spring Web Flow。在 Spring 应用上下文中，FlowHandlerMapping 的配置如下：

```xml
<bean class="org.springframework.webflow.mvc.servlet.FlowHandlerMapping">
  <property name="flowRegistry" ref="flowRegistry" />
</bean>
```

你可以看到，FlowHandlerMapping 装配了流程注册表的引用，这样它就能知道如何将请求的 URL 匹配到流程上。例如，如果我们有一 个 ID 为 pizza 的流程，FlowHandlerMapping 就会知道如果请求的 URL 模式（相对于应用程序的上下文路径）是 `/pizza` 的话，就要将其匹配到这个流程上。

然而，FlowHandlerMapping 的工作仅仅是将流程请求定向到 Spring Web Flow 上，响应请求的是 FlowHandlerAdapter。FlowHandlerAdapter 等同于 Spring MVC 的控制器，它会响应发送的流程请求并对其进行处理。FlowHandler-Adapter 可以像下面这样装配成一个 Spring bean，如下所示：

```xml
<bean class="org.springframework.webflow.mvc.servlet.FlowHandlerAdapter">
  <property name="flowExecutor" ref="flowExecutor" />
</bean>
```

这个处理适配器是 DispatcherServlet 和 Spring Web Flow 之间的桥梁。它会处理流程请求并管理基于这些请求的流程。在这里，它装配了流程执行器的引用，而后者是为所处理的请求执行流程的。

我们已经配置了 Spring Web Flow 所需的 bean 和组件。剩下就是真正定义流程了。我们随后将会进行这项工作。但首先，让我们先了解一下组成流程的元素。

## **流程的组件**

在 Spring Web Flow 中，流程是由三个主要元素定义的：状态、转移和流程数据。状态（State）是流程中事件发生的地点。如果你将流程想象成公路旅行，那状态就是路途上的城镇、路边饭店以及风景点。流程中的状态是业务逻辑执行、做出决策或将页面展现给用户的地方， 而不是在公路旅行中买 Doritos 薯片和健怡可乐的所在。

如果流程状态就像公路旅行中停下来的地点，那转移（transition）就是连接这些点的公路。在流程中，你通过转移的方式从一个状态到另 一个状态。

当你在城镇之间旅行的时候，你可能要买一些纪念品，留下一些记忆并在路上取一些空的零食袋。类似地，在流程处理中，它要收集一些数据：流程的当前状况。我很想将其称为流程的状态，但是在我们讨论流程的时候状态（state）已经有了另外的含义。

让我们仔细看一下在 Spring Web Flow 中这三个元素是如何定义的。

### **状态**

Spring Web Flow 定义了五种不同类型的状态，如表 8.1 所示。通过选择 Spring Web Flow 的状态几乎可以把任意的安排功能构造成会话式的 Web 应用。尽管并不是所有的流程都需要表 8.1 所描述的状态，但最终你可能会经常使用它们中的大多数。

|      状态类型      |                       它是用来做什么的                       |
| :----------------: | :----------------------------------------------------------: |
|   行为（Action）   |                 行为状态是流程逻辑发生的地方                 |
| 决策 （Decision）  | 决策状态将流程分成两个方向，它会基于流程数据的评估结果确定流程方向 |
|    结束（End）     |   结束状态是流程的最后一站。一旦进入End状态，流程就会终止    |
| 子流程 （Subflow） |   子流程状态会在当前正在运行的流程上下文中启动一个新的流程   |
|    视图（View）    |             视图状态会暂停流程并邀请用户参与流程             |

稍后我们将会看到如何将这些不同类型的状态组合起来形成一个完整的流程。但首先，让我们了解一下这些流程元素在 Spring Web Flow 定义中是如何表现的。

**视图状态**

视图状态用于为用户展现信息并使用户在流程中发挥作用。实际的视图实现可以是 Spring 支持的任意视图类型，但通常是用 JSP 来实现的。在流程定义的 XML 文件中，`<view-state>` 用于定义视图状态：

```xml
<view-state id="welcome" />
```

在这个简单的示例中，id 属性有两个含义。它在流程内标示这个状态。除此以外，因为在这里没有在其他地方指定视图，所以它也指定了流程到达这个状态时要展现的逻辑视图名为 welcome。

如果你愿意显式指定另外一个视图名，那可以使用 view 属性做到这一点：

```xml
<view-state id="welcome" view="greeting" />
```

如果流程为用户展现了一个表单，你可能希望指明表单所绑定的对象。为了做到这一点，可以设置 model 属性：

```xml
<view-state id="takePayment" model="flowScope.paymentDetails" />
```

这里我们指定 takePayment 视图中的表单将绑定流程作用域内的 payment-Details 对象。（稍后，我们将会更详细地介绍流程作用域和数据。）

**行为状态**

视图状态会涉及到流程应用程序的用户，而行为状态则是应用程序自身在执行任务。行为状态一般会触发 Spring 所管理 bean 的一些方法并根据方法调用的执行结果转移到另一个状态。

在流程定义 XML 中，行为状态使用 `<action-state>` 元素来声明。这里是一个例子：

```xml
<action-state id="saveOrder">
  <evaluate expression="pizzaFlowActions.saveOrder(order)" />
  <transition to="thankYou" />
</action-state>
```

尽管不是严格需要的，但是 `<action-state>` 元素一般都会有一个 `<evaluate>` 作为子元素。`<evalue>` 元素给出了行为状态要做的事情。expression 属性指定了进入这个状态时要评估的表达式。在本示例中，给出的 expression 是 SpEL 表达式，它表明将会找到 ID 为 pizzaFlowActions 的 bean 并调用其 saveOrder() 方法。

**Spring Web Flow 与表达式语言**

在这几年以来，Spring Web Flow 在选择的表达式语言方面，经过了一些变化。在 1.0 版本的时候，Spring Web Flow 使用的是对象图导航语言（Object-Graph Navigation Language，OGNL）。随后的 2.0 版本又换成了统一表达式语言（Unified Expression Language，Unified EL）。在 2.1 版本中，Spring Web Flow 使用的是 SpEL。

尽管可以使用上述的任意表达式语言来配置 Spring Web Flow，但 SpEL 是默认和推荐使用的表达式语言。因此，当定义流程的时候，我们会选择使用 SpEL，忽略掉其他的可选方案。

**决策状态**

有可能流程会完全按照线性执行，从一个状态进入另一个状态，没有其他的替代路线。但是更常见的情况是流程在某一个点根据流程的当前情况进入不同的分支。

决策状态能够在流程执行时产生两个分支。决策状态将评估一个 Boolean 类型的表达式，然后在两个状态转移中选择一个，这要取决于表达式会计算出 true 还是 false。在 XML 流程定义中，决策状态通过 `<decision-state>` 元素进行定义。典型的决策状态示例如下所示：

```xml
<decision-state id="checkDeliveryArea">
  <id test="pizzaFlowActions.checkDeliveryArea(customer.zipCode)"
      then="addCustomer"
      else="deliveryWarning" />
</decision-state>
```

你可以看到，`<decision-state>` 并不是独立完成工作的。`<if>` 元素是决策状态的核心。这是表达式进行评估的地方，如果表达式结果为 true，流程将转移到 then 属性指定的状态中，如果结果为 false，流程将会转移到 else 属性指定的状态中。

**子流程状态**

你可能不会将应用程序的所有逻辑写在一个方法中，而是将其分散到多个类、方法以及其他结构中。

同样，将流程分成独立的部分是个不错的主意。允许在一个正在执行的流程中调用另一个流程。这类似于在一个方法中调用另一个方法。

`<suflow-state>` 可以这样声明：

```xml
<subflow-state id="order" subflow="pizza/order">
  <input name="order" value="order" />
  <transition on="orderCreated" to="payment" />
</subflow-state>
```

在这里，<input> 元素用于传递订单对象作为子流程的输入。如果子流程结束的 `<end-state>` 状态 ID 为 orderCreated，那么流程将会转移到名为 payment 的状态。

在这里，我有点超出进度了，我们还没有讨论到 `<end-state>` 元素和转移。我们很快就会在 8.2.2 小节介绍转移。对于结束状态，这正是接下来要介绍的。

**结束状态**

最后，所有的流程都要结束。这就是当流程转移到结束状态时所做的。`<end-state>` 元素指定了流程的结束，它一般会是这样声明的：

```xml
<end-state id="customerReady" />
```

当到达 `<end-state>` 状态，流程会结束。接下来会发生什么取决于几个因素：

- 如果结束的流程是一个子流程，那调用它的流程将会从 `<subflow-state>` 处继续执行。`<end-state>` 的 ID 将会用作事件触发从 `<subflow-state>` 开始的转移。
- 如果 `<end-state>` 设置了 view 属性，指定的视图将会被渲染。视图可以是相对于流程路径的视图模板，如果添加 `externalRedirect:` 前缀的话，将会重定向到流程外部的页面，如果添加 `flowRedirect:` 将重定向到另一个流程中。
- 如果结束的流程不是子流程，也没有指定 view 属性，那这个流程只是会结束而已。浏览器最后将会加载流程的基本 URL 地址， 当前已没有活动的流程，所以会开始一个新的流程实例。

需要意识到流程可能会有不止一个结束状态。子流程的结束状态 ID 确定了激活的事件，所以你可能会希望通过多种结束状态来结束子流程，从而能够在调用流程中触发不同的事件。即使不是在子流程中，也有可能在结束流程后，根据流程的执行情况有多个显示页面供选择。

现在，已经看完了流程中的各个状态，我们应当看一下流程是如何在状态间迁移的。让我们看看如何在流程中通过定义转移来完成道路铺设的。

### **转移**

正如我在前面所提到的，转移连接了流程中的状态。流程中除结束状态之外的每个状态，至少都需要一个转移，这样就能够知道一旦这个状态完成时流程要去向哪里。状态可以有多个转移，分别对应于当前状态结束时可以执行的不同的路径。

转移使用 `<transition>` 元素来进行定义，它会作为各种状态元素（`<action-state>`、`<view-state>` 、`<subflow-state>`）的子元素。最简单的形式就是元素在流程中指定下一个状态：

```xml
<transition to="customerReady" />
```

属性 to 用于指定流程的下一个状态。如果只使用了 to 属性，那这个转移就会是当前状态的默认转移选项，如果没有其他可用转移的话，就会使用它。

更常见的转移定义是基于事件的触发来进行的。在视图状态，事件通常会是用户采取的动作。在行为状态，事件是评估表达式得到的结果。而在子流程状态，事件取决于子流程结束状态的 ID。在任意的事 件中（这里没有任何歧义），你可以使用 on 属性来指定触发转移的事件：

```xml
<transition on="phoneEntered" to="lookupCustomer" />
```

在本例中，如果触发了 phoneEntered 事件，流程将会进入 lookupCustomer 状态。

在抛出异常时，流程也可以进入另一个状态。例如，如果顾客的记录没有找到，你可能希望流程转移到一个展现注册表单的视图状态。以下的代码片段显示了这种类型的转移：

```xml
<transition on-exception="com.springinaction.pizza.service.CustomerNotFoundException"
            to="registraction" />
```

属性 on-exception 类似于 on 属性，只不过它指定了要发生转移的异常而不是一个事件。在本示例中，CustomerNotFoundException 异常将导致流程转移到 registrationForm 状态。

**全局转移**

在创建完流程之后，你可能会发现有一些状态使用了一些通用的转移。例如，如果在整个流程中到处都有如下的 `<transition>` 话， 我一点也不感觉意外：

```xml
<transition on="cancel" to="endState" />
```

与其在多个状态中都重复通用的转移，我们可以将 `<transition>` 元素作为的子元素，把它们定义为全局转移。例如：

```xml
<global-transitions>
  <transition on="cancel" to="endState" />
</global-transitions>
```

定义完这个全局转移后，流程中的所有状态都会默认拥有这个 cancel 转移。

我们已经讨论过了状态和转移。在我们开始编写流程之前，让我们看一下流程数据，这是 Web 流程三元素中的另一个成员。

### **流程数据**

如果你曾经玩过那种老式的基于文字的冒险游戏的话，那么当从一个地方转移到另一个地方时，你会偶尔发现散布在周围的一些东西，你可以把它们捡起来并带上。有时候，你会马上需要一件东西。其他的时候，你会在整个游戏过程中带着这些东西而不知道它们是做什么用的 —— 直到你到达游戏结束的时候才会发现它是真正有用的。

在很多方面，流程与这些冒险游戏是很类似的。当流程从一个状态进行到另一个状态时，它会带走一些数据。有时候，这些数据只需要很短的时间（可能只要展现页面给用户）。有时候，这些数据会在整个流程中传递并在流程结束的时候使用。

**声明变量**

流程数据保存在变量中，而变量可以在流程的各个地方进行引用。它能够以多种方式创建。在流程中创建变量的最简单形式是使用 `<var>` 元素：

```xml
<var name="customer" class="com.springinaction.pizza.domain.Customer" />
```

这里，创建了一个新的 Customer 实例并将其放在名为 customer 的变量中。这个变量可以在流程的任意状态进行访问。

作为行为状态的一部分或者作为视图状态的入口，你有可能会使用 `<evaluate>` 元素来创建变量。例如：

```xml
<evaluate result="viewScope.topppingsList" expression="T(com.springinaction.pizza.domain.Topping).asList()" />
```

在本例中，`<evaluate>` 元素计算了一个表达式（SpEL 表达式）并将结果放到了名为 toppingsList 的变量中，这个变量是视图作用域的（我们将会在稍后介绍关于作用域的更多概念）。

类似地，`<set>` 元素也可以设置变量的值：

```xml
<set name="flowScope.pizza" value="new com.springinaction.pizza.domain.Pizza()" />
```

`<set>` 元素与 `<evaluate>` 元素很类似，都是将变量设置为表达式计算的结果。这里，我们设置了一个流程作用域内的 pizza 变量，它的值是 Pizza 对象的新实例。

当我们在 8.3 小节开始构建真实工作的 Web 流程时，你会看到这些元素是如何具体应用在实际流程中的。但首先，让我们看一下变量的流程作用域、视图作用域以及其他的一些作用域是什么意思。

**定义流程数据的作用域**

流程中携带的数据会拥有不同的生命作用域和可见性，这取决于保存数据的变量本身的作用域。Spring Web Flow 定义了五种不同作用域，如表 8.2 所示。

|     范围     |                      生命作用域和可见性                      |
| :----------: | :----------------------------------------------------------: |
| Conversation | 最高层级的流程开始时创建，在最高层级的流程结束时销毁。被最高层级的流程和其所有的子流程所共享 |
|     Flow     | 当流程开始时创建，在流程结束时销毁。只有在创建它的流程中是可见的 |
|   Request    |          当一个请求进入流程时创建，在流程返回时销毁          |
|    Flash     | 当流程开始时创建，在流程结束时销毁。在视图状态渲染后，它也会被清除 |
|     View     | 当进入视图状态时创建，当这个状态退出时销毁。只在视图状态内是可见的 |

当使用 `<var>` 元素声明变量时，变量始终是流程作用域的，也就是在定义变量的流程内有效。当使用 `<set>` 或 `<evaluate>` 的时候，作用域通过 name 或 result 属性的前缀指定。例如，将一个值赋给流程作用域的 theAnswer 变量：

```xml
<set name="flowScope.theAnswer" value="42" />
```

到目前为止，我们已经看到了 Web 流程的所有原材料。是时候将其组装起来形成一个成熟且完整功能的 Web 流程了。当我们这样做的时候，请睁大你的眼睛观察，比如我是如何将数据存储在各作用域的变量中的。

## **组合起来：披萨流程**

正如我在本章前面所提到的，我们将暂时不用 Spittr 应用程序。取而代之，我们被要求做一个在线的披萨订购应用，饥饿的 Web 访问者可以在这里订购他们所喜欢的意大利派。

实际上，订购披萨的过程可以很好地定义在一个流程中。我们首先从构建一个高层次的流程开始，它定义了订购披萨的整体过程。接下来，我们会将这个流程拆分成子流程，这些子流程在较低的层次定义了细节。

### **定义基本流程**

一个新的披萨连锁店 Spizza 决定允许用户在线订购以减轻店面电话的压力。当顾客访问 Spizza 站点时，他们需要进行用户识别，选择一个或更多披萨添加到订单中，提供支付信息然后提交订单并等待热乎又新鲜的披萨送过来。图 8.2 阐述了这个流程。

![8.2 订购 pizza 流程](C:\Users\汤琛\Desktop\学习资料\Spring详解\images\8.2 订购 pizza 流程.jpg)

图中的方框代表了状态而箭头代表了转移。你可以看到，订购披萨的整个流程很简单且是线性的。在 Spring Web Flow中，表示这个流程是很容易的。使这个过程变得更有意思的就是前三个流程会比图中的简单方框更复杂。

以下的程序清单 8.1 展示了如何使用 Spring Web Flow 的 XML 流程定义来实现披萨订单的整体流程。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<flow xmlns="http://www.springframework.org/schema/webflow"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/webflow 
  http://www.springframework.org/schema/webflow/spring-webflow-2.3.xsd">

    <var name="order" class="com.springinaction.pizza.domain.Order"/>
    
    <subflow-state id="identifyCustomer" subflow="pizza/customer">
      <output name="customer" value="order.customer"/>
      <transition on="customerReady" to="buildOrder" />
    </subflow-state>
    
    <subflow-state id="buildOrder" subflow="pizza/order">
      <input name="order" value="order"/>
      <transition on="orderCreated" to="takePayment" />
    </subflow-state>

    <subflow-state id="takePayment" subflow="pizza/payment">
      <input name="order" value="order"/>
      <transition on="paymentTaken" to="saveOrder"/>      
    </subflow-state>
        
    <action-state id="saveOrder">
        <evaluate expression="pizzaFlowActions.saveOrder(order)" />
        <transition to="thankCustomer" />
    </action-state>
    
    <view-state id="thankCustomer">
      <transition to="endState" />
    </view-state>

    <end-state id="endState" />
    
    <global-transitions>
      <transition on="cancel" to="endState" />
    </global-transitions>
</flow>
```

在流程定义中，我们看到的第一件事就是 order 变量的声明。每次流程开始的时候，都会创建一个 Order 实例。Order 类会带有关于订单的所有信息，包含顾客信息、订购的披萨列表以及支付详情，如下面所示。

```java
package com.springinaction.pizza.domain;

import java.io.Serializable;
import java.util.ArrayList;
import java.util.List;

import org.springframework.beans.factory.annotation.Configurable;

@Configurable("order")
public class Order implements Serializable {
   private static final long serialVersionUID = 1L;
   private Customer customer;
   private List<Pizza> pizzas;
   private Payment payment;

   public Order() {
      pizzas = new ArrayList<Pizza>();
      customer = new Customer();
   }

   public Customer getCustomer() {
      return customer;
   }

   public void setCustomer(Customer customer) {
      this.customer = customer;
   }

   public List<Pizza> getPizzas() {
      return pizzas;
   }

   public void setPizzas(List<Pizza> pizzas) {
      this.pizzas = pizzas;
   }

   public void addPizza(Pizza pizza) {
      pizzas.add(pizza);
   }

   public float getTotal() {
      return 0.0f;//pricingEngine.calculateOrderTotal(this);
   }

   public Payment getPayment() {
      return payment;
   }

   public void setPayment(Payment payment) {
      this.payment = payment;
   }
}
```

流程定义的主要组成部分是流程的状态。默认情况下，流程定义文件中的第一个状态也会是流程访问中的第一个状态。在本例中，也就是 identifyCustomer状态（一个子流程）。但是如果你愿意的话，你可以通过 `<flow>` 元素的 start-state 属性将任意状态指定为开始状态。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<flow xmlns="http://www.springframework.org/schema/webflow"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/webflow 
    http://www.springframework.org/schema/webflow/spring-webflow-2.3.xsd"
  start-state="identifyCustomer">
...
</flow>
```

识别顾客、构造披萨订单以及支付这样的活动太复杂了，并不适合将其强行塞入一个状态。这是我们为何在后面将其单独定义为流程的原因。但是为了更好地整体了解披萨流程，这些活动都是以 `<subflow-state>` 元素来进行展现的。

流程变量 order 将在前三个状态中进行填充并在第四个状态中进行保存。identifyCustomer 子流程状态使用了元素来填充 order 的 customer 属性，将其设置为顾客子流程收到的输出。buildOrder 和 takePayment 状态使用了不同的方式，它们使用 <input> 将 order 流程变量作为输入，这些子流程就能在其内部填充 order 对象。

在订单得到顾客、一些披萨以及支付细节后，就可以对其进行保存了。saveOrder 是处理这个任务的行为状态。它使用来调用 ID 为 pizzaFlow-Actions 的 bean 的 saveOrder() 方法，并将保存的订单对象传递进来。订单完成保存后，它会转移到 thankCustomer。

thankCustomer 状态是一个简单的视图状态，后台使用了 `/WEB-INF/flows/pizza/thankCustomer.jsp` 这个 JSP 文件，如下所示：

```jsp
<html xmlns:jsp="http://java.sun.com/JSP/Page">
  <jsp:output omit-xml-declaration="yes" />
  <jsp:directive.page contentType="text/html;charset=UTF-8" />
  <head><title>Spizza</title></head>
  <body>
    <h2>Thank you for your order!</h2>
    <![CDATA[
    <a href="${flowExecutionUrl}&_eventId=finished">Finish</a>
    ]]>
  </body>
</html>
```

在“感谢”页面中，会感谢顾客的订购并为其提供一个完成流程的链接。这个链接是整个页面中最有意思的事情，因为它展示了用户与流程交互的唯一办法。

Spring Web Flow 为视图的用户提供了一个 flowExecutionUrl 变量，它包含了流程的 URL。结束链接将一个 `_eventId` 参数关联到 URL 上，以便回到 Web 流程时触发 finished 事件。这个事件将会让流程到达结束状态。

流程将会在结束状态完成。鉴于在流程结束后没有下一步做什么的具体信息，流程将会重新从 identifyCustomer 状态开始，以准备接受另一个披萨订单。

这涵盖了订购披萨的整体流程。但是这个流程并不仅仅是我们在代码清单 8.1 中所看到的这些。我们还需要定义 identifyCustomer、buildOrder、take-Payment 这些状态的子流程。让我们从识别用户开始构建这些流程。

### **收集顾客信息**

如果你曾经订购过披萨，你可能会知道流程。他们首先会询问你的电话号码。电话号码除了能够让送货司机在找不到你家的时候打电话给你，还可以作为你在这个披萨店的标识。如果你是回头客，他们可以使用这个电话号码来查找你的地址，这样他们就知道将你的订单派送到什么地方了。

对于一个新的顾客来讲，查询电话号码不会有什么结果。所以接下来，他们将询问你的地址。这样，披萨店的人就会知道你是谁以及将披萨送到哪里。但是在问你要哪种披萨之前，他们要确认你的地址在他们的配送范围之内。如果不在的话，你需要自己到店里并取走披 萨。

在每个披萨订单开始前的提问和回答阶段可以用图 8.3 的流程图来表示。

这个流程比整体的披萨流程更有意思。这个流程不是线性的而是在好几个地方根据不同的条件有了分支。例如，在查找顾客后，流程可能结束（如果找到了顾客），也有可能转移到注册表单（如果没有找到顾客）。同样，在 check-DeliveryArea 状态，顾客有可能会被警告也有可能不被警告他们的地址在配送范围之外。

![8.3 识别顾客流程](C:\Users\汤琛\Desktop\学习资料\Spring详解\images\8.3 识别顾客流程.jpg)

以下的程序清单展示了识别顾客的流程定义。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<flow xmlns="http://www.springframework.org/schema/webflow"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/webflow 
  http://www.springframework.org/schema/webflow/spring-webflow-2.0.xsd">

    <input name="order" required="true"/>
    
    <!-- Customer -->
    <view-state id="welcome">
        <transition on="phoneEntered" to="lookupCustomer"/>
        <transition on="cancel" to="cancel"/>
    </view-state>
    
    <action-state id="lookupCustomer">
        <evaluate result="order.customer" expression=
            "pizzaFlowActions.lookupCustomer(requestParameters.phoneNumber)" />
        <transition to="registrationForm" on-exception=
            "com.springinaction.pizza.service.CustomerNotFoundException" />
        <transition to="customerReady" />
    </action-state>
    
    <view-state id="registrationForm" model="order" popup="true" >
        <on-entry>
          <evaluate expression=
              "order.customer.phoneNumber = requestParameters.phoneNumber" />
        </on-entry>
        <transition on="submit" to="checkDeliveryArea" />
        <transition on="cancel" to="cancel" />
    </view-state>
    
    <decision-state id="checkDeliveryArea">
      <if test="pizzaFlowActions.checkDeliveryArea(order.customer.zipCode)" 
          then="addCustomer" 
          else="deliveryWarning"/>
    </decision-state>
    
    <view-state id="deliveryWarning">
        <transition on="accept" to="addCustomer" />
        <transition on="cancel" to="cancel" />
    </view-state>
    
    <action-state id="addCustomer">
        <evaluate expression="pizzaFlowActions.addCustomer(order.customer)" />
        <transition to="customerReady" />
    </action-state>
            
    <!-- End state -->
    <end-state id="cancel" />
    <end-state id="customerReady">
      <output name="customer" />
    </end-state>
    <global-transitions>
      <transition on="cancel" to="cancel" />
    </global-transitions>
</flow>
```

这个流程包含了几个新的技巧，包括我们首次使用的 `<decision-state>` 元素。因为它是 pizza 流程的子流程，所以它也可以接受 Order 对象作为输入。

与前面一样，我们还是将这个流程的定义分解成一个个的状态，让我们从 welcome 状态开始。

**询问电话号码**

welcome 状态是一个很简单的视图状态，它欢迎访问 Spizza 站点的顾客并要求他们输入电话号码。这个状态并没有什么特殊的。它有两个转移：如果从视图触发 phoneEntered 事件的话，转移会将流程定向到 lookupCustomer，另外一个就是在全局转移中定义的用来响应 cancel 事件的 cancel 转移。

welcome 状态的有趣之处在于视图本身。视图 welcome 定义在 `/WEBINF/flows/pizza/customer/welcome.jspx` 中，如下所示。

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<html>

  <head><title>Spring Pizza</title></head>

  <body>
    <h2>Welcome to Spring Pizza!!!</h2>
	
		<form:form>
      <input type="hidden" name="_flowExecutionKey" 
             value="${flowExecutionKey}"/>
		  <input type="text" name="phoneNumber"/><br/>
      <input type="submit" name="_eventId_phoneEntered" value="Lookup Customer" />
		</form:form>
	</body>
</html>
```

这个简单的表单提示用户输入其电话号码。但是表单中有两个特殊的部分来驱动流程继续。

首先要注意的是隐藏的 `flowExecutionKey` 输入域。当进入视图状态时，流程暂停并等待用户采取一些行为。赋予视图的流程执行 key（flow execution key）就是一种返回流程的“回程票”（claim ticket）。当用户提交表单时，流程执行 key 会 在 `_flowExecutionKey` 输入域中返回并在流程暂停的位置进行恢复。

还要注意的是提交按钮的名字。按钮名字的 `_eventId` 部分是提供给 Spring Web Flow 的一个线索，它表明了接下来要触发事件。当点击这个按钮提交表单时，会触发 phoneEntered 事件进而转移到 lookupCustomer。

**查找顾客**

当欢迎表单提交后，顾客的电话号码将包含在请求参数中并准备用于查询顾客。lookupCustomer 状态的 `<evaluate>` 元素是查找发生的地方。它将电话号码从请求参数中抽取出来并传递到 pizzaFlowActions bean 的 lookup-Customer() 方法中。

目前，lookupCustomer() 的实现并不重要。只需知道它要么返回 Customer 对象，要么抛出 CustomerNotFoundException 异常。

在前一种情况下，Customer 对象将会设置到 customer 变量中（通过 result 属性）并且默认的转移将把流程带到 customerReady 状态。但是如果不能找到顾客的话，将抛出 CustomerNotFoundException 并且流程被转移到 registrationForm 状态。

**注册新顾客**

registrationForm 状态是要求用户填写配送地址的。就像我们之前看到的其他视图状态，它将被渲染成 JSP。JSP 文件如下所示。

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<html>

  <head><title>Spring Pizza</title></head>

  <body>
    <h2>Customer Registration</h2>
    
    <form:form commandName="order">
      <input type="hidden" name="_flowExecutionKey" 
             value="${flowExecutionKey}"/>
      <b>Phone number: </b><form:input path="customer.phoneNumber"/><br/>
      <b>Name: </b><form:input path="customer.name"/><br/>
      <b>Address: </b><form:input path="customer.address"/><br/>
      <b>City: </b><form:input path="customer.city"/><br/>
      <b>State: </b><form:input path="customer.state"/><br/>
      <b>Zip Code: </b><form:input path="customer.zipCode"/><br/>
      <input type="submit" name="_eventId_submit" 
             value="Submit" />
      <input type="submit" name="_eventId_cancel" 
             value="Cancel" />
    </form:form>
	</body>
</html>
```

这并非我们在流程中看到的第一个表单。welcome 视图状态也为顾客展现了一个表单，那个表单很简单，并且只有一个输入域，从请求参数中获得输入域的值也很简单。但是注册表单就比较复杂了。

在这里不是通过请求参数一个个地处理输入域，而是以更好的方式将表单绑定到 Customer 对象上 —— 让框架来做所有繁杂的工作。

**检查配送区域**

在顾客提供其地址后，我们需要确认他的住址在配送范围之内。如果 Spizza 不能派送给他们，那么我们要让顾客知道并建议他们自己到店面里取走披萨。 

为了做出这个判断，我们使用了决策状态。决策状态 checkDeliveryArea 有一个元素，它将顾客的邮政编码传递到 pizzaFlowActions bean 的check-DeliveryArea() 方法中。这个方法将会返回一个 Boolean 值：如果顾客在配送区域内则为 true，否则为 false。

如果顾客在配送区域内的话，那流程转移到 addCustomer 状态。否则，顾客被带入到 deliveryWarning 视图状态。deliveryWarning 背后的视图就是`/WEB-INF/flows/pizza/customer/deliveryWarning.jspx`，如下所示：

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
  <head><title>Spring Pizza</title></head>
  
  <body>
		<h2>Delivery Unavailable</h2>
		
		<p>The address is outside of our delivery area. The order
		may still be taken for carry-out.</p>
		
		<a href="${flowExecutionUrl}&_eventId=accept">Accept</a> | 
		<a href="${flowExecutionUrl}&_eventId=cancel">Cancel</a>
  </body>
</html>
```

在 deliveryWarning.jspx 中与流程相关的两个关键点就是那两个链接，它们允许用户继续订单或者将其取消。通过使用与 welcome 状态相同的 flow-ExecurtionUrl 变量，这些链接分别触发流程中的 accept 或 cancel 事件。如果发送的是 accept 事件，那么流程会转移到 addCustomer 状态。否则，接下来会是全局的取消转移，子流程将会转移到 cancel 结束状态。

稍后我们将介绍结束状态。让我们先来看看 addCustomer 状态。

**存储顾客数据**

当流程抵达 addCustomer 状态时，用户已经输入了他们的地址。为了将来使用，这个地址需要以某种方式存储起来（可能会存储在数据库中）。add-Customer 状态有一个 `<evaluate>` 元素，它会调用 pizzaFlowActions bean 的 addCustomer() 方法，并将 customer 流程参数传递进去。

一旦这个过程完成，会执行默认的转移，流程将会转移到 ID 为 customer-Ready 的结束状态。

**结束流程**

一般来讲，流程的结束状态并不会那么有意思。但是这个流程中，它不仅仅只有一个结束状态，而是两个。当子流程完成时，它会触发一个与结束状态 ID 相同的流程事件。如果流程只有一个结束状态的话，那么它始终会触发相同的事件。但是如果有两个或更多的结束状态，流程能够影响到调用状态的执行方向。

当 customer 流程走完所有正常的路径后，它最终会到达 ID 为 customer-Ready 的结束状态。当调用它的披萨流程恢复时，它会接收到一个customer-Ready 事件，这个事件将使得流程转移到 buildOrder 状态。

要注意的是 customerReady 结束状态包含了一个 `<output>` 元素。在流程中这个元素等同于 Java 中的 return 语句。它从子流程中传递一些数据到调用流程。在本示例中，`<output>` 元素返回 customer 流程变量，这样在披萨流程中，就能够将 identifyCustomer 子流程的状态指定给订单。另一方面，如果在识别顾客流程的任意地方触发了 cancel 事件，将会通过 ID 为 cancel 的结束状态退出流程，这也会在披萨流程中触发 cancel 事件并导致转移（通过全局转移）到披萨流程的结束状态。

### **构建订单**

在识别完顾客之后，主流程的下一件事情就是确定他们想要什么类型的披萨。订单子流程就是用于提示用户创建披萨并将其放入订单中的，如图 8.4 所示。

![8.4 订单子流程](C:\Users\汤琛\Desktop\学习资料\Spring详解\images\8.4 订单子流程.jpg)

你可以看到，showOrder 状态位于订单子流程的中心位置。这是用户进入这个流程时看到的第一个状态，它也是用户在添加披萨到订单后要转移到的状态。它展现了订单的当前状态并允许用户添加其他的披萨到订单中。

要添加披萨到订单时，流程会转移到 createPizza 状态。这是另外一个视图状态，允许用户选择披萨的尺寸和面饼上面的配料。在这里，用户可以添加或取消披萨，两种事件都会使流程转移回 showOrder 状态。

从 showOrder 状态，用户可能提交订单也可能取消订单。两种选择都会结束订单子流程，但是主流程会根据选择不同进入不同的执行路径。

如下显示了如何将图中所阐述的内容转变成 Spring Web Flow 定义。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<flow xmlns="http://www.springframework.org/schema/webflow"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/webflow 
  http://www.springframework.org/schema/webflow/spring-webflow-2.0.xsd">

    <input name="order" required="true" />
    
    <!-- Order -->
    <view-state id="showOrder">
        <transition on="createPizza" to="createPizza" />
        <transition on="checkout" to="orderCreated" />
        <transition on="cancel" to="cancel" />
    </view-state>

    <view-state id="createPizza" model="flowScope.pizza">
        <on-entry>
          <set name="flowScope.pizza" 
              value="new com.springinaction.pizza.domain.Pizza()" />
              
          <evaluate result="viewScope.toppingsList" 
              expression="T(com.springinaction.pizza.domain.Topping).asList()" />
        </on-entry>
        <transition on="addPizza" to="showOrder">
          <evaluate expression="order.addPizza(flowScope.pizza)" />
        </transition>
        <transition on="cancel" to="showOrder" />
    </view-state>

        
    <!-- End state -->
    <end-state id="cancel" />
    <end-state id="orderCreated" />
</flow>
```

这个子流程实际上会操作主流程创建的 Order 对象。因此，我们需要以某种方式将 Order 从主流程传到子流程。你可能还记得在程序清单 8.1 中我们使用了 `<input>` 元素来将 Order 传递进流程。在这里，我们使用它来接收 Order 对象。如果你觉得这个流程与Java中的方法有 些类似地话，那这里使用的 `<input>` 元素实际上就定义了这个子流程的签名。这个流程需要一个名为 order 的参数。

接下来，我们会看到 showOrder 状态，它是一个基本的视图状态并具有三个不同的转移，分别用于创建披萨、提交订单以及取消订单。

createPizza 状态更有意思一些。它的视图是一个表单，这个表单可以添加新的 Pizza 对象到订单中。元素添加了一个新的 Pizza 对象到流程作用域内，当表单提交时，表单的内容会填充到该对象中。需要注意的是，这个视图状态引用的 model 是流程作用域内的同一个 Pizza 对象。Pizza 对象将绑定到创建披萨的表单中，如下所示。

```jsp
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<div>
  <h2>Create Pizza</h2>
  <form:form commandName="pizza">
    <input type="hidden" name="_flowExecutionKey" value="${flowExecutionKey}"/>
	<b>Size: </b><br/>
	<form:radiobutton path="size" label="Small (12-inch)" value="SMALL"/><br/>
    <form:radiobutton path="size" label="Medium (14-inch)" value="MEDIUM"/><br/>
    <form:radiobutton path="size" label="Large (16-inch)" value="LARGE"/><br/>
    <form:radiobutton path="size" label="Ginormous (20-inch)" value="GINORMOUS"/><br/>
	<br/>
	  
	<b>Toppings: </b><br/>
	<form:checkboxes path="toppings" items="${toppingsList}" delimiter="<br/>"/><br/>
	<br/>
	
	<input type="submit" class="button" name="_eventId_addPizza" value="Continue"/>
	<input type="submit" class="button" name="_eventId_cancel" value="Cancel"/>          
  </form:form>
</div>
```

当通过 Continue 按钮提交订单时，尺寸和配料选择将会绑定到 Pizza 对象中并且触发 addPizza 转移。与这个转移关联的 `<evaluate>` 元素表明在转移到 showOrder 状态之前，流程作用域内的 Pizza 对象将会传递给订单的 addPizza() 方法中。

有两种方法来结束这个流程。用户可以点击 showOrder 视图中的 Cancel 按钮或者 Checkout 按钮。这两种操作都会使流程转移到一个 `<end-state>`。但是选择的结束状态 id 决定了退出这个流程时触发事件，进而最终确定了主流程的下一步行为。主流程要么基于 cancel 事件要么基于 orderCreated 事件进行状态转移。在前者情况下，外边的主流程会结束；在后者情况下，它将转移 到 takePayment 子流程，这也是接下来我们要看的。

### **支付**

吃免费披萨这事儿并不常见。如果 Spizza 披萨店让他们的顾客不提供支付信息就订购披萨的话，估计他们也维持不了多久。在披萨流程要结束的时候，最后的子流程提示用户输入他们的支付信息。这个简单的流程如图 8.5 所示。

![8.5 支付子流程](C:\Users\汤琛\Desktop\学习资料\Spring详解\images\8.5 支付子流程.jpg)

像订单子流程一样，支付子流程也使用 `<input>` 元素接收一个 Order 对象作为输入。

你可以看到，进入支付子流程的时候，用户会到达 takePayment 状态。这是一个视图状态，在这里用户可以选择使用信用卡、支票或现金进行支付。提交支付信息后，将进入 verifyPayment 状态。这是一个行为状态，它将校验支付信息是否可以接受。

使用 XML 定义的支付流程如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<flow xmlns="http://www.springframework.org/schema/webflow"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/webflow 
  http://www.springframework.org/schema/webflow/spring-webflow-2.0.xsd">

    <input name="order" required="true"/>
    
    <view-state id="takePayment" model="flowScope.paymentDetails">
        <on-entry>
          <set name="flowScope.paymentDetails" 
              value="new com.springinaction.pizza.domain.PaymentDetails()" />

          <evaluate result="viewScope.paymentTypeList" 
              expression="T(com.springinaction.pizza.domain.PaymentType).asList()" />
        </on-entry>
        <transition on="paymentSubmitted" to="verifyPayment" />
        <transition on="cancel" to="cancel" />
    </view-state>

    <action-state id="verifyPayment">
        <evaluate result="order.payment" expression=
            "pizzaFlowActions.verifyPayment(flowScope.paymentDetails)" />
        <transition to="paymentTaken" />
    </action-state>
            
    <!-- End state -->
    <end-state id="cancel" />
    <end-state id="paymentTaken" />
</flow>
```

在流程进入 takePayment 视图时，元素将构建一个支付表单并使用 SpEL 表达式在流程作用域内创建一个 PaymentDetails 实例，这是支撑表单的对象。它也会创建视图作用域的 paymentTypeList 变量，这个变量是一个列表包含了 PaymentType 枚举（如程序清单 8.11 所示）的值。在这里，SpEL 的 T() 操作用于获得 PaymentType 类，这样就可以调用静态的 asList() 方法。

```java
package com.springinaction.pizza.domain;

import java.util.Arrays;
import java.util.List;

import org.apache.commons.lang3.text.WordUtils;

public enum PaymentType {
  CASH, CHECK, CREDIT_CARD;
  
  public static List<PaymentType> asList() {
    PaymentType[] all = PaymentType.values();
    return Arrays.asList(all);
  }
  
  @Override
  public String toString() {
    return WordUtils.capitalizeFully(name().replace('_', ' '));
  }
}
```

在面对支付表单的时候，用户可能提交支付也可能会取消。根据做出的选择，支付子流程将以名为 paymentTaken 或 cancel 的 `<end-state>` 结束。就像其他的子流程一样，不论哪种 `<end-state>` 都会结束子流程并将控制交给主流程。但是所采用 `<end-state>` 的 id 将决定主流程中接下来的转移。

现在，我们已经依次介绍了披萨流程及其子流程，并看到了 Spring Web Flow的很多功能。在我们结束 Spring Web Flow 话题之前，让我们快速了解一下如何对流程及其状态的访问增加安全保护。

## **保护 Web 流程**

在下一章中，我们将会看到如何使用 Spring Security 来保护 Spring 应用程序。但现在我们讨论的是 Spring Web Flow，让我们快速地看一下 Spring Web Flow 是如何结合 Spring Security 支持流程级别的安全性的。

Spring Web Flow 中的状态、转移甚至整个流程都可以借助 `<secured>` 元素实现安全性，该元素会作为这些元素的子元素。例如，为了保护对一个视图状态的访问，你可以这样使用 `<secured>`：

```xml
<view-state id="restricted">
  <secured attribute="ROLE_ADMIN" match="all" />
</view-state>
```

按照这里的配置，只有授予 ROLE_ADMIN 访问权限（借助 attributes 属性）的用户才能访问这个视图状态。attributes 属性使用逗号分隔的权限列表来表明用户要访问指定状态、转移或流程所需要的权限。match 属性可以设置为 any 或 all。如果设置为 any，那么用户必须至少具有一个 attributes 属性所列的权限。 如果设置为 all，那么用户必须具有所有的权限。你可能想知道用户如何具备 `<secured>` 元素所检验的权限，甚至最开始的时候用户是如何进行登录的？这些问题的答案将在第 9 章给出。

## **小结**

并不是所有的 Web 应用程序都是自由访问的。有时候，必须对用户进行指引、询问适当的问题并基于他们的响应将其引导到特定页面。在这些情况下，应用程序不太像一个菜单选项而更像应用程序与用户之间的对话。

在本章中，我们介绍了 Spring Web Flow，它是能够构建会话式应用程序的 Web 框架。在介绍的同时，我们构建了一个基于流程的披萨订单应用。我们先定义了应用程序的整体流程，从收集顾客信息开始到保存订单到系统中结束。

流程由多个状态和转移组成，它们定义了会话如何从一个状态到另一个状态。状态本身分为好多种：行为状态执行业务逻辑，视图状态涉及到流程中的用户，决策状态动态地引导流程执行，结束状态表明流程的结束，除此之外，还有子流程状态，它们自身是通过流程来定义 的。

最后，我们看到如何限制具有特定权限的用户才能访问流程、状态或转移。但是，我们还没有介绍应用程序对用户的认证以及如何授予用户权限。这就是 Spring Security 能够发挥作用的地方了，而 Spring Security 就是我们第 9 章将要介绍的内容。

