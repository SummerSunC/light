# spring Security 自定义自动登录 

tags: spring java 


定义自己的AuthenticationManager Bean ，可以直接继承父AuthenticationManager，只是为了在容器中能找到这个Bean
``` java
	@Bean(name="myAuthenticationManager")
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
```

需要引用的AuthenticationManager 的类中自动注入AuthenticationManager 
``` java
@Autowired
protected AuthenticationManager authenticationManager;

public void autoLoginByUsername(String loginName) {
		HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();

		UsernamePasswordAuthenticationToken token = new UsernamePasswordAuthenticationToken(loginName, genCodeSilent(loginName));
		token.setDetails(new WebAuthenticationDetails(request));
		Authentication authenticatedUser = authenticationManager.authenticate(token);
		SecurityContextHolder.getContext().setAuthentication(authenticatedUser);
		if (request.getSession() != null) {
			request.getSession().setAttribute(HttpSessionSecurityContextRepository.SPRING_SECURITY_CONTEXT_KEY, SecurityContextHolder.getContext());
		}
	}
```




