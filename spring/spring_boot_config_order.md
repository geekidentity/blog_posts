---
categories: spring

tags: 
  - Spring
  - Spring Boot

title: Spring Boot 属性加载顺序

date: 2018-06-12
---
Spring Boot 属性加载顺序，优先级按从高到低的顺序，数字越小优先级越高。

1. 在命令行中传入的参数。
2. SPRING_APPLICATION_JSON 中的属性。SPRING_APPLICATION_JSON 是以JSON 格式配置在系统环境变量中的内容。
3. java:comp/env 中的JNDI 属性。
4. Java的系统属性，可以通过System.getProperties() 获得的内容。
5. 操作系统的环境变量。
6. 通过random.* 配置的随机属性。
7. 位于当前应用jar 包之外，针对不同{profile}环境的配置文件内容，例如 application-{profile}.properties 或是YAML定义的配置文件。
8. 位于当前应用jar 包之内，针对不同{profile}环境的配置文件内容，例如 application-{profile}.properties 或是YAML定义的配置文件。
9. 位于当前应用jar 包之外的application.properties 和YAML 配置内容。
10. 位于当前应用jar 包之内的application.properties 和YAML 配置内容。
11. 在@Configuration 注解修改的类中，通过@PropertySource 注解定义的属性。
12. 应用默认属性，使用 SpringApplication.setDefaultProperties 定义的内容。