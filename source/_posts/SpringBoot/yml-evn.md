title: yml 文件中使用环境变量
date: 2016-10-25  
tags:
    - original
categories:
    - Spring Boot  
---

Spring Boot 中可以用 spring.profiles.active 参数来指定系统环境，让系统加载不同的配置文件。  
可以在程序启动的时候加上参数来指定需要的配置  

```
java -Dspring.profiles.active="dev" -jar user.jar
```

<!-- more -->

当然我们也可以事先设置好系统的环境变量

```
expoer SERVER_EVN=test
```

然后在 yml 文件中用 active: ${SERVER_EVN} 来动态的获取系统已设置好的数据。这样这台 test 服务器中的再启动 Spring Boot 项目的时候就可以不用每次都去设置参数了。  

同时 yml 也支持 ${SERVER_EVN:dev} 这样的方式来设置默认值，此时如果环境变量中没有 SERVER_EVN ， active就会默认设置为"dev"。  

<br>