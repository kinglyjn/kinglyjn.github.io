---
layout: post
title:  "springmvc-4.3.7.RELEASE 核心源码断点分析"
desc: "springmvc-4.3.7.RELEASE 核心源码断点分析"
keywords: "springmvc-4.3.7.RELEASE 核心源码断点分析,springmvc,java,kinglyjn"
date: 2017-04-15
categories: [Java]
tags: [java, springmvc]
icon: fa-coffee
---


### springmvc请求执行流程

<img src="http://img.blog.csdn.net/20170415210640025?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="100%"/><br><br>


### springmvc核心源码

#### 核心方法：DispatcherServlet.doDispatcher

该方法控制着springmvc处理和响应请求的核心流程，源码和注释如下：

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
	HttpServletRequest processedRequest = request;
	HandlerExecutionChain mappedHandler = null;
	boolean multipartRequestParsed = false;

	WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

	try {
		ModelAndView mv = null;
		Exception dispatchException = null;

		try {
			processedRequest = checkMultipart(request);
			multipartRequestParsed = (processedRequest != request);


			// 由 dispatcherServlet.handlerMappings 获取 HandlerExecutionChain 对象
			mappedHandler = getHandler(processedRequest);
			if (mappedHandler == null || mappedHandler.getHandler() == null) {
				noHandlerFound(processedRequest, response);
				return;
			}

			// 通过 HandlerExecutionChain 获取 HandlerAdaptor 对象
			HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

			// Process last-modified header, if supported by the handler.
			String method = request.getMethod();
			boolean isGet = "GET".equals(method);
			if (isGet || "HEAD".equals(method)) {
				long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
				if (logger.isDebugEnabled()) {
					logger.debug("Last-Modified value for [" + getRequestUri(request) + "] is: " + lastModified);
				}
				if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
					return;
				}
			}


			// 通过 HandlerExecutionChain 调用拦截器的 preHandle 方法	
			if (!mappedHandler.applyPreHandle(processedRequest, response)) {
				return;
			}

			// 通过 HandlerAdaptor 对象调用目标handler的目标方法得到ModelAndView
			mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

			if (asyncManager.isConcurrentHandlingStarted()) {
				return;
			}

			applyDefaultViewName(processedRequest, mv);

			//通过 HandlerExecutionChain 调用拦截器的postHandle方法
			mappedHandler.applyPostHandle(processedRequest, response, mv);
		}
		catch (Exception ex) { //上述调用的过程中如果发生异常
			dispatchException = ex;
		}
		catch (Throwable err) { //上述调用的过程中如果发生异常
			dispatchException = new NestedServletException("Handler dispatch failed", err);
		}

		// 由ViewResolver组件根据ModelAndView对象得到实际的View
		processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
	}
	catch (Exception ex) {
		triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
	}
	catch (Throwable err) {
		triggerAfterCompletion(processedRequest, response, mappedHandler,
				new NestedServletException("Handler processing failed", err));
	}
	finally {
		if (asyncManager.isConcurrentHandlingStarted()) {
			// Instead of postHandle and afterCompletion
			if (mappedHandler != null) {
				mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
			}
		}
		else {
			// Clean up any resources used by a multipart request.
			if (multipartRequestParsed) {
				cleanupMultipart(processedRequest);
			}
		}
	}
}
```
<br>

#### 相关方法

DispatcherServlet.getHandler<br>
由dispatcherServlet.handlerMappings 获取 HandlerExecutionChain 对象 <br>

```java
/**
 * Return the HandlerExecutionChain for this request.
 * <p>Tries all handler mappings in order.
 * @param request current HTTP request
 * @return the HandlerExecutionChain, or {@code null} if no handler could be found
 */
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
	for (HandlerMapping hm : this.handlerMappings) {
		if (logger.isTraceEnabled()) {
			logger.trace(
					"Testing handler map [" + hm + "] in DispatcherServlet with name '" + getServletName() + "'");
		}
		HandlerExecutionChain handler = hm.getHandler(request);
		if (handler != null) {
			return handler;
		}
	}
	return null;
}
```
<br>


HandlerExecutionChain.applyPreHandle <br>
通过 HandlerExecutionChain 调用拦截器的 preHandle 方法<br>


```java
/**
 * Apply preHandle methods of registered interceptors.
 * @return {@code true} if the execution chain should proceed with the
 * next interceptor or the handler itself. Else, DispatcherServlet assumes
 * that this interceptor has already dealt with the response itself.
 */
boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
	HandlerInterceptor[] interceptors = getInterceptors();
	if (!ObjectUtils.isEmpty(interceptors)) {
		for (int i = 0; i < interceptors.length; i++) {
			HandlerInterceptor interceptor = interceptors[i];
			if (!interceptor.preHandle(request, response, this.handler)) {
				triggerAfterCompletion(request, response, null);
				return false;
			}
			this.interceptorIndex = i;
		}
	}
	return true;
}
```
<br>


HandlerExecutionChain.applyPostHandle <br>
通过 HandlerExecutionChain 调用拦截器的postHandle方法 <br>

```java
/**
 * Apply postHandle methods of registered interceptors.
 */
void applyPostHandle(HttpServletRequest request, HttpServletResponse response, ModelAndView mv) throws Exception {
	HandlerInterceptor[] interceptors = getInterceptors();
	if (!ObjectUtils.isEmpty(interceptors)) {
		for (int i = interceptors.length - 1; i >= 0; i--) {
			HandlerInterceptor interceptor = interceptors[i];
			interceptor.postHandle(request, response, this.handler, mv);
		}
	}
}
```
<br>

HandlerExecutionChain 的 triggerAfterCompletion方法<br>
通过 HandlerExecutionChain 调用拦截器的afterCompletion方法<br>

```java
/**
 * Trigger afterCompletion callbacks on the mapped HandlerInterceptors.
 * Will just invoke afterCompletion for all interceptors whose preHandle invocation
 * has successfully completed and returned true.
 */
void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, Exception ex)
		throws Exception {

	HandlerInterceptor[] interceptors = getInterceptors();
	if (!ObjectUtils.isEmpty(interceptors)) {
		for (int i = this.interceptorIndex; i >= 0; i--) {
			HandlerInterceptor interceptor = interceptors[i];
			try {
				interceptor.afterCompletion(request, response, this.handler, ex);
			}
			catch (Throwable ex2) {
				logger.error("HandlerInterceptor.afterCompletion threw exception", ex2);
			}
		}
	}
}
```
<br>

HandlerExecutionChain的
applyAfterConcurrentHandlingStarted方法<br>

```java
/**
 * Apply afterConcurrentHandlerStarted callback on mapped AsyncHandlerInterceptors.
 */
void applyAfterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response) {
	HandlerInterceptor[] interceptors = getInterceptors();
	if (!ObjectUtils.isEmpty(interceptors)) {
		for (int i = interceptors.length - 1; i >= 0; i--) {
			if (interceptors[i] instanceof AsyncHandlerInterceptor) {
				try {
					AsyncHandlerInterceptor asyncInterceptor = (AsyncHandlerInterceptor) interceptors[i];
					asyncInterceptor.afterConcurrentHandlingStarted(request, response, this.handler);
				}
				catch (Throwable ex) {
					logger.error("Interceptor [" + interceptors[i] + "] failed in afterConcurrentHandlingStarted", ex);
				}
			}
		}
	}
}
```
<br>

DispatcherServlet的
processDispatchResult 方法<br>
渲染视图，最终执行转发操作<br>


* 上述调用的过程中是否发生异常，由HandlerExceptionResolver组件处理异常，得到新的ModelAndView对象<br>
* 通过 HandlerExecutionChain 调用拦截器的afterCompletion方法<br>

```java
/**
 * Handle the result of handler selection and handler invocation, which is
 * either a ModelAndView or an Exception to be resolved to a ModelAndView.
 */
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
		HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {

	boolean errorView = false;

	if (exception != null) {
		if (exception instanceof ModelAndViewDefiningException) {
			logger.debug("ModelAndViewDefiningException encountered", exception);
			mv = ((ModelAndViewDefiningException) exception).getModelAndView();
		}
		else {
			Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);

			// 上述调用的过程中是否发生异常，由HandlerExceptionResolver组件处理异常，得到新的ModelAndView对象
			mv = processHandlerException(request, response, handler, exception);
			errorView = (mv != null);
		}
	}

	// Did the handler return a view to render?
	// 由ViewResolver组件根据ModelAndView对象得到实际的View
	if (mv != null && !mv.wasCleared()) {
		render(mv, request, response);
		if (errorView) {
			WebUtils.clearErrorRequestAttributes(request);
		}
	}
	else {
		if (logger.isDebugEnabled()) {
			logger.debug("Null ModelAndView returned to DispatcherServlet with name '" + getServletName() +
					"': assuming HandlerAdapter completed request handling");
		}
	}

	if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
		// Concurrent handling started during a forward
		return;
	}

	if (mappedHandler != null) {
		// 通过 HandlerExecutionChain 调用拦截器的afterCompletion方法
		mappedHandler.triggerAfterCompletion(request, response, null);
	}
}
```
<br>

DispatcherServlet 的 processHandlerException 方法（由异常解析器处理异常，返回新的异常模型视图对象）

```java
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
	// 上述调用的过程中是否发生异常，由HandlerExceptionResolver组件处理异常，得到新的ModelAndView对象
	ModelAndView exMv = null;
	for (HandlerExceptionResolver handlerExceptionResolver : this.handlerExceptionResolvers) {
		exMv = handlerExceptionResolver.resolveException(request, response, handler, ex);
		if (exMv != null) {
			break;
		}
	}
	if (exMv != null) {
		if (exMv.isEmpty()) {
			request.setAttribute(EXCEPTION_ATTRIBUTE, ex);
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
<br>

DispatcherServlet.render（渲染视图具体执行方法）

* 由ViewResolver组件根据ModelAndView对象得到实际的View<br>

```java
/**
 * Render the given ModelAndView.
 * <p>This is the last stage in handling a request. It may involve resolving the view by name.
 * @param mv the ModelAndView to render
 * @param request current HTTP servlet request
 * @param response current HTTP servlet response
 * @throws ServletException if view is missing or cannot be resolved
 * @throws Exception if there's a problem rendering the view
 */
protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
	// Determine locale for request and apply it to the response.
	Locale locale = this.localeResolver.resolveLocale(request);
	response.setLocale(locale);

	View view;
	if (mv.isReference()) {

		// We need to resolve the view name.
		// 由ViewResolver组件根据ModelAndView对象得到实际的View
		view = resolveViewName(mv.getViewName(), mv.getModelInternal(), locale, request);
		if (view == null) {
			throw new ServletException("Could not resolve view with name '" + mv.getViewName() +
					"' in servlet with name '" + getServletName() + "'");
		}

	}
	else {
		// No need to lookup: the ModelAndView object contains the actual View object.
		view = mv.getView();
		if (view == null) {
			throw new ServletException("ModelAndView [" + mv + "] neither contains a view name nor a " +
					"View object in servlet with name '" + getServletName() + "'");
		}
	}

	// Delegate to the View object for rendering.
	if (logger.isDebugEnabled()) {
		logger.debug("Rendering view [" + view + "] in DispatcherServlet with name '" + getServletName() + "'");
	}
	try {
		if (mv.getStatus() != null) {
			response.setStatus(mv.getStatus().value());
		}
		// 渲染视图
		view.render(mv.getModelInternal(), request, response);
	}
	catch (Exception ex) {
		if (logger.isDebugEnabled()) {
			logger.debug("Error rendering view [" + view + "] in DispatcherServlet with name '" +
					getServletName() + "'", ex);
		}
		throw ex;
	}
}
```
<br>

DispatcherServlet.resolveViewName(由ViewResolver组件根据ModelAndView对象得到实际的View)

```java
/**
 * Resolve the given view name into a View object (to be rendered).
 * <p>The default implementations asks all ViewResolvers of this dispatcher.
 * Can be overridden for custom resolution strategies, potentially based on
 * specific model attributes or request parameters.
 * @param viewName the name of the view to resolve
 * @param model the model to be passed to the view
 * @param locale the current locale
 * @param request current HTTP servlet request
 * @return the View object, or {@code null} if none found
 * @throws Exception if the view cannot be resolved
 * (typically in case of problems creating an actual View object)
 * @see ViewResolver#resolveViewName
 */
protected View resolveViewName(String viewName, Map<String, Object> model, Locale locale,
		HttpServletRequest request) throws Exception {

	for (ViewResolver viewResolver : this.viewResolvers) {
		View view = viewResolver.resolveViewName(viewName, locale);
		if (view != null) {
			return view;
		}
	}
	return null;
}
```
<br>


AbstractView.render <br>


```java
/**
 * Prepares the view given the specified model, merging it with static
 * attributes and a RequestContext attribute, if necessary.
 * Delegates to renderMergedOutputModel for the actual rendering.
 * @see #renderMergedOutputModel
 */
@Override
public void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
	if (logger.isTraceEnabled()) {
		logger.trace("Rendering view with name '" + this.beanName + "' with model " + model +
			" and static attributes " + this.staticAttributes);
	}

	Map<String, Object> mergedModel = createMergedOutputModel(model, request, response);
	prepareResponse(request, response);
	//////////渲染视图的最后步骤
	renderMergedOutputModel(mergedModel, getRequestToExpose(request), response);
}
```
<br>


InternalResourceView 的 renderMergedOutputModel方法

```java
/**
 * Render the internal resource given the specified model.
 * This includes setting the model as request attributes.
 */
@Override
protected void renderMergedOutputModel(
		Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception {

	// Expose the model object as request attributes.
	exposeModelAsRequestAttributes(model, request);

	// Expose helpers as request attributes, if any.
	exposeHelpers(request);

	// Determine the path for the request dispatcher.
	String dispatcherPath = prepareForRendering(request, response);

	// Obtain a RequestDispatcher for the target resource (typically a JSP).
	RequestDispatcher rd = getRequestDispatcher(request, dispatcherPath);
	if (rd == null) {
		throw new ServletException("Could not get RequestDispatcher for [" + getUrl() +
				"]: Check that the corresponding file exists within your web application archive!");
	}

	// If already included or response already committed, perform include, else forward.
	if (useInclude(request, response)) {
		response.setContentType(getContentType());
		if (logger.isDebugEnabled()) {
			logger.debug("Including resource [" + getUrl() + "] in InternalResourceView '" + getBeanName() + "'");
		}
		rd.include(request, response);
	}

	else {
		// Note: The forwarded resource is supposed to determine the content type itself.
		if (logger.isDebugEnabled()) {
			logger.debug("Forwarding to resource [" + getUrl() + "] in InternalResourceView '" + getBeanName() + "'");
		}

		// 最终的请求转发代码
		rd.forward(request, response);
	}
}
```
<br>










