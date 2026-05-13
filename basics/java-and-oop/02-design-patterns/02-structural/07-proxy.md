# 七、代理模式

> _定义：_ 代理模式 (Proxy Pattern) 是一种为对象 **提供代理** 以控制该对象的访问，以达到保护、增强该目标对象之目的的设计模式。

## 7.0 基础思想

首先，我们需要明确被代理的对象能够提供什么服务。这个服务将会被总结为一个接口。

_服务接口_

```java
class IService {
    void service1(int n);
    double service2();
}
```

其次，被代理的对象和代理它的对象都需要实现该接口。

_被代理的对象_

```java
class Service implements IService {
    public void service1(int n) {/** ... */}
    public void service2(String s) {/** ... */}
}

```

_代理对象_

```java

class ServiceProxy implements IService {

    private Service service;

    public ServiceProxy(Service service) {
        this.service = service;
    }

    public void service1(int n) {
        before();
        service.service1(n);
        after();
    }

    public void service2(String s) {
        before();
        service.service2(s);
        after();
    }

    public void before() {/** */}
    public void after() {/** */}
}
```

## 7.1 静态代理 (Static Proxy)

