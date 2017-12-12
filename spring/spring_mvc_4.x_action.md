---
categories: spring

tags: 
  - Spring
  - Java

title: Spring MVC 4.x 知识点

date: 2017-03-16
---

# 前言
Spring Framework的Web层，由spring-web，spring-webmvc，spring-websocket和spring-webmvc-portlet模块组成。

很多人刚学时分不清Spring Web/Spring MVC有什么区别

* spring-web模块提供基本的面向Web的集成功能，例如多文件上传功能和使用Servlet监听器和面向Web的应用程序上下文来初始化IoC容器。 它还包含一个HTTP客户端和Web的相关部分的Spring的远程支持。
* spring-webmvc模块（也称为Web-Servlet模块）包含用于Web应用程序的Spring的模型视图控制器（MVC）和REST Web服务实现。 Spring的MVC框架提供了域模型代码和Web表单之间的分离，并与Spring框架的所有其他功能集成。

Spring 优势

•支持REST风格的URL
•添加更多注解，可完全注解驱动
•引入HTTP输入输出转换器（HttpMessageConverter）
•和数据转换、格式化、验证框架无缝集成
•对静态资源处理提供特殊支持
•更加灵活的控制器方法签名，可完全独立于Servlet API

# Spring MVC4.X新特性

* 可以使用Groovy DSL定义外部bean配置(4.0)。
* 新增@RestController注解(4.0)。
* 新增AsyncRestTemplate类，开发REST客户端时允许非阻塞异步支持(4.0)。
* JDK 1.8的java.util.Optional现在支持@RequestParam，@RequestHeader和@MatrixVariable控制器方法参数(4.1)。
* Jackson的@JsonView直接支持@ResponseBody和ResponseEntity控制器方法(4.1)。
* 内置对CORS的支持(4.2)。
* 新的@GetMapping，@PostMapping，@PutMapping，@DeleteMapping和@PatchMapping @RequestMapping的组合注释(4.3)。
* 新的@SessionAttribute @RequestAttribute注释用于访问session、request属性(4.3)。

# Spring MVC框架结构

![image](http://blog.geekidentity.com/images/spring_mvc_4.x_overview/spring_mvc_structure.png)

Spring MVC是围绕DispatcherServlet设计的，DispatcherServlet向处理程序分发各种请求。处理程序默认基于@Controller和@RequestMapping注释。

Spring MVC的设计原则是开闭原则，所以Spring MVC核心类中一些方法是final的。

## 代码示例


```Java
@Controller //1.将UserController变成一个Handler
@RequestMapping(“/user”) //2.指定控制器映射的URL
public class UserController {

    @RequestMapping(value = “/register”) // 3.处理方法对应的URL，相对于2处的URL
    public String register() {
        return “user/register”; // 4.返回逻辑视图名
    }
}
```

# HTTP请求地址映射

Spring MVC框架通过扫描将带有@Controller的类中的@RequestMapping的方法进行映射，然后调用映射的方法处理请求，这个分发过程默认是由DispaterServlet处理的。

## HTTP请求映射原理

![image](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/images/mvc.png.pagespeed.ce.tmIzOTr1gg.png)

## Spring MVC进行映射的依据

![image](http://blog.geekidentity.com/images/spring_mvc_4.x_overview/spring_mvc_mapping_gist.jpg)

## 通过URL限定：URL表达式

Spring MVC的地址映射支持标准的URL，同时默认支持是[ant风格](http://blog.geekidentity.com/spring/ant_style_path_patterns/)的URL。列如：

URL | 说明
--- | ---
/account/*/create | 匹配/account/aaa/create、/account/bbb/create等URL
/account/\*\*/create | 匹配/account/create、/account/aaa/bbb/create等URL
/account/create?? | 匹配/account/createaa、/account/createbb等URL
/account/{accountId} | 匹配account/123、account/abc等URL
/account/**/{userId} | 匹配account/aaa/bbb/123、account/aaa/456等URL
account/{accountId}/customer/{customerId}/detail | 匹配account/1234/customer/0000/detail等的URL

## 通过URL限定：绑定路径中{xxx}的值


```Java
@RequestMapping("/{accountId}")
    public ModelAndView showDetail(@PathVariable("accountId")String accountId){
        ModelAndView mav= new ModelAndView();
        mav.setViewName("user/showDetail");
        mav.addObject("user", userService.getUserById(userId));
    return mav;
}
```

URL中的{xxx}占位符可以通过@PathVariable("xxx")绑定到操作方法的入参中。

在3.X中如果@PathVariable不指定参数名，只有在编译时打开debug开关（javac -debug=no）时才可行！！（不建议），貌似4.x没有这个问题了。

## 通过请求方法限定:请求方法


请求方法 | 说明
---|---
GET | 使用GET方法检索一个表述（representation）——也就是对资源的描述。多次执行同一GET请求，不会对系统造成影响，GET方法具有幂等性[指多个相同请求返回相同的结果]。GET请求可以充分使用客户端的缓存。
POST | POST方法，通常表示“创建一个新资源”，但它既不安全也不具有幂等性（多次操作会产生多个新资源）。
DELETE | 表示删除一个资源，你也可以一遍又一遍地操作它，直到得出结果：删除不存在的东西没有任何问题
PUT | 幂等性同样适用于PUT（基本的含义是“更新资源数据，如果资源不存在的话，则根据此URI创建一个新的资源”）

* 示例1：

```Java
@RequestMapping(value=“/delete”)
public String delete(@RequestParam("accountId") String accountId){
    return "accountId/test1";
}
```

所有URL为<controllerURI>/delete的请求由delete处理(任何请求方法)
* 示例2：

```Java
// 在4.3以上版本中，可以用@DeleteMapping
@RequestMapping(value="/delete",method=RequestMethod.DELETE)
public String delete(@RequestParam("accountId") String userId){
return "accountId/test1";
}
```

所有URL为<controllerURI>/delete 且请求方法为DELETE 的请求由delete方法处理

## 模拟请求方法

通过在web.xml中配置一个org.springframework.web.filter.HiddenHttpMethodFilter
通过POST请求的_method参数指定请求方法，HiddenHttpMethodFilter动态更改HTTP头信息。
![image](http://blog.geekidentity.com/images/spring_mvc_4.x_overview/hidden_http_method_filter.jpg)

## 通过请求/请求头参数限定:示例

* 通过请求参数限定：

```Java
@RequestMapping(value="/delete", params="accountId")
public String test1(@RequestParam("accountId") String accountId){
    //TODO
}
```
* 通过请求头参数限定：

```Java
@RequestMapping(value="/show",headers="content-type=text/*")②
public String test2(@RequestParam("userId") String userId){
    //TODO
}
```
## 通过请求/请求头参数限定:更多

params和headers分别通过请求参数及报文头属性进行映射，它们支持简单的表达式，下面以params表达式为例说明，headers可以参照params进行理解。
- "param1"：表示请求必须包含名为param1的请求参数。
- "!param1"：表示请求不能包含名为param1的请求参数。
- "param1!=value1"：表示请求包含名为param1的请求参数，但其值不能为value1。
- {"param1=value1","param2"}：请求必须包含名为param1和param2的两个请求参数，且param1参数的值必须为value1。

# HTTP请求数据的绑定

## 通过注解绑定:示意图

![image](http://blog.geekidentity.com/images/spring_mvc_4.x_overview/data_bind.jpg)

## 通过注解绑定:小心抛出异常

@RequestParam有以下三个参数。
- value/name：参数名。
- required：是否必需，默认为true，表示请求中必须包含对应的参数名，如果不存在将抛出异常。
- defaultValue：默认参数值，设置该参数时，自动将required设为false。

```Java
@RequestMapping(value="/handle")
public String handle(@RequestParam("name") String name,){
    //TODO ...
}
```
上面的处理方法，如果HTTP请求不包含“name”参数时，**将产生异常！！** 因此，如果不能保证存在”userName”的参数，建议使用：

```
@RequestParam(value = "name", required = false)
```

## 使用命令/表单对象绑定

所谓命令/表单对象并不需要实现任何接口，仅是一个拥有若干属性的POJO。Spring MVC按：

**“HTTP请求参数名= 命令/表单对象的属性名”**

的规则，自动绑定请求数据，支持“级联属性名”，自动进行基本类型数据转换。

```Java
@RequestMapping(value = "/handle")public String handle14(User user) {
    //TODO ...
}
```
![image](http://blog.geekidentity.com/images/spring_mvc_4.x_overview/form_to_pojo.jpg)

## 使用Servlet API对象作为入参

在Spring MVC中，控制器类可以不依赖任何Servlet API对象，但是Spring MVC并不阻止我们使用Servlet API的类作为处理方法的入参。值得注意的是，**如果处理方法自行使用HttpServletResponse返回响应，则处理方法的返回值设置成void即可**。

```Java
@RequestMapping(value = "/handle")
public void handle(HttpServletRequestrequest,HttpServletResponseresponse) {
    String userName= WebUtils.findParameterValue(request, "userName");
    response.addCookie(new Cookie("userName", userName));
}
```

```Java
public String handle(HttpSession session) {
    session.setAttribute("sessionId", 1234);
    //TODO ...
    return "success";
}
```

```Java
public String handle(HttpServletRequest request,
@RequestParam("userName")String userName) {
    //TODO ...
    return "success";
}
```

## 使用Spring的Servlet API代理类

Spring MVC在org.springframework.web.context.request包中定义了若干个可代理Servlet原生API类的接口，如WebRequest和NativeWebRequest，它们也允许作为处理类的入参，通过这些代理类可访问请求对象的任何信息。

```Java
@RequestMapping(value = "/handle")
public String handle(WebRequest request) {
    String userName = request.getParameter("userName");
    return "success";
}
```
## 使用IO对象作为入参

Spring MVC允许控制器的处理方法使用java.io.InputStream/java.io.Reader及java.io.OutputStream/java.io.Writer作为方法的入参

```Java
@RequestMapping(value = "/handle")
public void handle(OutputStream os) throws IOException{
    Resource res = new ClassPathResource("/image.jpg");//读取类路径下的图片文件
    FileCopyUtils.copy(res.getInputStream(), os);//将图片写到输出流中
}
```
Spring MVC将获取ServletRequest的InputStream/Reader或ServletResponse的OutputStream/Writer，然后按类型匹配的方式，传递给控制器的处理方法入参。

## 其他类型的参数

控制器处理方法的入参除支持以上类型的参数以外，还支持java.util.Locale、java.security.Principal，可以通过Servlet的HttpServletRequest 的getLocale()及getUserPrincipal()得到相应的值。如果处理方法的入参类型为Locale或Principal，Spring MVC自动从请求对象中获取相应的对象并传递给处理方法的入参。

```Java
@RequestMapping(value = "/handle")
public void handle(Locale locale) throws IOException{
    //TODO ...
}
```
## HttpMessageConverter<T>

![image](http://blog.geekidentity.com/images/spring_mvc_4.x_overview/httpMessageConverter.jpg)

实现类：
![image](http://blog.geekidentity.com/images/spring_mvc_4.x_overview/httpMessageConverterImpls.jpg)

HttpMessageConverter所有实现类将会注册到AnnotationMethodHandlerAdapter

## 使用@RequestBody/@ResponseBody

其原理大概是将HttpServletRequest的getInputStream()内容绑定到入参，将处理方法返回值写入到HttpServletResponse的getOutputStream()中。

```Java
@RequestMapping(value = "/handle")
public String handle(@RequestBodyString requestBody) {
    System.out.println(requestBody);
    return "success";
}
```

```Java
@ResponseBody
@RequestMapping(value = "/handle/{imageId}")
public byte[] handle(@PathVariable("imageId") String imageId) throws IOException{
    System.out.println("load image of "+imageId);
    Resource res = new ClassPathResource("/image.jpg");
    byte[] fileData=FileCopyUtils.copyToByteArray(res.getInputStream();
    return fileData;
}
```
* 优点：处理方法签名灵活不受限
* 缺点：只能访问报文体，不能访问报文头

## 使用HttpEntity<T>/ResponseEntity<T>

```Java
@RequestMapping(value = "/handle")
public String handle(HttpEntity<String> httpEntity){
    long contentLen= httpEntity.getHeaders().getContentLength();
    System.out.println(httpEntity.getBody());
    return "success";
}
```

```Java
@RequestMapping(params = "method=login")
public ResponseEntity<String> doFirst(){
    HttpHeadersheaders = new HttpHeaders();
    MediaTypemt=new MediaType("text","html",Charset.forName("UTF-8"));
    headers.setContentType(mt);
    ResponseEntity<String> re=null;
    String return = new String("test");
    re=new ResponseEntity<String>(return,headers, HttpStatus.OK);
    return re;
}
```

* 优点：不但可以访问报文体，还能访问报文头
* 缺点：处理方法签名受限

对于服务端的处理方法而言，除使用@RequestBody/@ResponseBody或HttpEntity<T> /ResponseEntity<T>进行方法签名外，不需要进行任何额外的处理，借由Spring MVC中装配的HttpMessageConverter，它即拥有了处理XML及JSON的能力了。

# 数据转换、格式化、校验

![image](http://blog.geekidentity.com/images/spring_mvc_4.x_overview/data_binder.jpg)

低版本的Spring 只支持标准的PropertyEditor类型体系，不过PropertyEditor存在以下缺陷：
* 只能用于字符串和Java对象的转换，不适用于任意两个Java类型之间的转换；
* 对源对象及目标对象所在的上下文信息（如注解、所在宿主类的结构等）不敏感，在类型转换时不能利用这些上下文信息实施高级转换逻辑。

有鉴于此，Spring 3.0在核心模型中添加了一个通用的类型转换模块，ConversionService是Spring类型转换体系的核心接口。

Spring 3.0同时支持PropertyEditor和ConversionService 进行类型转换，在Bean配置、Spring MVC处理方法入参绑定中使用类型转换体系进行工作。

## 基于ConversionService体系，定义自定义的类型转换器

![image](http://blog.geekidentity.com/images/spring_mvc_4.x_overview/customer_converter.jpg)

```XML
<mvc:annotation-drivenconversion-service="conversionService"/>
<bean id="conversionService"
class="org.springframework.context.support.ConversionServiceFactoryBean">
    <property name="converters">
        <list>
            <bean class="com.geekidentity.StringToUserConverter"/>
        </list>
    </property>
</bean>
```
## 格式化：带格式字符串内部对象相互转换

![image](http://blog.geekidentity.com/images/spring_mvc_4.x_overview/formatter_interconversion.jpg)

```XML
<mvc:annotation-driven conversion-service="conversionService"/>
<bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
    <property name="converters">
        <list>
            <bean class="com.geekidentity.StringToUserConverter"/>
        </list>
    </property>
</bean>
```
值得注意的是，<mvc:annotation-driven/>标签内部默认创建的ConversionService实例就是一个FormattingConversionServiceFactoryBean，自动支持如下的格式化注解：
* @NumberFormatter：用于数字类型对象的格式化。
* @CurrencyFormatter：用于货币类型对象的格式化。
* @PercentFormatter：用于百分数数字类型对象的格式化。

## 数据校验框架

Spring 4.0拥有自己独立的数据校验框架，同时支持JSR 303标准的校验框架。Spring 的DataBinder在进行数据绑定时，可同时调用校验框架完成数据校验工作。在Spring MVC中，则可直接通过注解驱动的方式进行数据校验。
Spring的org.springframework.validation是校验框架所在的包

## JSR 303

JSR 303是Java为Bean数据合法性校验所提供的标准框架，它已经包含在Java EE 6.0中。JSR 303通过在Bean属性上标注类似于@NotNull、@Max等标准的注解指定校验规则，并通过标准的验证接口对Bean进行验证。
你可以通过http://jcp.org/en/jsr/detail?id=303了解JSR 303的详细内容。


注 解 | 功能说明
---|---
@Null | 被注释的元素必须为null
@NotNull | 被注释的元素必须不为null
@AssertTrue | 被注释的元素必须为true
@AssertFalse | 被注释的元素必须为false
@Min(value) | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值
@Max(value) | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值
@DecimalMin(value) | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值
@DecimalMax(value) | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值
@Size(max, min) | 被注释的元素的大小必须在指定的范围内
@Digits (integer, fraction) | 被注释的元素必须是一个数字，其值必须在可接受的范围内
@Past | 被注释的元素必须是一个过去的日期
@Future | 被注释的元素必须是一个将来的日期

注意：Spring本身没有提供JSR 303的实现，所以必须将JSR 303的实现者（如[Hibernate Validator](http://hibernate.org/validator/)）的jar文件放到类路径下，Spring将自动加载并装配好JSR 303的实现者。


```Java
@Controller
@RequestMapping("/user")
public class UserController{
    @RequestMapping(value = "/handle91")
        public String handle91(@Valid User user, BindingResultbindingResult){
        if(bindingResult.hasErrors()){
            return "/user/register";
        }else{
            return "/user/showUser";
        }
    }
}
```
在已经标注了JSR 303注解的表单/命令对象前标注一个@Valid，Spring MVC框架在将请求数据绑定到该入参对象后，就会调用校验框架根据注解声明的校验规则实施校验。

## 使用校验功能时，处理方法要如何签名？？

```Java
public String handle(@Valid User user, 
    BindingResult userBindingResult,
    String sessionId,ModelMap mm,
    @Valid Dept dept, Errors deptErrors){
```
Spring MVC是通过对处理方法签名的规约来保存校验结果的：
前一个表单/命令对象的校验结果保存在其后的入参中，这个保存校
验结果的入参必须是BindingResult或Errors类型，这两个类都位于
org.springframework.validation包中。

## 校验错误信息存放位置

![image](http://blog.geekidentity.com/images/spring_mvc_4.x_overview/valid_error_msg.jpg)

* 4.Spring MVC将HttpServletRequest对象数据绑定到处理方法的入
参对象中（表单/命令对象）；
* 5.将绑定错误信息、检验错误信息都保存到隐含模型中；
* 6.本次请求的对应隐含模型数据存放到HttpServletRequest的属性列
表中，暴露给视图对象。

## 页面显示错误信息

```HTML
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<html>
<head>
<title>注册用户</title>
    <style>.errorClass{color:red}</style>
</head>
<body>
    <form:form modelAttribute="user" action="user/handle91.html">
    <form:errors path="*"/>
        <table>
            <tr>
            <td>用户名：</td>
            <td>
                <form:errors path="userName" cssClass="errorClass"/>
                <form:input path="userName" />
            </td>
            </tr>
            …
        </table>
    </form:form>
</body>
</html>
```
Spring MVC还可以对错误信息进行很好的国际化。

# 数据模型控制

## 数据模型访问结构

![image](http://blog.geekidentity.com/images/spring_mvc_4.x_overview/data_model_access_structure.jpg)

## 访问数据模型:ModelAndView

通过ModelAndView：

```Java
@RequestMapping(method = RequestMethod.POST)
public ModelAndViewcreateUser(User user) {
    userService.createUser(user);
    ModelAndViewmav= new ModelAndView();
    mav.setViewName("user/createSuccess");
    mav.addObject("user", user);
    return mav;
}
```

## 访问数据模型：@ModelAttribute

1. 使用方式一

```
@RequestMapping(value = "/handle")
public String handle(@ModelAttribute("user")User user){
    user.setUserId("1000");
    return "/user/createSuccess";
}
```
Spring MVC将HTTP请求数据绑定到user入参中，然后再将user对象添加到数据模型中。

2. 使用方式二

```Java
@ModelAttribute("user")
public User getUser(){
    User user = new User();
    user.setUserId("1001");
    return user;
}
```
访问UserController中任何一个请求处理方法前，Spring MVC先执行该方法，并将返回值以user为键添加到模型中

## 访问数据模型：Map及Model

org.springframework.ui.Model和java.util.Map:

```Java
@RequestMapping(value = "/handle")
public String handle(ModelMap modelMap){
    modelMap.addAttribute("username","value");
    User user = (User)modelMap.get("user");
    user.setUserName("tom");
    return "/user/showUser";
}
```
Spring MVC一旦发现处理方法有Map或Model类型的入参，就会将请求内在的隐含模型对象的引用传给这些入参。

## 访问数据模型：@SessionAttributes

如果希望在多个请求之间共用某个模型属性数据，则可以在控制器类标注一个@SessionAttributes，Spring MVC会将模型中对应的属性暂存到HttpSession中:

```Java
@Controller
@RequestMapping("/user")
@SessionAttributes(“user”) //1. 将2 处的模型属性自动保存到HttpSession中
public class UserController {
    @RequestMapping(value = "/handle")
    public String handle1(@ModelAttribute(“user”) User user){ //2
        user.setUserName("geek");
        return "redirect:/user/handle1.html";
    }
    @RequestMapping(value = "/handle")
    public String handle2(ModelMap modelMap,SessionStatus sessionStatus){
        User user = (User)modelMap.get(“user”); //3. 读取模型中的数据
        if(user != null){
            user.setUserName("Jetty");
            sessionStatus.setComplete(); //4. 让Spring MVC清除本处理器对应的会话属性
        }
        return "/user/showUser";
    }
}
```

## 由@SessionAttributes引发的血案


```
org.springframework.web.HttpSessionRequiredException: 
    Session attribute 'user' required -not found in session...
```
对入参标注@ModelAttribute(“xxx”)的处理方法，Spring MVC按如下流程处理（handle1(@ModelAttribute(“user”) User user)）：

1. 如果隐含模型拥有名为xxx的属性，将其赋给该入参，再用请求消息填充该入参对象直接返回，否则到2步。
2. 如果xxx是会话属性，即在处理类定义处标注了@SessionAttributes("xxx")，则尝试从会话中获取该属性，并将其赋给该入参，然后再用请求消息填充该入参对象。如果在会话中找不到对应的属性，则抛出HttpSessionRequiredException异常。否则到3。
3. 如果隐含模型不存在xxx属性，且xxx也不是会话属性，则创建入参的对象实例，再用请求消息填充该入参。

# 视图及解析器

![image](http://blog.geekidentity.com/images/spring_mvc_4.x_overview/spring_mvc_parse_view.jpg)

## 视图解析器类型

单一解析逻辑的视图解析器：
* InternalResourceViewResolver
* FreeMarkerViewResolver
* BeanNameViewResolver
* XmlViewResolver
* ...

基于协商的视图解析器：
* ContentNegotiatingViewResolver

该解析器是Spring 3.0新增的，它不负责具体的视图解析，而是作为一个中间人的角色根据请求所要求的MIME类型，从上下文中选择一个适合的视图解析器，再将视图解析工作委托其负责

```XML
<bean class="org.springframework.web.servlet.view.ContentNegotiatingViewResolver"
    p:order="0" p:defaultContentType="text/html" p:ignoreAcceptHeader="true"
    p:favorPathExtension="false" p:favorParameter="true" p:parameterName="content">
    <property name="mediaTypes">
        <map>
            <entry key="html" value="text/html" />
            <entry key="xml" value="application/xml" />
            <entry key="json" value="application/json" />
        </map>
    </property>
    <property name="defaultViews">
        <list>
            <bean class="org.springframework.web.servlet.view.json.MappingJacksonJsonView"
            p:renderedAttributes="userList" />
            <bean class="org.springframework.web.servlet.view.xml.MarshallingView"
            p:modelKey="userList" p:marshaller-ref="xmlMarshaller" />
        </list>
    </property>
</bean>
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"
    p:order="100" p:viewClass="org.springframework.web.servlet.view.JstlView"
    p:prefix="/WEB-INF/views/" p:suffix=".jsp" />

```

# 静态资源处理

Spring MVC 可以提供的强大的静态资源处理和映射的功能。
这样可以更方便的实现REST风格API。

1. web.xml让所有请求都由Spring MVC处理

```XML
<servlet>
    <servlet-name>springServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:/spring/spring-mvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>springServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```
2. spring-mvc.xml让Web应用服务器处理静态资源

```XML
<mvc:default-servlet-handler/>
```
获取应用服务器的默认Servlet,大多数应用服务器的Servlet的名称都是“default”，如果默认不是“default”则使用

```XML
<mvc:default-servlet-handler default-servlet-name=“<defaultServletName>"/>
```
3. 物理静态资源路径映射逻辑资源路径

```XML
<mvc:resources mapping="/resources/**"
    location="/,classpath:/META-INF/publicResources/"/>
```

# 国际化

## 本地化:基础原理

一般，Web应用根据客户端浏览器的设置判断客户端的本地化类型，用户可以通过IE菜单：工具→Internet选项...→语言...在打开的“语言首选项”对话框中选择本地化类型。

浏览器中设置的本地化类型会包含在HTML请求报文头中发送给Web服务器，确切地说是通过报文头的Accept-Language参数将“语言首选项”对话框中选择的语言发送到服务器，成为服务器判别客户端本地化类型的依据。

## 本地化:Spring MVC的本地化解析器

* AcceptHeaderLocaleResolver：根据HTTP报文头的Accept-Language参数确定本地化类型，如果没有显式定义本地化解析器，Spring MVC默认采用AcceptHeader-LocaleResolver。
* CookieLocaleResolver：根据指定Cookie值确定本地化类型。
* SessionLocaleResolver：根据Session中特定的属性值确定本地化类型。
* LocaleChangeInterceptor：从请求参数中获取本次请求对应的本地化类型。

## LocaleChangeInterceptor：通过URL参数指定

很多国际型的网站都允许通过一个请求参数控制网站的本地化，如www.xxx.com? locale=zh_CN返回对应中国大陆的本地化网页，而www.xxx.com?locale=en返回本地化为英语的网页。这样，网站使用者可以通过URL的控制返回不同本地化的页面，非常灵活。

```
<bean id="localeResolver"class="org.springframework.web.servlet.i18n.CookieLocaleResolver"
    p:cookieName="clientLanguage"
    p:cookieMaxAge="100000"
    p:cookiePath="/"
    p:defaultLocale="zh_CN"/>
<mvc:interceptors>
    <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor" />
</mvc:interceptors>
```
