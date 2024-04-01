---
layout: post
title:  Spring 6 JdbcClient API指南
category: springdata
copyright: springdata
excerpt: Spring 6
---

## 1. 概述

在本教程中，我们将介绍[JdbcClient](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.html)接口，它是Spring Framework 6.1的最新成员。它为[JdbcTemplate和NamedParameterJdbcTemplate](https://www.baeldung.com/spring-jdbc-jdbctemplate)提供了一个具有统一外观的流式接口，这意味着现在它支持链接类型的操作。**我们现在可以以流式的API风格定义查询、设置参数并执行数据库操作**。

此功能简化了JDBC操作，使它们更具可读性且更易于理解。但是，我们必须使用旧的JdbcTemplate和NamedParameterJdbcTemplate类来进行JDBC批处理操作和存储过程调用。

在本文中，我们将使用[H2数据库](https://www.baeldung.com/spring-boot-h2-database)来展示JdbcClient的功能。

## 2. 必要的数据库设置

让我们首先看一下本教程中将使用的student表：

```sql
CREATE TABLE student (
    student_id INT AUTO_INCREMENT PRIMARY KEY,
    student_name VARCHAR(255) NOT NULL,
    age INT,
    grade INT NOT NULL,
    gender VARCHAR(10) NOT NULL,
    state VARCHAR(100) NOT NULL
);
-- Student 1
INSERT INTO student (student_name, age, grade, gender, state) VALUES ('John Smith', 18, 3, 'Male', 'California');

-- Student 2
INSERT INTO student (student_name, age, grade, gender, state) VALUES ('Emily Johnson', 17, 2, 'Female', 'New York');

--More insert statements...
```

上面的SQL脚本创建一个student表并在其中插入记录。

## 3. 创建JdbcClient

Spring Boot框架自动发现application.properties中的数据库连接属性，并在应用程序启动期间创建JdbcClient bean。此后，JdbcClient bean可以在任何类中自动装配。

以下是一个示例，我们在StudentDao类中注入JdbcClient bean：

```java

@Repository
class StudentDao {

    @Autowired
    private JdbcClient jdbcClient;
}
```

我们将在本文中使用StudentDao来定义理解JdbcClient接口的方法。

但是，**接口中也有一些静态方法，例如[create(DataSource dataSource)](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.html#create(javax.sql.DataSource))、[create(JdbcOperations jdbcTemplate)](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.html#create(org.springframework.jdbc.core.JdbcOperations))和[create(NamedParameterJdbcOperations jdbcTemplate)](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.html#create(org.springframework.jdbc.core.namedparam.NamedParameterJdbcOperations))，可以创建JdbcClient实例**。

## 4. 使用JdbcClient执行数据库查询

如前所述，JdbcClient是JdbcTemplate和NamedParameterJdbcTemplate的统一门面。因此，我们将看看它如何支持它们。

### 4.1 隐式支持位置参数

在本节中，我们将讨论对将位置SQL语句参数与占位符?绑定的支持。基本上，我们将了解它如何支持JdbcTemplate的功能。

让我们看一下StudentDao类中的以下方法：

```java
List<Student> getStudentsOfGradeStateAndGenderWithPositionalParams(int grade, String state, String gender) {
    String sql = "select student_id, student_name, age, grade, gender, state from student"
            + " where grade = ? and state = ? and gender = ?";
    return jdbcClient.sql(sql)
        .param(grade)
        .param(state)
        .param(gender)
        .query(new StudentRowMapper()).list();
}
```

在上面的方法中，参数grade、state和gender按照它们分配给方法[param()](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.StatementSpec.html#param(java.lang.Object))的顺序隐式注册。最后，当调用[query()](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.StatementSpec.html#query(org.springframework.jdbc.core.RowMapper))方法时，将执行该语句，并像JdbcTemplate中一样借助[RowMapper](https://www.baeldung.com/spring-jdbc-jdbctemplate#3-mapping-query-results-to-java-object)检索结果。

query()方法还支持[ResultSetExtractor](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.StatementSpec.html#query(org.springframework.jdbc.core.ResultSetExtractor))和[RowCallbackHandler](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.StatementSpec.html#query(org.springframework.jdbc.core.RowCallbackHandler))参数，我们将在接下来的部分中看到相关示例。

有趣的是，在调用方法[list()](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.MappedQuerySpec.html#list())之前，不会检索到任何结果。还支持其他终端操作，**例如[option()](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.MappedQuerySpec.html#optional())、[set()](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.MappedQuerySpec.html#set())、[single()](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.MappedQuerySpec.html#single())和[stream()](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.MappedQuerySpec.html#stream())**。

现在，我们来看看它是如何工作的：

```java
@Test
void givenJdbcClient_whenQueryWithPositionalParams_thenSuccess() {
    List<Student> students = studentDao.getStudentsOfGradeStateAndGenderWithPositionalParams(1, "New York", "Male");
    assertEquals(6, students.size());
}
```

让我们看看如何在[可变参数](https://www.baeldung.com/java-varargs)的帮助下完成相同的操作：

```java
Student getStudentsOfGradeStateAndGenderWithParamsInVarargs(int grade, String state, String gender) {
    String sql = "select student_id, student_name, age, grade, gender, state from student"
        + " where grade = ? and state = ? and gender = ? limit 1";
    return jdbcClient.sql(sql)
        .params(grade, state, gender)
        .query(new StudentRowMapper()).single();
}
```

正如我们在上面看到的，我们用params() 换了方法[param()](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.StatementSpec.html#params(java.lang.Object...))，它接收可变参数参数。此外，我们使用[single()](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.MappedQuerySpec.html#single())方法仅检索一条记录。

让我们看看它是如何工作的：

```java
@Test
void givenJdbcClient_whenQueryWithParamsInVarargs_thenSuccess() {
    Student student = studentDao.getStudentsOfGradeStateAndGenderWithParamsInVarargs(1, "New York", "Male");
    assertNotNull(student);
}
```

此外，**方法[params()](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.StatementSpec.html#params(java.util.List))也有一个重载版本，它接收参数列表**。让我们看一个例子：

```java
Optional<Student> getStudentsOfGradeStateAndGenderWithParamsInList(List params) {
    String sql = "select student_id, student_name, age, grade, gender, state from student"
        + " where grade = ? and state = ? and gender = ? limit 1";
    return jdbcClient.sql(sql)
        .params(params)
        .query(new StudentRowMapper()).optional();
}
```

**除了[params(Listvalues)](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.StatementSpec.html#params(java.util.List))之外，我们还看到了[option()](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.MappedQuerySpec.html#optional())方法，它返回Optional<Student\>对象**。这是上面的方法的实际应用：

```java
@Test
void givenJdbcClient_whenQueryWithParamsInList_thenSuccess() {
    List params = List.of(1, "New York", "Male");
    Optional<Student> optional = studentDao.getStudentsOfGradeStateAndGenderWithParamsInList(params);
    if(optional.isPresent()) {
        assertNotNull(optional.get());            
    } else {
        assertThrows(NoSuchElementException.class, () -> optional.get());
    }
}
```

### 4.2 通过索引显式支持位置参数

**如果我们需要设置SQL语句参数的位置怎么办？为此，我们将使用方法[param(int jdbcIndex, Object value)](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.StatementSpec.html#param(int,java.lang.Object))**：

```java
List<Student> getStudentsOfGradeStateAndGenderWithParamIndex(int grade, String state, String gender) {
    String sql = "select student_id, student_name, age, grade, gender, state from student"
        + " where grade = ? and state = ? and gender = ?";
    return jdbcClient.sql(sql)
        .param(1, grade)
        .param(2, state)
        .param(3, gender)
        .query(new StudentResultExtractor());
}
```

在该方法中，明确指定了参数的位置索引。此外，我们还使用了方法[query(ResultSetExtractor rse)](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.StatementSpec.html#query(org.springframework.jdbc.core.ResultSetExtractor))。

让我们看看实际效果：

```java
@Test
void givenJdbcClient_whenQueryWithParamsIndex_thenSuccess() {
    List<Student> students = studentDao.getStudentsOfGradeStateAndGenderWithParamIndex(1, "New York", "Male");
    assertEquals(6, students.size());
}
```

### 4.3 支持带有名称-值对的命名参数

JdbcClient还支持使用占位符:×绑定命名SQL语句参数，这是NamedParameterJdbcTemplate的一项功能。

**[param()](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.StatementSpec.html#param(java.lang.String,java.lang.Object))方法还可以将参数作为键值对**：

```java
int getCountOfStudentsOfGradeStateAndGenderWithNamedParam(int grade, String state, String gender) {
    String sql = "select student_id, student_name, age, grade, gender, state from student"
        + " where grade = :grade and state = :state and gender = :gender";
    RowCountCallbackHandler countCallbackHandler = new RowCountCallbackHandler();
    jdbcClient.sql(sql)
        .param("grade", grade)
        .param("state", state)
        .param("gender", gender)
        .query(countCallbackHandler);
    return countCallbackHandler.getRowCount();
}
```

在上面的方法中，我们使用了命名参数。此外，我们还使用了[query(RowCallbackHandler rch)](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.StatementSpec.html#query(org.springframework.jdbc.core.RowCallbackHandler))。让我们看看它的实际效果：

```java
@Test
void givenJdbcClient_whenQueryWithNamedParam_thenSuccess() {
    Integer count = studentDao.getCountOfStudentsOfGradeStateAndGenderWithNamedParam(1, "New York", "Male");
    assertEquals(6, count);
}
```

### 4.4 支持使用Map的命名参数

有趣的是，**我们还可以在[params(Map paramMap)](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.StatementSpec.html#params(java.util.Map))方法中在Map中传递参数名称-值对**：

```java
List<Student> getStudentsOfGradeStateAndGenderWithParamMap(Map<String, ?> paramMap) {
    String sql = "select student_id, student_name, age, grade, gender, state from student"
        + " where grade = :grade and state = :state and gender = :gender";
    return jdbcClient.sql(sql)
        .params(paramMap)
        .query(new StudentRowMapper()).list();
}
```

接下来，让我们看看它是如何工作的：

```java
@Test
void givenJdbcClient_whenQueryWithParamMap_thenSuccess() {
    Map<String, ?> paramMap = Map.of(
        "grade", 1,
        "gender", "Male",
        "state", "New York"
    );
    List<Student> students = studentDao.getStudentsOfGradeStateAndGenderWithParamMap(paramMap);
    assertEquals(6, students.size());
}
```

## 5. 使用JdbcClient执行数据库操作

就像查询一样，JdbcClient也支持数据库操作，例如创建、更新和删除记录。与前面的部分类似，我们还可以通过param()和params()方法的各种重载版本来绑定参数。因此，我们不会重复它们。

但是，**为了执行SQL语句而不是调用query()方法，我们将调用[update()](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.StatementSpec.html#update())方法**。

下面是向学生表中插入记录的示例：

```java
Integer insertWithSetParamWithNamedParamAndSqlType(Student student) {
    String sql = "INSERT INTO student (student_name, age, grade, gender, state)"
        + "VALUES (:name, :age, :grade, :gender, :state)";
    Integer noOfrowsAffected = this.jdbcClient.sql(sql)
        .param("name", student.getStudentName(), Types.VARCHAR)
        .param("age", student.getAge(), Types.INTEGER)
        .param("grade", student.getGrade(), Types.INTEGER)
        .param("gender", student.getStudentGender(), Types.VARCHAR)
        .param("state", student.getState(), Types.VARCHAR)
        .update();
    return noOfrowsAffected;
}
```

上面的方法使用[param(String name, Object value, int sqlType)](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.StatementSpec.html#param(java.lang.String,java.lang.Object,int))来绑定参数，它有一个额外的sqlType参数来指定参数的数据类型。此外，update()方法还返回受影响的行数。

让我们看看该方法的实际效果：

```java
@Test
void givenJdbcClient_whenInsertWithNamedParamAndSqlType_thenSuccess() {
    Student student = getSampleStudent("Johny Dep", 8, 4, "Male", "New York");
    assertEquals(1, studentDao.insertWithSetParamWithNamedParamAndSqlType(student));
}
```

在上面的方法中，getSampleStudent()返回一个Student对象。然后将Student对象传递给方法insertWithSetParamWithNamedParamAndSqlType()以在student表中创建新记录。

**与JdbcTemplate类似，JdbcClient具有[update(KeyHolder generatedKeyHolder)](https://docs.spring.io/spring-framework/docs/6.1.0-SNAPSHOT/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.StatementSpec.html#update(org.springframework.jdbc.support.KeyHolder))方法来检索执行插入语句时创建的自动生成的键**。

## 6. 总结

在本文中，我们了解了Spring Framework 6.1中引入的新接口JdbcClient。我们看到了这一接口如何执行之前由JdbcTemplate和NamedParameterJdbcTemplate执行的所有操作。此外，由于流式的API风格，代码也变得更易于阅读和理解。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-data-modules/spring-boot-jdbc-2)上获得。