---
title: Java 枚举类型在 switch 语句中的正确用法
date: 2009-01-21 10:41:57
tags: java
---

在 switch 语句中使用枚举类型想象起来好像很简单，但是看到很多人的写法都是有问题的。
所以下面通过一个例子谈一下正确的写法。

<!--more-->

假设现在有如下枚举类的定义：

```java
/**
 * 用户类型枚举类, 用于区分用户的身份.
 */
public enum UserType {

    STUDENT(1), TEACHER(2), PARENT(3), SCHOOL_ADMIN(4), UNKNOWN(-1);

    private int value;

    UserType(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }

    public String getStringValue() {
        return String.valueOf(value);
    }

    public static UserType valueOf(int value) {
        UserType userType = null;
        switch (value) {
        case 1:
            userType = STUDENT;
            break;
        case 2:
            userType = TEACHER;
            break;
        case 3:
            userType = PARENT;
            break;
        case 4:
            userType = SCHOOL_ADMIN;
            break;
        default:
            userType = UNKNOWN;
            break;
        }
        return userType;
    }

    public String getDescription() {
        String desc = null;
        switch (this) {
        case STUDENT:
            desc = "学生";
            break;
        case TEACHER:
            desc = "老师";
            break;
        case PARENT:
            desc = "家长";
            break;
        case SCHOOL_ADMIN:
            desc = "学校管理员";
            break;
        default:
            desc = "未知用户";
            break;
        }
        return desc;
    }
}
```

现在，假设知道某个用户的类型值（`int userType = ...`），需要根据类型来进行某些业务处理。

很多人也许会尝试写下这样的代码：

```java
UserType type = UserType.valueOf(userType);
switch (type) {
case UserType.STUDENT:
    ...
    break;
case UserType.TEACHER:
    ...
    break;
case UserType.PARENT:
    ...
    break;
...
}
```

不幸的是，这样的写法根本连编译都通不过。

那有些人也许会尝试这样写：

```java
switch (userType) {
case UserType.STUDENT.getValue():
    ...
    break;
case UserType.TEACHER.getValue():
    ...
    break;
case UserType.PARENT.getValue():
    ...
    break;
...
}
```

可这样写同样是编译通不过，编译器会提示说 case 后面跟的必须是一个常量。

最后，万般无奈之下也许就会这样写了：

```java
switch (userType) {
case 1:
    ...
    break;
case 2:
    ...
    break;
case 3:
    ...
    break;
...
}
```

这样的写法其实是没有利用到枚举类型。虽然在语法和功能上是都对的，但是因为出现了 magic 数字，所以是很不规范的代码。如果以后用户类型的值需要调整的话，除了修改枚举类里定义的数字，这里的数字也得调整，所以代码维护性很差。

正确的写法应该是这样的：

```java
UserType type = UserType.valueOf(userType);
switch (type) {
case STUDENT:
    ...
    break;
case TEACHER:
    ...
    break;
case PARENT:
    ...
    break;
...
}
```

即在 case 后面可以直接写枚举类型，不用加枚举类的类名。这样的写法看起来也比较简洁。
