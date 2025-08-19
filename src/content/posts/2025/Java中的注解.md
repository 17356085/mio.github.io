---
title: Java中的注解
published: 2025-08-19
updated: 2025-08-19
description: 了解Spring AOP的铺垫知识
author: mio
tags:
  - Java
  - 项目学习
category: 后端
---
# 注解

> Java中的注解的作用，主要有以下几个方面：
> 
> 1. 生成文档，通过代码里标识的元数据生成javadoc文档。
> 2. 编译检查，通过代码里标识的元数据让编译器在编译期间进行检查验证。
> 3. 编译时动态处理，编译时通过代码里标识的元数据动态处理，例如动态生成代码。
> 4. 运行时动态处理，运行时通过代码里标识的元数据动态处理，例如使用反射注入实例。

## 1. 基本语法

**组成元素**：
- `@` 符号：标识这是一个注解
- 注解名：注解的类型名称
- 参数列表：可选，用括号包含的键值对
```java
@注解名(参数列表) public class MyClass { // 类内容 }
```
## 2. Java中的内置注解

> 这些内置注解在实际开发中有什么用？
> 
### 2.1 标准注解

1️⃣ `@Override` ：用于标记重写父类，编译器会检查是否真正重写了父类方法
```java
public class Child extends Parent {
    @Override
    public void method() {
        // 重写父类方法
        System.out.println("Child implementation");
    }
}
```

2️⃣ `@Deprecated`：标记已过时的程序元素，编译器会产生警告。
```java
public class OldClass {
    @Deprecated
    public void oldMethod() {
        System.out.println("This method is deprecated");
    }
    
    public void newMethod() {
        System.out.println("Use this method instead");
    }
}
```

3️⃣`@SuppressWarnings`：抑制编译器警告。
```java
public class WarningExample {
    @SuppressWarnings("unchecked")
    public void method() {
        List list = new ArrayList(); // 原本会有unchecked警告
        list.add("item");
    }
    
    @SuppressWarnings({"deprecation", "unused"})
    public void multipleWarnings() {
        OldClass old = new OldClass();
        old.oldMethod(); // 使用了deprecated方法
        String unused = "never used"; // 未使用的变量
    }
}
```

4️⃣`SafeVarargs`：用于可变参数方法，抑制堆污染警告。
```java
public class VarargsExample {
    @SafeVarargs
    public static <T> void printAll(T... items) {
        for (T item : items) {
            System.out.println(item);
        }
    }
}
```

5️⃣`@FunctionalInterface`：标记函数式接口，确保接口只有一个抽象方法。
```java
@FunctionalInterface
public interface Calculator {
    int calculate(int a, int b);
    
    // 可以有默认方法
    default void printResult(int result) {
        System.out.println("Result: " + result);
    }
    
    // 可以有静态方法
    static void info() {
        System.out.println("This is a calculator interface");
    }
}
```

### 2.2 元注解

元注解是用来注解其他注解的注解。

1️⃣`@Target`

用来指定注解可以用在哪些程序元素上。

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Target;

@Target({ElementType.METHOD, ElementType.FIELD})
public @interface MyAnnotation {
    String value();
}
```
ElementType枚举值：
- `TYPE`：类、接口、枚举
- `FIELD`：字段
- `METHOD`：方法
- `PARAMETER`：方法参数
- `CONSTRUCTOR`：构造器
- `LOCAL_VARIABLE`：局部变量
- `ANNOTATION_TYPE`：注解类型
- `PACKAGE`：包

2️⃣`@Retention`

指定注解的保留策略

```java
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME)
public @interface RuntimeAnnotation {
    String value();
}
```
RetentionPolicy枚举值：
- `SOURCE`：只在源码中保留，编译时丢弃
- `CLASS`：编译到class文件中，运行时不可获取（默认）
- `RUNTIME`：运行时可以通过反射获取

3️⃣ `@Documented`

表示注解应该被javadoc记录

```java
import java.lang.annotation.Documented;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface DocumentedAnnotation {
    String author();
    String version() default "1.0";
}
```

4️⃣ `@Inherited`

表示注解可以被子类继承

```java
import java.lang.annotation.Inherited;

@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface InheritableAnnotation {
    String value();
}

@InheritableAnnotation("parent")
public class Parent {
}

// Child类会自动继承@InheritableAnnotation注解
public class Child extends Parent {
}
```
## 3. 自定义注解

### 3.1 定义

⚠️注意：
- 注解元素只能是基本类型、String、Class、enum、注解类型，以及这些类型的数组
- 不能有参数列表
- 不能有throws子句
- 返回值不能是void
- 可以有默认值，使用default关键字
```java
import java.lang.annotation.*;

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyCustomAnnotation {
    // 注解元素（属性）
    String value();                    // 必需参数
    String name() default "unknown";   // 可选参数，有默认值
    int priority() default 0;
    String[] tags() default {};        // 数组类型
    Class<?> targetClass() default Object.class; // Class类型
}
```

## 4. 注解处理

1️⃣ 运行时注解处理（反射）
```java
import java.lang.reflect.Method;

public class AnnotationProcessor {
    
    public static void processClass(Class<?> clazz) {
        // 检查类上的注解
        if (clazz.isAnnotationPresent(MyCustomAnnotation.class)) {
            MyCustomAnnotation annotation = clazz.getAnnotation(MyCustomAnnotation.class);
            System.out.println("Class annotation:");
            System.out.println("  Value: " + annotation.value());
            System.out.println("  Name: " + annotation.name());
            System.out.println("  Priority: " + annotation.priority());
            System.out.println("  Tags: " + String.join(", ", annotation.tags()));
        }
        
        // 检查方法上的注解
        Method[] methods = clazz.getDeclaredMethods();
        for (Method method : methods) {
            if (method.isAnnotationPresent(MyCustomAnnotation.class)) {
                MyCustomAnnotation annotation = method.getAnnotation(MyCustomAnnotation.class);
                System.out.println("Method " + method.getName() + " annotation:");
                System.out.println("  Value: " + annotation.value());
            }
        }
    }
    
    public static void main(String[] args) {
        processClass(TestClass.class);
    }
}
```

2️⃣编译时注解处理器
```java
import javax.annotation.processing.*;
import javax.lang.model.element.*;
import javax.lang.model.SourceVersion;
import java.util.Set;

@SupportedAnnotationTypes("com.example.MyCustomAnnotation")
@SupportedSourceVersion(SourceVersion.RELEASE_8)
public class MyAnnotationProcessor extends AbstractProcessor {
    
    @Override
    public boolean process(Set<? extends TypeElement> annotations, 
                          RoundEnvironment roundEnv) {
        
        for (Element element : roundEnv.getElementsAnnotatedWith(MyCustomAnnotation.class)) {
            MyCustomAnnotation annotation = element.getAnnotation(MyCustomAnnotation.class);
            
            // 在编译时处理注解
            processingEnv.getMessager().printMessage(
                Diagnostic.Kind.NOTE,
                "Processing element: " + element.getSimpleName() + 
                " with value: " + annotation.value()
            );
        }
        
        return true;
    }
}
```

## 5. 实际应用场景

### 5.1 Spring框架中的注解

1️⃣依赖注入
```java
@Component
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Value("${app.name}")
    private String appName;
    
    public User findById(Long id) {
        return userRepository.findById(id);
    }
}
```

2️⃣Web开发
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = userService.findById(id);
        return ResponseEntity.ok(user);
    }
    
    @PostMapping
    @Transactional
    public ResponseEntity<User> createUser(@RequestBody @Valid User user) {
        User savedUser = userService.save(user);
        return ResponseEntity.ok(savedUser);
    }
}
```

### 5.2 数据库映射（JPA）
```java
@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "username", unique = true, nullable = false)
    private String username;
    
    @Column(name = "email")
    @Email
    private String email;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<Order> orders;
    
    // getters and setters
}
```

### 5.3 数据验证
```java
public class UserDto {
    
    @NotNull(message = "用户名不能为空")
    @Size(min = 3, max = 20, message = "用户名长度必须在3-20之间")
    private String username;
    
    @Email(message = "邮箱格式不正确")
    @NotBlank(message = "邮箱不能为空")
    private String email;
    
    @Min(value = 18, message = "年龄不能小于18")
    @Max(value = 100, message = "年龄不能大于100")
    private Integer age;
    
    // getters and setters
}
```

### 5.4 测试注解
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserServiceTest {
    
    @Autowired
    private UserService userService;
    
    @MockBean
    private UserRepository userRepository;
    
    @Test
    @DisplayName("测试根据ID查找用户")
    public void testFindById() {
        // 准备测试数据
        User mockUser = new User();
        mockUser.setId(1L);
        mockUser.setUsername("testuser");
        
        // 模拟行为
        when(userRepository.findById(1L)).thenReturn(mockUser);
        
        // 执行测试
        User result = userService.findById(1L);
        
        // 验证结果
        assertThat(result).isNotNull();
        assertThat(result.getUsername()).isEqualTo("testuser");
    }
    
    @Test
    @Timeout(value = 2, unit = TimeUnit.SECONDS)
    public void testPerformance() {
        // 性能测试
        userService.performHeavyOperation();
    }
}
```

## 6. 实战示例

1️⃣ 缓存注解

```java
// 定义缓存注解
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Cacheable {
	// 缓存的键
    String key();
    // 过期时间（秒），默认5分钟
    int expireTime() default 300; 
}

// 缓存切面处理器
@Component
@Aspect
public class CacheAspect {
    
    private Map<String, CacheItem> cache = new ConcurrentHashMap<>();
    
    // 拦截所有带@Cacheable注解的方法
    @Around("@annotation(cacheable)")
    public Object handleCache(ProceedingJoinPoint joinPoint, Cacheable cacheable) throws Throwable {
	    // 1. 获取缓存键
        String key = cacheable.key();
        
        // 2. 检查缓存是否存在且未过期
        CacheItem item = cache.get(key);
        if (item != null && !item.isExpired()) {
            return item.getValue();
        }
        
        // 3. 缓存不存在或已过期，执行原方法
        Object result = joinPoint.proceed();
        
        // 4. 将结果存入缓存
        cache.put(key, new CacheItem(result, cacheable.expireTime()));
        
        return result;
    }
    
    private static class CacheItem {
	    // 缓存的值
        private Object value;
        // 过期时间戳
        private long expireTime;
        
        public CacheItem(Object value, int expireSeconds) {
            this.value = value;
            this.expireTime = System.currentTimeMillis() + expireSeconds * 1000L;
        }
        
        // 检查是否过期
        public boolean isExpired() {
            return System.currentTimeMillis() > expireTime;
        }
        
        public Object getValue() {
            return value;
        }
    }
}

// 使用缓存注解
@Service
public class UserService {
    // 缓存10分钟
    @Cacheable(key = "user_#{id}", expireTime = 600)
    public User findById(Long id) {
        // 模拟数据库查询
        return userRepository.findById(id);
    }
}
```

2️⃣ 日志注解
```java
// 定义日志注解
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogExecution {
    String value() default "";
    boolean logParams() default false;
    boolean logResult() default false;
}

// 日志切面处理器
@Component
@Aspect
public class LogAspect {
    
    private static final Logger logger = LoggerFactory.getLogger(LogAspect.class);
    
    @Around("@annotation(logExecution)")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint, LogExecution logExecution) throws Throwable {
        long startTime = System.currentTimeMillis();
        String methodName = joinPoint.getSignature().getName();
        
        // 记录方法开始执行
        if (logExecution.logParams()) {
	        // 获取方法参数
            Object[] args = joinPoint.getArgs();
            logger.info("方法 {} 开始执行，参数: {}", methodName, Arrays.toString(args));
        } else {
            logger.info("方法 {} 开始执行", methodName);
        }
        
        try {
            Object result = joinPoint.proceed();
            
            long endTime = System.currentTimeMillis();
            long executionTime = endTime - startTime;
            
            if (logExecution.logResult()) {
                logger.info("方法 {} 执行完成，耗时: {}ms，结果: {}", methodName, executionTime, result);
            } else {
                logger.info("方法 {} 执行完成，耗时: {}ms", methodName, executionTime);
            }
            
            return result;
        } catch (Exception e) {
            long endTime = System.currentTimeMillis();
            long executionTime = endTime - startTime;
            logger.error("方法 {} 执行异常，耗时: {}ms", methodName, executionTime, e);
            throw e;
        }
    }
}

// 使用日志注解
@Service
public class OrderService {
    
    @LogExecution(value = "创建订单", logParams = true, logResult = true)
    public Order createOrder(OrderDto orderDto) {
        // 业务逻辑
        Order order = new Order();
        order.setOrderNumber(generateOrderNumber());
        order.setAmount(orderDto.getAmount());
        return orderRepository.save(order);
    }
}
```