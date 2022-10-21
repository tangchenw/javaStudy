# Spring Security

本文章对Spring Security的解析主要通过Spring Boot集成Spring Security的demo进行讲解。

## 1.主要功能

- [x] 基于 `RBAC` 权限模型设计，详情参考数据库表结构设计 [`security.sql`](./sql/security.sql)
- [x] 支持**动态权限管理**，详情参考 [`RbacAuthorityService.java`](./src/main/java/com/xkcoding/rbac/security/config/RbacAuthorityService.java)
- [x] **登录 / 登出**部分均使用自定义 Controller 实现，未使用 `Spring Security` 内部默认的实现，适用于前后端分离项目，详情参考 [`SecurityConfig.java`](./src/main/java/com/xkcoding/rbac/security/config/SecurityConfig.java) 和 [`AuthController.java`](./src/main/java/com/xkcoding/rbac/security/controller/AuthController.java)
- [x] 持久化技术使用 `spring-data-jpa` 完成
- [x] 使用 `JWT` 实现安全验证，同时引入 `Redis` 解决 `JWT` 无法手动设置过期的弊端，并且保证同一用户在同一时间仅支持同一设备登录，不同设备登录会将，详情参考 [`JwtUtil.java`](./src/main/java/com/xkcoding/rbac/security/util/JwtUtil.java)
- [x] 在线人数统计，详情参考 [`MonitorService.java`](./src/main/java/com/xkcoding/rbac/security/service/MonitorService.java) 和 [`RedisUtil.java`](./src/main/java/com/xkcoding/rbac/security/util/RedisUtil.java)
- [x] 手动踢出用户，详情参考 [`MonitorService.java`](./src/main/java/com/xkcoding/rbac/security/service/MonitorService.java)
- [x] 自定义配置不需要进行拦截的请求，详情参考 [`CustomConfig.java`](./src/main/java/com/xkcoding/rbac/security/config/CustomConfig.java) 和 [`application.yml`](./src/main/resources/application.yml)

# 2.代码解析

### 2.1 POM文件解析（Maven项目）

[POM文件解析](C:\Users\汤琛\Desktop\学习资料\Spring详解\Spring Security\POM文件解析.md)