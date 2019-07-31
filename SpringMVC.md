### ViewResolver

```java
public interface ViewResolver{
    View resolveViewName(String viewName, Locale locale) throws Exception;
}
```

**AbstractCachingViewResolver会启用缓存功能，Spring中很多ViewResolver都继承这个**

- 面向单一视图类型：UrlBasedViewResolver

  通常指定视图模板的位置，这些ViewResolver就会按照逻辑视图名，抓取模板文件并构造View实例返回，但是一个ViewResolver对应一个View，数量为1:1

  - InternalResourceViewResolver：对应InternalResourceView，处理JSP模板类型的视图映射
  - FreeMarkerViewResolver：
  - ...

  用**prefix**属性指定路径，**suffix**指定模板文件后缀名，当有逻辑视图名后就可获得模板文件

  ```
  prefix+viewName+suffix
  ```

- 面向多视图类型：

  - ResourceBundleViewResolver：在properties或yml文件中管理逻辑名称与具体视图的映射关系
  - XmlViewResolver：xml配置文件加载映射信息
  - BeanNameViewResolver：可以被认为是XmlViewResolver的原型版或者简化版



### View

View是SpringMVC将视图渲染逻辑从DispatcherServlet中剥离出来

```java
public interface View{
    String getContentType();
    
    //实现渲染工作
    void render(Map model, HttpServletRequest request, HttpServletResponse response) throws Exception;
}
```

View的实现类就是使用相应的API将模板和最终提供的模型数据合并到一起，最终输出结果页面给客户端，SpringMVC提供的View实现类都直接或者间接地继承AbstractView





### Handler和HandlerAdaptor

HandlerMapping返回的不仅仅包括Controller一种类型，任何可以用于Web请求处理的处理对象都统称为Handler，因此HandlerExecutionChain返回的是一个Object类型的Handler对象

DispatcherServlet将不同Handler的调用职责交给了HandlerAdaptor

```java
public interface HandlerAdaptor{
    boolean supports(Object Handler);
    ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object Handler) throws Exception;
    long getLastModified(HttpServletRequest request, Object Handler);
}
```



### HandlerInterceptor

```java
public interface HandlerInterceptor{
    //HandlerAdaptor调用Handler之前执行
    boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;
    
    //HandlerAdaptor调用具体的Handler处理完Web请求之后，并在视图解析之前进行，因此可以获得ModelAndView
    void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception;
    
    //处理完之后执行
    void afterHandle(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception;
}
```



UserRoleAuthorizationInterceptor:通过HttpRequest的isUserInRole()方法，使用指定的一组用户角色对当前请求进行验证

WebContentInterceptor:

- 检查请求方法是否在支持方法内
- 检查必要的Session实例
- 检查缓存时间并这是相应的HTTP头的方式控制缓存行为

