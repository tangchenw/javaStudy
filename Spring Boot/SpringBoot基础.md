# Springboot基础

## Application启动类详解

### 启动引导Spring  

ReadingListApplication在Spring Boot应用程序里有两个作用：配置和启动引导。首先，这是主要的Spring配置类。虽然Spring Boot的自动配置免除了很多Spring配置，但你还需要进行少量配置来启用自动配置。正如代码清单2-1所示，这里只有一行配置代码。

代码清单2-1 ReadingListApplication.java不仅是启动引导类，还是配置类    

```java
package readinglist;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication //开启组件扫描和自动配置
public class ReadingListApplication {
public static void main(String[] args) {
SpringApplication.run(ReadingListApplication.class, args); //负责启动引导应用程序
}
}
```

@SpringBootApplication开启了Spring的组件扫描和Spring Boot的自动配置功能。实际上， @SpringBootApplication将三个有用的注解组合在了一起。  

- Spring的@Configuration：标明该类使用Spring基于Java的配置。虽然本书不会写太多配置，但我们会更倾向于使用基于Java而不是XML的配置。  
- Spring的@ComponentScan：启用组件扫描，这样你写的Web控制器类和其他组件才能被自动发现并注册为Spring应用程序上下文里的Bean。本章稍后会写一个简单的Spring MVC控制器，使用@Controller进行注解，这样组件扫描才能找到它。  
- Spring Boot 的 @EnableAutoConfiguration ： 这 个 不 起 眼 的 小 注 解 也 可 以 称 为@Abracadabra①，就是这一行配置开启了Spring Boot自动配置的魔力，让你不用再写成篇的配置了。  

在Spring Boot的早期版本中，你需要在ReadingListApplication类上同时标上这三个注解，但从Spring Boot 1.2.0开始，有@SpringBootApplication就行了  

如我所说， ReadingListApplication还是一个启动引导类。要运行Spring Boot应用程序有几种方式，其中包含传统的WAR文件部署。但这里的main()方法让你可以在命令行里把该应用程序当作一个可执行JAR文件来运行。这里向SpringApplication.run()传递了一个ReadingListApplication类的引用，还有命令行参数，通过这些东西启动应用程序。  

