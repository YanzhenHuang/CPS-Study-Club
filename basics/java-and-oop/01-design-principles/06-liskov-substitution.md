# 六、里氏替换原则

> _定义：_  设有 T1 和 T2 两个类，o1 和 o2 分别为他们的实例。设P为以程序。当P中所有的 o1 都替换成 o2时，若程序P的行为没有发生变化，则 T2 类型是 T1 的子类型。
>
> 优点：加强程序的健壮性、变更兼容性、可维护性于拓展性。降低需求变更时引入的风险。

**延申**

- 一个软件实体如果适用一个父类的话，就一适用用于其子类。
- 所有引用父类的地方必须能透明地使用其子类的对象，子类对象能够替换父类对象而程序逻辑不变。
    - 子类可以实现父类的抽象方法，但不能覆盖父类的非抽象方法。子类可以增加自己特有的方法——P父类中没有子类中的这些方法，故P必不会使用这些方法，增加这些方法断然不会改变P的行为。
    - 子类重载父类的方法时，入参需要比父类更宽松。
    - 子类重载、重写或实现父类的方法时，输出要比父类更严格。

## 6.1 违反里氏替换原则的例子

_父类：长方形_

```java
class Rectangle {
    protected int width;
    protected int height;

    public void setWidth(int width) {
        this.width = width;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public int getArea() {
        return width * height;
    }
}
```

_子类：正方形_

```java
class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        // 宽高必须一起变
        super.setWidth(width);
        super.setHeight(width);
    }

    @Override
    public void setHeight(int height) {
        super.setWidth(height);
        super.setHeight(height);
    }
}

```

_测试：_

```java
public class Test {
    public static void main(String[] args) {
        // 父类场景：长方形
        Rectangle rect = new Rectangle();
        rect.setWidth(4);
        rect.setHeight(5);
        System.out.println(rect.getArea()); // 20，符合预期

        // 用子类替换父类 → 逻辑出错
        Rectangle rect2 = new Square();
        rect2.setWidth(4);
        rect2.setHeight(5); // 宽度又被覆盖成 5 了
        System.out.println(rect2.getArea()); // 25，结果不对
    }
}
```

## 6.2 符合里氏替换原则的例子

正方形不是长方形的子类，而是二者共同继承一个更抽象的 `Quadrilateral`（四边形）类。

_抽象父类_

```java
abstract class Quadrilateral { 
    public abstract int getArea();
}
```

_子类：长方形_

```java
class Rectangle extends Shape {
    private int width;
    private int height;

    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public int getArea() {
        return width * height;
    }
}
```

_子类：正方形_

```java
class Square extends Shape {
    private int edge;

    public Square(int edge) {
        this.edge = edge;
    }

    @Override
    public int getArea() {
        return edge * edge;
    }
}
```

_测试：_

```java
public class Test {
    // 统一接收 Quadrilateral，子类可随意替换
    public static void printArea(Quadrilateral shape) {
        System.out.println(shape.getArea());
    }

    public static void main(String[] args) {
        Shape rect = new Rectangle(4, 5);
        Shape square = new Square(5);

        printArea(rect);  // 20
        printArea(square); // 25
    }
}
```