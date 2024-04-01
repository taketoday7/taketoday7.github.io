---
layout: post
title:  在Spring Boot中验证Boolean类型
category: springboot
copyright: springboot
excerpt: Spring Boot Validation
---

## 1. 简介

在本教程中，我们将学习如何在Spring Boot应用程序中验证Boolean类型，并了解执行验证的各种方法。此外，我们将在各个Spring Boot应用程序层(例如控制器或服务层)验证Boolean类型的对象。

## 2. 程序化验证

Boolean类提供了两个基本方法来创建该类的实例：Boolean.valueOf()和Boolean.parseBoolean()。

Boolean.valueOf()接收String和boolean值，它检查输入字段的值是true还是false，并相应地提供一个Boolean对象。Boolean.parseBoolean()方法仅接受字符串值。

**这些方法不区分大小写-例如，“true”、“True”、“TRUE”、“false”、“False”和“FALSE”都是可接受的输入**。

让我们通过单元测试验证String到Boolean的转换：

```java
@Test
void givenInputAsString_whenStringToBoolean_thenValidBooleanConversion() {
    assertEquals(Boolean.TRUE, Boolean.valueOf("TRUE"));
    assertEquals(Boolean.FALSE, Boolean.valueOf("false"));
    assertEquals(Boolean.TRUE, Boolean.parseBoolean("True"));
}
```

我们现在将验证从原始boolean值到布尔Boolean类的转换：

```java
@Test
void givenInputAsboolean_whenbooleanToBoolean_thenValidBooleanConversion() {
    assertEquals(Boolean.TRUE, Boolean.valueOf(true));
    assertEquals(Boolean.FALSE, Boolean.valueOf(false));
}
```

## 3. 使用自定义Jackson反序列化器进行验证

由于Spring Boot API通常处理JSON数据，我们还将了解如何通过数据反序列化来验证JSON到Boolean的转换。我们可以使用自定义反序列化器反序列化布尔值的自定义表示。

让我们考虑一个场景，我们想要使用表示布尔值的JSON数据，其中+(代表true)和–(代表false)。让我们编写一个JSON反序列化器来实现此目的：

```java
public class BooleanDeserializer extends JsonDeserializer<Boolean> {
    @Override
    public Boolean deserialize(JsonParser parser, DeserializationContext context) throws IOException {
        String value = parser.getText();
        if (value != null && value.equals("+")) {
            return Boolean.TRUE;
        } else if (value != null && value.equals("-")) {
            return Boolean.FALSE;
        } else {
            throw new IllegalArgumentException("Only values accepted as Boolean are + and -");
        }
    }
}
```

## 4. 使用注解进行Bean验证

[Bean Validation](https://www.baeldung.com/java-validation)约束是验证字段的另一种常用方法，要使用它，我们需要spring-boot-starter-validation依赖。在所有可用的验证注解中，其中三个可用于Boolean字段：

- **@NotNull**：如果Boolean字段为null，则产生错误 
- **@AssertTrue**：如果Boolean字段设置为false，则会产生错误 
- **@AssertFalse**：如果Boolean字段设置为true，则产生错误 

**需要注意的是，@AssertTrue和@AssertFalse都将null值视为有效输入**。这意味着，如果我们想确保只接受实际的布尔值，我们需要将这两个注解与@NotNull结合使用。

## 5. Boolean验证示例

为了演示这一点，我们将在控制器和服务层使用bean约束和自定义JSON反序列化器。让我们创建一个名为BooleanObject的自定义对象，它具有四个Boolean类型的参数，其中每个都会使用不同的验证方法：

```java
public class BooleanObject {

    @NotNull(message = "boolField cannot be null")
    Boolean boolField;

    @AssertTrue(message = "trueField must have true value")
    Boolean trueField;

    @NotNull(message = "falseField cannot be null")
    @AssertFalse(message = "falseField must have false value")
    Boolean falseField;

    @JsonDeserialize(using = BooleanDeserializer.class)
    Boolean boolStringVar;

    // getters and setters
}
```

## 6. 控制器中的验证

每当我们通过RequestBody将对象传递到REST端点时，我们都可以使用@Valid注解来验证该对象。**当我们将@Valid注解应用于方法参数时，我们指示Spring验证相应的用户对象**：

```java
@RestController
public class ValidationController {

    @Autowired
    ValidationService service;

    @PostMapping("/validateBoolean")
    public ResponseEntity<String> processBooleanObject(@RequestBody @Valid BooleanObject booleanObj) {
        return ResponseEntity.ok("BooleanObject is valid");
    }

    @PostMapping("/validateBooleanAtService")
    public ResponseEntity<String> processBooleanObjectAtService() {
        BooleanObject boolObj = new BooleanObject();
        boolObj.setBoolField(Boolean.TRUE);
        boolObj.setTrueField(Boolean.FALSE);
        service.processBoolean(boolObj);
        return ResponseEntity.ok("BooleanObject is valid");
    }
}
```

验证后，如果发现任何违规，Spring将抛出MethodArgumentNotValidException。为了处理这个问题，可以使用带有相关ExceptionHandler方法的[ControllerAdvice](https://www.baeldung.com/exception-handling-for-rest-with-spring)。让我们创建三个方法来处理控制器和服务层各自抛出的异常：

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(value = HttpStatus.BAD_REQUEST)
    public String handleValidationException(MethodArgumentNotValidException ex) {
        return ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(e -> e.getDefaultMessage())
                .collect(Collectors.joining(","));
    }

    @ExceptionHandler(IllegalArgumentException.class)
    @ResponseStatus(value = HttpStatus.BAD_REQUEST)
    public String handleIllegalArugmentException(IllegalArgumentException ex) {
        return ex.getMessage();
    }

    @ExceptionHandler(ConstraintViolationException.class)
    @ResponseStatus(value = HttpStatus.INTERNAL_SERVER_ERROR)
    public String handleConstraintViolationException(ConstraintViolationException ex) {
        return ex.getMessage();
    }
}
```

在测试REST功能之前，我们建议先[在Spring Boot中测试API](https://www.baeldung.com/spring-boot-testing)。让我们为控制器创建测试类的结构：

```java
@ExtendWith(SpringExtension.class)
@WebMvcTest(controllers = ValidationController.class)
class ValidationControllerUnitTest {

    @Autowired
    private MockMvc mockMvc;

    @TestConfiguration
    static class EmployeeServiceImplTestContextConfiguration {
        @Bean
        public ValidationService validationService() {
            return new ValidationService() {};
        }
    }

    @Autowired
    ValidationService service;
}
```

完成此操作后，我们现在可以测试在类中使用的验证注解。

### 6.1 验证@NotNull注解

让我们看看@NotNull是如何工作的。当我们传递带有null Boolean参数的BooleanObject时，@Valid注解将验证bean并抛出“400 Bad Request” HTTP响应：

```java
@Test
void whenNullInputForBooleanField_thenHttpBadRequestAsHttpResponse() throws Exception {
    String postBody = "{\"boolField\":null,\"trueField\":true,\"falseField\":false,\"boolStringVar\":\"+\"}";

    mockMvc.perform(post("/validateBoolean").contentType("application/json").content(postBody))
        .andExpect(status().isBadRequest());
}
```

### 6.2 验证@AssertTrue注解

接下来，我们将测试@AssertTrue。当我们传递带有false Boolean参数的BooleanObject时，@Valid注解将验证bean并抛出“400 Bad Request” HTTP响应。如果我们捕获响应正文，我们可以获取@AssertTrue注解中设置的错误消息：

```java
 @Test
void whenInvalidInputForTrueBooleanField_thenErrorResponse() throws Exception {
    String postBody = "{\"boolField\":true,\"trueField\":false,\"falseField\":false,\"boolStringVar\":\"+\"}";

    String output = mockMvc.perform(post("/validateBoolean").contentType("application/json").content(postBody))
        .andReturn()
        .getResponse()
        .getContentAsString();

    assertEquals("trueField must have true value", output);
}
```

让我们也检查一下如果我们提供null会发生什么。由于我们仅使用@AssertTrue标注该字段，而没有使用@NotNull注解，因此不会出现验证错误：

```java
@Test
void whenNullInputForTrueBooleanField_thenCorrectResponse() throws Exception {
    String postBody = "{\"boolField\":true,\"trueField\":null,\"falseField\":false,\"boolStringVar\":\"+\"}";

    mockMvc.perform(post("/validateBoolean").contentType("application/json").content(postBody))
        .andExpect(status().isOk());
}
```

### 6.3 验证@AssertFalse注解

我们现在将了解@AssertFalse的工作原理。当我们为@AssertFalse参数传递一个true值时，@Valid注解会抛出一个错误的请求。我们可以在响应正文中获取针对@AssertFalse注解设置的错误消息：

```java
@Test
void whenInvalidInputForFalseBooleanField_thenErrorResponse() throws Exception {
    String postBody = "{\"boolField\":true,\"trueField\":true,\"falseField\":true,\"boolStringVar\":\"+\"}";

    String output = mockMvc.perform(post("/validateBoolean").contentType("application/json").content(postBody))
        .andReturn()
        .getResponse()
        .getContentAsString();

    assertEquals("falseField must have false value", output);
}
```

同样，让我们看看如果我们提供null会发生什么。我们使用@AssertFalse和@NotNull标注该字段，因此，我们将收到验证错误：

```java
@Test
void whenNullInputForFalseBooleanField_thenHttpBadRequestAsHttpResponse() throws Exception {
    String postBody = "{\"boolField\":true,\"trueField\":true,\"falseField\":null,\"boolStringVar\":\"+\"}";
    
    mockMvc.perform(post("/validateBoolean").contentType("application/json").content(postBody))
        .andExpect(status().isBadRequest());
}
```

### 6.4 验证Boolean类型的自定义JSON反序列化器

让我们验证用我们的自定义JSON反序列化器标记的参数。自定义反序列化器仅接受值“+”和“-”，如果我们传递任何其他值，验证将失败并触发错误。让我们在输入JSON中传递“plus”文本值，看看验证是如何工作的：

```java
@Test
void whenInvalidBooleanFromJson_thenErrorResponse() throws Exception {
    String postBody = "{\"boolField\":true,\"trueField\":true,\"falseField\":false,\"boolStringVar\":\"plus\"}";

    String output = mockMvc.perform(post("/validateBoolean").contentType("application/json").content(postBody))
        .andReturn()
        .getResponse()
        .getContentAsString();

    assertEquals("Only values accepted as Boolean are + and -", output);
}
```

最后，让我们测试一下快乐的案例场景。我们将传递“+”号作为自定义反序列化字段的输入。由于它是有效的输入，因此验证将通过并给出成功的响应：

```java
@Test
void whenAllBooleanFieldsValid_thenCorrectResponse() throws Exception {
    String postBody = "{\"boolField\":true,\"trueField\":true,\"falseField\":false,\"boolStringVar\":\"+\"}";

    String output = mockMvc.perform(post("/validateBoolean").contentType("application/json").content(postBody))
        .andReturn()
        .getResponse()
        .getContentAsString();

    assertEquals("BooleanObject is valid", output);
}
```

## 7. 服务层中的验证

现在让我们看看服务层的验证。为了实现这一点，我们使用@Validated注解来标注服务类，并将@Valid注解放置在方法参数上。这两个注解的组合将导致Spring Boot验证对象。

与控制器层的@RequestBody不同，服务层的验证是针对一个简单的Java对象进行的，因此**框架会因验证失败而触发ConstraintViolationException**。在这种情况下，Spring框架将返回HttpStatus.INTERNAL_SERVER_ERROR作为响应状态。

**对于在控制器层创建或修改然后传递到服务层进行处理的对象，首选服务级别验证**。

让我们检查一下这个服务类的框架：

```java
@Service
@Validated
public class ValidationService {

    public void processBoolean(@Valid BooleanObject booleanObj) {
        // further processing
    }
}
```

在上一节中，我们创建了一个端点来测试服务层和异常处理程序方法来处理ConstraintViolationException。让我们编写一个新的测试用例来检查这一点：

```java
@Test
void givenAllBooleanFieldsValid_whenServiceValidationFails_thenErrorResponse() throws Exception {
    mockMvc.perform(post("/validateBooleanAtService").contentType("application/json"))
        .andExpect(status().isInternalServerError());
}
```

## 8. 总结

我们学习了如何使用三种方法在控制器和服务层验证Boolean类型：编程验证、bean验证和使用自定义JSON反序列化器。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-validations)上获得。