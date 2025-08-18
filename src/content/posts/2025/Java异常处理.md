---
title: Java异常处理
published: 2025-08-17
updated: 2025-08-17
description: Java异常处理
author: mio
tags:
  - 项目学习
  - Java
category: 后端
draft: false
---
## Java异常处理

### 1. 异常体系结构

```
Throwable
├── Error (错误)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── NoClassDefFoundError
└── Exception (异常)
    ├── RuntimeException (运行时异常/非检查异常)
    │   ├── NullPointerException
    │   ├── ArrayIndexOutOfBoundsException
    │   ├── IllegalArgumentException
    │   └── ClassCastException
    └── 检查异常 (Checked Exception)
        ├── IOException
        ├── SQLException
        ├── ClassNotFoundException
        └── InterruptedException
```

### 2 异常分类

**检查异常 (Checked Exception)**

- 编译时必须处理的异常
- 必须使用try-catch或throws声明
- 常见：IOException、SQLException、ClassNotFoundException

**非检查异常 (Unchecked Exception)**

- 运行时异常，编译时不强制处理
- 包括RuntimeException及其子类
- 常见：NullPointerException、ArrayIndexOutOfBoundsException

**错误 (Error)**

- JVM系统级错误，程序无法处理
- 常见：OutOfMemoryError、StackOverflowError

### 3 异常处理语法

#### try-catch-finally语句

```java
try {
    // 可能出现异常的代码
    int result = 10 / 0;
} catch (ArithmeticException e) {
    // 处理特定异常
    System.out.println("除零异常：" + e.getMessage());
} catch (Exception e) {
    // 处理其他异常
    System.out.println("其他异常：" + e.getMessage());
} finally {
    // 无论是否有异常都会执行
    System.out.println("清理资源");
}
```

#### try-with-resources (Java 7+)

```java
// 自动关闭资源
try (FileInputStream fis = new FileInputStream("file.txt");
     BufferedReader br = new BufferedReader(new InputStreamReader(fis))) {
    return br.readLine();
} catch (IOException e) {
    e.printStackTrace();
}
```

#### throws声明异常

```java
public void readFile(String fileName) throws IOException {
    FileInputStream fis = new FileInputStream(fileName);
    // 不在此方法中处理异常，向上抛出
}
```

#### throw抛出异常

```java
public void setAge(int age) {
    if (age < 0) {
        throw new IllegalArgumentException("年龄不能为负数");
    }
    this.age = age;
}
```

### 4 自定义异常

```java
// 自定义检查异常
public class CustomCheckedException extends Exception {
    public CustomCheckedException(String message) {
        super(message);
    }
}

// 自定义运行时异常
public class CustomRuntimeException extends RuntimeException {
    public CustomRuntimeException(String message) {
        super(message);
    }
}
```

### 5 异常处理最佳实践

5.1 **不要忽略异常**

```java
// 错误做法
try {
    // some code
} catch (Exception e) {
    // 什么都不做
}

// 正确做法
try {
    // some code
} catch (Exception e) {
    logger.error("处理失败", e);
    // 或者重新抛出
    throw new ServiceException("业务处理失败", e);
}
```

5.2 **具体化异常处理**

```java
// 好的做法：从具体到抽象
try {
    // some code
} catch (FileNotFoundException e) {
    // 处理文件未找到
} catch (IOException e) {
    // 处理其他IO异常
} catch (Exception e) {
    // 处理其他异常
}
```

5.3 **在合适的层级处理异常**

- DAO层：技术异常转换为业务异常
- Service层：业务异常处理
- Controller层：异常统一处理和响应格式化

### 6 异常处理建议

1. 尽早发现异常，就近处理
2. 不要忽略异常，至少要记录日志
3. 自定义异常要有意义，包含足够的上下文信息
4. 在finally或try-with-resources中释放资源
5. 避免在finally块中抛出异常

