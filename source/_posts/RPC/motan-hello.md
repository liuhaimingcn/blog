title: RPC框架motan使用
date: 2016-11-10  
tags:
    - original
    - motan
categories:
    - RPC
---

## 简介
* motan是新浪微博开源的一套轻量级、方便使用的RPC框架
* 项目地址：https://github.com/weibocom/motan

<!-- more -->

## Hello World

* 使用的过程分为Server端和Client端，Server提供RCP的服务接口，Client端发起调用获取结果。
* maven的pom文件配置  

``` xml
<properties>
    <motan.version>0.2.1</motan.version>
</properties>

<dependencies>
    <dependency>
        <groupId>com.weibo</groupId>
        <artifactId>motan-core</artifactId>
        <version>${motan.version}</version>
    </dependency>
    <dependency>
        <groupId>com.weibo</groupId>
        <artifactId>motan-transport-netty</artifactId>
        <version>${motan.version}</version>
    </dependency>
    <dependency>
        <groupId>com.weibo</groupId>
        <artifactId>motan-springsupport</artifactId>
        <version>${motan.version}</version>
    </dependency>
</dependencies>
```

### Server 端
* 暴露的接口
``` java
package com.raventech.user.motan;

/**
 * @author liuhaiming on 10/11/2016.
 */
public interface HelloService {
  String hello(String world);
}
```
* 暴露接口的实现类
``` java
package com.raventech.user.motan;

/**
 * @author liuhaiming on 10/11/2016.
 */
public class HelloServiceImpl implements HelloService {
  @Override
  public String hello(String world) {
    return "hello " + world;
  }
}
```
* xml配置文件，暴露接口
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:motan="http://api.weibo.com/schema/motan"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
        http://api.weibo.com/schema/motan http://api.weibo.com/schema/motan.xsd">

    <bean id="helloServiceImpl" class="com.raventech.user.motan.HelloServiceImpl"/>
    <motan:service interface="com.raventech.user.motan.HelloService" ref="helloServiceImpl" export="8002"/>
</beans>
```
* 启动服务的方法（运行main方法就可以启动服务了）
``` java
package com.raventech.user;

import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * @author liuhaiming on 10/11/2016.
 */
public class Server {
  public static void main(String[] args) {
    new ClassPathXmlApplicationContext("classpath:motan-server.xml");
    System.out.println("Server start ...");
  }
}
```

### Client 端
* 要请求的接口（不论包名还是类名都要和Server端的一样）
``` java
package com.raventech.user.motan;

/**
 * @author liuhaiming on 10/11/2016.
 */
public interface HelloService {
  String hello(String world);
}
```
* xml配置文件，获取接口信息
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:motan="http://api.weibo.com/schema/motan"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
        http://api.weibo.com/schema/motan http://api.weibo.com/schema/motan.xsd">

    <motan:referer id="helloService" interface="com.raventech.user.motan.HelloService" directUrl="127.0.0.1:8002"/>
</beans>
```
* 调用服务的方法（运行main方法就可以调用服务了）
``` java
package com.raventech.web;

import com.raventech.user.motan.HelloService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * @author liuhaiming on 10/11/2016.
 */
public class Client {
  public static void main(String[] args) {
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:motan-client.xml");
    HelloService fooService = (HelloService) applicationContext.getBean("helloService");
    System.out.println(fooService.hello("world"));
  }
}
```
* 调用响应结果
  ![](http://7xia33.com1.z0.glb.clouddn.com/2016-11-09%20at%2017.45.png)

## 使用Consul作为注册中心
* 在集群环境下使用motan需要依赖Consul等服务发现组件
* [Consul的介绍安装和使用](https://www.google.com)
* maven的pom文件配置(在上面的基础上增加consul)
``` xml
<dependency>
    <groupId>com.weibo</groupId>
    <artifactId>motan-registry-consul</artifactId>
    <version>${motan.version}</version>
</dependency>
```

### Server 端
* xml配置文件添加consul的注册
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:motan="http://api.weibo.com/schema/motan"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
        http://api.weibo.com/schema/motan http://api.weibo.com/schema/motan.xsd">

    <motan:registry regProtocol="consul" name="registry" address="127.0.0.1:8500"/>

    <bean id="helloServiceImpl" class="com.raventech.user.motan.HelloServiceImpl"/>
    <motan:service interface="com.raventech.user.motan.HelloService" ref="helloServiceImpl" registry="registry"  export="8002"/>
</beans>
```
* 启动服务的方法要在程序启动后调用心跳开关，将服务注册到consul，不然Client无法调用 （别的和上文Hello World一样不变，运行main方法启动服务）
``` java
package com.raventech.user;

import com.weibo.api.motan.common.MotanConstants;
import com.weibo.api.motan.util.MotanSwitcherUtil;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * @author liuhaiming on 10/11/2016.
 */
public class Server {
  public static void main(String[] args) {
    new ClassPathXmlApplicationContext("classpath:motan-server.xml");
    MotanSwitcherUtil.setSwitcherValue(MotanConstants.REGISTRY_HEARTBEAT_SWITCHER, true);
    System.out.println("Server start ...");
  }
}
```

### Client 端
* xml配置文件添加consul的服务发现
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:motan="http://api.weibo.com/schema/motan"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
        http://api.weibo.com/schema/motan http://api.weibo.com/schema/motan.xsd">

    <motan:registry regProtocol="consul" name="registry" address="127.0.0.1:8500"/>

    <motan:referer id="helloService" interface="com.raventech.user.motan.HelloService" registry="registry"/>
</beans>
```
* 别的和上文Hello World一样不变，运行Client类的main方法调用服务

## 使用注解的方式集成到Spring Boot项目中
* 项目改成Spring Boot后抛弃了繁琐的xml文件配置改为用注解的方式。motan也支持注解的方式进行配置，这样更加方便了代码的集成和风格的统一。
* 继续在前面的代码中进行修改，没提到的保持不变

### Server 端
* 删除motan-server.xml配置文件
* 用注解加载motan需要的配置
``` java
package com.raventech.user.config;

import com.weibo.api.motan.config.springsupport.AnnotationBean;
import com.weibo.api.motan.config.springsupport.BasicServiceConfigBean;
import com.weibo.api.motan.config.springsupport.ProtocolConfigBean;
import com.weibo.api.motan.config.springsupport.RegistryConfigBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author liuhaiming on 10/11/2016.
 */
@Configuration
public class MotanConfiguration {

  @Bean
  public AnnotationBean motanAnnotationBean() {
    AnnotationBean motanAnnotationBean = new AnnotationBean();
    motanAnnotationBean.setPackage("com.raventech.user.motan");
    return motanAnnotationBean;
  }

  @Bean(name = "motan")
  public ProtocolConfigBean protocolConfig1() {
    ProtocolConfigBean config = new ProtocolConfigBean();
    config.setDefault(true);
    config.setName("motan");
    config.setMaxContentLength(1048576);
    return config;
  }

  @Bean(name = "registry")
  public RegistryConfigBean registryConfig() {
    RegistryConfigBean config = new RegistryConfigBean();
    config.setRegProtocol("consul");
    config.setAddress("127.0.0.1:8500");
    return config;
  }

  @Bean
  public BasicServiceConfigBean baseServiceConfig() {
    BasicServiceConfigBean config = new BasicServiceConfigBean();
    config.setExport("motan:8002");
    config.setRegistry("registry");
    return config;
  }
}
```

* 暴露接口的实现类加上@MotanService注解，自动生成bean
``` java
package com.raventech.user.motan;

import com.weibo.api.motan.config.springsupport.annotation.MotanService;

/**
 * @author liuhaiming on 10/11/2016.
 */
@MotanService
public class HelloServiceImpl implements HelloService {
  @Override
  public String hello(String world) {
    return "hello " + world;
  }
}
```

* 启动服务的方法就是启动Spring Boot项目，并在在程序启动后调用心跳开关 (运行main方法启动服务)
``` java
package com.raventech.user;

import com.weibo.api.motan.common.MotanConstants;
import com.weibo.api.motan.util.MotanSwitcherUtil;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @author liuhaiming on 10/11/2016.
 */
@SpringBootApplication
public class Server {
  public static void main(String[] args) {
    SpringApplication.run(Server.class, args);
    MotanSwitcherUtil.setSwitcherValue(MotanConstants.REGISTRY_HEARTBEAT_SWITCHER, true);
    System.out.println("Server start ...");
  }
}
```

### Client 端
* 删除motan-client.xml配置文件和Client.java启动文件，已经没用了
* 用注解加载motan需要的配置
``` java
package com.raventech.web.config;

import com.weibo.api.motan.config.springsupport.AnnotationBean;
import com.weibo.api.motan.config.springsupport.BasicRefererConfigBean;
import com.weibo.api.motan.config.springsupport.ProtocolConfigBean;
import com.weibo.api.motan.config.springsupport.RegistryConfigBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author liuhaiming on 10/11/2016.
 */
@Configuration
public class MotanConfiguration {

  @Bean
  public AnnotationBean motanAnnotationBean() {
    AnnotationBean motanAnnotationBean = new AnnotationBean();
    # 添加用到motan注解的类的包名
    motanAnnotationBean.setPackage("com.raventech.web.controller");
    return motanAnnotationBean;
  }

  @Bean(name = "motan")
  public ProtocolConfigBean protocolConfig1() {
    ProtocolConfigBean config = new ProtocolConfigBean();
    config.setDefault(true);
    config.setName("motan");
    config.setMaxContentLength(1048576);
    return config;
  }

  @Bean(name = "registry")
  public RegistryConfigBean registryConfig() {
    RegistryConfigBean config = new RegistryConfigBean();
    config.setRegProtocol("consul");
    config.setAddress("127.0.0.1:8500");
    return config;
  }

  @Bean(name = "basicRefererConfig")
  public BasicRefererConfigBean basicRefererConfigBean() {
    BasicRefererConfigBean config = new BasicRefererConfigBean();
    config.setProtocol("motan");
    config.setRegistry("registry");
    config.setThrowException(true);
    return config;
  }
}
```
* 调用方法（在Controller中使用）
``` java
package com.raventech.web.controller;

import com.raventech.user.motan.HelloService;
import com.raventech.web.common.BaseController;
import com.weibo.api.motan.config.springsupport.annotation.MotanReferer;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author liuhaiming on 10/11/2016.
 */
@RestController
@RequestMapping("/motan")
public class HelloController extends BaseController {

  @MotanReferer(basicReferer = "basicRefererConfig")
  private HelloService helloService;

  @RequestMapping(value = "/hello", method = RequestMethod.GET)
  public String hello() throws Exception {
    return helloService.hello("world");
  }
}
```
* 启动Spring Boot项目，在浏览器中访问http://127.0.0.1:8080/motan/hello就可以获取验证结果

<br>