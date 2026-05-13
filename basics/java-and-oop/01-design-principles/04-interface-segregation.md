# 四、接口隔离原则

> _定义：_ 接口隔离原则 (Interface Segregation Principle) 要求尽量将一个总接口拆分成多个细分的接口。客户端通过依赖多个接口，排列组合出总的依赖。
> _即：_ 尽量细化接口，接口中的方法要尽可能少。


## 4.1 不符合接口隔离原则的情况

```java
interface Bird {
    void fly();
    void run();
    void noise();
}

interface Dog {
    void run();
    void noise();
}

interface Fish {
    void swim();
}
```

这个例子中，我们默认了“只要是鸟就一定会飞”、 “只有鱼才会游泳”。然而并非所有鸟都会飞——比如鸵鸟；且也有鸟类会游泳——比如天鹅。

- 若“鸵鸟”要实现 `Bird`，则`fly`方法是多余的；

- 单个接口的职责非常冗杂: “鸟”接口既要管“飞”，还要管“跑”和“叫”；

- 不同接口之间的方法有所重复：“鸟”和“狗”都有“跑”。

## 2.2 符合接口隔离原则的情况

```java
interface IFlyable {
    void fly();
}

interface ISwimmable {
    void swim();
}

interface INoiseable {
    void noise();
}

interface IRunnable {
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
