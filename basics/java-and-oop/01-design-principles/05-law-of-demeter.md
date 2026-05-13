# 五、迪米特法则（最小知道法则）

> _定义：_ 迪米特法则 (Law of Demeter) 要求：一个对象应该对其他对象保持 **最少了解**，以降低类之间的耦合。

**只和朋友交流，不和陌生人说话。** “朋友” 类指：出现在 1.成员变量、2.方法输入、3.输出参数 中的类。出现在方法内部的类不属于朋友类。

## 5.1 违反迪米特法则的情况

小明要花100块，钱包是小明的直接朋友，钱是钱包里的东西。

```java
// 钱
class Money { int amount = 100; }

// 钱包
class Wallet {
    private Money money = new Money();

    public Money getMoney() { return money; }
}

// 人
class Person {
    private Wallet wallet = new Wallet();

    public void pay(float amount) {
        // 直接获取到 Money 对象实例，操作朋友的内部细节。
        // 违反迪米特法则
        Money money = wallet.getMoney();
        money.amount -= amount;
    }
}
```

## 5.2 符合迪米特法则的情况

```java
// 钱
class Money { int amount = 100; }

// 钱包
class Wallet {
    private Money money = new Money();

    public Money pay(float amount) { 
        money.amount -= amount;
     }
}

// 人
class Person {
    private Wallet wallet = new Wallet();

    public void pay(float amount) {
        // 通知朋友操作细节，只和朋友说话
        // 符合迪米特法则
        wallet.pay(amount);
    }
}
```

