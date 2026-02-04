# 面向对象编程

## Java内部类

在 Java 中，可以将一个类定义在另外一个类里面或者一个方法里面，这样的类叫做内部类。

### 1）成员内部类

成员内部类是最常见的内部类，看下面的代码：

```java
class Wanger {
    int age = 18;
    
    class Wangxiaoer {
        int age = 81;
    }
}
```

看起来内部类 Wangxiaoer 就好像 Wanger 的一个成员，成员内部类可以无限制访问外部类的所有成员属性。

```java
public class Wanger {
    int age = 18;
    private String name = "沉默王二";
    static double money = 1;

    class Wangxiaoer {
        int age = 81;
        
        public void print() {
            System.out.println(name);
            System.out.println(money);
        }
    }
}
```

内部类可以随心所欲地访问外部类的成员，但外部类想要访问内部类的成员，就不那么容易了，必须先创建一个成员内部类的对象，再通过这个对象来访问：

```java
public class Wanger {
    int age = 18;
    private String name = "沉默王二";
    static double money = 1;

    public Wanger () {
        new Wangxiaoer().print();
    }

    class Wangxiaoer {
        int age = 81;

        public void print() {
            System.out.println(name);
            System.out.println(money);
        }
    }
}
```



这也就意味着，如果想要在静态方法中访问成员内部类的时候，就必须先得创建一个外部类的对象，因为内部类是依附于外部类的。

```java
public class Wanger {
    int age = 18;
    private String name = "沉默王二";
    static double money = 1;

    public Wanger () {
        new Wangxiaoer().print();
    }

    public static void main(String[] args) {
        Wanger wanger = new Wanger();
        Wangxiaoer xiaoer = wanger.new Wangxiaoer();
        xiaoer.print();
    }

    class Wangxiaoer {
        int age = 81;

        public void print() {
            System.out.println(name);
            System.out.println(money);
        }
    }
}
```

这种创建内部类的方式在实际开发中并不常用，因为内部类和外部类紧紧地绑定在一起，使用起来非常不便。



### 2）局部内部类

局部内部类是定义在一个方法或者一个作用域里面的类，所以局部内部类的生命周期仅限于作用域内。

```java
public class Champion {
    public void fight() {
        int damage = 100; // 局部变量

        // 定义在方法内部的类
        class Skill {
            public void execute() {
                System.out.println("造成伤害: " + damage);
            }
        }

        Skill s = new Skill();
        s.execute();
    }
}
```

局部内部类就好像一个局部变量一样，它是不能被权限修饰符修饰的，比如说 public、protected、private 和 static 等。

局部内部类就像是你在某个房间（方法）里临时请的一个装修工，一旦你出了这个房间，谁也不认识他。



<span style="color:red"> 面试：为什么局部变量必须是 `final`</span>

这是关于局部内部类**最常问、最经典**的面试题。

- **现象**：在 Java 8 之前，局部内部类访问的局部变量必须显式加上 `final`；Java 8 之后可以不加，但该变量必须是 **“事实上不可变”（Effectively Final）** 的（即你不能在后面修改它的值）。
- **底层原因（核心关键）**：**生命周期不一致导致的“数据备份”**。

1. **方法栈 vs 堆**：方法执行时，局部变量 `damage` 存在于**栈**中。方法跑完，栈帧销毁，变量就没了。
2. **对象永生**：局部内部类产生的对象 `s` 存在于**堆**中。方法跑完了，对象可能还没被垃圾回收（GC）。
3. **穿帮了怎么办？**：如果对象还在，但它依赖的变量已经没了，程序就会崩溃。
4. **解决办法**：Java 偷偷把这个变量**复制**了一份放进了内部类对象里。为了保证这份“备份”和原来的变量长得一模一样，Java 强制要求这个变量不能变（`final`）。



<span style="color:red">**面试常见问题清单（Q&A）**</span>

> **Q1：局部内部类和匿名内部类有什么区别？**

- **匿名内部类**：没有名字，通常是为了实现接口或继承类，随用随扔，最常用。
- **局部内部类**：有名字，可以多次实例化（在方法内），也可以定义构造方法。它更像是一个完整的类，只是限制了活动范围。

> **Q2：局部内部类能访问外部类的成员吗？**

- **能！** 它可以无障碍访问外部类的所有成员变量和方法（包括私有的）。因为它依然是在外部类这个“家”里定义的。

> **Q3：为什么局部内部类在实际开发中很少见？**

- 因为它太重了。如果只是为了临时用一下功能，大家通常首选 **Lambda 表达式**（Java 8+）或 **匿名内部类**。局部内部类只在“方法内需要定义复杂的逻辑、且需要复用、还要起个名字”这种极少数场景下才会出现。



### 3）匿名内部类

匿名内部类是我们平常用得最多的，尤其是启动多线程的时候，会经常用到，并且 IDE 也会帮我们自动生成。

```java
public class ThreadDemo {
    public static void main(String[] args) {
        Thread t = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName());
            }
        });
        t.start();
    }
}
```

匿名内部类就好像一个方法的参数一样，用完就没了，以至于我们都不需要为它专门写一个构造方法，它的名字也是由系统自动命名的。仔细观察编译后的字节码文件也可以发现，匿名内部类连名字都不配拥有，哈哈，直接借用的外部类，然后 `$1` 就搞定了。

![9.innerClass](../assets/javaAssets/9.innerClass.png)



匿名内部类是唯一一种没有构造方法的类。就上面的写法来说，匿名内部类也不允许我们为其编写构造方法，因为它就像是直接通过 new 关键字创建出来的一个对象。

匿名内部类的作用主要是用来继承其他类或者实现接口，并不需要增加额外的方法，方便对继承的方法进行实现或者重写。

**匿名内部类（Anonymous Inner Class）** 是 Java 内部类家族中地位最高、使用频率最广的“无名英雄”。

如果说局部内部类是“大楼里的临时工”，那么匿名内部类就是**“路边临时招募的志愿者”**——它连名字都没有，干完活就走，绝不留痕迹。

**1. 它的长相：创建与实例化“合体”**

通常我们要实现一个接口，得先写个类，再 `new` 它。匿名内部类直接把这两步合二为一：

```java
// 还是你之前的教练接口
interface Coach {
    void command();
}

public class Demo {
    public static void main(String[] args) {
        // 这里的代码块就是一个匿名内部类
        Coach c = new Coach() {
            @Override
            public void command() {
                System.out.println("临时教练说：大龙集合！");
            }
        }; 
        c.command();
    }
}
```

**语法拆解：**

- **`new Coach()`**：这看起来像是在实例化接口（其实不是），它实际上是在告诉 JVM：“我要创建一个实现 `Coach` 接口的类”。
- **`{ ... }`**：这个大括号里的内容就是这个**无名类**的类体。



**2. 匿名内部类的“三大准则”**

为了在面试和开发中不出错，你需要记住这三点：

1. **没有名字**：因为它没有名字，所以它**没有构造方法**（Constructor）。
2. **只能继承一个父类或实现一个接口**：它是单干户，不能既继承 A 又实现 B。
3. **变量捕获（重要！）**：和局部内部类一样，它访问的局部变量必须是 `final` 或 **事实上不可变（Effectively Final）** 的。



**3. 它存在的意义：为了“省事”**

如果你只需要用一次某个功能，特意去写一个 `.java` 文件或者写一个具名内部类就太沉重了。

**匿名内部类的优势：**

- **高内聚**：代码就在调用的地方，逻辑非常紧凑，不需要跳到别的文件去看实现。
- **减少命名污染**：不用为了给类起个“什么什么Impl”的名字而伤脑筋。



<span style="color:red">**4. 面试必考：匿名内部类 vs Lambda 表达式**</span>

在 Java 8 之后，匿名内部类的风头被 **Lambda** 抢走了不少。面试官经常会问它们的区别：

| **特性**          | **匿名内部类**                   | **Lambda 表达式**                            |
| ----------------- | -------------------------------- | -------------------------------------------- |
| **适用范围**      | 接口、抽象类、普通类             | **仅限函数式接口**（只有一个抽象方法的接口） |
| **关键字 `this`** | 指向内部类自己                   | 指向外部类                                   |
| **底层实现**      | 编译后产生一个 `.class` 文件     | 动态生成，通常性能更好                       |
| **多方法支持**    | 支持（可以重写接口里的多个方法） | **不支持**（只能实现一个方法）               |



### 4）静态内部类

静态内部类和成员内部类类似，只是多了一个 [static 关键字](https://javabetter.cn/oo/static.html)。

```java
public class Wangsi {
    static int age;
    double money;
    
    static class Wangxxiaosi {
        public Wangxxiaosi (){
            System.out.println(age);
        }
    }
}
```

由于 static 关键字的存在，静态内部类是不允许访问外部类中非 static 的变量和方法的，这一点也非常好理解：你一个静态的内部类访问我非静态的成员变量干嘛？

<div align="center">
  <img src="../assets/javaAssets/9.staticInnerClass.png" width="55%">
</div>

为什么要使用内部类呢？

这个问题问的非常妙

在《Think in java》中有这样一句话：

> 使用内部类最吸引人的原因是：每个内部类都能独立地继承一个（接口的）实现，所以无论外围类是否已经继承了某个（接口的）实现，对于内部类都没有影响。

在我们程序设计中有时候会存在一些使用接口很难解决的问题，这个时候我们可以利用内部类提供的、可以继承多个具体的或者抽象的类的能力来解决这些程序设计问题。可以这样说，接口只是解决了部分问题，而内部类使得多重继承的解决方案变得更加完整。

使用内部类还能够为我们带来如下特性：

- 1、内部类可以使用多个实例，每个实例都有自己的状态信息，并且与其他外围对象的信息相互独立。
- 2、在单个外部类中，可以让多个内部类以不同的方式实现同一个接口，或者继承同一个类。
- 3、创建内部类对象的时刻并不依赖于外部类对象的创建。
- 4、内部类并没有令人迷惑的“is-a”关系，他就是一个独立的实体。
- 5、内部类提供了更好的封装，除了该外围类，其他类都不能访问。



核心思想是：**内部类不是为了套娃而套娃，它是为了弥补 Java “单继承”的遗憾，并提供极致的封装。**

Java 规定一个类只能有一个亲爹（单继承）。但如果一个类既想表现得像 A，又想表现得像 B，该怎么办？

- **接口**：能解决一部分，但接口里不能存状态（变量）。
- **内部类**：这是“终极方案”。

**比喻：** 比如你（外部类）已经继承了“父亲”的基因。但你还想学习“裁缝”和“厨师”的技能，并且这两个技能都需要继承各自家族的“秘籍”（抽象类）。 你可以让 `内部类1` 继承 `裁缝秘籍`，`内部类2` 继承 `厨师秘籍`。这样，你这个整体就同时拥有了多个具体类的能力。



举例说明:

在 Java 中，一个类只能 `extends` 一个父类。假设我们有两个抽象类：`Father`（父亲，教你钓鱼）和 `Teacher`（老师，教你写代码）。如果我们想让一个 `Child` 类同时继承这两个人的本领，直接继承是不行的。

这时候，**内部类**就闪亮登场了。



```
// 抽象类 A：父亲的技能
abstract class Father {
    abstract void fish();
}

// 抽象类 B：老师的技能
abstract class Teacher {
    abstract void programming();
}

// 外部类：孩子
public class Child {
    
    // 特性 5：良好的封装，这两个内部类可以是 private，外界根本不知道它们的存在
    
    // 内部类 1：独立继承 Father
    private class FatherSide extends Father {
        @Override
        void fish() {
            System.out.println("继承了父亲的技能：去河边钓鱼");
        }
    }

    // 内部类 2：独立继承 Teacher
    private class TeacherSide extends Teacher {
        @Override
        void programming() {
            System.out.println("继承了老师的技能：在电脑前写 Java");
        }
    }

    // 暴露给外界的方法
    public void doFish() {
        new FatherSide().fish();
    }

    public void doProgram() {
        new TeacherSide().programming();
    }

    public static void main(String[] args) {
        Child xiaoming = new Child();
        // xiaoming 看起来只是一个 Child 实体
        // 但他通过内部类，同时拥有了 Father 和 Teacher 的能力
        xiaoming.doFish();
        xiaoming.doProgram();
    }
}
```

**结合你的 5 点深度拆解：**

**1. 独立继承与多继承（核心）**

> “内部类使得多重继承的解决方案变得更加完整。”

- **解释**：在上面的代码中，`Child` 类本身没有继承任何类，但它内部的两个小类分别继承了 `Father` 和 `Teacher`。这绕过了 Java 单继承的限制，让 `Child` 能够“白嫖”两个具体类的实现逻辑。

**2. 特性 1 & 2：多个实例与不同实现**

> “让多个内部类以不同的方式实现同一个接口...”

- **代码场景**：如果 `Father` 接口有一个“休息”方法，你可以在 `Child` 内部写两个内部类：`SummerRest` 和 `WinterRest`。
- **效果**：同一个外部类对象，可以根据需要产生两种完全不同的“休息”逻辑。如果是普通类实现接口，你只能写死一种逻辑。

**3. 特性 4：没有迷惑的 “is-a” 关系**

> “它就是一个独立的实体。”

- **解释**：如果 `Child extends Father`，逻辑上是“孩子是一个父亲”，这很荒谬。
- **内部类做法**：`Child` 内部有一个 `FatherSide`。这叫 **“组合（Has-a）”**。逻辑变成了“孩子拥有父亲的一部分能力”。这比强制继承更符合人类的逻辑直觉。

**4. 特性 5：完美的封装**

> “除了该外围类，其他类都不能访问。”

- **解释**：注意我代码里的 `private class`。
- **效果**：在外面你只能看到 `xiaoming.doFish()`，你根本不知道 xiaoming 内部是通过哪个类、哪行代码实现的。这种安全性是外部普通类无法提供的（普通类不能是 private）。





**这么做的意义是什么?**

在 Java 的世界里，法律规定：**一个孩子只能有一个亲生父亲（单继承）**。 如果你写 `class Child extends Father, Teacher`，编译器会直接报错。

**但是，现实需求往往很复杂：** 你既希望 `Child` 能像 `Father` 一样去钓鱼（继承 `Father` 的所有非抽象方法和属性），又希望他能像 `Teacher` 一样去写代码（继承 `Teacher` 的资源）。

**内部类完美解决了这个矛盾：**

- `Child` 本身可以空出来，去继承别的类。
- `FatherSide` 内部类作为“代理人”，替 `Child` 继承了 `Father`。
- `TeacherSide` 内部类作为另一个“代理人”，替 `Child` 继承了 `Teacher`。

**最终结果：** `Child` 这个类，通过两个“马甲”，间接拿到了两个抽象类的入场券。



**2. 实际生产生活中的场景**

这种设计在复杂的框架设计和底层开发中随处可见：

**场景 A：Android 开发中的“多路监听器”**

在 Android 中，一个界面（Activity）可能需要监听很多事件：点击、长按、滑动。这些监听器往往是抽象类（如 `GestureDetector.SimpleOnGestureListener`）。

- **问题**：Activity 已经继承了系统的 `Activity` 类，无法再继承这些手势抽象类。
- **方案**：在 Activity 内部定义多个私有的内部类，分别继承不同的手势抽象类。
- **意义**：Activity 变成了一个“大管家”，它内部雇佣了多个“专业小弟”来处理不同的手势。

**场景 B：多线程任务管理（Task Execution）**

假设你有一个“下载管理器”类（DownloadManager），它需要同时处理两个逻辑：

1. **心跳检测**：需要继承 `TimerTask` 来定时运行。
2. **文件写入**：需要继承某个 `Thread` 子类来处理 IO。

- **方案**：通过两个内部类，一个继承 `TimerTask` 负责跳动，一个继承 `Thread` 负责写盘。
- **意义**：`DownloadManager` 依然保持着它的独立性，不被任何一个父类“绑架”。

**场景 C：适配不同版本的第三方库**

想象你在做一个视频播放器，需要适配两个老旧的库：`LegacyPlayerA` 和 `LegacyPlayerB`。这两个库都要求你必须继承它们的 `BaseHandler` 才能用。

- **方案**：在你的播放器类里，写两个内部类分别继承 A 和 B 的基类。
- **意义**：你的主播放器类就像一个“适配器”，完美融合了两个完全不同的系统。



**那为什么不适用接口来完成呢?**

简单一句话：**接口只能解决“行为准则（做什么）”的问题，而内部类解决了“状态继承（是什么/怎么做）”的问题。**

**1. 状态（成员变量）的限制**

**接口**是“穷光蛋”，它不能拥有自己的**实例变量**。虽然 Java 8 之后接口有了 `default` 方法，但它依然存不了数据。

- **场景**：假设 `Father` 类里有一个 `int fishingRodCount`（鱼竿数量），每钓一次鱼，鱼竿数量就减一。
- **接口方案**：无法实现。接口里的变量默认都是 `static final` 的常量，你改不了它的值。
- **内部类方案**：内部类继承的是**真正的抽象类**，它可以拥有自己的属性，并记录状态的变化。



**2. 避免“强制实现”的臃肿**

如果你让 `Child` 直接实现两个接口，`Child` 类会变得非常混乱。

- **接口方案**：所有的实现逻辑都得堆在 `Child` 类的大括号里。如果这两个接口加起来有 20 个方法，`Child` 类瞬间就变成了一个“巨型垃圾场”。
- **内部类方案**：这叫**“逻辑拆分”**。钓鱼的逻辑全在 `FatherSide` 里，写代码的逻辑全在 `TeacherSide` 里。`Child` 只需要像插拔式 U 盘一样调用它们。这让代码结构非常清爽，符合“单一职责原则”。



**3. 处理“方法名冲突”**

这是一个非常阴险的场景：如果两个接口里恰好有两个**名字一模一样、但意义完全不同**的方法，接口就抓瞎了。

- **例子**：
  - `interface Cook { void heat(); }` （厨师的：加热食物）
  - `interface Electronics { void heat(); }` （电器的：机身发热）
- **接口方案**：如果 `Child` 同时实现这两个接口，它只能写一个 `void heat()` 方法。那么这个方法到底是做饭还是发烧？你没法区分。
- **内部类方案**：
  - `inner class MyCook implements Cook { void heat() { ... } }`
  - `inner class MyPhone implements Electronics { void heat() { ... } }`
  - **结果**：通过不同的内部类对象，你完美地区分了两个同名的动作。



**4. 访问外部类的“后门”权限**

这是内部类最核心的杀手锏。

- **接口方案**：如果你把实现逻辑写在外部的另一个普通类里，那个类是没法直接访问 `Child` 的 `private` 私有属性的。
- **内部类方案**：内部类就像是外部类的“影子”。它可以**无视任何权限修饰符**，直接拿取外部类的 `private` 数据。这种“亲密无间”的合作是接口无法提供的。



**5. 什么时候该用谁？（决策树）**

为了帮你理清思路，我们可以做个简单的判断：

| **需求场景**                         | **推荐方案**         | **理由**                                |
| ------------------------------------ | -------------------- | --------------------------------------- |
| 只是想给类加个标签（如：会跑、会跳） | **实现接口**         | 简单直接，符合 Java 标准。              |
| 需要继承某个现成的、有具体属性的类   | **内部类继承**       | 接口存不了数据，实现不了状态继承。      |
| 想要隐藏复杂的实现细节，不让外人看   | **内部类 + private** | 接口方法默认必须是 `public`，无法隐藏。 |
| 多个父类里有重名方法需要分别处理     | **内部类**           | 接口无法区分同名方法。                  |

**接口**像是一个**“法律合同”**，规定了你必须履行哪些职责； \**内部类\**则像是你雇佣的**“专业外包团队”**，它们带着自己的工具（属性）和经验（方法实现）进驻你的公司，替你处理那些复杂的、私密的、且可能互相冲突的任务。





## Java三大特性：封装、继承和多态

在谈 Java 面向对象的时候，不得不提到面向对象的三大特征：[封装](https://javabetter.cn/oo/encapsulation.html)、[继承](https://javabetter.cn/oo/extends-bigsai.html)、[多态](https://javabetter.cn/oo/polymorphism.html)。三大特征紧密联系而又有区别，合理使用继承能大大减少重复代码，**提高代码复用性。**

### 1）封装

封装从字面上来理解就是包装的意思，专业点就是信息隐藏，**是指利用抽象将数据和基于数据的操作封装在一起，使其构成一个不可分割的独立实体**。

数据被保护在类的内部，尽可能地隐藏内部的实现细节，只保留一些对外接口使之与外部发生联系。

其他对象只能通过已经授权的操作来与这个封装的对象进行交互。也就是说用户是无需知道对象内部的细节（当然也无从知道），但可以通过该对象对外的提供的接口来访问该对象。

使用封装有 4 大好处：

- 1、良好的封装能够减少耦合。
- 2、类内部的结构可以自由修改。
- 3、可以对成员进行更精确的控制。
- 4、隐藏信息，实现细节。

首先我们先来看两个类:

①Husband.java

```java
public class Husband {

    /*
     * 对属性的封装
     * 一个人的姓名、性别、年龄、妻子都是这个人的私有属性
     */
    private String name ;
    private String sex ;
    private int age ;
    private Wife wife;

    /*
     * setter()、getter()是该对象对外开发的接口
     */
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public void setWife(Wife wife) {
        this.wife = wife;
    }
}
```

②Wife.java

```java
public class Wife {
    private String name;
    private int age;
    private String sex;
    private Husband husband;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public void setHusband(Husband husband) {
        this.husband = husband;
    }

    public Husband getHusband() {
        return husband;
    }

}
```

可以看得出， Husband 类里面的 wife 属性是没有 `getter()`的，同时 Wife 类的 age 属性也是没有 `getter()`方法的。

没有哪个女人愿意别人知道她的年龄。

所以封装把一个对象的属性私有化，同时提供一些可以被外界访问的属性的方法，如果不想被外界方法，我们大可不必提供方法给外界访问。

但是如果一个类没有提供给外界任何可以访问的方法，那么这个类也没有什么意义了。

比如我们将一个房子看做是一个对象，里面有漂亮的装饰，如沙发、电视剧、空调、茶桌等等都是该房子的私有属性，但是如果我们没有那些墙遮挡，是不是别人就会一览无余呢？没有一点儿隐私！

因为存在那个遮挡的墙，我们既能够有自己的隐私而且我们可以随意的更改里面的摆设而不会影响到外面的人。

但是如果没有门窗，一个包裹的严严实实的黑盒子，又有什么存在的意义呢？所以通过门窗别人也能够看到里面的风景。所以说门窗就是房子对象留给外界访问的接口。

通过这个我们还不能真正体会封装的好处。现在我们从程序的角度来分析封装带来的好处。如果我们不使用封装，那么该对象就没有 `setter()`和 `getter()`，那么 Husband 类应该这样写：

```java
public class Husband {
    public String name ;
    public String sex ;
    public int age ;
    public Wife wife;
}
```

我们应该这样来使用它：

```java
Husband husband = new Husband();
husband.age = 30;
husband.name = "张三";
husband.sex = "男";    //貌似有点儿多余
```

但是哪天如果我们需要修改 Husband，例如将 age 修改为 String 类型的呢？你只有一处使用了这个类还好，如果你有几十个甚至上百个这样地方，你是不是要改到崩溃。如果使用了封装，我们完全可以不需要做任何修改，只需要稍微改变下 Husband 类的 `setAge()`方法即可。

```java
public class Husband {

    /*
     * 对属性的封装
     * 一个人的姓名、性别、年龄、妻子都是这个人的私有属性
     */
    private String name ;
    private String sex ;
    private String age ;    /* 改成 String类型的*/
    private Wife wife;

    public String getAge() {
        return age;
    }

    public void setAge(int age) {
        //转换即可
        this.age = String.valueOf(age);
    }

    /** 省略其他属性的setter、getter **/

}
```

其他的地方依然这样引用( `husband.setAge(22)` )保持不变。

到了这里我们确实可以看出，**封装确实可以使我们更容易地修改类的内部实现，而无需修改使用了该类的代码**。

我们再看这个好处：**封装可以对成员变量进行更精确的控制**。

还是那个 Husband，一般来说我们在引用这个对象的时候是不容易出错的，但是有时你迷糊了，写成了这样：

```java\
Husband husband = new Husband();
husband.age = 300;
```

也许你是因为粗心写成了这样，你发现了还好，如果没有发现那就麻烦大了，谁见过 300 岁的老妖怪啊！

但是使用封装我们就可以避免这个问题，我们对 age 的访问入口做一些控制(setter)如：

```java
public class Husband {

    /*
     * 对属性的封装
     * 一个人的姓名、性别、年龄、妻子都是这个人的私有属性
     */
    private String name ;
    private String sex ;
    private int age ;    /* 改成 String类型的*/
    private Wife wife;

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        if(age > 120){
            System.out.println("ERROR：error age input....");    //提示錯誤信息
        }else{
            this.age = age;
        }

    }

    /** 省略其他属性的setter、getter **/

}
```

上面都是对 setter 方法的控制，其实通过封装我们也能够对对象的出口做出很好的控制。例如性别在数据库中一般都是以 1、0 的方式来存储的，但是在前台我们又不能展示 1、0，这里我们只需要在 `getter()`方法里面做一些转换即可。

```java
public String getSexName() {
    if("0".equals(sex)){
        sexName = "女";
    }
    else if("1".equals(sex)){
        sexName = "男";
    }
    return sexName;
}
```

在使用的时候我们只需要使用 sexName 即可实现正确的性别显示。同理也可以用于针对不同的状态做出不同的操作。

```java
public String getCzHTML(){
    if("1".equals(zt)){
        czHTML = "<a href='javascript:void(0)' onclick='qy("+id+")'>启用</a>";
    }
    else{
        czHTML = "<a href='javascript:void(0)' onclick='jy("+id+")'>禁用</a>";
    }
    return czHTML;
}
```

### 2）继承

#### 01、什么是继承

**继承**（英语：inheritance）是面向对象软件技术中的一个概念。它使得**复用以前的代码非常容易。**

Java 语言是非常典型的面向对象的语言，在 Java 语言中**继承就是子类继承父类的属性和方法，使得子类对象（实例）具有父类的属性和方法，或子类从父类继承方法，使得子类具有父类相同的方法**。

我们来举个例子：动物有很多种，是一个比较大的概念。在动物的种类中，我们熟悉的有猫(Cat)、狗(Dog)等动物，它们都有动物的一般特征（比如能够吃东西，能够发出声音），不过又在细节上有区别（不同动物的吃的不同，叫声不一样）。

在 Java 语言中实现 Cat 和 Dog 等类的时候，就需要继承 Animal 这个类。继承之后 Cat、Dog 等具体动物类就是子类，Animal 类就是父类。

<div align="center">
  <img src="../assets/javaAssets/10.extendsPic.png" width="55%">
</div>

#### 02、为什么需要继承

你可能会问**为什么需要继承**？

如果仅仅只有两三个类，每个类的属性和方法很有限的情况下确实没必要实现继承，但事情并非如此，事实上一个系统中往往有很多个类并且有着很多相似之处，比如猫和狗同属动物，或者学生和老师同属人。各个类可能又有很多个相同的属性和方法，这样的话如果每个类都重新写不仅代码显得很乱，代码工作量也很大。

这时继承的优势就出来了：可以直接使用父类的属性和方法，自己也可以有自己新的属性和方法满足拓展，父类的方法如果自己有需求更改也可以重写。这样**使用继承不仅大大的减少了代码量，也使得代码结构更加清晰可见**。

<div align="center">
  <img src="../assets/javaAssets/10.extendsPic2.png" width="55%">
</div>

所以这样从代码的层面上来看我们设计这个完整的 Animal 类是这样的：

```java
class Animal
{
    public int id;
    public String name;
    public int age;
    public int weight;

    public Animal(int id, String name, int age, int weight) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.weight = weight;
    }
    //这里省略get set方法
    public void sayHello()
    {
        System.out.println("hello");
    }
    public void eat()
    {
        System.out.println("I'm eating");
    }
    public void sing()
    {
        System.out.println("sing");
    }
}
```

而 Dog，Cat，Chicken 类可以这样设计：

```java
class Dog extends Animal//继承animal
{
    public Dog(int id, String name, int age, int weight) {
        super(id, name, age, weight);//调用父类构造方法
    }
}
class Cat extends Animal{

    public Cat(int id, String name, int age, int weight) {
        super(id, name, age, weight);//调用父类构造方法
    }
}
class Chicken extends Animal{

    public Chicken(int id, String name, int age, int weight) {
        super(id, name, age, weight);//调用父类构造方法
    }
    //鸡下蛋
    public void layEggs()
    {
        System.out.println("我是老母鸡下蛋啦，咯哒咯！咯哒咯！");
    }
}
```

各自的类继承 Animal 后可以直接使用 Animal 类的属性和方法而不需要重复编写，各个类如果有自己的方法也可很容易地拓展。



#### 03、继承的分类

**单继承**

<div align="center">
  <img src="../assets/javaAssets/10.extendsPic3.png" width="55%">
</div>



单继承，一个子类只有一个父类，如我们上面讲过的 Animal 类和它的子类。**单继承在类层次结构上比较清晰，但缺点是结构的丰富度有时不能满足使用需求**。

**多继承**



<div align="center">
  <img src="../assets/javaAssets/10.extendsPic4.png" width="55%">
</div>

多继承，一个子类有多个直接的父类。这样做的好处是子类拥有所有父类的特征，**子类的丰富度很高，但是缺点就是容易造成混乱**。下图为一个混乱的例子。

<div align="center">
  <img src="../assets/javaAssets/10.extendsPic5.png" width="55%">
</div>

Java 虽然不支持多继承，但是 Java 有三种实现多继承效果的方式，**分别是**内部类、多层继承和实现接口。

[内部类](https://javabetter.cn/oo/inner-class.html)可以继承一个与外部类无关的类，保证了内部类的独立性，正是基于这一点，可以达到多继承的效果。

**多层继承：\**子类继承父类，父类如果还继承其他的类，那么这就叫\**多层继承**。这样子类就会拥有所有被继承类的属性和方法。

<div align="center">
  <img src="../assets/javaAssets/10.extendsPic6.png" width="55%">
</div>

[实现接口](https://javabetter.cn/oo/interface.html)无疑是满足多继承使用需求的最好方式，一个类可以实现多个接口满足自己在丰富性和复杂环境的使用需求。

类和接口相比，**类就是一个实体，有属性和方法，而接口更倾向于一组方法**。举个例子，就拿斗罗大陆的唐三来看，他存在的继承关系可能是这样的：

<div align="center">
  <img src="../assets/javaAssets/10.extendsPic7.png" width="75%">
</div>

#### 04、如何实现继承

**extends 关键字**

在 Java 中，类的继承是单一继承，也就是说一个子类只能拥有一个父类，所以**extends**只能继承一个类。其使用语法为

```java
class 子类名 extends 父类名{}
```

例如 Dog 类继承 Animal 类，它是这样的：

```java
class Animal{} //定义Animal类
class Dog extends Animal{} //Dog类继承Animal类
```

子类继承父类后，就拥有父类的非私有的**属性和方法**。如果不明白，请看这个案例，在 IDEA 下创建一个项目，创建一个 test 类做测试，分别创建 Animal 类和 Dog 类，Animal 作为父类写一个 sayHello()方法，Dog 类继承 Animal 类之后就可以调用 sayHello()方法。具体代码为：

```java
class Animal {
    public void  sayHello()//父类的方法
    {
        System.out.println("hello,everybody");
    }
}
class Dog extends Animal//继承animal
{ }
public class test {
    public static void main(String[] args) {
       Dog dog=new Dog();
       dog.sayHello();
    }
}
```

点击运行的时候 Dog 子类可以直接使用 Animal 父类的方法。

