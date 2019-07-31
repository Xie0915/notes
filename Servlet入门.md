## 1、简介

### **什么是servlet**

Java Servlet 是运行在 Web 服务器或应用服务器上的程序，它是作为来自 Web 浏览器或其他 HTTP 客户端的请求和 HTTP 服务器上的数据库或应用程序之间的中间层。

使用servlet，可以收集来自网页表单的用户输入，呈现来自数据库或者其他源的记录，还可以动态创建网页。

### Servlet架构

![1561084129373](C:\Users\xieyu\AppData\Roaming\Typora\typora-user-images\1561084129373.png)

### Servlet任务

- 读取客户端（浏览器）发送的显式的数据:html, applet...
- 读取客户端（浏览器）发送的隐式的 HTTP 请求数据: cookies, 媒体类型，压缩格式
- 处理数据并生成结果：可能需要访问数据库，或者直接计算得出对应的响应
- 发送显式的数据（即文档）到客户端（浏览器）：文本文件，二进制文件等
- 发送隐式的 HTTP 响应到客户端（浏览器）：被返回的文档类型（例如 HTML），设置 cookies 和缓存参数，以及其他类似的任务。

### Servlet包

运行在带有支持 Java Servlet 规范的解释器的 web 服务器上的 Java 类，Servlet 可以使用 **javax.servlet** 和 **javax.servlet.http** 包创建

## 2、Servlet生命周期

### init()

- 只调用一次，第一次创建servlet时调用
- Servlet 创建于用户第一次调用对应于该 Servlet 的 URL 时
- 当用户调用一个 Servlet 时，就会创建一个 Servlet 实例，**每一个用户请求都会产生一个新的线程**

```java
public void init() throws ServletException {
  // 初始化代码...
}
```

### service()

- 执行实际任务的主要方法,Servlet 容器（即 Web 服务器）调用 service() 方法来处理来自客户端（浏览器）的请求，并把格式化的响应写回给客户端。
- 每次服务器接收到一个 Servlet 请求时，服务器会产生一个新的线程并调用服务。service() 方法检查 HTTP 请求类型（GET、POST、PUT、DELETE 等），并在适当的时候调用 doGet、doPost、doPut，doDelete 等方法。

```java
public void service(ServletRequest request, 
                    ServletResponse response) throws ServletException, IOException{
}
```

- service() 方法由容器调用，只需要根据来自客户端的请求类型来重写 doGet() 或 doPost() 即可

#### doGet()

GET 请求来自于一个 URL 的正常请求，或者来自于一个未指定 METHOD 的 HTML 表单，它由 doGet() 方法处理

```java
public void doGet(HttpServletRequest request,
                  HttpServletResponse response)
    throws ServletException, IOException {
    // Servlet 代码
}
```

#### doPost()

POST 请求来自于一个特别指定了 METHOD 为 POST 的 HTML 表单，它由 doPost() 方法处理

```java
public void doPost(HttpServletRequest request,
                   HttpServletResponse response)
    throws ServletException, IOException {
    // Servlet 代码
}
```

### Destroy()

- destroy() 方法只会被调用一次，在 Servlet 生命周期结束时被调用
- 关闭数据库连接、停止后台线程、把 Cookie 列表或点击计数器写入到磁盘，并执行其他类似的清理活动
- 在调用 destroy() 方法之后，servlet 对象被标记为垃圾回收

```java
  public void destroy() {
    // 终止化代码...
  }
```

### 架构图

- 第一个到达服务器的 HTTP 请求被委派到 Servlet 容器。
- Servlet 容器在调用 service() 方法之前加载 Servlet。
- 然后 Servlet 容器处理由多个线程产生的多个请求，每个线程执行一个单一的 Servlet 实例的 service() 方法

## 3、表单数据

浏览器可以通过get和post方法传递信息到web服务器，可以通过get和post传递数据

- GET 方法是默认的从浏览器向 Web 服务器传递信息的方法，它会产生一个很长的字符串，出现在浏览器的地址栏中
- POST 方法不是把信息作为 URL 中 ? 字符后的文本字符串进行发送，而是把这些信息作为一个单独的消息。消息以标准输出的形式传到后台程序

### 使用Servlet读取表单数据：

- getParameter()：调用 request.getParameter() 方法来获取表单参数的值
- getParameterValues()：如果参数出现一次以上，则调用该方法，并返回多个值，例如复选框
- getParameterNames()：如果想要得到当前请求中的所有参数的完整列表，则调用该方法

对于数据中有中文的情况：request.getParameter("name").getBytes("ISO8859-1"),"UTF-8"



## 4、Http请求

**请求头：**

​	Accept, Accept-Charset, Accept-Language, Connection, Cookie等

**读取http中的请求头，通过 *HttpServletRequest*** 

```java
Cookie[] getCookies();
HttpSession getSession();
Enumeration getAttributeNames();
...
```

## 5、Http相应

**响应头：**

​	Allow, Connection, Content-Encoding等

**设置http中的响应头，通过 *HttpServletResponse***

```java
boolean isCommitted();
void reset();
void sendRedirect(String location);
...
```

## 6、Servlet过滤器（Filters）

动态地拦截请求和响应，以变换或使用包含在请求或响应中的信息

Servlet 过滤器是可用于 Servlet 编程的 Java 类，可以实现以下目的：

- 在客户端的请求访问后端资源之前，拦截这些请求
- 在服务器的响应发送回客户端之前，处理这些响应

根据规范建议的各种类型的过滤器：

- 身份验证过滤器（Authentication Filters）。
- 数据压缩过滤器（Data compression Filters）。
- 加密过滤器（Encryption Filters）。
- ....

**javax.servlet.Filter 接口**

```java
//完成实际的过滤操作
public void doFilter (ServletRequest, ServletResponse, FilterChain);

//web 应用程序启动时，web 服务器将创建Filter 的实例对象，并调用其init方法
public void init(FilterConfig filterConfig);

//销毁过滤器实例前调用该方法
public void destroy();
```

- 过滤器可以绑定到一个Servlet，也可以绑定到多个Servlet
- 可以创建多个filter，filter应用的顺序与xml中配置连接的顺序相同

## 7、Servlet异常处理

定义一个ErrorHandler同样集成HttpServlet，只是在xml文件中需要配置相应的错误与异常的mapping

先将ErrorHandler映射到某一个url，然后将其他的错误码或异常映射到这个url上

## 8、Cookie处理

Cookie 是存储在客户端计算机上的文本文件，并保留了各种跟踪信息：

- 服务器脚本向浏览器发送一组 Cookie。例如：姓名、年龄或识别号码等。
- 浏览器将这些信息存储在本地计算机上，以备将来使用。
- 当下一次浏览器向 Web 服务器发送任何请求时，浏览器会把这些 Cookie 信息发送到服务器，服务器将使用这些信息来识别用户。

Cookie通常存储在Http的头信息中，响应头中：

```java
HTTP/1.1 200 OK
Date: Fri, 04 Feb 2000 21:03:38 GMT
Server: Apache/1.3.9 (UNIX) PHP/4.0b3
Set-Cookie: name=xyz; expires=Friday, 04-Feb-07 22:03:38 GMT; 
                 path=/; domain=runoob.com
Connection: close
Content-Type: text/html
```

expires 字段是一个指令，告诉浏览器在给定的时间和日期之后"忘记"该 Cookie。

而在请求头中：

```java
GET / HTTP/1.0
Connection: Keep-Alive
User-Agent: Mozilla/4.6 (X11; I; Linux 2.2.6-15apmac ppc)
Host: zink.demon.co.uk:1126
Accept: image/gif, */*
Accept-Encoding: gzip
Accept-Language: en
Accept-Charset: iso-8859-1,*,utf-8
Cookie: name=xyz
```

一些cookie的方法：

```java
public void setDomain(String pattern);
public String getDomain();
public void setMaxAge(int expiry);
...

//cookie中不能包含[ ] ( ) = , " / ? @ : ;
Cookie cookie = new Cookie("key","value");
//最大保存时间，以秒作为单位
cookie.setMaxAge(60*60*24); 
response.addCookie(cookie);
```

## 9、Session跟踪

三种方式来维持 Web 客户端和 Web 服务器之间的 session 会话：

### Cookies：

分配一个唯一的 session 会话 ID 作为每个 Web 客户端的 cookie，对于客户端的后续请求可以使用接收到的 cookie 来识别。

### 隐藏的表单字段

一个 Web 服务器可以发送一个隐藏的 HTML 表单字段，以及一个唯一的 session 会话 ID

```xml
<input type="hidden" name="sessionid" value="12345">
```

表单被提交时，指定的名称和值会被自动包含在 GET 或 POST 数据中

### URL 重写

在每个 URL 末尾追加一些额外的数据来标识 session 会话，服务器会把该 session 会话标识符与已存储的有关 session 会话的数据相关联。

### HttpSession 对象

Servlet 容器使用**HttpSession接口**来创建一个 HTTP 客户端和 HTTP 服务器之间的 **session 会话**。会话持续一个指定的时间段，跨多个连接或页面请求。

通过调用 HttpServletRequest 的公共方法 **getSession()** 来获取 HttpSession 对象

```java
//获取一个Session
HttpSession session = request.getSession();

//返回在该 session 会话中具有指定名称的对象，如果没有指定名称的对象，则返回 null
public Object getAttribute(String name);

//使用指定的名称绑定一个对象到该 session 会话
public void setAttribute(String name, Object value) ;

//返回 String 对象的枚举，String 对象包含所有绑定到该 session 会话的对象的名称。
public Enumeration getAttributeNames();

//移除一个属性
public void removeAttribute(String name);

//丢弃整个session对象
public void invalidate();
```

## 10、网页重定向

文档移动到新的位置，我们需要向客户端发送这个新位置时，我们需要用到网页重定向

## 11、点击计数器

网站的某个特定页面上的总点击量。使用 Servlet 来计算这些点击量是非常简单的，因为一个 Servlet 的生命周期是由它运行所在的容器控制的

一个简单的基于 Servlet 生命周期的网页点击计数器需要采取的步骤：

- 在 init() 方法中初始化一个全局变量。
- 每次调用 doGet() 或 doPost() 方法时，都增加全局变量。
- 如果需要，可以使用一个数据库表来存储全局变量的值在 destroy() 中。在下次初始化 Servlet 时，该值可在 init() 方法内被读取。这一步是可选的。
- 如果只想对一个 session 会话计数一次页面点击，那么请使用 isNew() 方法来检查该 session 会话是否已点击过相同页面。这一步是可选的。
- 可以通过显示全局计数器的值，来在网站上展示页面的总点击量。这一步是可选的。

## 12、自动刷新页面

假设有一个网页，它是显示现场比赛成绩或股票市场状况或货币兑换率。对于所有这些类型的页面，需要定期刷新网页

使用响应对象的方法 **setIntHeader()**

```java
public void setIntHeader(String header, int headerValue);

header = "Refresh"
```

## 13、Servlet包

```
/myapp
    /images
    /WEB-INF
        /classes
        /lib
```

- WEB-INF 子目录中包含应用程序的部署描述符，名为 web.xml。所有的 HTML 文件都位于顶级目录 *myapp* 下
- WEB-INF/classes 目录包含了所有的 Servlet 类和其他类文件

## 14、URL通配符

“/helloworld/index?”可以匹配“/helloworld/indexA”、“/helloworld/indexB”，但不能匹配“/helloworld/index”也不能匹配“/helloworld/indexAA”；

“/helloworld/index*”可以匹配“/helloworld/index”、“/helloworld/indexA”、“/helloworld/indexAA”但不能匹配“/helloworld/index/A”；

“/helloworld/index/*”可以匹配“/helloworld/index/”、“/helloworld/index/A”、“/helloworld/index/AA”、“/helloworld/index/AB”但不能匹配    “/helloworld/index”、“/helloworld/index/A/B”;

“/helloworld/index/**”可以匹配“/helloworld/index/”下的多有子路径，比如：“/helloworld/index/A/B/C/D”;

如果现在有“/helloworld/index”和“/helloworld/*”，如果请求地址为“/helloworld/index”那么将如何匹配？Spring MVC会按照最长匹配优先原则（即和映射配置中哪个匹配的最多）来匹配，所以会匹配“/helloworld/index”，下面来做测试：



## 15、ServletConfig对象

**通过此对象可以读取web.xml中配置的初始化参数**



## 16、ServletContext对象

当Tomcat启动的时候，就会创建一个ServletContext对象。它**代表着当前web站点**，多个Servlet可以通过ServletContext来进行通信

**==ServletContext.setAttribute(String name,Object obj)==**

**==ServletContext.getAttribute(String name)==**

## 17、request

**HttpServeletRequest的应用：**

- 防盗链

- **表单提交数据（Post方法）**：

  主要用到两个方法：

  ```java
  request.getParameter(String name);
  request.getParameterValues(String name);
  用name标志
  ```

- 超链接方式提交数据：超链接 <a> 和重定向：sendRedirect()

- 中文乱码问题：

  - 在Post方法中，需要使用 request.setCharacterEncoding("UTF-8")，因为http会将表单参数封装到form data中，http请求携带了实体主体，而request封装了这个http请求，就可以对其操作

  - Get方法中，数据从消息行带走，需要在Servlet中反向编码一次，即首先按照ISO-8859-1将参数解码（Bytes）然后用UTF8编码

    ```java
    //此时得到的数据已经是被ISO 8859-1编码后的字符串了，这个是乱码
    String name = request.getParameter("username");
    //乱码通过反向查ISO 8859-1得到原始的数据
    byte[] bytes = name.getBytes("ISO8859-1");
    //通过原始的数据，设置正确的码表，构建字符串
    String value = new String(bytes, "UTF-8");
    ```

  - 提交数据能用Post就用Post！

- 转发：

  ```java
  //获取到requestDispatcher对象，跳转到index.jsp
  RequestDispatcher requestDispatcher = request.getRequestDispatcher("/index.jsp");
  //调用requestDispatcher对象的forward()实现转发,传入request和response方法
  requestDispatcher.forward(request, response);
  ```

  request也是一个域对象，与ServletContext不同，这是针对单一Http请求的域对象，在一次Http请求中可访问，尽量使用request来实现Servlet的通信，这个消耗资源更少

  

**转向时的流程图**：

![img](https://segmentfault.com/img/remote/1460000013228834?w=915&h=547)



## 18、response

Tomcat收到客户端的http请求，会针对每一次请求，分别创建一个**代表请求的request对象**、和**代表响应的response对象**

**HttpServletResponse对象就封装了http响应的信息。**

**Response的应用：**

- ServletOutputStream()向浏览器输出数据：

  print和write方法，print接收字符串，write接收字节数组

- PrintWriter向浏览器输出字符数据，而不是二进制数据

- 文件下载：读取文件--设置消息头--读取内容回送

  java的文件上传下载都是通过io流来完成的

  ```java
  //读取文件
  String path = this.getServletContext().getRealPath("/download/1.png");
  FileInputStream fileInputStream = new FileInputStream(path);
  String fileName = path.substring(path.lastIndexOf("\\") + 1);
  
  //设置消息头
  response.setHeader("Content-Disposition", "attachment; filename="+fileName);
  
  //读取内容回送
  int len = 0;
  byte[] bytes = new byte[1024];
  ServletOutputStream servletOutputStream = response.getOutputStream();
  while ((len = fileInputStream.read(bytes)) > 0) {
      servletOutputStream.write(bytes, 0, len);
  }
  servletOutputStream.close();
  fileInputStream.close();
  ```

  中文的文件名，需要对URL进行编码

  ```java
  response.setHeader("Content-Disposition", "attachment; filename=" + URLEncoder.encode(fileName, "UTF-8"));
  ```

- 自动刷新：

  修改消息头

  ```java
  response.getWriter().write("time is :" + System.currentTimeMillis());
  response.setHeader("Refresh", "3;url='/index.jsp'");
  ```

- 设置缓存：

  设置消息头

- 设置压缩：

- 重定向跳转：

  ```java
  //重定向到index.jsp页面
  response.sendRedirect("/index.jsp");
  ```

  **重定向是通过302状态码和跳转地址实现的**

  因此，可以通过设置状态码与消息头实现重定向：

  ```java
  //设置状态码是302
  response.setStatus(302);
  //HttpServletResponse把常用的状态码封装成静态常量了，所以我们可以使用SC_MOVED_TEMPORARILY代表着302
  response.setStatus(HttpServletResponse.SC_MOVED_TEMPORARILY);
  //跳转的地址是index.jsp页面
  response.setHeader("Location", "/index.jsp");
  ```

  因此sendRedirect是setStatus和setHeader的封装





## 一些注释：

### 1、Servlet是单例的

浏览器多次对Servlet的请求，一般情况下，服务器只创建一个Servlet对象，也就是说，Servlet对象一旦创建了，就会驻留在内存中，为后续的请求做服务，直到服务器关闭。对于每次访问请求，Servlet引擎都会创建一个新的HttpServletRequest请求对象和一个新的HttpServletResponse响应对象，然后将这两个对象作为参数传递给它调用的Servlet的service()方法，service方法再根据请求方式分别调用doXXX方法。当多个用户访问Servlet的时候，服务器会为每个用户创建一个线程。当多个用户并发访问Servlet共享资源的时候就会出现线程安全问题。

### 2、Servlet加载静态资源

- 通过绝对路径
- 通过Servlet从WEB-INF下读取
- 通过类加载器

### 3、转发和重定向的区别

|          | 转发                                                         | 重定向                                                       |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 发生位置 | 服务器，浏览器地址栏无变化（对浏览器透明）                   | 浏览器，浏览器地址栏变化，两个http请求                       |
| 资源地址 | **服务器用的直接从资源名开始写**request.getRequestDispatcher("/资源名 URI").forward(request,response) | **浏览器用的要把应用名写上**response.send("/web应用/资源名 URI"); |
| URL范围  | 服务器跳转，当前WEB应用的资源                                | 浏览器跳转，任意资源                                         |
| 跳转时间 | 执行完跳转语句后立刻跳转                                     | 整个页面执行完后进行跳转                                     |

怎么选择：

**转发是带着转发前的请求的参数的。重定向是新的请求**。

典型的应用场景：

1. 转发: 访问 Servlet 处理业务逻辑，然后 forward 到 jsp 显示处理结果，浏览器里 URL 不变
2. 重定向: 提交表单，处理成功后 redirect 到另一个 jsp，防止表单重复提交，浏览器里 URL 变了