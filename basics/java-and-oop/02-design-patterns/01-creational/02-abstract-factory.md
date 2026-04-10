# 二、抽象工厂

> _定义：_ 抽象工厂模式 (Abstract Factory Pattern) 是 一种提供一个创建 **一系列相关/相互依赖的对象** 的接口而无需指定具体类的设计模式。

我们从简单工厂出发，不难发现一些不足之处：我们的 UI 控件不仅仅有 **按钮**，还有 **对话框** 、**输入框** 等控件——这些控件在简单工厂的语境下需要各自维护一个工厂：

```java
class ButtonFactory { /*...*/ }
class DialogueFactory { /*...*/ }
class InputFactory { /*...*/ }
```

这么做会引起一系列问题：

其一，每个工厂种各自都有判断平台的 `switch` 选择代码： *新增产品需要修改其判断逻辑，违背开闭原则。*

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

**抽象工厂** 的思想是，将每一个操作系统平台的各种 UI 控件看作一个 **产品族** ，每个产品族中都有对应的具体产品的实现（如 Windows 产品族 和 MacOS 产品族 都有 Button、Dialogue 等）, 因此我们可以根据各个产品族总结出一个 **抽象产品族** 。抽象工厂将抽象产品族包装成接口，每个产品族对应会有一个具体工厂对象来实现这个接口。因此，我们可以根据产品族来选择性创建工厂对象，进而创建对应族群的产品。

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

| 平台 \ UI 控件 | 按钮 <br> `createButton()` | 对话框 <br> `createDialogue()` | 输入框 <br> `createInput()`| 对应工厂类/接口 <br> `GUIFactory` |
|-|-|-|-|-|
| Windows | Windows 按钮 | Windows 对话框 | Windows 输入框 | `WindowsGUIFactory` |
| MacOS | MacOS 按钮 | MacOS 对话框 | MacOS 输入框 | `MacOSGUIFactory` |

_(横行：产品族；纵行：单类产品继承体系下，各个产品族的此类产品)_

**在抽象工厂模式中：**

- 一系列抽象产品（对应到按钮接口、对话框接口和输入框接口）组成一个 **抽象产品族** 。
- 每个操作系统平台对应一系列抽象产品接口的具体实现（对应到Windows版的按钮、对话框和输入框），组成一个 **具体产品族** 。 
- 抽象产品族对应到 **抽象工厂接口** ，每个实例产品族对应到 **具体工厂类** 。
- 具体工厂类的选择仍然依赖于 `switch case` 语句。
    - 运行时操作系统平台是绝对不会变的，因此理论上只需要判断一次。抽象工厂模式能够只用一次 `switch` 判断便完成，节省了不必要的判断开销和代码冗余；
    - 但这也有一定的问题：它仍旧违反了开闭原则，当我需要增加平台的时候仍然需要修改该 `switch` 语句。

**总结**

- 适用场景：客户端不依赖于产品类实例创建和实现的细节；
- 优点：具体产品在应用层代码隔离，无需关心创建细节；将一系列产品统一到一起创建；
- 缺点：抽象产品族限定死了可能被创建的产品的集合，产品组中拓展新的产品需要修改所有的抽象工厂类；