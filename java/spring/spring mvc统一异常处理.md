# Spring @ExceptionHandler 异常处理

## 代码实现
``` java
/**
*   拦截所有异常包括ajax
*/
@ExceptionHandler(Exception.class)
	protected String exception(HttpServletRequest request, HttpServletResponse response, Exception excep) {
	    // 判断是否是ajax 异常
		if (request.getHeader("X-Requested-With") != null && "XMLHttpRequest".equals(request.getHeader("X-Requested-With").toString())) {
			try {
				PrintWriter writer = response.getWriter();
				if (excep instanceof IllegalArgumentException) {

				} else if (excep instanceof SQLException) {

				} else if (excep instanceof RuntimeException) {

				}
				writer.write(excep.getMessage());
				writer.flush();
			} catch (IOException e) {
				e.printStackTrace();
			}
			return null;
		}
		// 非ajax异常直接返回错误页
		return "/error";
	}
```

---
## spring mvc 异常处理机制
spring 会维护一个**handlerExceptionResolvers**列表，这其中包括我们的自定义exceptionHandler,也包括默认的DefaultHandlerExceptionResolver等等，处理异常时遍历列表中的所有handler，找到一个可以处理异常的handler

spring 主servlet(DispatcherServlet) 中 处理自定义异常代码 ： 
``` java
/**
    * Determine an error ModelAndView via the registered HandlerExceptionResolvers.
    * @param request current HTTP request
    * @param response current HTTP response
    * @param handler the executed handler, or {@code null} if none chosen at the time of the exception
    * (for example, if multipart resolution failed)
    * @param ex the exception that got thrown during handler execution
    * @return a corresponding ModelAndView to forward to
    * @throws Exception if no error ModelAndView found
    */
    protected ModelAndView processHandlerException(HttpServletRequest request, HttpServletResponse response,
            Object handler, Exception ex) throws Exception {

        // Check registered HandlerExceptionResolvers...
        ModelAndView exMv = null;
        for (HandlerExceptionResolver handlerExceptionResolver : this.handlerExceptionResolvers) {
            exMv = handlerExceptionResolver.resolveException(request, response, handler, ex);
            if (exMv != null) {
                break;
            }
        }
        if (exMv != null) {
            if (exMv.isEmpty()) {
                return null;
            }
            // We might still need view name translation for a plain error model...
            if (!exMv.hasView()) {
                exMv.setViewName(getDefaultViewName(request));
            }
            if (logger.isDebugEnabled()) {
                logger.debug("Handler execution resulted in exception - forwarding to resolved error view: " + exMv, ex);
            }
            WebUtils.exposeErrorRequestAttributes(request, ex, getServletName());
            return exMv;
        }

        throw ex;
    }
```
excpetionHandler的公共父类AbstractHandlerExceptionResolver中会调用**shouldApplyTo**来查看这个handler是否可以处理这个异常。
``` java
/**
    * Checks whether this resolver is supposed to apply (i.e. the handler matches
    * in case of "mappedHandlers" having been specified), then delegates to the
    * {@link #doResolveException} template method.
    */
    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response,
            Object handler, Exception ex) {

        if (shouldApplyTo(request, handler)) {
            // Log exception, both at debug log level and at warn level, if desired.
            if (logger.isDebugEnabled()) {
                logger.debug("Resolving exception from handler [" + handler + "]: " + ex);
            }
            logException(ex, request);
            prepareResponse(ex, response);
            return doResolveException(request, response, handler, ex);
        }
        else {
            return null;
        }
    }
```




