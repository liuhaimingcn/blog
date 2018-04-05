title: Spring Boot 项目中只能有一个main方法
date: 2016-10-24  
tags:
    - original
categories:
    - Spring Boot  
---

对Spring Boot 项目用maven进行打包的时候报错以下错误  

[ERROR] Failed to execute goal org.springframework.boot:spring-boot-maven-plugin:1.4.1.RELEASE:repackage (default) on project nlp-user: Execution default of goal org.springframework.boot:spring-boot-maven-plugin:1.4.1.RELEASE:repackage failed: Unable to find a single main class from the following candidates [com.raventech.user.Application, com.raventech.user.util.Utils] -> [Help 1]  

<!-- more -->

原来是因为 Spring Boot 项目中只能有一个main方法，不然 spring-boot-maven-plugin 在打包的过程中会扫描到了多个 main 方法，然后就懵逼不知道用哪个作为启动方法了。  

以前总喜欢在 Utils 中写个main方法来调试静态方法，看来以后用完就得随手把它给删除了。

<br>