## JSP

JSP（全称Java Server Pages）是一种使软件开发者可以响应客户端请求，而动态生成 HTML、XML 或其他格式文档的Web网页的技术标准。 JSP 技术是以 Java 语言作为脚本语言的，JSP 网页为整个服务器端的 Java 库单元提供了一个接口来服务于HTTP的应用程序。JSP文件后缀名为 *.jsp 。可以使用JSP标签在HTML网页中插入Java代码，标签通常以 `<%` 开头以 `%>` 结束 。

#### 1、JSP处理过程：

- 浏览器向服务器发送一个HTTP请求；
- WEB服务器识别出这是一个对JSP网页的请求，并将该请求转发给JSP引擎，通过使用URL或者.jsp文件来完成；
- JSP引擎载入JSP文件，然后将它们转化为Servlet，这种转化只是简单地将模板文本改用 `println()` 语句，并将所有JSP元素转换成java代码；
- JSP引擎将Servlet编译成可执行类，并将原始请求传递给Servlet引擎；
- WEB 服务器的某组件将会调用 Servlet 引擎，然后载入并执行 Servlet 类。在执行过程中，Servlet 产生 HTML 格式的输出并将其内嵌于 HTTP response 中上交给 Web 服务器；
- WEB 服务器以静态 HTML 网页的形式将 HTTP response 返回到您的浏览器中；
- 最终，Web 浏览器处理 HTTP response 中动态产生的HTML网页，就好像在处理静态网页一样 。

#### 2、指令元素：

- **page指令：**

  主要用来定义当前JSP页面的属性参数，一般用来定义在页面的最上面，可以定义多个 `page` 指令。

  ```jsp
  <%@ page contentType="text/html;charset=UTF-8"%>
  import：指明该JSP可以使用那些java api
  pageEncoding：指明该JSP网页的编码方式
  contentType：表示MIME类型和JSP的编码方式
  buffer = "none/size KB"：设置缓冲区，默认是8KB
  Language：指明脚本片段使用的默认语言，默认java
  ```

- **include指令：**

  `<jsp:include  page =" "> ` 动作脚本，主要用来包含动态JSP页面，page：用来指定JSP页面地址。

  `<%@include file =" "%>` 静态脚本，主要用来复用JSP页面中的资源，file指定静态资源路径。

  ```jsp
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <html>
  <head>
      <title>Main</title>
  </head>
  <body>
      <a href="main.jsp?pageNum=1">第一页</a>
      <a href="main.jsp?pageNum=2">第二页</a>
      <a href="main.jsp?pageNum=3">第三页</a>
      <%
          String pageNum = request.getParameter("pageNum");
          if (pageNum != null) {
              pageNum = "1".equals(pageNum) ? "div1.jsp" : "2".equals(pageNum) ? "div2.jsp" : "div3.jsp";
          } else {
              pageNum = "div1.jsp";
          }
      %>
      <div>
          <jsp:include page="<%= pageNum%>"></jsp:include>
      </div>
  </body>
  </html>
  ```

- **taglib指令：**

  `<%@ taglib ... %> ` 引入标签库的定义。

  ```jsp
  <%@ taglib prefix="c" uri="http://java.sun.com/jstl/core_rt"%>
  ```


#### 3、JSP语法

- **脚本语法：** 脚本程序可以包含任意量的java语句、变量、方法或表达式，只要它们在脚本语言中是有效的。

  ```jsp
  <% 代码片段 %>
  <html>
  <head><title>Hello World</title></head>
  <body>
  Hello World!<br/>
  <% out.println("Your IP address is " + request.getRemoteAddr()); %>
  </body>
  </html>
  ```

- **JSP声明：**一个声明语句可以声明一个或多个变量、方法，以供后面的java程序使用，这些变量相当于全局变量。

  ```jsp
  <%! 声明变量或方法 %>
  ```

- **JSP表达式：**一个JSP表达式中包含的脚本语言表达式，先被转化成String，然后插入到表达式出现的地方，表达式元素中可以包含任何符合Java语言规范的表达式，但是不能使用分号来结束表达式 。

  ```jsp
  <%= 表达式 %>
  ```

  ```jsp
  <%@ page language="java" contentType="text/html; charset=UTF-8"
      pageEncoding="UTF-8"%>
  <!DOCTYPE html>
  <html>
  <head>
  <meta charset="utf-8">
  <title>时间日期</title>
  </head>
  <body>
  <p>
     今天的日期是: <%= (new java.util.Date()).toLocaleString()%>
  </p>
  </body> 
  </html>
  ```

- **JSP注释：**

  `<%-- 注释 --%> ` JSP注释，注释内容不会被发送至浏览器甚至不会被编译 ；

  `<!-- 注释 --> ` HTML注释，通过浏览器查看网页源代码时可以看见注释内容；

- **JSP行为：**

  |      语法       |                            描述                            |
  | :-------------: | :--------------------------------------------------------: |
  |   jsp:include   |             用于在当前页面中包含静态或动态资源             |
  |   jsp:useBean   |                寻找和初始化一个JavaBean组件                |
  | jsp:setProperty |                   设置 JavaBean组件的值                    |
  | jsp:getProperty |             将 JavaBean组件的值插入到 output中             |
  |   jsp:forward   | 从一个JSP文件向另一个文件传递一个包含用户请求的request对象 |
  |   jsp:plugin    |       用于在生成的HTML页面中包含Applet和JavaBean对象       |
  |   jsp:element   |                    动态创建一个XML元素                     |
  |  jsp:attribute  |                定义动态创建的XML元素的属性                 |
  |    jsp:body     |                定义动态创建的XML元素的主体                 |
  |    jsp:text     |                      用于封装模板数据                      |

- **JSP隐含对象：**

  |      对象       |                             描述                             |
  | :-------------: | :----------------------------------------------------------: |
  |   **request**   |                  HttpServletRequest类的实例                  |
  |  **response**   |                 HttpServletResponse类的实例                  |
  |     **out**     |         PrintWriter类的实例，用于把结果输出至网页上          |
  |   **session**   |                     HttpSession类的实例                      |
  | **application** |           ServletContext类的实例，与应用上下文有关           |
  |   **config**    |                    ServletConfig类的实例                     |
  | **pageContext** | PageContext类的实例，提供对JSP 页面所有对象以及命名空间的访问 |
  |    **page**     |                  类似于java类中的this关键字                  |
  |  **Exception**  |   Exception类的对象，代表发生错误的JSP页面中对应的异常对象   |


#### 4、EL表达式

- **语法：** ${EL表达式}

- **作用：** 

  1、用于表达式的运算；

  2、用作从作用域中取出数据；

- **获取数据：** 

  |  作用域  |              Java代码              |         EL表达式          |
  | :------: | :--------------------------------: | :-----------------------: |
  |  页面域  | pageContext.getAttribute(“color”); |    ${pageScope.color}     |
  |  请求域  |   request.getAttribute(“color”);   |   ${requestScope.color}   |
  |  会话域  |   session.getAttribute(“color”)    |   ${sessionScope.color}   |
  | 上下文域 | application.getAttribute(“color”)  | ${applicationScope.color} |
  | 自动查找 | pageContext.findAttribute(“color”) |         ${color}          |

- **EL中的隐式对象：**

  |   隐含对象名称   |                             描述                             |
  | :--------------: | :----------------------------------------------------------: |
  |   pageContext    |         代表页面上下文对象，可以在页面上调用get方法          |
  |    pageScope     |                    代表页面域中的Map对象                     |
  |   requestScope   |                    代表请求域中的Map对象                     |
  |   sessionScope   |                    代表会话域中的Map对象                     |
  | applicationScope |                   代表上下文域中的Map对象                    |
  |      param       |     得到表单提交的数据，功能与request.getParameter()相同     |
  |   paramValues    | 得到表单提交参数，功能与String[] request.getParameterValues()相同 |
  |      header      |             得到请求头数据，request.getHeader()              |
  |   headerValues   |             得到请求头数据，request.getHeader()              |
  |      cookie      |                     得到请求的Cookie信息                     |
  |    initParam     |    相当于config.getInitParameter()得到web.xml中的配置参数    |

- **pageContext调用的get方法：**

  |     作用     |              JSP表达式               |               EL表达式                |
  | :----------: | :----------------------------------: | :-----------------------------------: |
  | 当前工程路径 |   < %= request.getContextPath()% >   | ${pageContext.request.getContextPath} |
  | 请求资源路径 |   < %= request.getContextURL()% >    |   ${pageContext.request.contextURL}   |
  |  访问者的IP  |   < %= request.getRemoteAddr()% >    |   ${pageContext.request.remoteAddr}   |
  | 当前会话的ID | < %= request.getSession().getID()% > |   ${pageContext.request.session.id}   |

- **访问集合中的元素：**

  1、访问集合中的数据：\$ {requestScope.list} ；访问集合中第一个Student对象的sname属性值pageScope.stulist[0].sname：${pageScope.stulist[0].sname}

  2、访问map集合中的数据：访问Map中所有的Key=value对象pageScope.mapstring：\${pageScope.mapstring}；访问Map中指定的Key的对象pageScope.mapstring['key']：\${pageScope.mapstring['en1']} ；

#### 5、JSP标准标签库（JSTL）

JSP标准标签库（JSTL）是一个JSP标签集合，它封装了JSP应用的通用核心功能；JSTL支持通用的、结构化的任务，比如迭代，条件判断，XML文档操作，国际化标签，SQL标签。

- **核心标签：**

  |        标签         |                             描述                             |
  | :-----------------: | :----------------------------------------------------------: |
  |    **< c:out >**    |             用于在JSP中显示数据，相当于< %= % >              |
  |    **< c:set >**    |                         用于保存数据                         |
  |  **< c:remove >**   |                         用于删除数据                         |
  |    **< c:if >**     |                     和一般程序中的if一样                     |
  |  **< c:choose >**   |     只当做 **< c:when >**和**< c:otherwise >**  的父标签     |
  |   **< c:when >**    |        **< c:choose >**的子标签，用来判断条件是否成立        |
  | **< c:otherwise >** | **< c:choose >**的子标签，接在**< c:when >**标签后，当**< c:when >**标签判断为false时被执行 |
  |  **< c:forEach >**  |                基础迭代标签，接受多种集合类型                |

  **< c:forEach >标签的属性：**

  - **items：**要被循环的信息；
  - **var：**代表当前条目的变量名称；
  - **varStatus：**代表循环状态的变量名称，.index 从0开始的迭代索引，.count 从1开始的迭代索引；

  **< c:set >标签的属性：**

  - **value：** 要存储的值；
  - **var：** 存储信息的变量；
  - **scope：** var属性的作用域；

  **< c:when >标签的属性：**

  - **test：** 判断的条件；

  ```jsp
  <c:set var="salary" scope="session" value="${2000*2}"/>
  <p>你的工资为 : <c:out value="${salary}"/></p>
  <c:choose>
      <c:when test="${salary <= 0}">
         太惨了。
      </c:when>
      <c:when test="${salary > 1000}">
         不错的薪水，还能生活。
      </c:when>
      <c:otherwise>
          什么都没有。
      </c:otherwise>
  </c:choose>
  ```

- **格式化标签：**

  |         **标签**         |               **描述**               |
  | :----------------------: | :----------------------------------: |
  | **< fmt:formatNumber >** |    使用指定的格式或精度格式化数字    |
  |  **< fmt:formatDate >**  | 使用指定的风格或模式格式化日期和时间 |

  **< fmt:formatNumber >标签的属性：** 

  - **value：** 要显示的数字；
  - **pattern：** 指定一个自定义的格式化模式用于输出

  **< fmt:formatDate >标签的属性：** 

  - **value：** 要显示的日期；
  - **pattern：** 自定义格式模式；
  - **timeZone：** 显示日期的时区；