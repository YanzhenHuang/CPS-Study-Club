## 结构型设计模式

---

## 一、 适配器模式

当一个方法的输出需要作为另一个方法的输入时，若出现格式不匹配的情况，则可以添加一个适配器在从中作转换，是为 **适配器模式** 。

1. 适配器实现其中一个接收数据的实例的接口。
2. 接收数据的实例可以安全调用适配器的方法。

_例1_ 举例，我们有一个 `DataReciever` 接口，其中的方法 `receiveData()` 需要接收一个 `Json` 示例的入参。然而，我们的数据提供方 `DataProvider` 只能返回 `Xml` 类型的实例数据。

```java
// 数据接收方，只能接收 Json 类型的数据
interface DataReceiver {
    void receiveData(Json json);
}

// 数据提供方：只能提供 Xml 类型的数据
class DataProvider {
    public Xml sendData() {
        Xml xml = new Xml();
        /** */
        return xml;
    }
}
```

我们添加一个 **实例适配器** `DataAdapter` 。它继承自 `DataProvider` ，并通过重载 `getData()` 方法，将 `Xml` 转换成 `Json` 。

```java
class DataAdapter extends DataProvider implements DataReceiver {

    // 转换方法：将 Xml 转换成 Json
    private Json convert(Xml xml) { /* ... */ }

    @Override
    public Json sendData() {
        Xml xml = super.sendData();
        Json json = convert(xml);
        return json;
    }
}
```

在 **实例适配器** 之外，还有 **类适配器** 。但是它需要多继承工作，而并非所有语言都支持多继承。因此此处不多介绍。

---

## 二、 桥接模式

假设你是 `Minecraft` 的开发者。你定义怪物的 AI 需要包含两种行为：攻击 (Attack) 和 移动 (Move) ，二者分别由多个种类。如，攻击包含击打 (hit) 和 射箭 (shoot) 等；移动包含走路 (walk) 和 爬 (crawl) 等。

桥接模式能够允许生物和AI解耦：任意生物都可以执行任意AI的行为。

```java
// 根据 AI 的继承树定义不同的 AI 类
interface MobAI {
    void attack();
    void move();
}

class MeleeAI implements MobAI {
    public void attack() { hit(); }
    public void move() { walk(); }
}

class RangedAi implements MobAI { 
    public void attack() { shoot(); }
    public void move() { walk(); }
}

class ExplodeAi implements MobAI { 
    public void attack() { explode(); }
    public void move() { crawl(); }
}

// 根据 Mob 的继承树定义不同的 Mob 类

abstract class Mob {
    protected MobAI ai; // 关键：持有 AI 的引用

    class Mob (MobAI ai) {
        this.ai = ai;
    }

    public void setAIBehavior(MobAI ai) {
        this.ai = ai;
    }

    void performAttack() {
        aiBehavior.attack();
    }
}

class Zombie extends Mob {
    public Zombie(MobAI ai) { super(ai); }
    // ...
}

class Skeleton extends Mob {
    public Skeleton(MobAI ai) { super(ai); }
    // ...
}

class Creeper extends Mob { 
    public Creeper(MobAI ai) { super(ai); }
    // ...
}

// 自由组合

public class Main {
    public static void main(String[] args) {
        Zombie zombie = new Zombie(new MeleeAi());
        Skeleton skeleton = new Skeleton(new RangedAi());
        Creeper creeper = new Creeper(new ExplodeAi());

        // 生成会自爆的僵尸也是可以的
        Zombie zombie = new Zombie(new ExplodeAi());
    }
}
```

桥接模式的结构本质上就是 “两个继承树，其中一个持有另一个的引用”。它是由主从关系的：持有方位主，被持有方为从；抽象方为主，实现方位从。

如果我们简化 AI 的继承树（如只有一层继承），桥接模式就退化为策略模式。

---

## 三、 组合模式

组合模式中，元素可以递归地持有。一个常见的应用场景就是抽象语法树。

---

## 四、 装饰模式

---

## 五、 外观模式

---

## 六、 享元模式

---

## 七、代理模式
