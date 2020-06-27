---
title: Spring MVC
top: true
cover: true
author: 张文军
date: 2020-05-13 19:34:24
tags: Spring MVC
category: Spring
summary: Spring MVC
---
<center>更多内容请关注：</center>

![Java快速开发学习](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1586869254.png)

<center><a href="https://it.njauit.cn">锁清秋</a></center>

----

# Spring MVC

## 基本概念

### MVC模型

MVC即Model-View-Controller，将应用按照Model（模型）、View（视图）、Controller（控制）这样的方式分离。

#### 视图(View)

视图(View)：代表用户交互界面，对于Web应用来说，可以是HTML，也可能是jsp、XML和Applet等。一个应用可能有很多不同的视图，MVC设计模式对于视图的处理仅限于视图上数据的采集和处理，以及用户的请求，而不包括在视图上的业务流程的处理。业务流程的处理交予模型(Model)处理。

SpringMVC 框架提供了很多的 View 视图类型的支持，包括： jstlView、 freemarkerView、 pdfView等。我们最常用的视图就是 jsp。一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由程序员根据业务需求开发具体的页面。

#### 模型(Model)

模型(Model)：是业务的处理以及业务规则的制定。模型接受视图请求的数据，并返回最终的处理结果。业务模型的设计是MVC最主要的核心。MVC设计模式告诉我们，把应用的模型按一定的规则抽取出来，抽取的层次很重要，抽象与具体不能隔得太远，也不能太近。MVC并没有提供模型的设计方法，而只是组织管理这些模型，以便于模型的重构和提高重用性。

#### 控制(Controller)

控制(Controller)：可以理解为从用户接收请求, 将模型与视图匹配在一起，共同完成用户的请求。划分控制层的作用也很明显，它清楚地告诉你，它就是一个分发器，选择什么样的模型，选择什么样的视图，可以完成什么样的用户请求。控制层并不做任何的数据处理。

### Spring MVC基本概念

1. 是一种基于Java实现的MVC设计模型的请求驱动类型的轻量级WEB框架。
2. Spring MVC属于SpringFrameWork的后续产品，已经融合在Spring Web Flow里面。Spring 框架提供了构建 Web 应用程序的全功能 MVC 模块。
3. 使用 Spring 可插入的 MVC 架构，从而在使用Spring进行WEB开发时，可以选择使用Spring的SpringMVC框架或集成其他MVC开发框架，如Struts1(现在一般不用)，Struts2等。

>springmvc是单例模式的框架,但它是线程安全的,因为springmvc没有成员变量,所有参数的封装都是基于方法的,属于当前线程的私有变量. 因此是线程安全的框架。所以效率高。

## Spring MVC 执行流程

### 基本流程

1. 当启动Tomcat服务器的时候，因为配置了load-on-startup标签，所以会创建DispatcherServlet(前端控制器)对象，就会加载springmvc.xml配置文件
2. springmvc.xml 中开启了注解扫描，那么 ControllerBean 对象（添加了@Controller的类）就会被创建
3. 从index.jsp发送请求，请求会先到达DispatcherServlet核心控制器，根据配置@RequestMapping注解
找到执行的具体方法
4. 根据执行方法的返回值，再根据配置的视图解析器(viewResolver)，去指定的目录下查找指定名称的JSP文件
5. Tomcat服务器渲染页面，做出响应
![执行过程](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1589394686.png)

## Spring MVC 执行流程详解

### Spring MVC 中重要的组件

#### DispatcherServlet：前端控制器

用户请求到达前端控制器，它就相当于 mvc 模式中的 c， dispatcherServlet 是整个流程控制的中心，由它调用其它组件处理用户的请求， dispatcherServlet 的存在降低了组件之间的耦合性。

#### HandlerMapping：处理器映射器

HandlerMapping 负责根据用户请求找到 Handler 即处理器， SpringMVC 提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。（即，根据前端控制器发过来的用户的请求参数找到相应的Controller，并将执行链返回给前端控制器）

#### Handler：处理器

它就是我们开发中要编写的具体业务控制器。由 DispatcherServlet 把用户请求转发到 Handler。由Handler 对具体的用户请求进行处理 （即：Controller类）

#### HandlAdapter：处理器适配器

通过 HandlerAdapter 对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。
    ![适配器](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1589395325.png)

#### View Resolver：视图解析器

View Resolver 负责将处理结果生成 View 视图， View Resolver 首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成 View 视图对象，最后对 View 进行渲染将处理结果通过页面展示给用户。

### 执行流程详解

![Spring MVC 执行流程](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1589395974.png)

1. 用户发送请求至前端控制器DispatcherServlet。

2. DispatcherServlet收到请求调用HandlerMapping处理器映射器。

3. 处理器映射器找到具体的处理器(可以根据xml配置. 注解进行查找)，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。

4. DispatcherServlet调用HandlerAdapter处理器适配器。

5. HandlerAdapter经过适配调用具体的处理器(Controller，也叫后端控制器)。

6. Controller执行完成返回ModelAndView。

7. HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet。

8. DispatcherServlet将ModelAndView传给ViewReslover视图解析器。

9. ViewReslover解析后返回具体View.

10. DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）。

11. DispatcherServlet响应用户。

## SpringMVC 常用注解及其作用

|注解|作用|
| --- | --- |
| @Controller： | 标识这个类是一个控制器 |
| @RequestMapping： | 给控制器方法绑定一个uri |
| @ResponseBody： | 将java对象转成json，并且发送给客户端 |
| @RequestBody： | 将客户端请求过来的json转成java对象 |
| @RequestParam： | 当表单参数和方法形参名字不一致时，做一个名字映射 |
| @PathVarible： | 拥有绑定url中的占位符的。例如：url中有/delete/{id}，{id}就是占位符 |
| Rest风格的新api |  |
| @RestController  | 相当于@Controller+ @ResponseBody |
| @GetMapping、@DeleteMapping、@PostMapping、@PutMapping  | 相当于 @Get + @requestMapping 。。。 |
| 其他注解 |  |
| @SessionAttribute： | 声明将什么模型数据存入session |
| @CookieValue： | 获取cookie值 |
| @ModelAttribute： | 将方法返回值存入model中 |
| @HeaderValue： | 获取请求头中的值 |
