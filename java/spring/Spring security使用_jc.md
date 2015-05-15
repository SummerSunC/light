# Spring security


---
## spring security 个人总结
- spring security 登录成功/失败都是通过handler来完成具体操作的，handler依赖filter来拦截请求。用户自定的handler在每个filter中执行的顺序是最后一个。自定义filter 可以通过addFilterAfter/before来指定顺序
- spring security 的核心是**AuthenticationManager** ，但是**AuthenticationManager** 只是一个接口，我们依赖于具体实现，默认实现spring security 的是 **ProviderManager** ，**ProviderManager**包含了一个**AuthenticationProvider**列表，这里只要有一个AuthenticationProvider **不通过则验证通过**，所以我们在自定义鉴权的时候只要实现**AbstractUserDetailsAuthenticationProvider**验证就好了，spring security默认的使用**DaoAuthenticationProvider** 这里所有的鉴权都是通过spring security **UserDetails**对象实现的。
- 我们通过 UserDetailsService loadUserByUsername 加载对象和分配权限
### 槽点
> 刚开始我是这样写的
> ``` java
> //不生效，使用的successHandler还是默认的SavedRequestAwareAuthenticationSuccessHandler
> .and().formLogin().successHandler(loginSuccessHandler).loginPage("/wap/login").permitAll().defaultSuccessUrl("/wap/")
> //之后改成这样,尼玛 就能运行了，顺序很重要
> .and().formLogin().loginPage("/wap/login").permitAll().defaultSuccessUrl("/wap/").successHandler(loginSuccessHandler)
> ```


## config
``` java
@Configuration
@Order(SecurityProperties.ACCESS_OVERRIDE_ORDER)
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecureConfig extends WebSecurityConfigurerAdapter {

	@Autowired
	private WapUserDetailsService detailsService;

	@Autowired
	private SecurityProperties security;

	@Autowired
	private DataSource dataSource;
	
	@Autowired
	private WapLogoutHandler logoutHandler;
	
	@Autowired
	private WapAuthenticationProvider wapAuthProvider;
	
	@Autowired
	private WapLoginSuccessHandler loginSuccessHandler;
	
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.exceptionHandling().accessDeniedPage("/wap/accessDenied")
			.and().authorizeRequests()
			.antMatchers(UrlConstants.URL_SYS + "/**").hasAuthority(RoleConstants.ROLE_SYS)
			.antMatchers("/").permitAll()
			.antMatchers("/wap/welcome").permitAll()
			.antMatchers("/wap/genCode").permitAll()
			.antMatchers("/wap/scan/**").permitAll()
			.antMatchers("/wap/task/callback/**").permitAll()
			.antMatchers("/wap/").permitAll()
			.antMatchers("/wap/task/taskDetail").permitAll()
			.antMatchers("/wap/errorCode").permitAll()
			.antMatchers("/wap/getcode").permitAll()
			.antMatchers("/wap/clearSession").permitAll()
			.antMatchers(UrlConstants.URL_WAP + "/**").hasAuthority(RoleConstants.ROLE_NORMAL)
			.anyRequest().denyAll()
			.and().formLogin().loginPage("/wap/login").permitAll().defaultSuccessUrl("/wap/").successHandler(loginSuccessHandler)
			.and().logout().addLogoutHandler(logoutHandler).logoutUrl("/logout")
			.and().sessionManagement().maximumSessions(1).expiredUrl("/wap/error/expired")
			.and()
			.and().csrf().disable();
	}

	@Override
	public void configure(WebSecurity web) throws Exception {
		web.ignoring().antMatchers("/static/**").antMatchers("/error/expired").antMatchers("/druid/**").antMatchers("/**/favicon.ico");
	}

	@Override
	public void configure(AuthenticationManagerBuilder auth) throws Exception {
		auth.authenticationProvider(defaultAuthenticationProviderBean()).authenticationProvider(wapAuthProvider);
	}

	@Bean
	public AuthenticationProvider defaultAuthenticationProviderBean() {
		DaoAuthenticationProvider authProvider = new DaoAuthenticationProvider();
		authProvider.setUserDetailsService(detailsService);
		authProvider.setPasswordEncoder(new BCryptPasswordEncoder());
		return authProvider;
	}
		
```

## logoutHandler

``` java 
@Component
public class WapLogoutHandler implements LogoutHandler {
	private static final Logger logger = LoggerFactory.getLogger(WapLogoutHandler.class);

	@Autowired
	private UserService userService;

	/**
	 * 只有在微信中登录的用户，退出登录时才会解绑。
	 */
	@Override
	public void logout(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {
		try {
			String wxOpenId = (String) request.getSession().getAttribute(WxConstants.SESSION_KEY_WX_OPEN_ID);

			if (StringUtils.isNoneBlank(wxOpenId)) {
				userService.removeWxOpenId(wxOpenId);
			}

			if (authentication == null) {
				logger.info("无登录用户");
			} else {
				logger.info(authentication.getName() + "退出登录");
			}
		} catch (Exception e) {
			logger.error(e.getMessage());
		}
	}

}
```

## AuthenticationSuccessHandler 

``` java
@Component
public class WapLoginSuccessHandler extends SavedRequestAwareAuthenticationSuccessHandler {
	@Autowired
	private UserService userService;

	@Override
	public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {

		updateWxOpenId(request, authentication);
		super.onAuthenticationSuccess(request, response, authentication);

	}

	private void updateWxOpenId(HttpServletRequest request, Authentication authentication) {
		Assert.notNull(authentication);

		String loginName = authentication.getName();

		if (StringUtils.isBlank(loginName)) {
			return;
		}

		User user = userService.findByLoginName(loginName);

		String wxOpenId = (String) request.getSession().getAttribute(WxConstants.SESSION_KEY_WX_OPEN_ID);

		if (user != null && StringUtils.isNotBlank(wxOpenId) && !wxOpenId.equals(user.getWxOpenId())) {
			user.setWxOpenId(wxOpenId);
			userService.updateUserWxOpenId(user);
			logger.info("更新" + user.getLoginName() + " wx_open_id : " + wxOpenId);
		}
	}
}
```

## AuthenticationProvider

``` java
@Component
public class WapAuthenticationProvider extends AbstractUserDetailsAuthenticationProvider {

	@Autowired
	private UserService userService;

	@Autowired
	private WapUserDetailsService userDetailsService;

	@Override
	protected void additionalAuthenticationChecks(UserDetails userDetails, UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {

	}

	@Override
	protected UserDetails retrieveUser(String username, UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {
		UserDetails loadedUser = null;

		if (StringUtils.isBlank(username)) {
			HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
			Object openId = request.getSession().getAttribute(WxConstants.SESSION_KEY_WX_OPEN_ID);
			if (openId != null) {
				User user = userService.findByWxOpenId((String) openId);
				username = user.getLoginName();
				// authentication = new
				// UsernamePasswordAuthenticationToken(username,
				// userService.genCodeSilent(username));
			}

			try {
				loadedUser = userDetailsService.loadUserByUsername(username);
			} catch (UsernameNotFoundException notFound) {
				throw notFound;
			} catch (Exception repositoryProblem) {
				throw new InternalAuthenticationServiceException(repositoryProblem.getMessage(), repositoryProblem);
			}

		}

		if (loadedUser == null) {
			throw new InternalAuthenticationServiceException("UserDetailsService returned null, which is an interface contract violation");
		}

		return loadedUser;

	}

}
```


## 问题 
```  
使用http.addFilter() 和http.addFilterAfter(new CustumLogoutFilter('/logout',custumLogoutHandler),LogoutFilter)都没有起作用（应该会起作用）
最终使用http.logout().addLogoutHander(custumLogoutHandler)解决了自定义登出时间

```
[关于custum-filter网友提问][1]


  [1]: http://stackoverflow.com/questions/24122586/how-to-represent-the-spring-security-custom-filter-using-java-configuration