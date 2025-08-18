---
title: Java异常处理&线程
published: 2025-08-17
updated: 2025-08-17
description: Java异常处理
author: mio
tags:
  - 未归档
category: 未归档
draft: true
---
# Java异常与线程知识点复盘

## 一、Java异常处理

### 1.1 异常体系结构

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

### 1.2 异常分类

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

### 1.3 异常处理语法

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

### 1.4 自定义异常

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

### 1.5 异常处理最佳实践

1. **不要忽略异常**

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

2. **具体化异常处理**

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

3. **在合适的层级处理异常**

- DAO层：技术异常转换为业务异常
- Service层：业务异常处理
- Controller层：异常统一处理和响应格式化

## 二、Java线程

### 2.1 线程基础概念

**进程 vs 线程**

- 进程：系统分配资源的基本单位
- 线程：CPU调度的基本单位，同一进程内的线程共享内存空间

**线程状态**

```
NEW (新建) → RUNNABLE (就绪/运行) → BLOCKED (阻塞)
                ↓                      ↑
            TERMINATED (终止) ← WAITING/TIMED_WAITING (等待)
```

### 2.2 创建线程的方式

#### 方式1：继承Thread类

```java
public class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("线程执行：" + Thread.currentThread().getName());
    }
}

// 使用
MyThread thread = new MyThread();
thread.start(); // 注意：调用start()而不是run()
```

#### 方式2：实现Runnable接口

```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("线程执行：" + Thread.currentThread().getName());
    }
}

// 使用
Thread thread = new Thread(new MyRunnable());
thread.start();

// 或使用Lambda表达式
Thread thread2 = new Thread(() -> {
    System.out.println("Lambda线程执行");
});
thread2.start();
```

#### 方式3：实现Callable接口

```java
public class MyCallable implements Callable<String> {
    @Override
    public String call() throws Exception {
        return "执行结果：" + Thread.currentThread().getName();
    }
}

// 使用
FutureTask<String> futureTask = new FutureTask<>(new MyCallable());
Thread thread = new Thread(futureTask);
thread.start();

// 获取结果
try {
    String result = futureTask.get(); // 阻塞等待结果
    System.out.println(result);
} catch (InterruptedException | ExecutionException e) {
    e.printStackTrace();
}
```

#### 方式4：线程池

```java
// 创建线程池
ExecutorService executor = Executors.newFixedThreadPool(5);

// 提交任务
executor.submit(() -> {
    System.out.println("线程池执行任务");
});

// 关闭线程池
executor.shutdown();
```

### 2.3 线程同步

#### synchronized关键字

```java
public class Counter {
    private int count = 0;
    
    // 同步方法
    public synchronized void increment() {
        count++;
    }
    
    // 同步代码块
    public void decrement() {
        synchronized (this) {
            count--;
        }
    }
    
    // 静态同步方法
    public static synchronized void staticMethod() {
        // 锁的是类对象
    }
}
```

#### Lock接口

```java
public class CounterWithLock {
    private int count = 0;
    private final ReentrantLock lock = new ReentrantLock();
    
    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock(); // 确保释放锁
        }
    }
    
    public boolean tryIncrement() {
        if (lock.tryLock()) {
            try {
                count++;
                return true;
            } finally {
                lock.unlock();
            }
        }
        return false;
    }
}
```

#### volatile关键字

```java
public class VolatileExample {
    private volatile boolean flag = false;
    
    public void setFlag() {
        flag = true; // 写操作立即对其他线程可见
    }
    
    public void checkFlag() {
        while (!flag) {
            // 等待flag变为true
        }
        System.out.println("Flag已设置");
    }
}
```

### 2.4 线程通信

#### wait/notify机制

```java
public class ProducerConsumer {
    private final Object lock = new Object();
    private int data = 0;
    private boolean hasData = false;
    
    // 生产者
    public void produce(int value) throws InterruptedException {
        synchronized (lock) {
            while (hasData) {
                lock.wait(); // 等待消费者消费
            }
            data = value;
            hasData = true;
            System.out.println("生产：" + value);
            lock.notify(); // 通知消费者
        }
    }
    
    // 消费者
    public int consume() throws InterruptedException {
        synchronized (lock) {
            while (!hasData) {
                lock.wait(); // 等待生产者生产
            }
            int result = data;
            hasData = false;
            System.out.println("消费：" + result);
            lock.notify(); // 通知生产者
            return result;
        }
    }
}
```

#### BlockingQueue

```java
public class BlockingQueueExample {
    private final BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(10);
    
    // 生产者
    public void producer() {
        try {
            for (int i = 0; i < 10; i++) {
                queue.put(i); // 队列满时阻塞
                System.out.println("生产：" + i);
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    // 消费者
    public void consumer() {
        try {
            while (true) {
                Integer item = queue.take(); // 队列空时阻塞
                System.out.println("消费：" + item);
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

### 2.5 线程池详解

#### 核心参数

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    5,                      // corePoolSize：核心线程数
    10,                     // maximumPoolSize：最大线程数
    60L,                    // keepAliveTime：空闲线程存活时间
    TimeUnit.SECONDS,       // timeUnit：时间单位
    new LinkedBlockingQueue<>(100),  // workQueue：工作队列
    Executors.defaultThreadFactory(), // threadFactory：线程工厂
    new ThreadPoolExecutor.AbortPolicy() // handler：拒绝策略
);
```

#### 常用线程池类型

```java
// 固定大小线程池
ExecutorService fixed = Executors.newFixedThreadPool(5);

// 缓存线程池
ExecutorService cached = Executors.newCachedThreadPool();

// 单线程池
ExecutorService single = Executors.newSingleThreadExecutor();

// 定时任务线程池
ScheduledExecutorService scheduled = Executors.newScheduledThreadPool(2);
```

### 2.6 线程安全的集合类

```java
// 并发集合
ConcurrentHashMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();
CopyOnWriteArrayList<String> copyOnWriteList = new CopyOnWriteArrayList<>();
ConcurrentLinkedQueue<String> concurrentQueue = new ConcurrentLinkedQueue<>();

// 同步包装器
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
Map<String, Integer> syncMap = Collections.synchronizedMap(new HashMap<>());
```

### 2.7 ThreadLocal

```java
public class ThreadLocalExample {
    private static final ThreadLocal<String> threadLocal = new ThreadLocal<String>() {
        @Override
        protected String initialValue() {
            return "默认值";
        }
    };
    
    public static void set(String value) {
        threadLocal.set(value);
    }
    
    public static String get() {
        return threadLocal.get();
    }
    
    public static void remove() {
        threadLocal.remove(); // 防止内存泄漏
    }
}
```

## 三、异常与线程的结合使用

### 3.1 线程中的异常处理

```java
public class ThreadExceptionHandler implements Thread.UncaughtExceptionHandler {
    @Override
    public void uncaughtException(Thread t, Throwable e) {
        System.err.println("线程 " + t.getName() + " 发生未捕获异常：");
        e.printStackTrace();
    }
}

// 使用
Thread thread = new Thread(() -> {
    throw new RuntimeException("线程异常");
});
thread.setUncaughtExceptionHandler(new ThreadExceptionHandler());
thread.start();
```

### 3.2 线程池中的异常处理

```java
ExecutorService executor = Executors.newFixedThreadPool(2);

// submit方式：异常被包装在Future中
Future<?> future = executor.submit(() -> {
    throw new RuntimeException("任务异常");
});

try {
    future.get(); // 这里会抛出ExecutionException
} catch (ExecutionException e) {
    Throwable cause = e.getCause(); // 获取原始异常
    System.out.println("任务执行异常：" + cause.getMessage());
}

// execute方式：异常直接抛出给UncaughtExceptionHandler
executor.execute(() -> {
    throw new RuntimeException("任务异常");
});
```

## 四、实践建议

### 4.1 异常处理建议

1. 尽早发现异常，就近处理
2. 不要忽略异常，至少要记录日志
3. 自定义异常要有意义，包含足够的上下文信息
4. 在finally或try-with-resources中释放资源
5. 避免在finally块中抛出异常

### 4.2 多线程编程建议

1. 优先使用线程池而不是直接创建线程
2. 正确使用同步机制，避免死锁
3. 合理设置线程池参数
4. 及时清理ThreadLocal，防止内存泄漏
5. 使用concurrent包下的线程安全集合
6. 注意线程间的协作和通信
