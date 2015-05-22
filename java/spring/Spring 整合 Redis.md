# Spring 整合 Redis

---
## redis简介与小结

### 小结
- 在使用spring 注解时要特别注意，注解必须加载直接调用的方法上，如果使用嵌套的方式则注解会失效！！！！！

``` java
	@CacheEvict(value = "userCache", key = RedisKeyPrefixConstants.FXT_USER_LOGIN_NAME_PREFIX + "#loginName", beforeInvocation = true)
	public void cleanUserCache(String loginName) {
		logger.debug("成功清除userCache:" + loginName);
	}
    
    public void testPublicClean(String loginName) {
		cleanUserCache(loginName);
	}
    
    
    // 无法清除缓存
    @Test
	public void testPublicCleanCache() {
		final String loginName = "xxxxx";
		userService.testPublicClean(loginName);
	}
```

### Serializer
　因为redis是以key-value的形式将数据存在内存中，key就是简单的string，key似乎没有长度限制，不过原则上应该尽可能的短小且可读性强，无论是否基于持久存储，key在服务的整个生命周期中都会在内存中，因此减小key的尺寸可以有效的节约内存，同时也能优化key检索的效率。
　　value在redis中，存储层面仍然基于string，在逻辑层面，可以是string/set/list/map，不过redis为了性能考虑，使用不同的“encoding”数据结构类型来表示它们。(例如：linkedlist，ziplist等)。
　　所以可以理解为，其实redis在存储数据时，都把数据转化成了byte[]数组的形式，那么在存取数据时，需要将数据格式进行转化，那么就要用到序列化和反序列化了，这也就是为什么需要配置Serializer的原因。

SDR支持的序列化策略
> * JdkSerializationRedisSerializer
> * StringRedisSerializer
> * JacksonJsonRedisSerializer
> * OxmSerializer

其中JdkSerializationRedisSerializer和StringRedisSerializer是最基础的序列化策略，其中“JacksonJsonRedisSerializer”与“OxmSerializer”都是基于stirng存储，因此它们是较为“高级”的序列化(最终还是使用string解析以及构建java对象)。
　基本推荐使用JdkSerializationRedisSerializer和StringRedisSerializer，因为其他两个序列化策略使用起来配置很麻烦，如果实在有需要序列化成Json和XML格式，可以使用java代码将String转化成相应的Json和XML
### redisTemplate

---

## pom.xml
``` xml
<!--spring-session-data-redis已经包含redis及和spring继承所需的所有包-->
<dependency>
	<groupId>org.springframework.session</groupId>
	<artifactId>spring-session-data-redis</artifactId>
	<version>1.0.0.RELEASE</version>
	<type>pom</type>
</dependency>
```
### spring-context-cache.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:cache="http://www.springframework.org/schema/cache"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/cache http://www.springframework.org/schema/cache/spring-cache.xsd"
	default-lazy-init="true">
	
	<context:property-placeholder location="classpath:/redis.properties" ignore-unresolvable="true" />	
	
	<bean id="redisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
		<property name="hostName" value="${redis.host}"/>
		<property name="password" value="${redis.password}"/>
		<property name="port" value="${redis.port}"/>
	</bean>
	
	<!--替代默认serializer 增加性能-->
	<bean id="stringRedisSerializer" class="org.springframework.data.redis.serializer.StringRedisSerializer"/>
	
	<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
		<property name="connectionFactory" ref="redisConnectionFactory"/>
		<property name="keySerializer" ref="stringRedisSerializer"/>
	</bean>
	
	<!-- 如果自定义了redisTemplate 需要增加以下配置 -->
	<bean id="redisCacheManager" class="org.springframework.data.redis.cache.RedisCacheManager" >
		<constructor-arg name="template" ref="redisTemplate"/>
	</bean>
	
	<!-- !important 启动注解 -->
	<cache:annotation-driven cache-manager="redisCacheManager"/>	
</beans>
```
## redis.properties
``` properties
# Redis settings  
redis.host=192.168.135.128
redis.port=6379
redis.password=  
```
## JAVA code
``` java
/**
*使用缓存
*/
@Cacheable(value = "userCache", key = "'user:loginName:'+#loginName")
public User getUserByLoginName(String loginName) {
	Assert.notNull(loginName, "登录名不能为空");
	return userDao.getUserByLoginName(loginName);
}

/**
* 清除缓存
*/
@CacheEvict(value = "userCache", key = "'user:'+#user.id")
public void saveOrUpdate(User user) {
	Assert.notNull(user);
	if (StringUtils.isBlank(user.getId())) {
		user.setId(UUIDUtil.generateUUID());
		userDao.insert(user);
	} else {
		userDao.updateByPrimaryKeySelective(user);
	}
}
```
> * key中""内一定要加''，否则无法识别






