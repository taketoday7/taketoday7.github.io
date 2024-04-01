---
layout: post
title:  Spring MVC面试题
category: interview
copyright: interview
excerpt: Spring MVC
---

## 1. 简介

Spring MVC是基于Servlet API构建的Spring的原始Web框架，它提供了可用于开发灵活的Web应用程序的模型-视图-控制器架构。

在本教程中，我们将重点讨论与之相关的问题，因为它通常是Spring开发人员工作面试的主题。

更多关于Spring框架的问题，可以查看我们[面试题系列](https://www.baeldung.com/tag/interview/)的另一篇[Spring相关文章](https://www.baeldung.com/spring-interview-questions)。

## 2. Spring MVC基础题

### Q1. 为什么要使用Spring MVC？

**Spring MVC实现了明确的关注点分离，使我们能够轻松地开发和单元测试我们的应用程序**。

这些概念包括：

-   DispatcherServlet
-   控制器
-   视图解析器
-   视图、模型
-   ModelAndView
-   模型和会话属性

彼此完全独立，只负责一件事。

因此，**MVC为我们提供了相当大的灵活性**。它基于接口(提供了实现类)，我们可以使用自定义接口配置框架的每个部分。

**另一件重要的事情是我们不依赖于特定的视图技术(例如JSP)，而是可以选择我们最喜欢的技术**。

此外，**我们不仅在Web应用程序开发中使用Spring MVC，在创建RESTful Web服务时也使用它**。

### Q2. @Autowired注解的作用是什么？

**@Autowired注解可以与字段或方法一起使用，以按类型注入bean**。此注解允许Spring解析协作bean并将其注入到你的bean中。

有关更多详细信息，请参阅有关[Spring @Autowired](https://www.baeldung.com/spring-autowire)的教程。

### Q3. 解释模型属性

@ModelAttribute注解是Spring MVC中最重要的注解之一。**它将方法参数或方法返回值绑定到命名模型属性，然后将其公开给Web视图**。

如果我们在方法级别使用它，则表明该方法的目的是添加一个或多个模型属性。

另一方面，当用作方法参数时，它表示应从模型中检索参数。当不存在时，我们应该首先实例化它，然后将它添加到模型中。一旦出现在模型中，我们应该从所有具有匹配名称的请求参数中填充参数字段。

有关此注解的更多信息，请参阅与[@ModelAttribute注解相关的文章](https://www.baeldung.com/spring-mvc-and-the-modelattribute-annotation)。

### Q4. 解释@Controller和@RestController之间的区别

@Controller和@RestController注解之间的主要区别是**@ResponseBody注解自动包含在@RestController中**，这意味着我们不需要使用@ResponseBody标注我们的处理程序方法。如果我们想将响应类型直接写入HTTP响应主体，我们需要在@Controller类中执行此操作。

### Q5. 描述PathVariable

**我们可以使用@PathVariable注解作为处理程序方法参数，以便提取URI模板变量的值**。

例如，如果我们想从www.mysite.com/user/123中按id获取用户，我们应该将控制器中的方法映射为/user/{id}：

```java
@RequestMapping("/user/{id}")
public String handleRequest(@PathVariable("id") String userId, Model map) {}
```

**@PathVariable只有一个名为value的元素，它是可选的，我们使用它来定义URI模板变量名**。如果我们省略value元素，那么URI模板变量名称必须与方法参数名称相匹配。

允许有多个@PathVariable注解，通过一个接一个地声明它们：

```java
@RequestMapping("/user/{userId}/name/{userName}")
public String handleRequest(@PathVariable String userId, @PathVariable String userName, Model map) {}
```

或者将它们全部放在Map<String, String\>或MultiValueMap<String, String\>中：

```java
@RequestMapping("/user/{userId}/name/{userName}")
public String handleRequest(@PathVariable Map<String, String> varsMap, Model map) {}
```

### Q6. 使用Spring MVC进行验证

**Spring MVC默认支持JSR-303规范，我们需要将JSR-303及其实现依赖项添加到我们的Spring MVC应用程序中**。例如，Hibernate Validator是我们可以使用的JSR-303实现之一。

JSR-303是用于bean验证的Java API规范，是Jakarta EE和JavaSE的一部分，它使用@NotNull、@Min和@Max等注解确保bean的属性满足特定标准。有关验证的更多信息，请参阅[Java Bean验证基础知识](https://www.baeldung.com/javax-validation)一文。

**Spring提供了@Validator注解和BindingResult类**。当我们输入无效数据时，Validator实现将在控制器请求处理程序方法中引发错误，然后我们可以使用BindingResult类来获取这些错误。

除了使用现有的实现之外，我们还可以创建自己的实现。为此，我们首先创建一个符合JSR-303规范的注解。然后，我们实现Validator类。另一种方法是实现Spring的[Validator](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/validation/Validator.html)接口，并通过Controller类中的@InitBinder注解将其设置为验证器。

要了解如何实现和使用你自己的验证器，请参阅有关Spring MVC中[自定义验证器](https://www.baeldung.com/spring-mvc-custom-validator)的教程。

### Q7. @RequestBody和@ResponseBody注解是什么？

**@RequestBody注解用作处理程序方法参数，将HTTP请求主体绑定到传输或域对象**。Spring使用Http消息转换器自动将传入的HTTP请求反序列化为Java对象。

**当我们在Spring MVC控制器中的处理程序方法上使用@ResponseBody注解时，它表示我们将方法的返回类型直接写入HTTP响应主体**。我们不会将它放在Model中，Spring也不会将其解释为视图名称。

请查看有关[@RequestBody和@ResponseBody](https://www.baeldung.com/spring-request-response-body)的文章，了解有关这些注解的更多详细信息。

### Q8. 解释Model、ModelMap和ModelAndView

**Model接口定义了模型属性的容器。ModelMap具有类似的用途，能够传递值集合。然后它将这些值视为在Map中**。我们应该注意，在模型(ModelMap)中我们只能存储数据，我们放入数据并返回视图名称。

另一方面，**对于ModelAndView，我们返回对象本身**。我们在要返回的对象中设置所有必需的信息，例如数据和视图名称。

你可以在有关[Model、ModelMap和ModelView](https://www.baeldung.com/spring-mvc-model-model-map-model-view)的文章中找到更多详细信息。

### Q9. 解释SessionAttributes和SessionAttribute

**@SessionAttributes注解用于在用户会话中存储模型属性**，我们在控制器类级别使用它，如我们关于[Spring MVC中的会话属性](https://www.baeldung.com/spring-mvc-session-attributes)的文章所示：

```java
@Controller
@RequestMapping("/sessionattributes")
@SessionAttributes("todos")
public class TodoControllerWithSessionAttributes {

    @GetMapping("/form")
    public String showForm(Model model, @ModelAttribute("todos") TodoList todos) {
        // method body
        return "sessionattributesform";
    }

    // other methods
}
```

在前面的示例中，如果@ModelAttribute和@SessionAttributes具有相同的name属性，则模型属性“todos”将被添加到会话中。

**如果我们想从全局管理的会话中检索现有属性，我们将使用@SessionAttribute注解作为方法参数**：

```java
@GetMapping
public String getTodos(@SessionAttribute("todos") TodoList todos) {
    // method body
    return "todoView";
}
```

### Q10. @EnableWebMVC的目的是什么？

**@EnableWebMvc注解的目的是通过Java配置启用Spring MVC**，它相当于XML配置中的\<mvc:annotation-driven\>。此注解从WebMvcConfigurationSupport导入Spring MVC配置，它支持使用@RequestMapping将传入请求映射到处理程序方法的@Controller注解类。

你可以在我们的[Spring @Enable注解](https://www.baeldung.com/spring-enable-annotations)指南中了解有关此注解和类似注解的更多信息。

### Q11. Spring中的ViewResolver是什么？

**ViewResolver通过将视图名称映射到实际视图**，使应用程序能够在浏览器中呈现模型，而无需将实现绑定到特定的视图技术。

有关ViewResolver的更多详细信息，请查看我们的[Spring MVC中的ViewResolver指南](https://www.baeldung.com/spring-mvc-view-resolver-tutorial)。

### Q12. 什么是BindingResult？

**BindingResult是org.springframework.validation包中的一个接口，表示绑定结果。我们可以使用它来检测和报告提交的表单中的错误**，它很容易调用-我们只需要确保将它作为参数放在我们正在验证的表单对象之后。可选的Model参数应该在BindingResult之后，如[自定义验证器教程](https://www.baeldung.com/spring-mvc-custom-validator)中所示：

```java
@PostMapping("/user")
public String submitForm(@Valid NewUserForm newUserForm, BindingResult result, Model model) {
    if (result.hasErrors()) {
        return "userHome";
    }
    model.addAttribute("message", "Valid form");
    return "userHome";
}
```

**当Spring看到@Valid注解时，它会首先尝试为被验证的对象找到验证器，然后它将获取验证注解并调用验证**器。最后，它将发现的错误放在BindingResult中，并将后者添加到视图模型中。

### Q13. 什么是表单支持对象？

**表单支持对象或命令对象只是一个从我们提交的表单中收集数据的POJO**。

我们应该记住它不包含任何逻辑，只包含数据。

要了解如何在Spring MVC中将表单支持对象与表单一起使用，请查看我们关于[Spring MVC中的表单](https://www.baeldung.com/spring-mvc-form-tutorial)文章。

### Q14. @Qualifier注解的作用是什么？

**它与@Autowired注解同时使用，以避免在存在多个bean类型实例时发生混淆**。

让我们看一个例子，我们在XML配置中声明了两个类似的bean：

```xml
<bean id="person1" class="cn.tuyucheng.taketoday.Person" >
    <property name="name" value="Joe" />
</bean>
<bean id="person2" class="cn.tuyucheng.taketoday.Person" >
    <property name="name" value="Doe" />
</bean>
```

当我们尝试注入bean时，我们会得到一个org.springframework.beans.factory.NoSuchBeanDefinitionException。要修复它，我们需要使用@Qualifier来告诉Spring应该注入哪个bean：

```java
@Autowired
@Qualifier("person1")
private Person person;
```

### Q15. @Required注解的作用是什么？

**@Required注解用于setter方法，它指示必须在配置时填充具有此注解的bean属性。否则，Spring容器将抛出BeanInitializationException异常**。

此外，@Required与@Autowired不同-因为它仅限于setter，而@Autowired不是。@Autowired也可用于构造函数和字段，而@Required仅检查属性是否已设置。

让我们看一个例子：

```java
public class Person {
    private String name;

    @Required
    public void setName(String name) {
        this.name = name;
    }
}
```

现在，需要在XML配置中设置Person bean的name，如下所示：

```xml
<bean id="person" class="cn.tuyucheng.taketoday.Person">
    <property name="name" value="Joe" />
</bean>
```

请注意，**默认情况下，@Required不适用于基于Java的@Configuration类**。如果你需要确保设置了所有属性，则可以在使用@Bean注解方法创建bean时执行此操作。

### Q16. 描述前端控制器模式

**在前端控制器模式中，所有请求都将首先转到前端控制器而不是Servlet。它将确保响应准备就绪并将它们发送回浏览器，这样我们就有了一个地方可以控制来自外部的一切请求**。

前端控制器将识别应该首先处理请求的Servlet。然后，当它从Servlet取回数据时，它会决定要呈现哪个视图，最后，它会将呈现的视图作为响应发回：

[![前端控制器](https://www.baeldung.com/wp-content/uploads/2018/12/front_end_controller.png)](https://www.baeldung.com/wp-content/uploads/2018/12/front_end_controller.png)

要了解实现细节，请查看我们的[Java前端控制器模式指南](https://www.baeldung.com/java-front-controller-pattern)。

### Q17. 什么是模型1和模型2架构？

在设计Java Web应用程序时，模型1和模型2代表了两种常用的设计模型。

**在模型1中，请求到达Servlet或JSP并在那里得到处理**。Servlet或JSP处理请求、处理业务逻辑、检索和验证数据并生成响应：

[![模型 1-1](https://www.baeldung.com/wp-content/uploads/2018/12/Model_1-1.png)](https://www.baeldung.com/wp-content/uploads/2018/12/Model_1-1.png)

由于这种架构易于实现，因此我们通常在小型和简单的应用程序中使用它。

另一方面，对于大型Web应用程序来说，它并不方便。这些功能通常在业务和表示逻辑耦合的JSP中重复。

**模型2基于模型视图控制器设计模式，它将视图与操作内容的逻辑分开**。

**此外，我们可以区分MVC模式中的三个模块：模型、视图和控制器**。该模型表示应用程序的动态数据结构，它负责数据和业务逻辑操作。视图负责显示数据，而控制器则充当前两者之间的接口。

在模型2中，请求被传递到控制器，控制器处理所需的逻辑以获得应该显示的正确内容。然后控制器将内容放回请求中，通常作为JavaBean或POJO。它还决定哪个视图应该呈现内容并最终将请求传递给它。然后，视图呈现数据：

[![型号 2](https://www.baeldung.com/wp-content/uploads/2018/12/Model_2.png)](https://www.baeldung.com/wp-content/uploads/2018/12/Model_2.png)

## 3. Spring MVC进阶题

### Q18. Spring中的@Controller、@Component、@Repository和@Service注解有什么区别？

根据Spring官方文档，@Component是任何Spring管理的组件的通用构造型。@Repository、@Service和@Controller是针对更具体用例(例如，分别在持久性、服务和表示层)的@Component专用化。

让我们来看看它们的具体用例：

-   **@Controller**：表示该类充当控制器的角色，并检测类中的@RequestMapping注解
-   **@Service**：表示该类持有业务逻辑并调用Repository层中的方法
-   **@Repository**：表示该类定义了一个数据仓库；它的工作是捕获特定于平台的异常并将它们作为Spring统一的非受检异常之一重新抛出

### Q19. 什么是DispatcherServlet和ContextLoaderListener？

简而言之，在前端控制器设计模式中，单个控制器负责将传入的HttpRequest定向到应用程序的所有其他控制器和处理程序。

**Spring的DispatcherServlet实现了这种模式，因此负责将HttpRequest正确协调到正确的处理程序**。

另一方面，ContextLoaderListener启动并关闭Spring的根WebApplicationContext。它将ApplicationContext的生命周期与ServletContext的生命周期联系起来，我们可以使用它来定义跨不同Spring上下文工作的共享bean。

有关DispatcherServlet的更多详细信息，请参阅[本教程](https://www.baeldung.com/spring-dispatcherservlet)。

### Q20. 什么是MultipartResolver以及我们应该何时使用它？

**MultipartResolver接口用于上传文件**。Spring框架提供了一个用于Commons FileUpload的MultipartResolver实现和另一个用于Servlet 3.0多部分请求解析的实现。

使用这些，我们可以在我们的Web应用程序中支持文件上传。

### Q21. 什么是Spring MVC拦截器以及如何使用它？

Spring MVC拦截器允许我们拦截客户端请求并在三个位置处理它-在处理请求之前、处理之后或请求完成(呈现视图时)之后。

拦截器可用于横切关注点并避免重复的处理程序代码，如日志记录、更改Spring模型中全局使用的参数等。

有关详细信息和各种实现，请查看[Spring MVC HandlerInterceptor简介](https://www.baeldung.com/spring-mvc-handlerinterceptor)一文。

### Q22. 什么是InitBinder？

**使用@InitBinder标注的方法用于自定义请求参数、URI模板和支持/命令对象**。我们在控制器中定义它，它有助于控制请求。**在这个方法中，我们注册并配置我们的自定义PropertyEditor、格式化程序和验证器**。

注解具有“value”元素，如果我们不设置它，@InitBinder注解的方法将在每个HTTP请求上被调用。如果我们设置值，这些方法将仅应用于名称对应于“value”元素的特定命令/表单属性和/或请求参数。

**重要的是要记住，其中一个参数必须是WebDataBinder**。其他参数可以是处理程序方法支持的任何类型，命令/表单对象和相应的验证结果对象除外。

### Q23. 解释ControllerAdvice

**@ControllerAdvice注解允许我们编写适用于各种控制器的全局代码**，我们可以将控制器的范围绑定到选定的包或特定的注解。

默认情况下，**@ControllerAdvice适用于用@Controller(或@RestController)标注的类**。如果我们想要更具体，我们也可以使用一些属性。

**如果我们想将适用的类限制在一个包中，我们应该将包的名称添加到注解中**：

```java
@ControllerAdvice("my.package")
@ControllerAdvice(value = "my.package")
@ControllerAdvice(basePackages = "my.package")
```

也可以使用多个包，这样我们需要使用数组而不是String。

**除了通过名称限制包之外，我们还可以通过使用该包中的类或接口之一来实现**：

```java
@ControllerAdvice(basePackageClasses = MyClass.class)
```

'assignableTypes'元素将@ControllerAdvice应用于特定类，而'annotations'则针对特定注解。

**值得注意的是，我们应该将它与@ExceptionHandler一起使用**。这种组合将使我们能够配置一个全局的和更具体的错误处理机制，而不需要每次都为每个控制器类实现它。

### Q24. @ExceptionHandler注解有什么作用？

**@ExceptionHandler注解允许我们定义一个处理异常的方法，我们可以单独使用此注解，但将它与@ControllerAdvice一起使用是一个更好的选择。这样，我们就可以建立一个全局的错误处理机制，不需要在每个Controller里面都编写异常处理的代码**。

让我们看一下我们关于[使用Spring进行REST错误处理](https://www.baeldung.com/exception-handling-for-rest-with-spring)的文章中的示例：

```java
@ControllerAdvice
public class RestResponseEntityExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler(value = { IllegalArgumentException.class, IllegalStateException.class })
    protected ResponseEntity<Object> handleConflict(RuntimeException ex, WebRequest request) {
        String bodyOfResponse = "This should be application specific";
        return handleExceptionInternal(ex, bodyOfResponse, new HttpHeaders(), HttpStatus.CONFLICT, request);
    }
}
```

我们还应该注意，这将为所有抛出IllegalArgumentException或IllegalStateException的控制器提供@ExceptionHandler方法。使用@ExceptionHandler声明的异常应该与用作方法参数的异常相匹配，否则，异常解析机制将在运行时失败。

**这里要记住的一件事是，可以为同一个异常定义多个@ExceptionHandler。我们不能在同一个类中这样做，因为Spring会通过抛出异常并在启动时失败来抱怨**。

另一方面，**如果我们在两个单独的类中定义它们，应用程序会正常启动，但它会使用它找到的第一个处理程序，这可能是错误的处理程序**。

### Q25. Web应用程序中的异常处理

在Spring MVC中，我们有三种异常处理选项：

-   每个异常
-   每个控制器
-   全局范围

如果在Web请求处理过程中抛出非受检异常，服务器将返回HTTP 500响应。为防止这种情况，**我们应该使用@ResponseStatus注解来标注我们的任何自定义异常，这种异常由HandlerExceptionResolver解决**。

当控制器方法抛出我们的异常时，这将导致服务器返回具有指定状态码的适当HTTP响应。我们应该记住，我们不应该为了让这种方法起作用而在其他地方处理我们的异常。

**处理异常的另一种方法是使用@ExceptionHandler注解**。我们将@ExceptionHandler方法添加到任何控制器，并使用它们来处理从该控制器内部抛出的异常。这些方法可以在没有@ResponseStatus注解的情况下处理异常，将用户重定向到专用错误视图，或构建完全自定义的错误响应。

我们还可以传入与Servlet相关的对象(HttpServletRequest、HttpServletResponse、HttpSession和Principal)作为处理程序方法的参数。但是，我们应该记住，我们不能直接将Model对象作为参数。

**处理错误的第三个选项是@ControllerAdvice类**。它将允许我们应用相同的技术，只是这次是在应用程序级别，而不仅仅是特定的控制器。为此，我们需要同时使用@ControllerAdvice和@ExceptionHandler。这样异常处理程序将处理任何控制器抛出的异常。

有关此主题的更多详细信息，请阅读[使用Spring进行REST错误处理](https://www.baeldung.com/exception-handling-for-rest-with-spring)一文。

## 4. 总结

在本文中，我们探讨了在Spring开发人员的技术面试中可能出现的一些与Spring MVC相关的问题。你应该将这些问题作为进一步研究的起点，因为这绝不是一个详尽的清单。