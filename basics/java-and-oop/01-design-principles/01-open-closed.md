# 一、开闭原则

> _定义：_ 开闭原则 (Open-Closed Principle) 要求一个软件实体（类、方法、接口等）应当对拓展 **开放** ，对修改 **关闭** 。
> _延申：_ 用抽象构建框架，用实现完善细节。

## 1.1 场景

假设我们有一个 `Animal` 接口，里面只有一个 `say()` 方法。每个动物都需要实现这个接口，并具体化 `say()` 方法。

_`Animal` 接口_

```java
interface Animal {
    void say();
}
```

_具体动物类_

```java
class Dog implements Animal {
    public void say() { System.out.println("roof"); }
}

class Cat implements Animal {
    public void say() {System.out.println("meow");}
}
```

使用时，我们将 “哪个动物” 传入一个 `Voice` 类中的 `make()` 方法中来实现让动物叫的效果。

```java
class Voice {
    // 具体实现请看 1.2 和 1.3
}

public class Client {
    public static void main(String[] args) {
        // 具体实现请看 1.2 和 1.3 
    }
}
```

## 1.2 违背开闭原则

```java
class Voice {
    public void make(String type) {
        switch(type) {
            case "Dog":
                new Dog().say();
                break;
            case "Cat":
                new Cat().say();
                break;
            default:
                throw new Exception("Unknown Animal.");
        }
    }
}

public class Client {
    public static void main(String[] args) {
        Voice v = new Voice();
        v.make("Dog");
        v.make("Cat");
        // v.make("Pig");  // 添加 Pig 类需要修改 Voice 类
    }
}
```

`Voice` 类此时 **依赖具体实现** ，违背开闭原则：后续添加动物时，必须修改 `Voice` 类，不满足 “对修改关闭”。

## 1.3 符合开闭原则

```java
class Voice {
    public void make(Animal animal) {
        animal.say();
    }
}

public class Client {
    public static void main(String[] args) {
        Voice v = new Voice();
        v.make(new Dog());
        v.make(new Cat());
        v.make(new Pig());  // 添加Pig类无需修改 Voice 类
    }
}
```

`Voice` 类此时接收 `Animal` 接口，**依赖抽象**，满足“对修改关闭”：新增动物时，只需在客户端内向 `Voice` 实例中传入新扩展的动物类，而无需修改 `Voice` 类。