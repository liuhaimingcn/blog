title: Spring Boot 中使用 Jedis 来操作 Redis
date: 2016-10-27  
tags:
    - original
categories:
    - Spring Boot
---

把之前老的项目切换到用 Spring Boot 时，由于抛弃了 xml 配置文件的使用，需要把之前 Jedis 配置现在用注解的形式重新实现一遍。  

### 老的代码
* config.properties

``` properties
# redis数据库连接配置(covert)
redis.url=redis://:name@host:6379/2
# 最大实例数
redis.maxTotal=100
# 最大空闲实例数
redis.maxIdle=10
# (创建实例时)最大等待时间
redis.maxWaitMillis=10000
# (创建实例时)是否验证
redis.testOnBorrow=true
```

<!-- more -->

* spring.xml

``` xml
<!--加载外部数据库配置-->
<context:property-placeholder location="classpath:config.properties" file-encoding="utf-8" ignore-unresolvable="true"/>

<!-- 配置redis池，依次为最大实例数，最大空闲实例数，(创建实例时)最大等待时间，(创建实例时)是否验证 -->
<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
    <property name="maxTotal" value="${redis.maxTotal}"/>
    <property name="maxIdle" value="${redis.maxIdle}"/>
    <property name="maxWaitMillis" value="${redis.maxWaitMillis}"/>
    <property name="testOnBorrow" value="${redis.testOnBorrow}"/>
</bean>
<bean id="convertShardInfo" class="redis.clients.jedis.JedisShardInfo">
    <constructor-arg name="host" value="${redis.url}"/>
</bean>
<bean id="convertJedisPool" class="redis.clients.jedis.ShardedJedisPool">
    <constructor-arg index="0" ref="jedisPoolConfig"/>
    <constructor-arg index="1">
        <list>
            <ref bean="convertShardInfo"/>
        </list>
    </constructor-arg>
</bean>
```

### 新的代码

* application.yml

``` yml
# redis数据库连接配置(covert)
redisConfig:
  url: "redis://:name@host:6379/2"
  # 最大实例数
  maxTotal: 100
  # 最大空闲实例数
  maxIdle: 10
  # (创建实例时)最大等待时间
  maxWaitMillis: 10000
  # (创建实例时)是否验证
  testOnBorrow: true
```

* RedisConfig.java

``` java
package com.raventech.web.models.yml;

@Component
@ConfigurationProperties(prefix = "redisConfig")
public class RedisConfig implements Serializable {
    private static final long serialVersionUID = 1097752157567754456L;
    private String url;
    private Integer maxTotal;
    private Integer maxIdle;
    private Long maxWaitMillis;
    private Boolean testOnBorrow;
    ......
```

* JedisConfiguration.java

``` java
@Configuration
@ComponentScan({"com.raventech.web.models.yml"}) // 解决 Configuration 注解中使用 Autowired 注解 IDE 报错
public class JedisConfiguration {
	@Autowired
	RedisConfig redisConfig;

	@Bean
	public ShardedJedisPool convertJedisPool() {
		JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
		jedisPoolConfig.setMaxTotal(redisConfig.getMaxTotal());
		jedisPoolConfig.setMaxIdle(redisConfig.getMaxIdle());
		jedisPoolConfig.setMaxWaitMillis(redisConfig.getMaxWaitMillis());
		jedisPoolConfig.setTestOnBorrow(redisConfig.getTestOnBorrow());
		List<JedisShardInfo> jedisShardInfoList = new ArrayList<>();
		jedisShardInfoList.add(new JedisShardInfo(redisConfig.getUrl()));
		return new ShardedJedisPool(jedisPoolConfig, jedisShardInfoList);
	}
}
```

### 应用

```
@Autowired
private ShardedJedisPool convertJedisPool;

public String convertRedisGet(String key) {
	ShardedJedis resource = convertJedisPool.getResource();
	String result = resource.get(key);
	resource.close();
	return result;
}
}
```


<br>