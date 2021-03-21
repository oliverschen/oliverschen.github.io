---
title: Tomcat
date: 2020-01-08 23:25:55
tags: Tomcat
category: Tomcat
comments: true
---

![Photo by MatrizenDesign on wallhaven.cc](/tomcat.png)

Tomcat 是一款轻量级 Web 容器，也是目前使用最广泛的应用服务器之一，Springboot 内嵌的默认服务器就是 Tomcat。

<!--more-->
## Servlet

### 介绍

Sun公司在其API中提供了一个servlet接口，用户若想用发一个动态web资源(即开发一个Java程序向浏览器输出数据)，需要完成以下2个步骤：

1. 编写一个Java类，实现servlet接口
2. 把开发好的Java类部署到web服务器中

### 调用过程

Servlet 程序是由WEB服务器调用，web 服务器收到客户端的 Servlet 访问请求后：

1. Web 服务器首先检查是否已经装载并创建了该 Servlet 的实例对象。如果是，则直接执行第 4 步，否则，执行第 2 步。
2. 装载并创建该 Servlet 的一个实例对象。
3. 调用 Servlet 实例对象的 init() 方法。
4. 创建一个用于封装 HTTP 请求消息的 HttpServletRequest 对象和一个代表 HTTP 响应消息的HttpServletResponse 对象，然后调用 Servlet 的 service() 方法并将请求和响应对象作为参数传递进去。
5. WEB 应用程序被停止或重新启动之前，Servlet 引擎将卸载 Servlet，并在卸载之前调用 Servlet 的 destroy() 方法。

### 实现类

Servlet 接口 SUN 公司定义了两个默认实现类，分别为：GenericServlet、HttpServlet。

1. HttpServlet 指能够处理 HTTP 请求的 Servlet，它在原有 Servlet 接口上添加了一些与 HTTP 协议处理方法，它比 Servlet 接口的功能更为强大。因此开发人员在编写 Servlet 时，通常应继承这个类，而避免直接去实现 Servlet 接口。
2. HttpServlet 在实现 Servlet 接口时，覆写了 service 方法，该方法体内的代码会自动判断用户的请求方式，如为 GET 请求，则调用 HttpServlet 的doGet 方法，如为 Post 请求，则调用 doPost 方法。因此，在编写 Servlet 时，通常只需要覆写 doGet 或 doPost 方法，而不要去覆写 service 方法。

### URL 映射 

由于客户端是通过 URL 地址访问 web 服务器中的资源，所以 Servlet 程序若想被外界访问，必须把 Servlet 程序映射到一个 URL 地址上，这个工作在 `web.xml` 文件中使用 `<servlet>` 元素和 `<servlet-mapping>` 元素完成。

#### <servlet> 

该标签用于注册 Servlet，它包含有两个主要的子标签：`<servlet-name>` 和 `<servlet-class>` ，分别用于设置 Servlet 的注册名称和Servlet 的完整类名。

#### <servlet-mapping> 

该标签用于映射一个已注册的 Servlet 的一个对外访问路径，它包含有两个子元素：

`<servlet-name>` 和 `<url-pattern>` ，分别用于指定 Servlet 的注册名称和 Servlet 的对外访问路径。

```
<web-app>
  <servlet>
    <servlet-name>ck</servlet-name>
    <servlet-class>HelloServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>ck</servlet-name>
    <url-pattern>/ck.html</url-pattern>
  </servlet-mapping>
</web-app>
```

#### 注意

1. 同一个 Servlet 可以被映射到多个 URL 上，即多个 `<servlet-mapping>` 元素的 `<servlet-name>` 子元素的设置值可以是同一个 Servlet 的注册名。
2. 在 Servlet 映射到的 URL 中也可以使用 `*` 通配符，但是只能有两种固定的格式：一种格式是 `*.扩展名` ，另一种格式是以正斜杠 `/` 开头并以 `/*` 结尾。

```
<url-pattern>*.do</url-pattern>
<url-pattern>/action/*</url-pattern>
```

#### 说明

1. Servlet是一个供其他 Java 程序（Servlet 引擎）调用的 Java 类，它不能独立运行，它的运行完全由 Servlet 引擎来控制和调度。
2. 针对客户端的多次 Servlet 请求，通常情况下，服务器只会创建一个 Servlet 实例对象，也就是说 Servlet 实例对象一旦创建，它就会驻留在内存中，为后续的其它请求服务，直至 web 容器退出，Servlet 实例对象才会销毁。
3. 在 Servlet 的整个生命周期内，Servlet 的 init 方法只被调用一次。而对一个 Servlet 的每次访问请求都导致 Servlet 引擎调用一次 Servlet 的 service方法。对于每次访问请求，Servlet 引擎都会创建一个新的 HttpServletRequest 请求对象和一个新的 HttpServletResponse 响应对象，然后将这两个对象作为参数传递给它调用的 Servlet 的 service() 方法，service 方法再根据请求方式分别调用 doXXX 方法。
4. 在 <servlet> 元素中配置了一个 <load-on-startup> 元素，那么 WEB 应用程序在启动时，就会装载并创建 Servlet 的实例对象、以及调用 Servlet 实例对象的 init() 方法。
5. 如果某个 Servlet 的映射路径仅仅为一个正斜杠 `/` ，那么这个 Servlet 就成为当前 Web 应用程序的默认 Servlet。
6. 凡是在 web.xml 文件中找不到匹配的 <servlet-mapping> 元素的 URL，它们的访问请求都将交给缺省 Servlet 处理，也就是说，缺省 Servlet 用于处理所有其他 Servlet 都不处理的访问请求。
7. 在 <tomcat的安装目录>\conf\web.xml 文件中，注册了一个名称为 org.apache.catalina.servlets.DefaultServlet 的 Servlet，并将这个 Servlet 设置为了缺省 Servlet。
8. 当访问 Tomcat 服务器中的某个静态 HTML 文件和图片时，实际上是在访问这个缺省 Servlet。
9. 当多个客户端并发访问同一个 Servlet 时，web 服务器会为每一个客户端的访问请求创建一个线程，并在这个线程上调用 Servlet 的 service 方法，因此 service 方法内如果访问了同一个资源的话，就有可能引发线程安全问题。

### ServletConfig

1. 在 Servlet 的配置文件中，可以使用一个或多个 <init-param> 标签为 servlet 配置一些初始化参数。
2. 当 servlet 配置了初始化参数后，web 容器在创建 servlet 实例对象时，会自动将这些初始化参数封装到 ServletConfig 对象中，并在调用 servlet 的init 方法时，将 ServletConfig 对象传递给 servlet。进而通过 ServletConfig 对象就可以得到当前 servlet 的初始化参数信息。

### ServletContext

1. WEB 容器在启动时，它会为每个 WEB 应用程序都创建一个对应的 ServletContext 对象，它代表当前 web 应用。
2. ServletConfig 对象中维护了 ServletContext 对象的引用，开发人员在编写 servlet 时，可以通过 ServletConfig.getServletContext 方法获得ServletContext 对象。
3. 由于一个 WEB 应用中的所有 Servlet 共享同一个 ServletContext 对象，因此 Servlet 对象之间可以通过 ServletContext 对象来实现通讯。ServletContext 对象通常也被称之为 context 域对象。

#### 应用

1. 多个 Servlet 通过 ServletContext 对象实现数据共享。
2. 获取 WEB 应用的初始化参数。
3. 实现 Servlet 的转发。
4. 对于不经常变化的数据，在 servlet 中可以为其设置合理的缓存时间值，以避免浏览器频繁向服务器发送请求，提升服务器的性能。
5. 利用 ServletContext 对象读取资源文件。
6. 得到文件路径
7. 读取资源文件的三种方式
8. properties文件（属性文件）


### Filter

Servlet API 中提供了一个 Filter 接口，开发 web 应用时，如果编写的 Java 类实现了这个接口，则把这个 java 类称之为过滤器 Filter。通过 Filter 技术，开发人员可以实现用户在访问某个目标资源之前，对访问的请求和响应进行拦截。

#### 拦截流程

Filter 接口中有一个 doFilter()，配置对哪个 web 资源进行拦截后，WEB 服务器每次在调用 web 资源的 **service() 方法之前**，都会先调用一下 Filter 的 doFilter()。

#### FilterChain

1. web 服务器在调用 doFilter 方法时，会传递一个 filterChain 对象进来，filterChain 对象是 filter 接口中最重要的一个对象，它也提供了一个doFilter 方法，开发人员可以根据需求决定是否调用此方法，调用该方法，则 web 服务器就会调用 web 资源的 service 方法，即 web 资源就会被访问，否则 web 资源不会被访问。
2. 在一个 web 应用中，可以开发编写多个 Filter，这些 Filter 组合起来称之为一个 Filter 链。
3. web 服务器根据 Filter 在 web.xml 文件中的注册顺序，决定先调用哪个 Filter，当第一个 Filter 的 doFilter 方法被调用时，web 服务器会创建一个代表 Filter 链的 FilterChain 对象传递给该方法。在 doFilter 方法中，如果调用了 FilterChain 对象的 doFilter 方法，则 web 服务器会检查 FilterChain 对象中是否还有 filter，如果有，则调用第 2 个 filter，如果没有，则调用目标资源。

#### 生命周期

> init(FilterConfig filterConfig)throws ServletException：

1. 和 Servlet 程序一样，Filter 的创建和销毁由 WEB 服务器负责。 web 应用程序启动时，web 服务器将创建 Filter 的实例对象，并调用其 init 方法，完成对象的初始化功能，从而为后续的用户请求作好拦截的准备工作「注：filter 对象只会创建一次，init 方法也只会执行一次。」通过 init 方法的参数，可获得代表当前 filter 配置信息的 FilterConfig 对象
2. destroy()：在Web容器卸载 Filter 对象之前被调用。该方法在 Filter 的生命周期中仅执行一次。在这个方法中，可以释放过滤器使用的资源。

#### FilterConfig

用户在配置 filter 时，可以使用 `<init-param>` 为 filter 配置一些初始化参数，当 web 容器实例化 Filter 对象，调用其 init 方法时，会把封装了 filter初始化参数的 filterConfig 对象传递进来。因此通过 filterConfig 对象的方法，就可获得：

1. String getFilterName()：得到filter的名称。
2. String getInitParameter(String name)： 返回在部署描述中指定名称的初始化参数的值。如果不存在返回 null.
3. Enumeration getInitParameterNames()：返回过滤器的所有初始化参数的名字的枚举集合。
4. public ServletContext getServletContext()：返回 Servlet 上下文对象的引用。

#### 应用

##### 统一全站字符编码的过滤器

通过配置参数 encoding 指明使用何种字符编码,以处理 Html Form 请求参数的中文问题

##### 禁止浏览器缓存所有动态页面的过滤器：

有 3 个 HTTP 响应头字段都可以禁止浏览器缓存当前页面，它们在 Servlet 中的示例代码如下：

```
response.setDateHeader("Expires",-1);
response.setHeader("Cache-Control","no-cache");
response.setHeader("Pragma","no-cache");
```

并不是所有的浏览器都能完全支持上面的三个响应头，因此最好是同时使用上面的三个响应头。

1. Expires 数据头：值为GMT时间值，为-1指浏览器不要缓存页面
2. Cache-Control 响应头有两个常用值
3. no-cache 指浏览器不要缓存当前页面
4. max-age xxx指浏览器缓存页面xxx秒

##### 使用 Filter 实现 URL 级别的权限认证

把一些执行敏感操作的 servlet 映射到一些特殊目录中，并用 filter 把这些特殊目录保护起来，限制只能拥有相应访问权限的用户才能访问这些目录下的资源。从而在我们系统中实现一种 URL 级别的权限功能。

## JSP

### 运行原理

1. 每个JSP 页面在第一次被访问时，WEB 容器都会把请求交给 JSP 引擎（即一个 Java 程序）去处理。JSP 引擎先将 JSP 翻译成一个 _jspServlet(实质上也是一个 servlet ) ，然后按照 servlet 的调用方式进行调用。
2. 由于 JSP 第一次访问时会翻译成 servlet，所以第一次访问通常会比较慢，但第二次访问，JSP 引擎如果发现 JSP 没有变化，就不再翻译，而是直接调用，所以程序的执行效率不会受到影响。
3. JSP 引擎在调用 JSP 对应的 _jspServlet  时，会传递或创建 9 个与 web 开发相关的对象供 _jspServlet 使用。JSP 技术的设计者为便于开发人员在编写 JSP 页面时获得这些 web 对象的引用，特意定义了 9 个相应的变量，开发人员在 JSP 页面中通过这些变量就可以快速获得这 9 大对象的引用。

### 9大内置对象

1. Request
2. Response
3. Session
4. Application
5. Config
6. Page
7. Exception 
8. Out
9. PageContext

***

<center>何以为家</center>

