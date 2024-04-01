---
layout: post
title:  Java 8 Stream：根据另一个List中的值从一个List中查找元素
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个快速教程中，我们将学习如何**使用[Java 8 Stream](https://www.baeldung.com/java-8-streams-introduction)根据另一个列表中的值从一个列表中查找元素**。

## 2. 使用Java 8 Stream

让我们从两个实体类开始-Employee和Department：

```java
class Employee {
    Integer employeeId;
    String employeeName;

    // getters and setters
}

class Department {
    Integer employeeId;
    String department;

    // getters and setters
}
```

**这里的想法是根据Department对象列表过滤Employee对象列表**。更具体地说，我们希望从列表中找到所有Employee：

-   将“sales”作为他们的部门，并且
-   在Department的列表中有相应的employeeId

为了实现这一点，我们实际上会在另一个列表中过滤一个：

```java
@Test
public void givenDepartmentList_thenEmployeeListIsFilteredCorrectly() {
    Integer expectedId = 1002;

    populate(emplList, deptList);

    List<Employee> filteredList = emplList.stream()
        .filter(empl -> deptList.stream()
            .anyMatch(dept -> 
                dept.getDepartment().equals("sales") && 
                empl.getEmployeeId().equals(dept.getEmployeeId())))
            .collect(Collectors.toList());

    assertEquals(1, filteredList.size());
    assertEquals(expectedId, filteredList.get(0).getEmployeeId());
}
```

填充完这两个列表后，我们只需将一个Employee的Stream对象传递给Department的Stream对象。

**接下来，要根据我们的两个条件过滤记录，我们使用anyMatch谓词**，在其中我们组合了所有给定的条件。

最后，我们将结果收集到filteredList中。

## 3. 总结

在本文中，我们学习了如何：

-   使用Collection#stream将一个列表的值流式传输到另一个列表中，并且
-   使用anyMatch()谓词组合多个过滤条件

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-2)上获得。