# spring Security 自定义自动登录 


定义自己的AuthenticationManager Bean ，可以直接继承父AuthenticationManager，只是为了在容器中能找到这个Bean

## 配置文件
``` java
	package com.vcooline.fxt.wap;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.security.SecurityProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.session.SessionRegistry;
import org.springframework.security.core.session.SessionRegistryImpl;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.web.authentication.session.ConcurrentSessionControlAuthenticationStrategy;

import com.vcooline.fxt.common.constant.RoleConstants;
import com.vcooline.fxt.common.constant.UrlConstants;
import com.vcooline.fxt.wap.security.handler.WapLoginSuccessHandler;
import com.vcooline.fxt.wap.security.handler.WapLogoutHandler;
import com.vcooline.fxt.wap.service.auth.WapUserDetailsService;

@Configuration
@Order(SecurityProperties.ACCESS_OVERRIDE_ORDER)
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecureConfig extends WebSecurityConfigurerAdapter {

	@Autowired
	private WapUserDetailsService detailsService;

	@Autowired
	private WapLogoutHandler logoutHandler;

	@Autowired
	private WapLoginSuccessHandler loginSuccessHandler;

	private SessionRegistry sessionRegistry;

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		/**
		 * 友情提示：修改配置时注意顺序，顺序不一样会导致配置无效
		 */
		http.exceptionHandling().accessDeniedPage("/wap/accessDenied")
			.and().authorizeRequests()
			.antMatchers(UrlConstants.URL_SYS + "/**").hasAuthority(RoleConstants.ROLE_SYS)
			.antMatchers("/").permitAll()
			.antMatchers("/wap/welcome").permitAll()
			.antMatchers("/wap/genCode").permitAll()
			.antMatchers("/wap/scan/**").permitAll()
			.antMatchers("/wap/task/callback/**").permitAll()
			.antMatchers("/wap/").permitAll()
			.antMatchers("/wap/loadDoingAndTopAd").permitAll()
			.antMatchers("/wap/loadAdList").permitAll()
			.antMatchers("/wap/task/taskDetail").permitAll()
			.antMatchers("/wap/errorCode").permitAll()
			.antMatchers("/wap/getcode").permitAll()
			.antMatchers("/wap/clearSession").permitAll()
			.antMatchers("/wap/getwxconfig").permitAll()			
			.antMatchers("/wap/getcitycode").permitAll()			
			.antMatchers(UrlConstants.URL_WAP + "/**").hasAuthority(RoleConstants.ROLE_NORMAL)
			.anyRequest().denyAll()
			.and().formLogin().loginPage("/wap/login").permitAll().defaultSuccessUrl("/wap/").successHandler(loginSuccessHandler)
			.and().logout().addLogoutHandler(logoutHandler).logoutUrl("/logout")
			.and().sessionManagement().sessionAuthenticationStrategy(new ConcurrentSessionControlAuthenticationStrategy(sessionRegistry()))
			.maximumSessions(1).expiredUrl("/wap/error/expired").sessionRegistry(sessionRegistry())
			.and()
			.and().csrf().disable();
	}

	@Override
	public void configure(WebSecurity web) throws Exception {
		web.ignoring().antMatchers("/static/**").antMatchers("/error/expired").antMatchers("/druid/**").antMatchers("/**/favicon.ico");
	}

	@Override
	public void configure(AuthenticationManagerBuilder auth) throws Exception {
		auth.userDetailsService(detailsService).passwordEncoder(new BCryptPasswordEncoder());
	}

	@Bean
	@Override
	public AuthenticationManager authenticationManagerBean() throws Exception {
		return super.authenticationManagerBean();
	}

	@Bean
	public SessionRegistry sessionRegistry() {
		if (sessionRegistry == null) {
			sessionRegistry = new SessionRegistryImpl();
		}
		return sessionRegistry;
	}

}
```

## 登录拦截filter
```
package com.vcooline.fxt.wap.web.filter;

import java.io.IOException;
import java.util.Properties;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.util.Assert;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.support.WebApplicationContextUtils;

import com.alibaba.fastjson.JSONObject;
import com.vcooline.fxt.common.constant.WxConstants;
import com.vcooline.fxt.common.util.HttpUtil;
import com.vcooline.fxt.common.util.SpringUtil;
import com.vcooline.fxt.wap.service.auth.WapUserDetailsService;

public class WapWxFilter implements Filter {

	private static final Logger logger = LoggerFactory.getLogger(WapWxFilter.class);

	private String appId;

	private String appSecret;

	private final static String GET_ACCESS_TOKEN_URL = "https://api.weixin.qq.com/sns/oauth2/access_token";// 获取access_token的url接口

	private static final String REDIRECT_URL_PREFIX = "/wap/getcode?scope=snsapi_base&redirectUrl=";

	private static final String WX_APP_ID_KEY = "WxAppID";

	private static final String WX_APP_SECRET_KEY = "WxAppSecret";

	private WapUserDetailsService userDetailService;

	@Override
	public void init(FilterConfig filterConfig) throws ServletException {
		ServletContext servletContext = filterConfig.getServletContext();
		WebApplicationContext wac = WebApplicationContextUtils.getWebApplicationContext(servletContext);

		userDetailService = wac.getBean(WapUserDetailsService.class);

		Properties prop = SpringUtil.getProperties(wac);
		appId = prop.getProperty(WX_APP_ID_KEY);
		appSecret = prop.getProperty(WX_APP_SECRET_KEY);

	}

	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
		HttpServletRequest req = (HttpServletRequest) request;
		HttpServletResponse resp = (HttpServletResponse) response;
		HttpSession session = req.getSession(false);

		if (session == null) {
			chain.doFilter(request, response);
			return;
		}

		final String wxOpenIdInSession = (String) session.getAttribute(WxConstants.SESSION_KEY_WX_OPEN_ID);

		if (!HttpUtil.isWxBrowser(req) || StringUtils.isNotBlank(wxOpenIdInSession)) {
			chain.doFilter(request, response);
			return;
		}

		final String code = req.getParameter("code");

		if (StringUtils.isNotBlank(code)) {
			String wxOpenId = setWxOpenIdByWxCode(code, session);

			try {
				userDetailService.autoLoginByWxOpenId(wxOpenId);
				logger.debug("wxOpenId:{},自动登录成功", wxOpenId);
			} catch (Exception e) {
				logger.error("wxOpenId:{},自动登录失败", wxOpenId);
			}
			chain.doFilter(request, response);
		} else {
			String url = REDIRECT_URL_PREFIX + req.getRequestURL().toString();
			resp.sendRedirect(url);
		}

	}

	@Override
	public void destroy() {
		// TODO Auto-generated method stub

	}

	protected String setWxOpenIdByWxCode(String wxCode, HttpSession session) {
		Assert.notNull(wxCode);

		String wxOpenId = getWxOpenIdFromWx(wxCode);

		if (StringUtils.isNotBlank(wxOpenId)) {
			session.setAttribute(WxConstants.SESSION_KEY_WX_OPEN_ID, wxOpenId);
			return wxOpenId;
		}

		return null;
	}

	private String getWxOpenIdFromWx(String wxCode) {
		final String ERR_CODE_KEY = "errcode";
		final String OPEN_ID_KEY = "openid";

		String wxOpenId = null;
		String method = "GET";
		String params = "appid=" + appId + "&secret=" + appSecret + "&code=" + wxCode + "&grant_type=authorization_code";
		String result = HttpUtil.httpsRequest(GET_ACCESS_TOKEN_URL, method, params);

		if (StringUtils.isNotBlank(result)) {
			JSONObject json = JSONObject.parseObject(result);
			if (StringUtils.isBlank(json.getString(ERR_CODE_KEY)) || WxConstants.WX_REQUEST_CODE_SUCCESS.equals(json.getString(ERR_CODE_KEY))) {
				wxOpenId = json.getString(OPEN_ID_KEY);
			} else {
				logger.debug("根据code:{}获取wx_open_id失败", wxCode);
			}

		}
		logger.debug("成功获取wx_open_id:" + wxOpenId);
		return wxOpenId;
	}

}

```



需要引用的AuthenticationManager 的类中自动注入AuthenticationManager 
## 自动登录service
``` java
package com.vcooline.fxt.wap.service.auth;

import java.util.HashSet;
import java.util.List;
import java.util.Set;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;

import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.security.authentication.AuthenticationDetailsSource;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.session.SessionInformation;
import org.springframework.security.core.session.SessionRegistry;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.security.web.context.HttpSessionSecurityContextRepository;
import org.springframework.session.data.redis.RedisOperationsSessionRepository;
import org.springframework.stereotype.Component;
import org.springframework.util.Assert;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import com.vcooline.fxt.common.constant.RedisKeyPrefixConstants;
import com.vcooline.fxt.common.constant.RoleConstants;
import com.vcooline.fxt.common.model.User;
import com.vcooline.fxt.common.service.user.UserService;

@Component
public class WapUserDetailsService implements UserDetailsService {

	@Autowired
	private UserService userService;

	@Autowired
	private StringRedisTemplate stringRedisTemplate;

	@Autowired
	private AuthenticationManager authManager;

	@Autowired
	private SessionRegistry sessionRegistry;

	@Autowired
	private RedisOperationsSessionRepository sessionRepository;

	private AuthenticationDetailsSource<HttpServletRequest, ?> authenticationDetailsSource = new WebAuthenticationDetailsSource();

	@Override
	public UserDetails loadUserByUsername(String loginName) {
		if (StringUtils.isBlank(loginName)) {
			throw new UsernameNotFoundException("loginName is blank");
		}

		User user = userService.findByLoginName(loginName);
		if (user == null) {
			user = new User();
			user.setLoginName(loginName);
		}
		user.setPassword(getPwd(loginName));

		Set<GrantedAuthority> authorities = new HashSet<>();
		authorities.add(new SimpleGrantedAuthority(RoleConstants.ROLE_NORMAL));

		return new org.springframework.security.core.userdetails.User(user.getLoginName(), user.getPassword(), true,// 是否可用
				true,// 账户未过期为true
				true,// 证书不过期为true
				true,// 账户未锁定为true
				authorities);
	}

	private String getPwd(String phone) {
		String key = RedisKeyPrefixConstants.FXT_RANDOMCODE_PREFIX + phone;
		String value = stringRedisTemplate.opsForValue().get(key);

		return value == null ? "" : value;
	}

	public void autoLoginByWxOpenId(String wxOpenId) throws Exception {
		Assert.notNull(wxOpenId, "自动登录时wxOpenId不能为空");

		User user = userService.findByWxOpenId(wxOpenId);
		if (user != null && StringUtils.isNotBlank(user.getLoginName())) {
			autoLoginByUsername(user.getLoginName());
		}
	}

	public void autoLoginByUsername(String loginName) {
		HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
		UsernamePasswordAuthenticationToken token = new UsernamePasswordAuthenticationToken(loginName, userService.genCodeSilent(loginName));
		token.setDetails(authenticationDetailsSource.buildDetails(request));
		Authentication authenticatedUser = authManager.authenticate(token);
		SecurityContextHolder.getContext().setAuthentication(authenticatedUser);

		if (request.getSession() != null) {
			expireSession(request, authenticatedUser);
			request.getSession().setAttribute(HttpSessionSecurityContextRepository.SPRING_SECURITY_CONTEXT_KEY, SecurityContextHolder.getContext());
		}

	}

	public void expireSession(HttpServletRequest request, Authentication authUser) {
		HttpSession session = request.getSession(false);
		List<SessionInformation> sessionInfos = sessionRegistry.getAllSessions(authUser.getPrincipal(), false);
		if (session != null) {

			if (sessionRegistry.getSessionInformation(session.getId()) == null) {
				sessionRegistry.registerNewSession(session.getId(), authUser.getPrincipal());
			}

			for (SessionInformation si : sessionInfos) {
				if (si != null && !si.getSessionId().equals(session.getId())) {
					si.expireNow();
				}
			}
		}
	}

	public void setSessionWxRelationToRedis(String wxOpenId, String sessionId) {
		Assert.notNull(wxOpenId);
		Assert.notNull(sessionId);

		clearSessionByWxOpenId(wxOpenId, sessionId);

		String key = RedisKeyPrefixConstants.FXT_SESSION_WX_MAP_KEY_PREFIX + wxOpenId;
		stringRedisTemplate.opsForSet().add(key, sessionId);

	}

	private void clearSessionByWxOpenId(String wxOpenId, String currentSessionId) {
		String key = RedisKeyPrefixConstants.FXT_SESSION_WX_MAP_KEY_PREFIX + wxOpenId;
		Set<String> sessionIds = stringRedisTemplate.opsForSet().members(key);

		if (sessionIds == null) {
			return;
		}

		for (String sessionId : sessionIds) {
			if (!sessionId.equals(currentSessionId)) {
				sessionRepository.delete(sessionId);
			}
		}
	}
}

```







