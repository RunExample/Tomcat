# Tomcat

Study Tomcat by Example.

**Tomcat是Java Web Service**，其可以读取常见的静态文件，以及Java里特殊的如Servlet

这里将通过例子演示Tomcat的用法。

## tomcat 目录结构

此代码直接下载自tomcat官方网站 <http://tomcat.apache.org>

版本为 `9.0.29`

其目录分类如下:

```
tomcat
├── BUILDING.txt
├── CONTRIBUTING.md
├── LICENSE
├── NOTICE
├── README.md
├── RELEASE-NOTES
├── RUNNING.txt
├── bin
├── conf
├── lib
├── logs
├── temp
├── webapps
└── work
```

其中：

1. `bin`目录为可执行脚本目录，运行tomcat服务脚本来自此目录
2. `conf`目录为各种配置，例如运行 Server的端口号`8080` 配置于 `conf/server.xml`
2. `logs`目录打印各种日志

logs 目录地址 来自于 `conf/server.xml` 的一项配置

```xml
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
    prefix="localhost_access_log" suffix=".txt"
    pattern="%h %l %u %t &quot;%r&quot; %s %b" />
```

## 启动、停止Tomcat

Linux系统下调试运行如下所示，<kbd>Ctrl</kbd> + <kbd>c</kbd>结束

```sh
 ./tomcat/bin/catalina.sh run
```

Linux系统下后台运行如下所示

```sh
 ./tomcat/bin/catalina.sh start
```

结束后台运行如下所示

```sh
 ./tomcat/bin/catalina.sh stop
```

本项目主要用于测试Tomcat用法，所以最好用 `catalina.sh run`的方式前台运行，还能看到出错信息

## 原理分析

首先运行tomcat `./tomcat/bin/catalina.sh run`，默认运行端口为`8080`

本质上，tomcat是以特定的目录为http的根目录，该目录为`webapps/`

该目录之所以特殊是因为`conf/server.xml`中有一项配置:

```xml
<Host name="localhost"  appBase="webapps"
```

所有HTTP请求操作都是对该根目录文件的访问处理。

例如http `GET /examples/index.html` Tomcat将读取`./tomcat/webapps/examples/index.html`文件并返回给客户端

访问 <http://localhost:8080/examples/index.html> 将读取该文件

如上tomcat可以作为一个静态文件服务器

### Servlet

`tomcat/webapps/`目录下的各个文件夹可视为不同的项目

每个项目中都有一个特殊的名为`WEB-INF`(Web Info)的文件夹，该文件夹是安全目录，`GET /project/WEB-INF/abc.txt`之类的文件将无法获取到。

例如 <http://localhost:8080/examples/WEB-INF/web.xml>

将得到`HTTP 404 Not Found`

在`WEB-INF`目录下又有一个特殊的文件`web.xml`

此文件来自于配置 `conf/contents.xml`

```xml
<Context>

    <!-- Default set of monitored resources. If one of these changes, the    -->
    <!-- web application will be reloaded.                                   -->
    <WatchedResource>WEB-INF/web.xml</WatchedResource>
    <WatchedResource>WEB-INF/tomcat-web.xml</WatchedResource>
    <WatchedResource>${catalina.base}/conf/web.xml</WatchedResource>

    <!-- Uncomment this to disable session persistence across Tomcat restarts -->
    <!--
    <Manager pathname="" />
    -->
</Context>
```

此文件有许多重要的设置，其中就有设置`Servlet 路由`

#### Servlet 路由

Servlet是Http请求动态执行的Java文件，那么问题在于怎样的规则触发那个Servlet执行?

熟悉web api的都知道通常web api 例如 `GET /user/list` web服务器将通过 HTTP动作 `GET` 以及 url `/user/list` 两个条件查找某个函数执行

`动作+url` 作为key，执行函数作为value的一个mapping。

servlet可以看做是一个执行函数，我们还需要一个mapping规则，这个规则就在`web.xml`中设置，

例如查看`./tomcat/webapps/examples/WEB-INF/web.xml`文件，里面有如下mapping规则

```xml
<servlet>
    <servlet-name>HelloWorldExample</servlet-name>
    <servlet-class>HelloWorldExample</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>HelloWorldExample</servlet-name>
    <url-pattern>/servlets/servlet/HelloWorldExample</url-pattern>
</servlet-mapping>
```

当调用HTTP `GET /examples/servlets/servlet/HelloWorldExample`

```sh
curl -i localhost:8080/examples/servlets/servlet/HelloWorldExample
```

将执行`HelloWroldExample.class`的`doGet` 方法

返回结果为：

```
HTTP/1.1 200
Content-Type: text/html;charset=UTF-8
Content-Length: 395
Date: Fri, 22 Nov 2019 16:51:47 GMT

<!DOCTYPE html><html>
<head>
<meta charset="UTF-8" />
<title>你好，世界.</title>
</head>
<body bgcolor="white">
<a href="../helloworld.html">
<img src="../images/code.gif" height=24 width=24 align=right border=0 alt="view code"></a>
<a href="../index.html">
<img src="../images/return.gif" height=24 width=24 align=right border=0 alt="return"></a>
<h1>你好，世界.</h1>
</body>
</html>
```

现在我们找一下改返回结果的调用出处:

文件参见 `./tomcat/webapps/examples/WEB-INF/classes/HelloWorldExample.java`

```java
    @Override
    public void doGet(HttpServletRequest request,
                      HttpServletResponse response)
        throws IOException, ServletException
    {
        ResourceBundle rb =
            ResourceBundle.getBundle("LocalStrings",request.getLocale());
        response.setContentType("text/html");
        response.setCharacterEncoding("UTF-8");
        PrintWriter out = response.getWriter();

        out.println("<!DOCTYPE html><html>");
        out.println("<head>");
        out.println("<meta charset=\"UTF-8\" />");

        String title = rb.getString("helloworld.title");

        out.println("<title>" + title + "</title>");
        out.println("</head>");
        out.println("<body bgcolor=\"white\">");

        out.println("<a href=\"../helloworld.html\">");
        out.println("<img src=\"../images/code.gif\" height=24 " +
                    "width=24 align=right border=0 alt=\"view code\"></a>");
        out.println("<a href=\"../index.html\">");
        out.println("<img src=\"../images/return.gif\" height=24 " +
                    "width=24 align=right border=0 alt=\"return\"></a>");
        out.println("<h1>" + title + "</h1>");
        out.println("</body>");
        out.println("</html>");
    }
```

#### 小结

Servlet (确切说是HttpServlet) 相对于web api的handler，由Tomcat(视为Servlet Container)动态执行，

其路由(mapping)常配置在`WEB-INF/web.xml`中。
