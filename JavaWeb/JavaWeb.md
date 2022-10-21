## Servlet接口实现类

1. Servlet接口来自于Servlet规范下的一个接口，这个接口存在于Http服务器（tomcat）提供的jar包下

2. Tomcat服务器的lib文件夹下有一个servlet-api.jar存放servlet接口（javax.servlet.Servlet接口）

3. Servlet规范中，Http服务器（tomcat）能调用的动态资源文件必须是一个servlet接口实现类

   > 换句话说类必须implements Servlet才能够被tomcat调用。

### Servlet接口实现类开发步骤

1. 先让类实现httpServlet接口.
2. 重写接口的两个方法，doGet(),doPost().分别用于处理get请求，post请求
3. 将servlet接口实现类信息注册到tomcat服务器（也就是web.xml下去声明）

```java
<servlet>
    <servlet-name>mm</servlet-name>//声明一个变量mm存储serlvet接口实现类的路径
    <servlet-class>com.cn.show.xxxServlet</servlet-class>//声明注册的接口实现类
</servlet>
<servlet-mapping>
        <servlet-name>mm<servlet-name>
        <url-pattern>/one<url-pattern>//设置简短请求别名，书写时以/开头，访问时直接localhost:端口号:前缀
</servlet-mappding>
```

### Servlet接口实现类的生命周期

该实现类的实例对象是不需要用户去创建的（也就是说又tomcat服务器去创建【动态代理】），当用户手动使用<load-on-startup>数字</load-on-startup>会自动创建。

如果没有手动使用，则Tomcat服务器收到响应的serlvet接口实现类的对应请求时候会被自动创建实例对象，并且只会被创建一次（单例），而且Tomcat关闭的时候被销毁。

### HttpServletResponse和HttpServletRequest接口

HttpServletResponse接口的功能：

out.writer方法可以将"字符"、‘字符串’、‘ASCII码’写入响应体

ASCII: a------------------97   65-------------------A       0-----------------48

***

response.sendRedirect(result);//通过响应对象将地址值给响应头中的location属性。【重定向】

> doPost获取请求体的时候，默认使用的是request对象的编码格式，iso编码
>
> doGet获取请求头的时候，默认使用的是tomcat服务器的编码格式，utf-8

### Http状态码

100-599；五大类

1xx:特征值100，通知浏览器本次返回的资源文件并不是一个独立的资源文件，还需要其它文件。

2xx:200，请求成功。

3xx:特征值302：重定向

4xx:404：资源文件未找到，servlet实现类未找到

​		405：使用错了请求方式

5xx:500：后端请求处理出现异常。



### 多个Servlet之间的调用

#### 重定向

response.sendRedirect("/网站名/资源名");

调用资源文件接收的请求方式一定是get请求。

缺点：需要多次在浏览器和http服务器之间进行往返，浪费大量时间。

#### 请求转发

request.getRequestDispatcher("/资源文件名").forward(当前请求对象，当前响应对象);

> 浏览器只发送了一次请求，也就只有一个请求协议包，所有servlet共享这个请求协议包，因此这些servlet接收的请求方式
>
> 都与浏览器的请求方式一致。



## 多个Servlet之间数据共享方案

Servlet规范中提供四种数据共享方案

### ServletContext接口

ServletContext实例对象（全局作用域对象）【存储方式为map存储】

全局作用域对象生命周期：Http服务器启动（tomcat）会自动在内存中创建一个全局作用域对象且只有一个，在Http服务器关闭的时候被销毁。

> 用法：
>
> ​	ServletContext application = request.getServletContext();通过请求对象获取全局作用域对象
>
> ​    application.setAttribute("key1",数据)；
>
> 另外一个servlet中取
>
> ​	ServletContext application = request.getServletContext();
>
> ​	Object data=application.getAttribute("key1")；

### Cookie接口

原理：Http服务器第一次接收请求之后，创建一个cookie存储相关的数据，将cookie写入响应头返回给浏览器，cookie将存储在浏览器的cookie当中，用户下次再接收同样的请求时，就可以直接通过cookie读取响应数据。

**代码**

```java
//1.创建cookie对象
Cookie cookie=new Cookie("key1","value1");
//cookie键值对只能存入String字符串，且key不能为中文
//2.将cookie写入HttpServletReponse响应对象,交给浏览器
resp.addCookie(cookie);

---------------------------------------------
    //调用
    Cookie  cookieArray[] = request.getCookies();
	for(Cookie cookie:cookieArray){
        String key = cookie.getName();//获取key
        cookie.getValue();//获取value
    }
```

> 默认情况cookie只会放在浏览器的内存当中，只要浏览器关闭，则cookie对象被销毁。但是大部分网站的账号等信息都会设置为在客户端计算机硬盘上存储，只有设置的存活时间(cookie.setMaxAge(s))到达才会被销毁。

#### HttpSession对象

HttpSession简称（Session)会话作用域对象。

**特点**

- 存储在计算机内存当中
- 可以存储任意类型的共享数据Object
- 可以存储任意数量的共享数据（map集合存储）

**用法**

```java
//1.向Http服务器获取Session对象
HttpSession session= requset.getSession();
//2.存储数据
session.setAttribute("key1",Object);

----------------------------------------------
    另一个Servlet
    //3.获取Session对象
    HttpSession session= requset.getSession();
	//4.获取存储数据
	Object data= session.getAttribute("key1");

----------------------------------------------
    
    获取session全部数据
    //全部key存入枚举对象
    Enumertion data=session.getAttributeNames();
	//遍历取出
	whlie(data.hasMoreElements){
        String key = data.nextElement();
        Object value = data.getAttribute("key");
    }
```

> Http服务器（tomcat)在创建session对象的时候会生成一个编号（JSESSIONId），这个编号会被Tomcat保存到cookie对象当中，然后推送到浏览器缓存当中.当同一用户多次访问同一请求时，则通过该编号确定是哪一个用户。

**getSession()与getSession(false)**

- getSession()如果当前用户已经有session对象则返回该对象，如果没有则创建session对象。
- getSession(false) 同样有则返回，没有对象则返回Null。

[^由于用户和Session关联时使用了cookie存放在浏览器缓存而不是服务器中，所以当浏览器关闭时，用户和HttpSession之间的联系就断了，但是TomcatHttp服务器又不知道服务器是否关闭，因此即使关闭了HttpSession对象也不会被销毁，所以Tomcat为每一个HttpSession对象都设置了一个默认的销毁时间（30min）,如果用户超过30min未使用HttpSession则Tomcat销毁该对象。]: 手动配置：web.xml

```java
<session-config>
	<session-timeout>5<session-timeout>//5min
</session-config>
```

#### HttpServletRequest接口

该数据共享方案只试用于两个servlet之间进行请求转发。

## 监听器

**作用**

- 该监听器接口在Http服务器中没有实现类，需要手动实现
- 用于监控【作用域对象的生命周期变化时刻】和【作用域对象共享数据变化时刻】

[^作用域对象]: 在服务器内存当中为两个Servlet之间提供数据共享方案的对象被称之为作用域对象。由于Cookie存储在浏览器缓存当中，所以它并不是作用域对象。

**监听器接口（ServletContextListener）**

- 检测全局作用域对象被初始化时刻以及被销毁时刻

- 监听事件方法：`public void contextInitlized()`全局作用域对象被Http服务器初始化的时候被调用；

  `public void contextDestory()`全局作用域对象被Http服务器销毁的时候被调用

**共享数据变化监听接口（ServletContextAttributeListener）**

- 监听全局作用域对象共享数据变化时刻

- 监听事件方法：`public void contextAdd()`:在全局作用域对象添加共享数据的时候被调用

  `public void contextReplaced()`：更新数据的时候被调用

  `public void contextRemove()`：对象输出数据的时候被调用

## 过滤器

**作用**

- 该过滤器接口在Http服务器中没有实现类，需要手动实现
- Fiter接口在Http服务器调用资源文件之前，对Http服务器进行拦截。
- 检测请求的合法性。
- 对当前请求进行增强操作。

**用法**

- 创建一个java类实现Filter接口
- 重写Filter接口中的doFilter方法
- web.xml将Filter过滤器接口实现类注册到Http服务器

```java
<filter-mapping>
    <filter-name>oneFilter</filter-name>
    <url-pattern>/img/mm.jpg</url-pattern>
<filter-mapping>
```

## Ajax

**作用**

- 全局刷新：整个浏览器被新的数据覆盖。在网络中传输大量数据，页面需要刷新。
- 局部刷新：在浏览器的内部发起请求，获取数据改变页面内容。（ajax就是做局部刷新的）

**用法**

1. 创建核心对象(异步请求对象）：`var xmlHttp =new XMLHttpRequest()`
2. 初始化异步请求对象：xmlHttp.open(请求方式，请求地址，true);
3. 异步对象发送请求：xmlHttp.send()

## JQuery

**特点**

- Jquery表示的对象都是数组

**语法**

```javascript
$("#id").click(function(){
    //todo
})
```

---

### Jquery之发送ajax请求

```javascript
$.ajax({
    url: "", //请求的地址
    asnyc:true, //异步
    type: GET,  //请求类型
    contentType:"application/json", //设置请求头格式
    data: {name:"",age:20}, //发送的数据
    dataType: "json",  //响应的数据格式
    error:function(){
        //请求出错时的处理
    }，
    success: function(resp){
    //请求成功时的处理
}
})
```

>jquery中ajax传递参数为数组的时候需要反深度序列化将traditonal设置为true才能将数组传递给servlet接收。

