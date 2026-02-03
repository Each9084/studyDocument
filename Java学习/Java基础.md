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









