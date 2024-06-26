---
layout: post
title:  枚举类型的验证
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 简介

在教程[Java Bean Validation基础](https://www.baeldung.com/javax-validation)中，我们了解了如何使用[JSR 380](https://beanvalidation.org/2.0/)将javax验证应用于各种类型。在教程[Spring MVC 自定义验证](https://www.baeldung.com/spring-mvc-custom-validator)中，我们看到了如何创建自定义验证。

在下一个教程中，**我们将重点介绍如何使用自定义注解为枚举构建验证**。

## 2. 验证枚举

**不幸的是，大多数标准注解不能应用于枚举**。

例如，当将@Pattern注解应用于枚举时，我们会收到与HibernateValidator类似的错误：

```shell
javax.validation.UnexpectedTypeException: HV000030: No validator could be found for constraint 
 'javax.validation.constraints.Pattern' validating type 'cn.tuyucheng.taketoday.javaxval.enums.demo.CustomerType'. 
 Check configuration for 'customerTypeMatchesPattern'
```

实际上，唯一可以应用于枚举的标准注解是@NotNull和@Null。

## 3. 验证枚举的模式

**让我们首先定义一个注解来验证枚举的模式**：

```java
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE})
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = EnumNamePatternValidator.class)
public @interface EnumNamePattern {
    String regexp();
    String message() default "must match \"{regexp}\"";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

现在我们可以简单地使用正则表达式将这个新注解添加到我们的CustomerType枚举中：

```java
@EnumNamePattern(regexp = "NEW|DEFAULT")
private CustomerType customerType;
```

**正如我们所见，注解实际上并不包含验证逻辑。因此，我们需要提供一个ConstraintValidator**：

```java
public class EnumNamePatternValidator implements ConstraintValidator<EnumNamePattern, Enum<?>> {
    private Pattern pattern;

    @Override
    public void initialize(EnumNamePattern annotation) {
        try {
            pattern = Pattern.compile(annotation.regexp());
        } catch (PatternSyntaxException e) {
            throw new IllegalArgumentException("Given regex is invalid", e);
        }
    }

    @Override
    public boolean isValid(Enum<?> value, ConstraintValidatorContext context) {
        if (value == null) {
            return true;
        }

        Matcher m = pattern.matcher(value.name());
        return m.matches();
    }
}
```

在此示例中，实现与标准@Pattern验证器非常相似。**但是，这一次，我们匹配枚举的名称**。

## 4. 验证枚举的子集

将枚举与正则表达式匹配不是类型安全的。**相反，与枚举的实际值进行比较更有意义**。

但是，由于注解的限制，这样的注解不能通用。这是因为注解的参数只能是特定枚举的具体值，而不能是枚举父类的实例。

让我们看看如何为我们的CustomerType枚举创建特定的子集验证注解：

```java
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE})
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = CustomerTypeSubSetValidator.class)
public @interface CustomerTypeSubset {
    CustomerType[] anyOf();
    String message() default "must be any of {anyOf}";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

然后可以将此注解应用于CustomerType类型的枚举：

```java
@CustomerTypeSubset(anyOf = {CustomerType.NEW, CustomerType.OLD})
private CustomerType customerType;
```

**接下来，我们需要定义CustomerTypeSubSetValidator来检查给定枚举值列表是否包含当前值**：

```java
public class CustomerTypeSubSetValidator implements ConstraintValidator<CustomerTypeSubset, CustomerType> {
    private CustomerType[] subset;

    @Override
    public void initialize(CustomerTypeSubset constraint) {
        this.subset = constraint.anyOf();
    }

    @Override
    public boolean isValid(CustomerType value, ConstraintValidatorContext context) {
        return value == null || Arrays.asList(subset).contains(value);
    }
}
```

虽然注解必须特定于某个枚举，但我们当然可以在不同的验证器之间[共享代码](https://github.com/eugenp/tutorials/tree/master/javaxval/src/main/java/com/baeldung/javaxval/enums)。

## 5. 验证字符串是否与枚举的值匹配

除了验证枚举以匹配字符串之外，我们还可以做相反的事情。为此，我们可以创建一个注解来检查字符串是否对特定枚举有效。

```java
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE})
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = ValueOfEnumValidator.class)
public @interface ValueOfEnum {
    Class<? extends Enum<?>> enumClass();
    String message() default "must be any of enum {enumClass}";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

可以将此注解添加到String字段，我们可以传递任何枚举类。

```java
@ValueOfEnum(enumClass = CustomerType.class)
private String customerTypeString;
```

**让我们定义ValueOfEnumValidator来检查String(或任何CharSequence)是否包含在枚举中**：

```java
public class ValueOfEnumValidator implements ConstraintValidator<ValueOfEnum, CharSequence> {
    private List<String> acceptedValues;

    @Override
    public void initialize(ValueOfEnum annotation) {
        acceptedValues = Stream.of(annotation.enumClass().getEnumConstants())
                .map(Enum::name)
                .collect(Collectors.toList());
    }

    @Override
    public boolean isValid(CharSequence value, ConstraintValidatorContext context) {
        if (value == null) {
            return true;
        }

        return acceptedValues.contains(value.toString());
    }
}
```

在使用JSON对象时，此验证尤其有用。因为在将不正确的值从JSON对象映射到枚举时会出现以下异常：

```shell
Cannot deserialize value of type CustomerType from String value 'UNDEFINED': value not one
 of declared Enum instance names: [...]
```

我们当然可以处理这个异常。但是，这不允许我们一次报告所有违规行为。

我们可以将它映射到String而不是将值映射到枚举。然后我们将使用我们的验证器来检查它是否匹配任何枚举值。

## 6. 整合一切

现在，我们可以使用我们的任何新验证注解来验证bean。最重要的是，我们所有的验证都接受空值。因此，我们也可以将它与注解@NotNull结合起来：

```java
public class Customer {
    @ValueOfEnum(enumClass = CustomerType.class)
    private String customerTypeString;

    @NotNull
    @CustomerTypeSubset(anyOf = {CustomerType.NEW, CustomerType.OLD})
    private CustomerType customerTypeOfSubset;

    @EnumNamePattern(regexp = "NEW|DEFAULT")
    private CustomerType customerTypeMatchesPattern;

    // constructor, getters etc.
}
```

在下一节中，我们将了解如何测试新注解。

## 7. 测试枚举的Javax验证

为了测试我们的验证器，我们将设置一个验证器，它支持我们新定义的注解。我们将为所有测试使用Customer bean。

首先，我们要确保有效的Customer实例不会导致任何违规行为：

```java
@Test 
public void whenAllAcceptable_thenShouldNotGiveConstraintViolations() { 
    Customer customer = new Customer(); 
    customer.setCustomerTypeOfSubset(CustomerType.NEW); 
    Set violations = validator.validate(customer); 
    assertThat(violations).isEmpty(); 
}
```

其次，我们希望我们的新注解支持和接受空值。我们只希望有一次违规。这应该通过@NotNull注解在customerTypeOfSubset上报告：

```java
@Test
public void whenAllNull_thenOnlyNotNullShouldGiveConstraintViolations() {
    Customer customer = new Customer();
    Set<ConstraintViolation> violations = validator.validate(customer);
    assertThat(violations.size()).isEqualTo(1);

    assertThat(violations)
        .anyMatch(havingPropertyPath("customerTypeOfSubset")
        .and(havingMessage("must not be null")));
}
```

最后，当输入无效时，我们验证验证器以报告违规行为：

```java
@Test
public void whenAllInvalid_thenViolationsShouldBeReported() {
    Customer customer = new Customer();
    customer.setCustomerTypeString("invalid");
    customer.setCustomerTypeOfSubset(CustomerType.DEFAULT);
    customer.setCustomerTypeMatchesPattern(CustomerType.OLD);

    Set<ConstraintViolation> violations = validator.validate(customer);
    assertThat(violations.size()).isEqualTo(3);

    assertThat(violations)
        .anyMatch(havingPropertyPath("customerTypeString")
        .and(havingMessage("must be any of enum class com.baeldung.javaxval.enums.demo.CustomerType")));
    assertThat(violations)
        .anyMatch(havingPropertyPath("customerTypeOfSubset")
        .and(havingMessage("must be any of [NEW, OLD]")));
    assertThat(violations)
        .anyMatch(havingPropertyPath("customerTypeMatchesPattern")
        .and(havingMessage("must match \"NEW|DEFAULT\"")));
}
```

## 8. 总结

在本教程中，**我们介绍了使用自定义注解和验证器验证枚举的三个选项**。

首先，我们学习了如何使用正则表达式验证枚举的名称。

其次，我们讨论了对特定枚举值的子集的验证。我们还解释了为什么我们不能构建通用注解来执行此操作。

最后，我们还研究了如何为字符串构建验证器。为了检查String是否符合给定枚举的特定值。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-validation-2)上获得。