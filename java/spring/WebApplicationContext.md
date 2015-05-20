# WebApplicationContext
## tips
通过WebApplicationContext getBean获得的bean可以获取其相关依赖的Bean
## spring中
在spring中可以通过ContextLoader.getCurrentWebApplicationContext获得

## filter
在filter中可以在init()通过FilterConfig..getServletContext();WebApplicationContextUtils.getWebApplicationContext(servletContext); 获得。