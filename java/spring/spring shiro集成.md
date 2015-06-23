# spring shiro集成
## spring boot 继承注意点
要在webconfig 中注册一个shiroFilter且需要放在第一个

WebConfig.java
``` java
……
@Bean
public FilterRegistrationBean shiroFilter() {
	FilterRegistrationBean reg = new FilterRegistrationBean();
	reg.setFilter(new DelegatingFilterProxy(BeanNameConstants.SHIRO_FILTER_BEAN));
	reg.addUrlPatterns("/*");
	reg.setDispatcherTypes(DispatcherType.FORWARD, DispatcherType.INCLUDE, DispatcherType.REQUEST);
	return reg;
}
……
```

shiroSecurityConfig.java
``` java
……
@Bean(name = BeanNameConstants.SHIRO_FILTER_BEAN)
public ShiroFilterFactoryBean shiroFilterBean() {
	ShiroFilterFactoryBean shiroFilter = new ShiroFilterFactoryBean();
	Map<String, String> filterChainDefinitionMapping = new HashMap<String, String>();

	// filterChainDefinitionMapping.put("/*", "authc");
	filterChainDefinitionMapping.put("/admin/*", "authc");
	filterChainDefinitionMapping.put("/logout", "logout");

	shiroFilter.setLoginUrl("/login");
	shiroFilter.setSuccessUrl("/index");
	shiroFilter.setFilterChainDefinitionMap(filterChainDefinitionMapping);

	shiroFilter.setSecurityManager(securityManager());
	return shiroFilter;
}
……
```

## pom.xml
``` xml
<!-- SECURITY begin -->
<dependency>
	<groupId>org.apache.shiro</groupId>
	<artifactId>shiro-spring</artifactId>
	<version>${shiro.version}</version>
</dependency>

<dependency>
	<groupId>org.apache.shiro</groupId>
	<artifactId>shiro-cas</artifactId>
	<version>${shiro.version}</version>
	<exclusions>
		<exclusion>
			<groupId>commons-logging</groupId>
			<artifactId>commons-logging</artifactId>
		</exclusion>
	</exclusions>
</dependency>

<dependency>
	<groupId>org.apache.shiro</groupId>
	<artifactId>shiro-ehcache</artifactId>
	<version>${shiro.version}</version>
</dependency>

<dependency>
	<groupId>commons-codec</groupId>
	<artifactId>commons-codec</artifactId>
	<version>1.10</version>
</dependency>
<!-- SECURITY end -->
```
## web.xml
``` xml
<!-- Apache Shiro -->
<filter>   
    <filter-name>shiroFilter</filter-name>  
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>  
    <init-param>  
         <param-name>targetFilterLifecycle</param-name>   
         <param-value>true</param-value>   
    </init-param>   
</filter>

<filter-mapping>
	<filter-name>shiroFilter</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
```

## spring-context-shiro.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
		http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context-4.0.xsd"
	default-lazy-init="true">

	<description>Shiro Configuration</description>

	<!-- 加载配置属性文件 -->
	<context:property-placeholder
		ignore-unresolvable="true" location="classpath:/application.properties" />

	<!-- Shiro权限过滤过滤器定义 -->
	<bean name="shiroFilterChainDefinitions" class="java.lang.String">
		<constructor-arg>
			<value>
				/login = authc
				/logout = logout
				/admin/** = authc
			</value>
		</constructor-arg>
	</bean>
	
	<bean id="sessionIdCookie" class="org.apache.shiro.web.servlet.SimpleCookie">
	    <constructor-arg name="name" value="sun-snow.session.id"/>
	</bean>

	<!-- 安全认证过滤器 -->
	<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
		<property name="securityManager" ref="securityManager" />
		<property name="loginUrl" value="/login" />
		<property name="successUrl" value="/?login" />
		<property name="filters">
			<map>
				<entry key="authc" value-ref="formAuthenticationFilter" />
				<entry key="logout" value-ref="logoutFilter"/>
			</map>
		</property>
		<property name="filterChainDefinitions">
			<ref bean="shiroFilterChainDefinitions" />
		</property>
	</bean>
	
	<!-- 重定向登出跳转页面，如果不指定默认路径为 “/”  -->
	<bean id="logoutFilter" class="org.apache.shiro.web.filter.authc.LogoutFilter">
		<property name="redirectUrl" value="/login"/>
	</bean>
	
	<bean id="formAuthenticationFilter" class="org.apache.shiro.web.filter.authc.FormAuthenticationFilter" />

	<bean id="cacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager">
		<!-- <property name="cacheManagerConfigFile" value="classpath:/../configs/cache/ehcache.xml" /> -->
	</bean>

	<bean id="sunAuthorizingRealm" class="org.sun.snow.web.security.SunAuthorizingRealm" >
		<property name="userService" ref="userService"/>
		<property name="roleService" ref="roleService"/>
	</bean>

	<!-- session id 生成器 -->
	<bean id="sessionIdGenerator"
		class="org.apache.shiro.session.mgt.eis.JavaUuidSessionIdGenerator" />

	<bean id="sessionDAO"
		class="org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO">
		<property name="activeSessionsCacheName" value="shiro-activeSessionCache" />
		<property name="sessionIdGenerator" ref="sessionIdGenerator" />
	</bean>

	<bean id="sessionManager"
		class="org.apache.shiro.web.session.mgt.DefaultWebSessionManager">
		<property name="sessionDAO" ref="sessionDAO" />
		<!-- 会话超时时间，单位：毫秒 -->
		<property name="globalSessionTimeout" value="1800000" />
		<property name="deleteInvalidSessions" value="true" />
		<property name="sessionValidationSchedulerEnabled" value="true" />
		<property name="sessionIdCookie" ref="sessionIdCookie" />
		<property name="sessionIdCookieEnabled" value="true" />
	</bean>

	<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
		<property name="realms">
			<list>
				<ref bean="sunAuthorizingRealm" />
			</list>
		</property>
		<property name="sessionManager" ref="sessionManager" />
		<property name="cacheManager" ref="cacheManager" />
	</bean>

	<!-- 相当于调用SecurityUtils.setSecurityManager(securityManager) -->
	<bean
		class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
		<property name="staticMethod"
			value="org.apache.shiro.SecurityUtils.setSecurityManager" />
		<property name="arguments" ref="securityManager" />
	</bean>

	<!-- Shiro生命周期处理器 -->
	<bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor" />
</beans>
```

## realm
``` java
public class SunAuthorizingRealm extends AuthorizingRealm {
	@Autowired
	private UserService userService;
	@Autowired
	private RoleService roleService;

	private static final Logger logger = LoggerFactory.getLogger(SunAuthorizingRealm.class);

	/**
	 * 授权
	 */
	@Override
	protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
		if (principals == null)
			throw new AuthorizationException("principals 不能为空");

		String username = (String) getAvailablePrincipal(principals);

		SimpleAuthorizationInfo info = null;
		if (StringUtils.isNotBlank(username)) {
			info = new SimpleAuthorizationInfo();

			List<Role> roles = roleService.getAllRoleByLoginName(username);

			for (Role role : roles) {
				info.addRole(role.getRoleName());
			}

		}

		return info;
	}

	/**
	 * 身份验证/登录
	 */
	@Override
	protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken auchcToken) throws AuthenticationException {
		if (auchcToken == null)
			throw new AuthenticationException("权限验证失败");

		UsernamePasswordToken token = (UsernamePasswordToken) auchcToken;

		if (StringUtils.isBlank(token.getUsername()))
			throw new AuthenticationException("用户名不能为空");

		User user = null;

		user = userService.getUserByLoginName(token.getUsername());

		// 交给AuthenticatingRealm使用CredentialsMatcher进行密码匹配
		SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(user.getLoginName(), // 用户名
				user.getPassword().toCharArray(), // 密码
				// ByteSource.Util.bytes(user.getName()),// salt=username+salt(这里为了方便，使用name作为salt)
				getName() // realm name
		);

		return authenticationInfo;
	}
}
```







