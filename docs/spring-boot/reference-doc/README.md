# Spring Boot参考指南

> [Spring Boot参考指南 v2.1.1.RELEASE](https://springcloud.cc/spring-boot.html)
>
> [Spring Boot Reference Guide v2.1.4.RELEASE](https://docs.spring.io/spring-boot/docs/2.1.4.RELEASE/reference/htmlsingle/)
>
> 备注：本指南旨在挑选出实际项目中涉及部分，优化阅读体验，无意改编原文。

## 目录

### Ⅰ. Spring Boot 文件

- **1.关于文档**
- **2.获得帮助**
- **3.第一步**
- **4.使用 Spring Boot**
- **5.了解 Spring Boot 功能**
- **6.转向生产**
- **7.高级主题**

### Ⅱ. 入门

- **8.介绍 Spring Boot**
- **9.系统要求**
- **10.安装 Spring Boot**
- **11.开发你的第一个 Spring Boot 应用程序**
- **12.接下来要阅读的内容**

### III. 使用 Spring Boot

- **13.构建系统**
- **14.构建你的代码**
- **15.配置类**
- **16.自动配置**
- **17.Spring Beans 和依赖注入**
- **18.使用 @SpringBootApplication 注解**
- **19.运行你的应用程序**
- **20.开发人员工具**
- **21.包装你的生产应用程序**
- **22.接下来要阅读的内容**

### IV. Spring Boot 功能

- **23.Spring Application**
- **24.外部配置**
- **25.简介**
- **26.记录**
- **27.JSON**
- **28.开发 Web 应用程序**
- **29.安全**
- **30.使用 SQL 数据库**
- **[31.使用 NoSQL 技术](./31.使用NoSQL技术.md)**
  - 31.1 Redis
- **32.缓存**
- **33.消息传递**
- **34.使用 RestTemplate 调用 REST 服务**
- **35.使用 WebClient 调用REST服务**
- **36.验证**
- **37.发送电子邮件**
- **38.使用 JTA 分布式事务**
- **39.Hazelcast**
- **40.Quartz 调度器**
- **41.任务执行和调度**
- **42.Spring 集成**
- **43.Spring Session**
- **44.对 JMX 的检测和管理**
- **45.测试**
- **46.WebSockets**
- **47.网络服务**
- **48.使用 WebServiceTemplate 调用 Web 服务**
- **49.创建自己的自动配置**
- **50.Kotlin 的支持**
- **51.接下来要阅读的内容**

### V. Spring Boot Actuator：生产就绪功能

- **52.启动生产就绪功能**
- **53.终点**
- **54.通过 HTTP 进行监控和管理**
- **55.对 JMX 的监测和管理**
- **56.记录器**
- **57.度量标准**
- **58.审计**
- **59.HTTP 跟踪**
- **60.过程检测**
- **61.Cloud Foundry 支持**
- **62.接下来要阅读的内容**

### VI. 部署 Spring Boot 应用程序

- **63.部署到云端**
- **64.安装 Spring Boot 应用程序**
- **65.接下来要阅读的内容**

### Ⅶ. Spring Boot CLI

- **66.安装 CLI**
- **67.使用 CLI**
- **68.使用 Groovy Beans DSL 开发应用程序**
- **69.使用  settings.xml 配置 CLI**
- **70.接下来要阅读的内容**

### Ⅷ. 构建工具插件

- **71.Spring Boot Maven 插件**
- **72.Spring Boot Gradle 插件**
- **73.Spring Boot AntLib 模块**
- **74.支持其他构建系统**
- **75.接下来要阅读的内容**

### Ⅸ. 'How-to' 指南

- **76.Spring Boot 申请**
- **77.属性和配置**
- **78.嵌入式 Web 服务器**
- **79.Spring MVC**
- **80.使用 Spring 安全性进行测试**
- **81.Jersey**
- **82.HTTP 客户端**
- **83.记录**
- **84.数据访问**
- **85.数据库初始化**
- **86.消息传递**
- **87.批量申请**
- **88.执行器**
- **89.安全**
- **90.热插拔**
- **91.建立**
- **92.传统部署**
  - 92.4 使用 Jedis 代替 Lettuce

### Ⅹ.  附录

- **A.常见应用程序属性**
- **B.配置元数据**
- **C.自动配置类**
- **D.测试自动配置注释**
- **E.可执行的Jar格式**
- **F.依赖版本**
