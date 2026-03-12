## 创建型设计模式

## 一、简单工厂

设想以下场景。我要设计一套跨平台的系统：在不同的操作系统上，我需要调用不同的底层去实现同一套逻辑。以按钮空间为例，我需要在每个平台都实现 `Button` 接口：

```java
// 假想的UI控件接口，包含渲染方法
interface UIElements {
    void render();
}
// 假想的按钮接口，包含点击回调方法
interface Button extends UIElements {
    void onClick();
}
```

此时，我们可以为不同的平台设计对应的 **按钮** 类，让这个类只实现 `Button` 接口。

```java
// Windows 版本按钮
class WindowsButton implements Button {
    @Override
    void render() { /* ... */ }

    @Override
    void onClick() { /* ... */ }
}

// MacOS 版本按钮
class MacOSButton implements Button {
    @Override
    void render() { /* ... */ }

    @Override
    void onClick() { /* ... */ }
}
```

那么我们该怎么根据平台来决定使用哪个 `Button` 接口的实现类呢？

### 1.1 幼稚方法

一个显而易见的方法如下。在运行时，我们获取到所在平台的信息，并判断实例化哪个类。对应到的UI类逻辑如下：

```java
class UI {
    public void run(){
        Button button;
        String platform = getPlatform();

        switch (platform) {
            case "Windows": // Windows平台，获取Windows版按钮
                button = new WindowsButton();
                break;
            case "MacOS":   // MacOS平台，获取MacOS版按钮
                button = new MacOSButton();
                break;
            case default:
                throw new Exception("Unknown OS.");
        }

        button.render();    // 不论哪个平台，调用对应底层
    }
}
```

不难发现，我们在主程序里进行了一个 `switch` 选择操作。虽然这很简单，但是这种方式引入了一个问题：如果我们想要渲染多个按钮，就会将主程序逻辑撑的非常冗杂。此外，如果我们想添加操作系统平台，就需要 **修改主程序** ，徒增维护的成本。

### 1.2 简单工厂

简单工厂的思想是，我们把这份“运行时选择实际类”的逻辑抽离到 **工厂类** 中。这样有两个好处：其一，我们将switch 选择操作封装起来，让主程序专注于与其相关的逻辑实现；其二，我们将“拓展新平台”的地点从主程序迁出，当拓展新平台时只需要修改工厂类，而非主程序，显著提高了可维护性。

```java
class ButtonFactory {
    // 选择逻辑迁移到工厂！
    Button createButton(String platform) {
        Button button;
        switch (platform){
            case "Windows": // Windows平台，渲染Windows版按钮
                button = new WindowsButton();
                break;
            case "MacOS":   // MacOS平台，渲染MacOS版按钮
                button = new MacOSButton();
                break;
/**         
 *          新代码可以通过拓展工厂来增加        
 *          例子如下：
 *          case "Ubuntu":
 *              button = new UbuntuButton();
 *              break; 
*/
            case default:
                throw new Exception("Unknown OS.");
        }
        return button;
    }
}
```

修改后，我们将 **按钮工厂** 通过构造函数注入进 `UI` 类中。在主程序中，获取按钮则通过工厂的 `createButton` 方法。

```java
class UI {
    private ButtonFactory buttonFactory;

    public UI(ButtonFactory bf) {
        this.buttonFactory = bf;
    }

    public void run() {
        Button button = buttonFactory.createButton(getPlatform());
        button.render(); // 无论添加多少平台，这部分逻辑都不会变！
    }
}
```

---

## 二、抽象工厂

我们从简单工厂出发，不难发现一些不足之处：我们的 UI 控件不仅仅有 **按钮**，还有 **对话框** 、**输入框** 等控件——这些控件在简单工厂的语境下需要各自维护一个工厂：

```java
class ButtonFactory { /*...*/ }
class DialogueFactory { /*...*/ }
class InputFactory { /*...*/ }
```

这么做会引起一系列问题：

其一，回忆：每个工厂种各自都有判断平台的 `switch` 选择代码——代码冗余非常严重。上面的代码块中省略了这部分以提升观感。

其二，不难想象，在 `UI` 类中，这些控件会显得 `UI` 类的构造函数很臃肿。此外，开发者还需要为每一种“不需要某个控件”情况设计一种构造方法的重载。

```java
class UI {
    private ButtonFactory buttonFactory;
    private DialogueFactory dialogureFactory;
    private InputFactory inputFactory;

    // 全部都要
    public UI (
        ButtonFactory bf, 
        DialogueFactory df, 
        InputFactory _if) { /* ... */ }

    // 不需要ButtonFactory的情况
    public UI (
        DialogueFactory df, 
        InputFactory _if) { /* ... */ }

    // ...
}
```

**抽象工厂** 的思想是，我们把每一种工厂的对应的 `switch case` 下的创建逻辑都统一起来：每一个 `switch` 的 `case` 下，对应一套 `create` 代码，这一套代码组合起来作为当前 `switch case` 下的工厂。

```java
// GUI 工厂接口
interface GUIFactory {
    Button createButton();
    Dialogue createDialogue();
    Input createInput();
}

// Windows GUI 工厂
class WindowsGUIFactory implements GUIFactory {
    @Override
    Button createButton() { /* ... */ }

    @Override
    Dialogue createDialogue() { /* ... */ }

    @Override
    Input createInput() { /* ... */ }
}

// MacOS GUI 工厂
class MacOSGUIFactory implements GUIFactory {
    @Override
    Button createButton() { /* ... */ }

    @Override
    Dialogue createDialogue() { /* ... */ }

    @Override
    Input createInput() { /* ... */ }
}
```

最后，在 `UI` 类中，我们只需要注入 `GUIFactory` 类即可。

```java
class UI {
    private GUIFactory guiFactory;

    public UI(GUIFactory gf) {
        this.guiFactory = gf;
    }

    public void run(String[] args) {
        // 生成UI 控件
        Button button = guiFactory.createButton();
        Dialogue dialogue = guiFactory.createDialogue();
        Input input = guiFactory.createInput();

        // 执行渲染
        button.render();
        dialogue.render();
        input.render();
    }
}
```

注意，`UI` 的构造方法中，入参的形参类型是 `GUIFactory`，代表它可以接收任何实现了该接口的类实例作为实参。也就是说，我们可以在外面判断好平台之后，将对应的实参放入 `UI` 的构造方法中。

```java
class Main {
    public static void main() {
        String platform = getPlatform();
        GUIFactory gf;

        // 根据平台决定具体实现哪个工厂
        switch(platform) {
            case "Windows":
                gf = new WindowsGUIFactory();
                break;
            case "MacOS":
                gf = new MacOSGUIFactory();
                break;
            case default:
                throw Exception("Unknown OS!");
        }

        UI ui = new UI(gf); // 放入UI类中
        ui.run();           // 跑UI线程
    }
}
```

**总结：** 在抽象工厂模式中：
- 一系列抽象产品（对应到按钮接口、对话框接口和输入框接口）组成一个 **抽象产品族** 。
- 每个操作系统平台对应一系列抽象产品接口的具体实现（对应到Windows版的按钮、对话框和输入框），组成一个 **具体产品族** 。 
- 抽象产品族对应到 **抽象工厂接口** ，每个实例产品族对应到 **具体工厂类** 。
- 具体工厂类的选择仍然依赖于 `switch case` 语句。
    - 运行时操作系统平台是绝对不会变的，因此理论上只需要判断一次。抽象工厂模式能够只用一次 `switch` 判断便完成，节省了不必要的判断开销和代码冗余；
    - 但这也有一定的问题：它仍旧违反了开闭原则，当我需要增加平台的时候仍然需要修改该 `switch` 语句。

| 平台 \ UI 控件 | 按钮 <br> `createButton()` | 对话框 <br> `createDialogue()` | 输入框 <br> `createInput()`| 对应工厂类/接口 <br> `GUIFactory` |
|-|-|-|-|-|
| Windows | Windows 按钮 | Windows 对话框 | Windows 输入框 | `WindowsGUIFactory` |
| MacOS | MacOS 按钮 | MacOS 对话框 | MacOS 输入框 | `MacOSGUIFactory` |

---

## 三、生成器模式

抽象工厂模式解决了产品族多样化的问题；而 **生成器模式** 解决产品本身多样化的问题。

看一个具体的例子：想像你是 `Minecraft` 的开发者，你需要生成一匹马——然而一匹马的参数非常多：

| 参数 | 含义 | 是否必填 |
|-|-|-|
| `String breed` | 品种 | 是 |
| `String color` | 毛色 | 是 |
| `double speed` | 速度 | 默认 `14.0` |
| `int endurance` | 耐力 | 默认 `100` |
| `boolean hasSaddle` | 是否有马鞍 | 默认 `false` |
| `Equipment equipment` | 装备 | 默认 `null` |

过多的参数会引来两个问题。其一，构造函数会过于冗杂；其二，我们需要给任意的“不需要某些入参”的情况都设计一份构造函数的重载。这严重提高了维护的成本。

**生成器模式** 的思想是，我们将“传参”的形式 **过程化** —— 即以链式调用的形式传递参数，并进行一次最终的实例化。一个符合生成器模式思想的 `Horse` 类如下：

```java
public class Horse {
    private final String breed;         // 品种，必填
    private final String color;         // 毛色，默认"棕色"
    private final double speed;         // 速度，默认14.0
    private final int endurance;        // 耐力，默认100
    private final boolean hasSaddle;    // 有无马鞍，默认false
    private final Equipment equipment;  // 装备，默认无

    // 私有构造函数，只给Builder用
    private Horse(Builder builder) {
        this.breed = builder.breed;
        this.color = builder.color;
        this.speed = builder.speed;
        this.endurance = builder.endurance;
        this.hasSaddle = builder.hasSaddle;
        this.gender = builder.gender;
        this.age = builder.age;
        this.equipment = builder.equipment;
    }

    // 静态Builder内部类
    public static class Builder {
        // 必填参数
        private final String breed;
        
        // 可选参数 + 默认值
        private String color = "棕色";
        private double speed = 14.0;
        private int endurance = 100;
        private boolean hasSaddle = false;
        private Equipment equipment = null;

        // Builder构造函数：只收必填参数
        public Builder(String breed) {
            this.breed = Objects.requireNonNull(breed, "品种必填");
        }

        // 链式setter，返回this
        public Builder color(String color) { this.color = color; return this; }
        public Builder speed(double speed) { this.speed = speed; return this; }
        public Builder endurance(int endurance) { this.endurance = endurance; return this; }
        public Builder hasSaddle(boolean hasSaddle) { this.hasSaddle = hasSaddle; return this; }
        public Builder equipment(Equipment equipment) { this.equipment = equipment; return this; }

        // build()创建不可变Horse对象
        public Horse build() {
            return new Horse(this);
        }
    }
}
```

那么在生成一个新的 `Horse` 实例时，我们可以根据需要选择性决定是否调用某个 `setter`，从而达到选择性入参的目的。

```java
class Main {
    public static void main(String[] args) {
        Horse.Builder horseBuilder = new Horse.Builder("plain");

        Horse horse = horseBuilder  // 链式调用，构造
                        .color("brown")
                        .speed(13)
                        .hasSaddle(true);
                        .build(); // 这一刻才真正实例化
            // 选择性不调用equipment，即保持默认
    }
}
```

这样的构造过程就非常简洁。

---

## 四、原型模式

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

    // 嵌套对象
    private Wing wing;
    
    public Bird(String name, int endurance, int health) {
        this.name = name;
        this.endurance = endurance;
        this.health = health;
        this.wing = new Wing(21);
    }   

    @Override
    public void fly() { /* ... */ }

    @Override
    public Bird clone() {
        // 浅拷贝基础字段
        Bird clonedBird = (Bird) super.clone();
        
        // 深拷贝嵌套对象，避免共享引用！
        cloned.wing = new Wing(this.wing.getSpan());

        return cloned;
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

---

## 五、单例模式

**单例模式** 保证一个类只有一个实例，并提供一个访问该实例的全局节点。

假设你是一个游戏开发者，你需要设计一个全局的场景管理器。单例模式就是最好的选择。

单例模式下，类的构造器被私有化，而一个对外开放的 `get` 函数则成为了哪个静态的初始化节点。

**懒汉式写法**

懒汉式写法线程不安全：若在 `new SceneManager()` 处发生竞态条件，`instance` 则有可能被覆盖，或者造成多实例现象，从而违反单例模式原则。

当然，我们可以给 `get` 方法添加 `synchronize` 字段来保证同步代码，但是这会导致显著的性能降级。

```java
public class SceneManager {
    private static final SceneManager instance;

    private SceneManager() {}

    public static SceneManager get() {
        // 全局实例在第一次访问时才被加载
        if (this.instance == null)
            this.instance = new SceneManager();

        return this.instance;
    }
}
```

**饿汉式写法**

饿汉式写法是线程安全的。因为实例在类加载时创建，而类加载时JVM会为静态变量加锁。

```java
public class SceneManager {
    // 全局实例在类加载阶段就被创建了
    private static final SceneManager instance = new SceneManager();

    private SceneManager() {}

    public static SceneManager get() {
        return this.instance;
    }
}
```