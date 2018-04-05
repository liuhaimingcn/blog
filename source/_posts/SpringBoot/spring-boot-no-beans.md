title: Configuration 注解中使用 Autowired 注解 IDE 报错
date: 2016-10-26  
tags:
    - original
categories:
    - Spring Boot  
---

在 Spring Boot 项目中会用 @Configuration 注解来初始化配置, 这时可以通过 @autowired 自动注入封装好的model对象, 方便使用yml中的配置的数据。	 
这样做代码运行没问题，通过该对象也可以成功的获取yml配置文件中的数据，但是 IDE 却给出 “Could not autowird. No beans of'RedisConfig' type found.” 的错误提示。	

![](http://7xia33.com1.z0.glb.clouddn.com/2016-10-26%20at%2016.18.png)  

<!-- more -->

我们手动的在 @Configuration 注解下面添加 @ComponentScan 注解并指定所需model类的包地址就可以解决整个问题了。	  
原因估计是因为在项目的启动的最初阶段，IDE 还没有扫描到model类，无法发现对应的 bean，于是就需要我们手动的给其指定需要扫描的包了。	  

![](http://7xia33.com1.z0.glb.clouddn.com/2016-10-26%20at%2016.19.png)

<br>