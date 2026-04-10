# 五、单例模式

**目录**

5.1 [饿汉式单例](#51-饿汉式单例)
&nbsp; 5.1.1 [直接赋值写法](#511-直接赋值写法)
&nbsp; 5.1.2 [静态块写法](#512-静态块写法)
5.2 [懒汉式单例](#52-懒汉式单例)
&nbsp; 5.2.1 [线程不安全写法](#521-线程不安全写法)
&nbsp; 5.2.2 [单检查写法](#522-单检查写法)
&nbsp; 5.2.3 [双检查写法](#523-双检查写法)
&nbsp; 5.2.4 [静态内部类写法](#524-静态内部类写法)
5.3 [注册式单例](#53-注册式单例)
&nbsp; 5.3.1 [枚举式单例](#531-枚举式单例)
&nbsp; 5.3.2 [容器式单例](#532-容器式单例)
5.4 [`ThreadLocal` 单例](#54-threadlocal-单例)


> _定义：_ 单例模式 (Singleton Pattern) 是一种 保证一个类在任何情况下只有 **一个** 实例，并提供一个访问该实例的 **全局节点** 的设计模式。

单例模式下，类的构造器被私有化，开放一个静态的 `getInstance()` 函数作为获取该实例的全局节点。

单例模式的适用场景有很多，如服务器配置获取、游戏场景管理等。本文以游戏场景管理器为例。

## 5.1 饿汉式单例

在单例类 **首次加载** 时就创建实例。

### 5.1.1 直接赋值写法

```java
public class SceneManager {
    // 类加载阶段创建全局实例
    private static final SceneManager instance = new SceneManager();

    // 私有构造函数，防止外部实例化
    private SceneManager() {}
    public static SceneManager getInstance() {
        return this.instance;
    }
}
```

### 5.1.2 静态块写法

_注：先静态后动态；先上后下；先属性后方法。_

```java
public class SceneManager {
    // 类加载阶段创建全局实例
    private static final SceneManager instance;
    static {
        instance = new SceneManager();
    }

    // 私有构造函数，防止外部实例化
    private SceneManager() {}
    public static SceneManager getInstance() {
        return this.instance;
    }
}
```

- 优点：无锁，执行效率高；线程安全；
- 缺点：从类加载阶段到初次使用时间段的内存浪费（尤其在单例类极多的情况下）。


## 5.2 懒汉式单例

在单例类 **首次被外部访问时** 才创建实例。

### 5.2.1 线程不安全写法

```java
public class SceneManager {
    private static final SceneManager instance;

    private SceneManager() {}

    public static SceneManager getInstance() {
        // 第一次访问时，全局实例为null，遂创建全局实例；
        // 否则直接返回已有的实例。
        if (this.instance == null)
            this.instance = new SceneManager();

        return this.instance;
    }
}
```

- 优点：低内存浪费，不需要时不创建实例；
- 缺点：线程不安全。

懒汉式单例线程不安全：若在 `new SceneManager()` 处发生竞态条件，`instance` 则有可能被覆盖，造成不同线程访问到不同实例的现象，从而违反单例模式原则。

```log
0: Thread 1 [RUNNING] find   instance is null.
1: context switch.
2: Thread 2 [RUNNING] find   instance is null.
3: Thread 2 [RUNNING] create instance 0x2f3a79.
4: Thread 2 [RUNNING] get    instance 0x2f3a79.
5: context switch.
6: Thread 1 [RUNNING] create instance 0x2d3018.
7: Thread 1 [RUNNING] get    instance 0x2d3018.
```

当然，`Thread1` 创建的实例覆盖了 `Thread2` 创建的实例，但是二者获取到的实例 **不一致** ！

### 5.2.2 单检查写法

我们将`getInstance()` 方法中 创建实例的代码块 加锁，保证其原子性。

```java
public class SceneManager {
    private static final SceneManager instance;

    private SceneManager() {}

    // 加锁
    public static SceneManager getInstance() {
        synchronized (SceneManager.class) {
            if (this.instance == null)
                this.instance = new SceneManager();
        }

        return this.instance;
    }
}
```

在创建实例时，阻塞另一个进入访问的线程，杜绝了竞态条件发生的可能性。

```log
0: Thread 1 [RUNNING] find   instance is null.
1: context switch.
2: Thread 2 [MONITOR] waits.
3: context switch.
4: Thread 1 [RUNNING] create instance 0x2d3018.
5: Thread 1 [RUNNING] get    instance 0x2d3018.
6: context switch.
7: Thread 2 [RUNNING] find   instance is not null.
8: Thread 2 [RUNNING] get    instance 0x2d3018.
```

问题：这是一个频繁调用的方法，这样做会导致每次访问时发生线程阻塞，引起严重的性能下降。

### 5.2.3 双检查写法

第一次检查的是是否要阻塞。当且仅当 `instance` 为 `null` 时，才进入阻塞。

第二次检查的是是否要重新创建实例。

```java
public class SceneManager {
    private volatile static final SceneManager instance;

    private SceneManager() {}

    public static SceneManager getInstance() {
        // 当且仅当 instance 为 null 时，才进入阻塞
        if (this.instance == null) 
        {
            synchronized (SceneManager.class) {
                if (this.instance == null)
                    this.instance = new SceneManager();
            }
        }

        return this.instance;
    }
}
```

- 优点：低内存浪费，性能高，线程安全；
- 缺点：双重 `if` 检查不够优雅，可读性差。

例子如下。第8个时间片时，`Thread 1` 二次检查发现 `instance` 已经和初次检查时不一样，被创建了。因此选择不再创建新的实例，而获取现有的实例。

```log
0: Thread 1 [RUNNING] find   instance is null.
1: context switch.
2: Thread 2 [RUNNING] find   instance is null.
3: Thread 2 [RUNNING] create instance 0x2f3a79.
4: context switch.
5: Thread 1 [MONITOR] waits.
6: Thread 2 [RUNNING] get    instance 0x2f3a79.
7: context switch.
8: Thread 1 [RUNNING] find   instance is not null.
9: Thread 1 [RUNNING] get    instance 0x2f3a79.
```

`volatile` 解决指令重排问题。

### 5.2.4 静态内部类写法

利用Java语法特点“投机取巧”。类加载阶段，加载 `SceneManager` (`SceneManager.class`) 时，并不会加载内部类 `SceneManagerHolder`。当且仅当第一次访问 `getInstance()` 时，才会加载内部类 `SceneManagerHolder` (`SceneManager$SceneManagerHolder.class`)，并创建静态成员变量 `INSTANCE`。

```java
public class SceneManager {
    // 静态内部类
    private static class SceneManagerHolder {
        private static final SceneManager INSTANCE = new SceneManager();
    }

    private SceneManager() {}

    public static SceneManager getInstance() {
        // 读时加载
        return SceneManagerHolder.INSTANCE;
    }
}
```

- 优点：低内存浪费，线程安全，性能高，写法优雅；
- 缺点：**能够被反射破坏。** （前面的均可）

**反射破坏**

```java
public class ReflectDestroySingleton {
    public static void main(String[] args) throws Exception {
        Class<?> clazz = SceneManager.class;
        // 强制获取单例的构造器
        Constructor c = clazz.getDeclaredConstructor(null);

        // 强制访问，创建了新的实例！
        c.setAccessible(true);
        SceneManager instance1 = (SceneManager) c.newInstance();
        SceneManager instance2 = (SceneManager) c.newInstance();
        SceneManager instance3 = (SceneManager) c.newInstance();
    }
}
```

解决方案：在构造器中添加判断，若实例已经存在则抛出异常。

```java
public class SceneManager {
    private static class SceneManagerHolder {
        private static final SceneManager INSTANCE = new SceneManager();
    }

    private SceneManager() {
        if (SceneManagerHolder.INSTANCE != null)
            throw new RuntimeException("单例构造器禁止反射调用！！！");
    }

    public static SceneManager getInstance() {
        return SceneManagerHolder.INSTANCE;
    }
}
```

- 优点：低内存浪费，线程安全，性能高，不能被反射破坏；
- 缺点：构造器内抛异常，不够优雅。

## 5.3 注册式单例

将每一个实例都缓存到统一的容器中，使用唯一标识获取实例。

### 5.3.1 枚举式单例

通过将单例类继承自 `Enum` 类实现单例。

```java
public enum SceneManager {  // 继承 Enum 类
    // 实例本身
    INSTANCE;

    // 私有字段
    private Object data;
    public Object getData() { 
        return data; 
    }

    // 全局访问节点
    public static SceneManager getInstance() {
        return INSTANCE;
    }
}
```

_使用方法：_

```java
public class EnumSingletonTest {
    public static void main(String[] args) {
        EnumSingleton instance1 = EnumSingleton.getInstance();
        instance1.setData(new Object());
    }
}
```

- 优点：高性能，线程安全，防止反射破坏，写法优雅；
- 缺点：内存浪费。

**线程安全**

声明枚举时，声明的 `INSTANCE` 作为常量存入 `Map` 中，和饿汉式一样。虽然线程安全且高性能，但是仍然会有内存浪费。

**防止反射破坏**

枚举类在 JDK 底层就被禁止通过反射创建！

_例：仍旧尝试通过反射来强行创建_

```java
public class EnumSingletonTest { 
    public static void main(String[] args) throws Exception { 
        Class<?> clazz = SceneManager.class;

        // Enum 没有无参构造器
        Constructor c = clazz.getDeclaredConstructor(String.class, int.class);
        c.setAccessible(true);

        // 尝试创建枚举对象
        SceneManager instance1 = (EnumSingleton) c.newInstance();
    }
}
```

_报错：不能用反射创建枚举对象。_

```log
java.lang.IllegalArgumentException: Cannot reflectively create enum objects
```

这个行为是定义在 `Constructor` 类中的，而非枚举类构造器中定义的。这是JDK的底层要求。

### 5.3.2 容器式单例

_定义方法：_

```java
public class Container {
    private Container() {}

    private static Map<String, Object> ioc = new HashMap<>();

    private static Object getInstance(String className) {

        if (!ioc.containsKey(className)) {
            Object instance = Class.forName(className).newInstance();
            ioc.put(className, instance);
        }

        return ioc.get(className);
    }
}
```

_使用方法：_

```java
public class ContainerSingletonTest { 
    public static void main(String[] args) { 
        Object instance1 = Container.getInstance("com.example.SceneManager");
    }
}
```

**序列化/反序列化破坏单例**

在单例中添加 `readResolve()` 方法，返回单例对象。在 `ObjectInputStream` 中，会调用 `readResolve()` 方法，返回单例对象，否则会重新创建对象而破坏单例。

```java
public class SceneManager implements Serializable { 
    private static INSTANCE = new SceneManager();
    private SceneManager() {}
    public static SceneManager getInstance() { 
        return INSTANCE;
    }

    // 防止序列化、反序列化破坏单例
    private Object readResolve() { 
        return INSTANCE;
    }
}
```

## 5.4 `ThreadLocal` 单例

保证线程内部全局唯一，且天生线程安全。

```java
public class SceneManager {
    private static final ThreadLocal<SceneManager> INSTANCE = 
        new ThreadLocal<SceneManager>() {   // 匿名实现
            @Override
            protected SceneManager initialValue() { 
                return new SceneManager();
            }

            private SceneManager() {}

            public static SceneManager getInstance() { 
                return INSTANCE.get();
            }
        };
}
```

特殊性：不同线程内部的单例是不同的。

```java

// 定义每个线程内部的操作为访问该单例。
public class AccessSceneManager implements Runnable {
    public void run() { 
        SceneManager instance = SceneManager.getInstance();

        System.out.printf("%s: %s",
            Thread.currentThread().getName(),
            instance
        );
    }
}

public class ThreadLocalSingletonTest { 
    public static void main(String[] args) { 
        for (int i = 0; i < 10; i++) {
            new Thread(new AccessSceneManager()).start();
        }
        // 每个线程内的SceneManager单例不是同一个
    }
}
```