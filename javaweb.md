# 基础知识

## C/S & B/S

### C/S(Client-Server)

客户端-服务器架构

- 不灵活
- 升级维护麻烦
- 可以做的很cool

### B/S(Browser-Server)

浏览器-服务器架构

- 轻量（既是优点也是缺点）
- 不安全

# B/S

## 通信步骤

1. 输入网址

2. 域名解析，解析后就是IP地址http://110.242.68.3:80/index.html

3. 浏览器在网络中搜索域名110.242.68.3这一台主机

4. 定位主机后，定位端口是80对应的软件，80对应的资源名是：index.html

5. 服务器找到浏览器想要的资源名即index.html，并把index.html的内容输出响应到浏览器上

6. 浏览器接收到服务器的代码(html css js)

7. 浏览器渲染，执行html css js代码

   ![image-20240528150013180](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240528150013180.png)

## WEB服务器

- 服务器软件

  - Tomcat(WEB服务器)

  - jetty(WEB服务器)

  - jBOSS(应用服务器)

  - WebLogic(应用服务器)

  - WebSphere(应用服务器)

- 区别：
  - 应用服务器实现了java的所有（13个）规范
  - web服务器只实现了servlet + jsp两个核心规范

### tomcat

tomcat是用Java写的

一定要配置JAVA_HOME和CATALINA_HOME的路径

- 启动：startup.bat
- 关闭：shutout.bat（已更改为stop.bat）

#### 目录

- bin：命令文件目录：打开/关闭tomcat（**.bat文件可以批量编写Windows的dos命令**，.sh文件则是linux专属的shell命令，启动本质）
- conf：tomcat服务器配置存放目录（默认端口号是8080）
- lib：核心程序目录，java写的
- logs：日志目录
- temp：服务器临时目录
- webapps：存放大量的webapp(web应用)
- work：存放jsp文件翻译之后的java文件及编译之后的class文件

## 实现web

startup启动后，把.html文件放在tomcatwebapps文件夹下

1. 写完类实现servlet接口
2. 配置好web.properties文件（文件名和路径名已被写死），配置好请求**路径和类名**的关系
3. 目录结构、配置文件路径、Java程序路径都已被servlet写死

### 动态网页技术

使.html文件不再被写死，而使用Java程序

对于动态web，一个请求和响应的过程有多少角色参与，角色与角色之间有多少协议

#### 角色

1. Browser软件的开发商(Edge, Chrome...)
2. WEB Server的开发团队(Tomcat, JBOSS...)
3. DB Server的开发团队(MySQL, Oracle...)
4. Webapp的开发团队(我开发的)

#### 协议

1. Webapp与Web Server开发团队有一套规范：JavaEE规范之**Servlet规范**
   - 作用：WebServer和Webapp的解耦合
2. Broswer和WebServer的传输协议: **HTTP协议(**超文本传输协议)
3. Webapp开发团队和数据库团队使**JDBC规范**

![image-20240603203543159](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240603203543159.png)

# Servlet

jakarta ee9(Oracle捐给Eclipse，javee改名了)，tomcat10.0对应的最新版本是**jakarta.servlet.Servlet**

*乱码问题：W:\Code\apache-tomcat-10.1.24\conf\logging.properties里更改ConsoleHandler.encoding = **GBK***

![image-20240603213506136](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240603213506136.png)

### 文件配置

#### WEB-INF

WEB-INF目录下资源是受保护的，不能通过浏览器直接访问，因此像HTML CSS JS等静态资源一定要放到WEB-INF之外

##### classes

存放.class文件的

##### lib

存放外置jar包，非必须

##### web.xml

注册Servlet类

```xml
<web-app xmlns="https://jakarta.ee/xml/ns/jakartaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="https://jakarta.ee/xml/ns/jakartaee
                      https://jakarta.ee/xml/ns/jakartaee/web-app_6_0.xsd"
  version="6.0"
  metadata-complete="true">

</web-app>
```

![image-20240603212943630](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240603212943630.png)

#### html

#### css

#### javascript

#### image

### JDBC连接数据库

1. Servlet是Java程序，因此完全可以在Servlet中编写JDBC代码连接数据库

2. 在webapp中去连接数据库，需要将驱动jar包(com.mysql.cj.jdbc.Driver)放在WEB-INF/lib目录下

3. 在web.xml文件中完成对应StudentServlet的注册

4. 写一个student.html页面(不能放到WEB-INF目录里面，只能放到WEB-INF外面)，在页面中编写超链接，Tomcat执行后台的StudentServlet

   ![image-20240604205643491](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240604205643491.png)

5. IDEA关联Tomcat

   ![image-20240604210219818](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240604210219818.png)

### Servlet生命周期

1. servlet对象由Tomcat全权负责的，tomcat服务器又称为web容器来管理servlet对象的死活
2. new的servlet对象不受tomcat管理，只有web容器创建的servlet对象（被放在一个统一的hashmap中）才受tomcat管理
3. 启动tomcat时，不会实例化自己new的servlet对象，只有在用户发送请求时时才会实例化，要启动要在web.xml对应的servlet加入<load-on-startup>0（一定是整数）</load-on-startup>
4. 用户发送请求时，实例化servlet对象（构造器），会调用init方法和service方法，但init和构造器只会被调用一次，service每发送一次请求就调用一次；关闭tomcat服务器后被tomcat服务器调用destroy函数一次，destroy方法执行结束之后，servlet对象被销毁，内存被释放
   - 说明servlet对象是单例的（对象是单例，但不属于单例模式，属于假单例。因为这是servlet对象由tomcat创建的，用户管不着，真单例模式构造方法是私有化的）
   - 过程：构造器->init->service->destroy
   - 不建议自己编写构造方法，定义不当就会导致无法实例化，建议在init方法中写构造方法要写的代码
   - destroy方法：写抓紧时间要保存的资源。。。

### GenericServlet

使用适配器模式：编写一个标准通用GenericServlet类，不直接实现servlet而是继承GenericServlet

#### 自己定义的

```java
public abstract class GenericServlet implements Servlet {
    private ServletConfig servletConfig;
    // 防止子类重写带servletConfig的init
    @Override
    public final void init(ServletConfig servletConfig) throws ServletException {
        this.servletConfig = servletConfig;
        this.init();
    }
    // 这样子类也能重写无参init了，防止改变servletConfig导致tomcat调用出错
	public void init(){
        
    }
    @Override
    public ServletConfig getServletConfig() {
        return null;
    }
	
    // 核心方法抽象化：service
    @Override
    public abstract void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException;

    @Override
    public String getServletInfo() {
        return "";
    }

    @Override
    public void destroy() {

    }
}
```

#### 源代码

```java
package jakarta.servlet;
// 此处还实现了servletConfig，因此可以直接this.(config中的方法例如getInitParameterNames)
public abstract class GenericServlet implements Servlet, ServletConfig, Serializable {
    private static final long serialVersionUID = 1L;
    private transient ServletConfig config;

    public GenericServlet() {
    }

    public void destroy() {
    }

    public String getInitParameter(String name) {
        return this.getServletConfig().getInitParameter(name);
    }

    public Enumeration<String> getInitParameterNames() {
        return this.getServletConfig().getInitParameterNames();
    }

    public ServletConfig getServletConfig() {
        return this.config;
    }

    public ServletContext getServletContext() {
        return this.getServletConfig().getServletContext();
    }

    public String getServletInfo() {
        return "";
    }

    public void init(ServletConfig config) throws ServletException {
        this.config = config;
        this.init();
    }

    public void init() throws ServletException {
    }

    public void log(String message) {
        ServletContext var10000 = this.getServletContext();
        String var10001 = this.getServletName();
        var10000.log(var10001 + ": " + message);
    }

    public void log(String message, Throwable t) {
        this.getServletContext().log(this.getServletName() + ": " + message, t);
    }

    public abstract void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    public String getServletName() {
        return this.config.getServletName();
    }
}

```



### ServletConfig

是一个接口，tomcat会实现其具体对象

实例化servlet对象时会实例化一个servletconfig，一个servlet对象对应一个servletconfig

```java
public interface ServletConfig {
    // 获取servlet名
    String getServletName();

    ServletContext getServletContext();
	// 获取<init-param>的key对应的value
    String getInitParameter(String var1);
	// 包装<init-param>的信息，是以一种hashmap的形式
    Enumeration<String> getInitParameterNames();
}
```

tomcat会解析web.xml并把对应的<servlet></servlet>标签中的配置信息，包装进servletconfig中

## ServletContext

servlet对象的上下文对象，可以理解为web.xml文件，又名**应用域**

- 对于一个webapp来说，**servletcontext对象只有一个**；webapp启动时创建，关闭时销毁。
- **tomcat是一个webapp容器**，一个容器可以放多个webapp，一个webapp有对应一个servletcontext

### 应用域

用于存放用户共享的数据，要求数据量小，几乎不修改（涉及并发线程安全问题）

- 向应用域绑定数据时，相当于把数据放到了缓存（cache）中，然后访问的时候直接从缓存中访问减少IO操作，大大提升系统性能
  - 缓存技术
    1. 字符串常量池("abc")
    2. 数据库连接池(提前创建好N个连接对象，将连接对象放到集合中，连接对象时直接从缓存中拿，省去连接对象的创建过程，提升效率)
    3. 线程池（tomcat预先创建好N个线程对象，然后将线程对象存到集合中）
- 应用域数据要求所有用户共享数据
- 共享数据量很小
- 共享数据修改操作很小

### 源代码

```java
package jakarta.servlet;
public interface ServletContext {
    String TEMPDIR = "jakarta.servlet.context.tempdir";
    String ORDERED_LIBS = "jakarta.servlet.context.orderedLibs";
    // 重要！
	// getContextPath：获取项目的根路径
    String getContextPath();

    // 记录日志
    // 默认路径：%CATLINA_HOME%\logs目录下
    // IDEA中：%CATLINA_BASE%\logs目录下
    // 日志一般有
    // catalina.2024-12-25.log
    // localhost.2024-12-25.log
    // local
    void log(String var1);

    void log(String var1, Throwable var2);
	// getRealPath：获取文件的绝对路径
    String getRealPath(String var1);

    String getServerInfo();
	// 与ServletConfig同款<context-param></context-param>
    // 不同的在于是全局信息，servletconfig是局部信息
    String getInitParameter(String var1);

    Enumeration<String> getInitParameterNames();

    // CRUD
    boolean setInitParameter(String var1, String var2);
	// 取value
    Object getAttribute(String var1);
	// 取key
    Enumeration<String> getAttributeNames();
	// 存
    void setAttribute(String var1, Object var2);
	// 删
    void removeAttribute(String var1);

}

```

### 缓存机制

- 堆内存中的字符串常量池

  "abc"

- 堆内存中的整数型常量池

  [-128, 127]

- 连接池(Connection Cache)

- 线程池

  Tomcat服务器启动时预先创建N多个线程Thread对象，然后将线程对象放到集合中，用户发送请求时就把线程分配给用户

- redis

## HttpServlet

### ServletRequest

- request对象实际上又称为**请求域**，生命周期短很多（只在一次请求中有效），对象范围小很多

- Servletcontext是**应用域**对象（服务器启动时启动），用于获取前端数据Map集合
- 原则：尽量使用小的域对象，占用资源少

```java
public interface ServletRequest {
    // 最常用！获取value一维数组中的第一个元素
    String getParameter(String var1);
	// 获取keys
    Enumeration<String> getParameterNames();
	// 获取key对应的value(是一维数组)
    String[] getParameterValues(String var1);
	// 获取map
    Map<String, String[]> getParameterMap();
    
    // Attribute本质还是Map<String, String[]>
    // 从域中获取数据
    Object getAttribute(String var1);
    // 向域中绑定数据
    void setAttribute(String var1, Object var2);
	// 移除域中数据
    void removeAttribute(String var1);
    
    // 获取请求转发器对象
    // 相当于把var1路径包装到请求转发器，实际上把下一个跳转的资源路径告诉tomcat
	RequestDispatcher getRequestDispatcher(String var1);
    
    // 其他常用方法
    // 获取客户端IP地址
    String getRemoteAddr();
    // 设置POST方法的“请求体”编码方式
	void setCharacterEncoding(String var1) throws UnsupportedEncodingException;

}
Map<String, String[]> parametermap= request.getParameterMap();
Enumeration<String> names = request.getParameterNames();
String[] values = request.getParameterValues();
String value = request.getParameter();

// 转发操作,/b是另一个servlet的路径；当然也可以是.html页面
RequestDispatcher dispatcher = getRequestDispatcher("/b");
dispatcher.forward(request, response);
// 只能设置POST请求体，GET请求没用
request.setCharacterEncoding("UTF-8");
// tomca10之后默认是UTF-8，tomcat10之前request和response中文有乱码，解决方法:
request.setCharacterEncoding("UTF-8");
respnse.setContentType("text/html:charset=UTF-8");
//修改CATLINA_HOME\conf\server.xml配置文件
<Connector URIEncoding = "UTF-8"/>
```

```java
package jakarta.servlet;
import java.io.IOException;
public interface RequestDispatcher {
    ...
    void forward(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    void include(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;
}
```

- getParameter获取的是用户在浏览器提交的数据
- getAttribute获取的是请求域中绑定的数据

### ServletResponse



### 源代码

```java

package jakarta.servlet.http;

public abstract class HttpServlet extends GenericServlet{
    
    // 模板方法
    // 该方法定义核心算法骨架，具体实现交由给子类
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // getMethod获得7种请求指令之一（GET HEAD POST...）
        String method = req.getMethod();
        long lastModified;
        if (method.equals("GET")) {
            lastModified = this.getLastModified(req);
            if (lastModified == -1L) {
                // 可以在子类中重写doGet方法，即可完成方法更迭，这就是模板模式的核心
                this.doGet(req, resp);
            } else {
                long ifModifiedSince;
                try {
                    ifModifiedSince = req.getDateHeader("If-Modified-Since");
                } catch (IllegalArgumentException var9) {
                    ifModifiedSince = -1L;
                }

                if (ifModifiedSince < lastModified / 1000L * 1000L) {
                    this.maybeSetLastModified(resp, lastModified);
                    this.doGet(req, resp);
                } else {
                    resp.setStatus(304);
                }
            }
        } else if (method.equals("HEAD")) {
            lastModified = this.getLastModified(req);
            this.maybeSetLastModified(resp, lastModified);
            this.doHead(req, resp);
        } else if (method.equals("POST")) {
            // 可以在子类中重写doPost方法，即可完成方法更迭，这就是模板模式的核心
            this.doPost(req, resp);
        } else if (method.equals("PUT")) {
            this.doPut(req, resp);
        } else if (method.equals("DELETE")) {
            this.doDelete(req, resp);
        } else if (method.equals("OPTIONS")) {
            this.doOptions(req, resp);
        } else if (method.equals("TRACE")) {
            this.doTrace(req, resp);
        } else {
            String errMsg = lStrings.getString("http.method_not_implemented");
            Object[] errArgs = new Object[]{method};
            errMsg = MessageFormat.format(errMsg, errArgs);
            resp.sendError(501, errMsg);
        }

    }
	// service方法
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        HttpServletRequest request;
        HttpServletResponse response;
        try {
            // 向下转型为具体的实现类HttpServletRequest和HttpServletResponse
            request = (HttpServletRequest)req;
            response = (HttpServletResponse)res;
        } catch (ClassCastException var6) {
            throw new ServletException(lStrings.getString("http.non_http"));
        }
		// 重载service方法
        this.service(request, response);
    }

}

```





### HTTP

由W3C联盟制定，超文本（html）传输协议（通信协议）：支持文本、视频。。。流媒体

##### 请求协议(B -> S)

浏览器向web服务器发送数据



- 请求行

  1. 请求方式(常用GET/POST)

  2. URI

     URL（统一资源定位符，表示某个资源的路径，可以找到）包括URI（统一资源标识符，表示某个资源名，但找不到）

  3. HTTP版本号(1.1)

- 请求头

  1. 请求的主机
  2. 主机端口
  3. 浏览器信息
  4. 品牌信息

- 空白行

  分割请求头与请求体

- 请求体

###### GET请求

![image-20240605161831227](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240605161831227.png)

###### POST请求

![image-20240605162054201](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240605162054201.png)

##### 响应协议(S -> B)

web服务器向浏览器发送数据

![image-20240605161543438](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240605161543438.png)

- ##### 状态行

  1. HTTP版本号(HTTP/1.1)
  2. 状态码(HTTP协议中规定的响应状态号)
     - 200：请求响应成功
     - 404：资源不存在：要么路径错了；要么服务器资源没打开
     - 500：服务器程序出现异常
     - 4开头：浏览器出错
     - 5开头：服务器出错
  3. 状态的描述信息
     - ok 成功
     - not found 资源找不到

- 响应头

  1. 响应内容类型
  2. 响应内容长度
  3. 响应时间

- 空白行

  用于分隔响应头和响应体

- 响应体

  响应的正文，这些内容是一个长的字符串，这个字符串被浏览器渲染解释并执行，最终展示效果（F12可看）

#### GET

- 在浏览器上地址栏直接输入URL，敲回车，属于GET
- 浏览器点击超链接
- 使用form表单提交数据时，form标签没有method属性，默认是get

#### POST

只有一种情况：使用form的method属性为post时，才使用post；避免敏感信息回显在浏览器地址栏的情况

#### 区别

1. get请求发送数据时，数据会挂在URI后面加一个？，导致数据回显在浏览器的地址栏上(pan.baidu.com/pwd = 4396)
2. post发送数据时，不会会回显到浏览器的地址栏上
3. 发送的请求数据格式是统一的：name = value & name = value & name = value...
   - 举例：此处的name是“用户名”，value对应你定义的用户名比如"yiga"
4. get请求只能发普通的字符串，有长度限制无法发送大数据量；post请求可以发任何类型数据，大数据量，没有长度限制
5. get适合从服务器获取数据（打开网页）；post适合向服务器传送数据（上传文件）
6. get请求是绝对安全，因为是从服务器获取数据；post请求是危险的，因为向服务器提交数据，大部分情况会监听post请求
7. get请求支持缓存，被浏览器缓存起来，一个 get请求路径对应一个资源；post请求不支持缓存

#### HttpServletRequest

```java
package jakarta.servlet.http;
public interface HttpServletRequest extends ServletRequest {
	// 动态获取servlet根路径
	String getContextPath();
    // 获取请求方式（GET或者POST）
	String getMethod();
    // 获取请求URI（带项目名）
    String getRequestURI(); // / "project01/aServlet"
    // 获取servlet路径（不带项目名 // "/aServlet"
    String getServletPath();
}
```

在service中HttpServletRequest由tomcat实现org.apache.catalina.connector.**RequestFacade**，封装了HTTP的请求协议

#### HttpServletResponse

生命周期只在被调用时存在

### 欢迎页面

1. 不指定具体的资源时，默认跳转的页面；如果不设置，tomcat会默认配置（在%CATLINA_HOME%\conf\web/xml下）选$index.html$ $index.htm$ $index.jsp$作为欢迎页面（全局）

   ![image-20240606151608559](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240606151608559.png)

   

2. 在web.xml下修改：（**局部，就近原则**，局部优先于全局）

```xml
<welcome-file-list>
    <!--不需要加'/'，默认从webapp的根路径开始-->
	<welcome-file>hello.html<welcome-file>
    <!--越往上优先级越高-->
    <welcome-file>hello2.html<welcome-file>
</welcome-file-list>
```

### Servlet完成CRUD

1. 准备一张数据表

   ```sql
   
   create table dept(
   	deptno int primary key,
   	dname varchar(255),
   	loc varchar(255)
   )
   insert into dept(deptno, danme, loc) values(10, '市场', '北京');
   insert into dept(deptno, dname, loc) values(20, '研发', '广东');
   insert into dept(deptno, dname, loc) values(30, '技术', '苏州');
   
   ```

2. 准备html页面

   - 欢迎页面: index.html

   - 列表页面: list.html，核心

   - 新增页面: add.html
   - 修改页面: edit.html
   - 详情页面: detail.html

3. 功能：操作能够连接数据库

   - 查看部门列表
   - 新增部门
   - 删除部门
   - 查看部门详细信息
   - 跳转修改页面
   - 修改部门

4. 在IDEA中搭建开发环境

   - 创建webapp
   - 向web-app\lib中添加jar包（mysql-connector-java-8.0.28.jar）
   - JDBC的工具类
   - 将所有html页面拷贝到web目录下

5. 实现功能：查看部门列表

   - 修改前端页面超链接，用户点击的就是这个超链接

     ```
     <a href="/oa/dept/list">查看部门列表</a>
     ```

     

   - 编写web.xml文件

     ```
     <se
     ```

     

   - 编写DeptListServlet类继承HttpServlet类，重写doGet方法



# MVC

Model View Controller，数据/业务展示控制器

1个司令+2个秘书

- M：处理业务/数据的秘书
- C：绝对核心，是控制器/核心，司令官
- V：展示页面的一个秘书

![image-20240614182950768](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240614182950768.png)

model包括：

pojo, bean, domain, service, dao

**三层架构：表示层-业务逻辑层-持久化层**

![image-20240614191746033](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240614191746033.png)

![image-20240614191726586](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240614191726586.png)

## 不用MVC缺点

- 一个servelet同时负责数据接收；核心的业务处理；数据库表的CRUD；页面的数据展示，太杂糅

1. 代码复用性太低，没有职能分工和独立组件概念
2. 耦合度高，代码难以拓展
3. 操作数据库代码和业务逻辑混在一起，容易出错

## 优点

1. 代码分层，各司其职，耦合度降低，拓展力增强，组件可复用性增强

![image-20240614204255469](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240614204255469.png)

## **DAO**

(Data Access Object，数据访问对象)

属于JAVE EE的一种设计模式，只负责数据库表的CRUD，**没有任何业务逻辑**

```java
public class AccountDao{
	// 增
	public int insert(Account act){
        Connection conn = null;
        PreparedStatement ps = null;
        try{
            conn = DBUtil.getConnection();
            String sql = "insert into t_act(actno, balance) values(?, ?)" ;
            ps = conn.prepareStatement(sql);
            ps.setString(1, act.getActno());
            ps.setDouble(2, act.getBalance());
            int i = ps.executeUpdate();
        } catch(SQLException e){
            throw new RuntimeException(e);
        } finally{
            DBUtil.close(conn, ps, rs);
        }
		return 0;
	}
	// 删
	public int deleteById(String id){
        Connection conn = null;
        PreparedStatement ps = null;
		int count = 0;
        try{
            conn =DBUtil.getConnection();
            String sql = "delete from t_act where id = ?";
            ps = conn.prepareStatement(sql);
            ps.setLong(1, id);
            count = ps.executeUpdate();
        } catch (SQLException e) {
            thro new RuntimeException(e);
        } finally {
            DBUtil.close(conn, ps, rs);
        }
        return count;
	}
	// 改
	public int update(Account act){
        Connection conn = null;
        PreparedStatement ps = null;
		int count = 0;
        try{
            conn =DBUtil.getConnection();
            String sql = "update t_act set balance = ?, actno = ? where id = ?";
            ps = conn.prepareStatement(sql);
            ps.setDouble(1,act.getBalance());
            ps.setString(2, act.getActno());
            ps.setLong(3, act.getId());
            count = ps.executeUpdate();
        } catch (SQLException e) {
            thro new RuntimeException(e);
        } finally {
            DBUtil.close(conn, ps, rs);
        }
        return count;
	}
	// 查
	public Account selectByActno(String actno){
        Connection conn = null;
        PreparedStatement ps = null;
		ResultSet rs = null;
        Account act = null;
        try{
            conn =DBUtil.getConnection();
            String sql = "select id, balance from t_act where actno = ?";
            ps = conn.prepareStatement(sql);
            ps.setString(2, act.getActno());
            rs = ps.executeQuery();
            if(rs.next()){
                Long id = rs.getLong("id");
                act = new Account();
                act.setId(id);
                act.setActno(actno);
                act.setBalance(balance);
            }
        } catch (SQLException e) {
            thro new RuntimeException(e);
        } finally {
            DBUtil.close(conn, ps, rs);
        }
        return act;
	}
    
    // 查"所有"
    public List<Account> selectAll(){
        Connection conn = null;
        PreparedStatement ps = null;
		ResultSet rs = null;
        List<Account> list = new ArrayList<>();
        try{
            conn =DBUtil.getConnection();
            String sql = "select id, actno, balance from t_act where actno = ?";
            ps = conn.prepareStatement(sql);
            ps.setString(1, actno);
            rs = ps.executeQuery();
            if(rs.next()){
                Long id = rs.getLong("id");
                Double balance = rs.getDouble("balance");
                String actno = rs.getString("actno");
                Account account = new Account(id, actno, balance);
                //account.setId(id);
                //account.setActno(actno);
                //act.setId(id);
                list.add(account);
            }
        } catch (SQLException e) {
            thro new RuntimeException(e);
        } finally {
            DBUtil.close(conn, ps, rs);
        }
        return list;
    }
}
```

创建一个类来存储SQL的数据的实例对象，简单的对象类又称为**POJO(Plain Ordinary Java Object，简单的JAVA对象，也有称为beam对象，意思一样；也有称为专门封装数据的对象，称为领域模型对象，domain)对象**

```java
public class Account{
	// 一般选用包装类而不是基本数据类型，防止null带来的错误
	private Long id;
	private String actno;
	private Double balance;
}
```

## AccountService

(service译为业务，专门做业务逻辑)

```java
// 失败的例子，业务和CRUD(使用了connection)混在了一起，并且与DAO的connection对象不同，很麻烦
public class AccountService{
    private AccountDao accountDao = new AccountDao();
    // 具体的转账业务
    public void transfer(String fromActno, String toActno, double money){
        try(Connection connection = DBUtil.getConnection()){
            connection.setAutoCommit(false); 
            Account fromAct = accountDao.selectByActno(fromActno);
            // 查询余额是否不足
            if(fromAct.getBalance() < money){
                throw new MoneyNotEnoughException("余额不足");
            }
            Account toAct = accountDao.selectByActno(toActno);
            // 修改余额
            fromAct.setBalance(fromAct.getBalanc() - money);
            int count = accountDao.update(fromAct);
            count += accountDao.update(toAct);
            if (count != 2){
                throw new ApplicationExcepiton("账号转账异常","账号接收异常");
            } 
            // connection要在servlet中写
            connection.commit();
        } catch(SQLException e){
            throw new Appection("事务异常");
        } finally{
        }
    }
}
```

```java
// 成功的例子，使用了递归中线程是同一对象这一特点；但还是使用了事务connection，不要在业务层使用事务
public class AccountService{
    private AccountDao accountDao = new AccountDao();
    // 具体的转账业务
    public void transfer(String fromActno, String toActno, double money){
        connection.setAutoCommit(false); 
        Account fromAct = accountDao.selectByActno(fromActno);
        // 查询余额是否不足
        if(fromAct.getBalance() < money){
            throw new MoneyNotEnoughException("余额不足");
        }
        Account toAct = accountDao.selectByActno(toActno);
        // 修改余额
        fromAct.setBalance(fromAct.getBalanc() - money);
        int count = accountDao.update(fromAct);
        count += accountDao.update(toAct);
        if (count != 2){
            throw new ApplicationExcepiton("账号转账异常","账号接收异常");
        } 
        // connection要在servlet中写
    }
}
```

## Controller

```java
// 记得加注解
@WebServlet("/transfer")
public class AccountServlet extends HttpServlet{
	@Overwrite
	protected void doPost(HttpServletRequest request, HttpServletResponse response)
    	throws ServletException, IOException{
        // 接收数据
		String fromActno = request.getParameter("fromActno");
		String toActno = request.getParameter("toActno"); 
		double money = Double.parseDouble(request.getParameter("money"));
        AccountService accountService = new AccountService();
        try{
            // 调用业务方法处理业务
            accountService.transfer(fromActno, toActno, money);
            // 执行到这里说明成功了
            response.sendRedirect(request.getContextPath() + "/success.jsp");
        // 钱不够了
        } catch(MoneyNotEnoughtException e){
        // 执行到这里，说明失败了
        } catch(Exception e){
            response.sendRedirect(request.getContextPath() + "/error.jsp");
        }
	}
}
```

## MyThreadLocal

利用线程对象是一样的，这样就不要connection处理事务了

```java
public class MyThreadLocal<T>{
	private Map<Thread, T> map = new Hashmap<>();
	public void set(T obj){
		map.put(Thread.currentThread(), obj);
	}
    public T get(){
        return map.get(Thread.currentThread());
    }
    public T remove(){
        map.remove(Thread.currentThread());
    }
}
```

在DBUtil中

```java
public class DButil{
	private MyThreadLocal<Connection> local = new MyThreadLocal<>();
    public static Connection getConnection(){
        Connection connection = local.get();
        if(connection == null){
            connection  = new Connection();
            local.set(connection);
        }
        return connection
    }
}
```

## 面向接口编程

降低耦合度，提高拓展力

其实就是把之前的类抽象成接口，具体实现类再放在impl文件夹下

```
-dao(M，持久层)
	-impl
-pojo(M)
	-impl
-exceptions(M)
	-impl
-service(M，业务层)
	-impl
-utils
	-impl
-web(C V)
	-impl
```

