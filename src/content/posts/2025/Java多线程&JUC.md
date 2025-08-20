---
title: Java多线程处理
published: 2025-08-20
updated: 2025-08-20
description: Java线程处理
author: mio
tags:
  - 后端开发
  - 项目学习
  - Java
category: 后端
draft: true
---
> 1. 并行：在同一时刻，有多个指令在多个CPU上同时执行。
> 2. 并发：在同一时刻，有多个指令在单个CPU上交替执行。
> 3. 进程：正在运行的程序
> 	1. 独立性：进程是一个能独立运行的基本单位，同时也是系统分配资源和调度的独立单位。
> 	2. 动态性：进程的实质是程序的一次执行过程，进程是动态产生，动态消亡的。
> 	3. 并发性：进程的实质是程序的一次执行过程，进程是动态产生，动态消亡的
> 4. 线程：是进程中的单个顺序控制流，是一条执行路径
> 	1. 单线程：一个进程如果只有一条执行路径。
> 	2. 多线程：是指从软件或者硬件上实现多个线程并发执行的技术。具有多线程能力的计算机因有硬件支持而能够在同一时间执行多个线程，提升性能。

## 1. 实现多线程的方式

1️⃣ 继承Thread类

| 方法           | 说明                       |
| ------------ | ------------------------ |
| void run()   | 在线程开启后，此方法将被调用执行         |
| void start() | 使此线程开始执行，java虚拟机会调用run方法 |
```java
public class MyThread extends Thread {
    @Override
    public void run() {
        for(int i=0; i<100; i++) {
            System.out.println(i);
        }
    }
}
public class MyThreadDemo {
    public static void main(String[] args) {
        MyThread my1 = new MyThread();
        MyThread my2 = new MyThread();

//        my1.run();
//        my2.run();

        //void start() 导致此线程开始执行; Java虚拟机调用此线程的run方法
        my1.start();
        my2.start();
    }
}
```

2️⃣ 实现Runnable接口

| 方法                                   | 说明             |
| ------------------------------------ | -------------- |
| Thread(Runnable target)              | 分配一个新的Thread对象 |
| Thread(Runnable target, String name) | 分配一个新的Thread对象 |
```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        for(int i=0; i<100; i++) {
            System.out.println(Thread.currentThread().getName()+":"+i);
        }
    }
}
public class MyRunnableDemo {
    public static void main(String[] args) {
        //创建MyRunnable类的对象
        MyRunnable my = new MyRunnable();

        //创建Thread类的对象，把MyRunnable对象作为构造方法的参数
        //Thread(Runnable target)
//        Thread t1 = new Thread(my);
//        Thread t2 = new Thread(my);
        //Thread(Runnable target, String name)
        Thread t1 = new Thread(my,"坦克");
        Thread t2 = new Thread(my,"飞机");

        //启动线程
        t1.start();
        t2.start();
    }
}
```

3️⃣ 实现Callable接口

| 方法                               | 说明                                |
| -------------------------------- | --------------------------------- |
| V call                           | 计算结果，如果无法计算结果，则抛出一个异常             |
| FutureTask(Callable<V> callable) | 创建一个FutureTask，一旦运行就执行给定的Callable |
| V get()                          | 如有必要等待计算完成，然后获取其结果                |
```java
public class MyCallable implements Callable<String> {
    @Override
    public String call() throws Exception {
        for (int i = 0; i < 100; i++) {
            System.out.println("跟女孩表白" + i);
        }
        //返回值就表示线程运行完毕之后的结果
        return "答应";
    }
}
public class Demo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //线程开启之后需要执行里面的call方法
        MyCallable mc = new MyCallable();

        //Thread t1 = new Thread(mc);

        //可以获取线程执行完毕之后的结果.也可以作为参数传递给Thread对象
        FutureTask<String> ft = new FutureTask<>(mc);

        //创建线程对象
        Thread t1 = new Thread(ft);

        String s = ft.get();
        //开启线程
        t1.start();

        //String s = ft.get();
        System.out.println(s);
    }
}
```