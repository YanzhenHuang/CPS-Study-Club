# 四、原型模式

**原型模式** 想要解决“复制对象实例”相关的问题。

1. 我们需要复制某个实例时，要遍历其所有的成员变量，且无法访问其内部的私有变量。因此，我们没法通过创建一个新对象然后同步参数值的形式来完全复制某个实例；
2. 当我们复制该实例时，我们必须要知道它所属的类——但有时，我们只知道它所实现的接口，并不知道它具体的类（如动态绑定的问题）。

```java
class Game {
    void run() {
        // 生成随机可飞行生物的工厂
        Flyable flyable = getRandomFlyable();

        // 不知道具体实现类，无法复制 
        Flyable anotherFlyable = new ?????();
    }
}
```

我们将 `Flyable` 接口改造成符合原型模式的标准：

```java

interface Prototype extends Clonable {
    Prototype clone();
}

// 每一个 Flyable 实现类都要保证有原型
interface Flyable extends Prototype { 
    void fly();
}

class Bird implements Flyable {
    
    // 基础字段
    private int endurance;
    private int health;
    private String name;
    private boolean isDeep = true;

    // 嵌套对象
    private Wing wing;
    
    public Bird(String name, int endurance, int health, boolean isDeep) {
        this.name = name;
        this.endurance = endurance;
        this.health = health;
        this.wing = new Wing(21);
        this.isDeep = isDeep;
    }

    @Override
    public void fly() { /* ... */ }

    /**
     * 选项1：浅拷贝，对齐字段，新创建访问对象避免共享引用
     */
    @Override
    public Bird shallowCopy() {
        // 浅拷贝基础字段
        Bird clonedBird = Bird(name, endurance, health);
        
        // 深拷贝嵌套对象，避免共享引用！
        cloned.wing = new Wing(this.wing.getSpan());

        return cloned;
    }

    /**
     * 选项2：在字节码层面深拷贝
     */
    @Override
    public Bird deepCopy() {
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(bos);

        oos.writeObject(this);

        ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(bis);

        return (Bird) ois.readObject();
    }

    public Prototype clone() {
        if (isDeep) {
            return deepCopy();
        } else {
            return shallowCopy();
        }
    }
}
```

在使用时，我们就可以直接透过原型接口复制，不需要知道其具体类。

```java
// 使用示例
class Game {
    void run() {
        // 生成随机可飞行生物的工厂
        Flyable flyable = getRandomFlyable();

        // 使用原型模式复制，无需知道具体类
        Flyable anotherFlyable = ((Prototype) flyable).clone();
        anotherFlyable.fly();
    }
}
```