# 三、生成器模式

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