# 二、单一职责原则

> _定义：_ 单一职责原则 (Simple Responsibility Principle) 要求不要存在多于一个导致类发生变更的原因，一个软件实体（类、接口、方法）只负责一项职责。

## 2.1 不符合单一职责原则的情况

从接口设计的角度来看，单一职责的要求就是尽量细化接口。这需要谨慎对待“只要A就一定B”的想法。比如：

```java
interface Bird {
    double flyVelocity;
    double runVelocity;
    void fly();
    void run();
    void noise();
}

interface Dog {
    double runVelocity;
    void run();
    void noise();
}

interface Fish {
    double swimVelocity;
    void swim();
}
```

这个例子中，我们默认了“只要是鸟就一定会飞”、 “只有鱼才会游泳”。然而并非所有鸟都会飞——比如鸵鸟；且也有鸟类会游泳——比如天鹅。这就导致如果需要兼容“不会飞得鸟”和“会游泳得鸟”，要么得把所有方法全部写在 `Bird` 接口中，要么就要新建 `UnFlyableBird` 和 `SwimmableBird` 等特殊类。这样的设计违反单一职责原则：

- 单个接口的职责非常冗杂: “鸟”接口既要管“飞”，还要管“跑”和“叫”；

- 不同接口之间的职责有所重复：“鸟”和“狗”都有“跑”。

## 2.2 符合单一职责原则的情况

```java
interface IFlyable {
    double flyVelocity;
    void fly();
}

interface ISwimmable {
    double swimVelocity;
    void swim();
}

interface INoiseable {
    void noise();
}

interface IRunnable {
    double runVelocity;
    void run();
}
```

我们将所有的动作全部拆分到对应的接口中，最大化地细化接口的颗粒度。

```java
// 金丝雀
class Canary implements IFlyable, INoiseable, IRunnable {/* ... */}
// 天鹅，会游泳
class Swan implements IFlyable, INoiseable, IRunnable, ISwimmable {/* ... */}
// 鸵鸟，不会飞
class Ostrich implements INoisable, IRunnable {/* ... */}
```