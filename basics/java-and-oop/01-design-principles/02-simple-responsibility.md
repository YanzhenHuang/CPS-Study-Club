# 二、单一职责原则

> _定义：_ 单一职责原则 (Simple Responsibility Principle) 要求不要存在多于一个导致类发生变更的原因，一个软件实体（类、接口、方法）只负责一项职责。

## 2.1 违反单一职责原则的情况

```java
public class Modifier {
    // 职责不清晰，可以拆
    private void modifyUserInfo(String userName, String address, String ... fields) {
        // ...
    }
}
```

## 2.2 符合单一职责原则的情况

拆都不能再拆了，就符合单一职责原则。

```java
public class Modifier { 

    // 职责清晰，无法拆分。
    private void modifyUserName(String userName) {/* ... */}
    private void modifyUserAddress(String address) {/* ... */}
    // ...
}
```