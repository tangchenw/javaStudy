



# Spring Cloud工程创建

先点击文件，再新建项目

![image-20220317100441405](C:\Users\汤琛\Desktop\学习资料\Spring cloud\images\image-20220317100441405.png)

按如图进行好配置之后直接点完成即可。

![QQ截图20220317100741](C:\Users\汤琛\Desktop\学习资料\Spring cloud\images\QQ截图20220317100741.png)

### 微服务的基本组成

**生产者：提供服务**

**消费者：消费服务**

**服务注册/发现中心：服务注册，发现，监控**

## Eureka

服务注册Eureka的配置文件

```yaml
spring:
  security:
    basic:
      enabled: true
    user:
      name: admin
      password: 123456
#      服务端口
server:
  port: 8761
#
eureka:
#  instance:
#    hostname: localhost
  client:
    register-with-eureka: false #是否注册到eureka服务中，自己就是注册中心，不用注册自己
    fetch-registry: false  #是否拉取其它服务，自己就是注册中心，不用去注册中心获取其它服务,针对单点，如果为集群则需要获取其它服务器中的注册中心服务。
    service-url:
      defaultZone: http://admin:123456@localhost:8761/eureka #注册的url
#      defaultZone: http://${spring.security.user.name}:${spring.security.user.password}@${eureka.instance.hostname}:${server.port}/eureka/ #基于配置的写法
```

Eureka启动类配置

```java
@EnableEurekaServer //Eureka服务
```

pom依赖配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>eureka</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>eureka</name>
    <description>eureka</description>
    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.SR1</spring-cloud.version>
    </properties>
    <dependencies>

        <!-- hystrix断路器 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>

        <!--eureka-server-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <!--LOMBOK-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.22</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
<!--            <dependency>-->
<!--                <groupId>org.springframework.cloud</groupId>-->
<!--                <artifactId>spring-cloud-context</artifactId>-->
<!--                <version>3.0.1</version>-->
<!--            </dependency>-->

            <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-commons -->
<!--            <dependency>-->
<!--                <groupId>org.springframework.cloud</groupId>-->
<!--                <artifactId>spring-cloud-commons</artifactId>-->
<!--                <version>3.0.1</version>-->
<!--            </dependency>-->
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

## Eureka服务客户端（生产者）

provider.yml

```yml
#应用端口
server:
  port: 7901
spring:
  application:
    name: provider-user

#eureka 配置
eureka:
  client:
    service-url:
      defaultZone: http://admin:123456@localhost:8761/eureka
#日志基本
logging:
  level:
    root: INFO
```

provider启动类

```java
@EnableEurekaClient //Eureka服务客户端注册注解
```

controller类

```java
@RestController
@RequestMapping("/user")
public class UserController {
    @RequestMapping("/say")
    public String Hello(){
        return "Hello World!";
    }

    @RequestMapping("/Hi")
    public String Hi(){
        return "Hi World!";
    }

    @RequestMapping("/HaHa")
    public String HaHa(){
        return "HaHa World!";
    }
}
```

pom配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>provider</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>provider</name>
    <description>provider</description>
    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.SR1</spring-cloud.version>
    </properties>
    <dependencies>

        <!--eureka-client 依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <!--LOMBOK-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.22</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.2.6.RELEASE</version>
            <scope>compile</scope>
        </dependency>


    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>


    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

## 消费者

yml配置

```yml
server:
  port: 8001
spring:
  application:
    name: consumer

# eureka配置
eureka:
  client:
    service-url:
      defaultZone: http://admin:123456@localhost:8761/eureka

logging:
  level:
    root: INFO
```

controller配置

```java
@RestController
public class SayController {
    @Bean
    @LoadBalanced
    public RestTemplate getResttemplate(){
        return new RestTemplate();
    }
    @Autowired
    private RestTemplate resttemplate;

    @RequestMapping("/hello")
    public String hello(){
        //指出服务地址   http://{服务提供者应用名名称}/{具体的controller}
        String url="http://provider-user/user/say";

        //返回值类型和我们的业务返回值一致
        return resttemplate.getForObject(url, String.class);

    }
    @RequestMapping("/hi")
    public String hi(){
        //指出服务地址   http://{服务提供者应用名名称}/{具体的controller}
        String url="http://provider-user/user/Hi";

        //返回值类型和我们的业务返回值一致
        return resttemplate.getForObject(url, String.class);

    }
    @RequestMapping("/haha")
    public String haha(){
        //指出服务地址   http://{服务提供者应用名名称}/{具体的controller}
        String url="http://provider-user/user/Haha";
        //返回值类型和我们的业务返回值一致
        return resttemplate.getForObject(url, String.class);
    }

}
```

## Feign

yml配置

```yml
server:
  port: 8082
spring:
  application:
    name: fegin

eureka:
  client:
    service-url:
      defaultZone: http://admin:123456@localhost:8761/eureka
```

生产者接口配置

```java
@FeignClient("PROVIDER-USER")
public interface UserClient {
    /**
     * Feign中没有原生的@GetMapping/@PostMapping/@DeleteMapping/@PutMapping，要指定需要用method进行
     *
     *
     * 接口上方用requestmapping指定是服务提供者的哪个controller提供服务
     */
    @RequestMapping(value="/user/say",method= RequestMethod.GET)
    public String sayHello();

    @RequestMapping(value="/user/Hi",method=RequestMethod.GET)
    public String sayHi();

    @RequestMapping(value="/user/Haha",method=RequestMethod.GET)
    public String sayHaha();

}
```

启动类配置

```java
@EnableEurekaClient  //表明这是一个eureka客户端
@EnableFeignClients(basePackages = "com.example.*") //开启feign
```

controller配置

```java
/*
这个接口相当于把原来的服务提供者项目当成一个Service类
* */
@RestController
public class HelloController {
    @Autowired
    private UserClient userClient;


    /**
     * 此处的mapping是一级controller，调用方法里边绑定了二级的conroller，相当于用http完成一次转发
     * @return
     */
    @GetMapping("/hello")
    public String hello(){
        return userClient.sayHello();
    }

    @GetMapping("/hi")
    public String hi(){
        return userClient.sayHi();
    }

    @GetMapping("/haha")
    public String haha(){
        return userClient.sayHaha();
    }

}
```

pom配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.6.4</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>feign</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>feign</name>
    <description>feign</description>
    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>2021.0.1</spring-cloud.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>

        <!--eureka-server-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- boot-web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```
