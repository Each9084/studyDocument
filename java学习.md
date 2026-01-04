# Java基础

## 基础语法

### Abstract

#### 1.abstract 是什么？

`abstract` 是 Java 中的一个关键字，用来声明**抽象类**和**抽象方法**。

**①. 抽象类**

- 用 `abstract` 修饰的类
- **不能直接创建对象**（不能实例化）
- 可以有抽象方法和普通方法

**②. 抽象方法**

- 用 `abstract` 修饰的方法
- **只有方法声明，没有方法体**（没有大括号和实现）
- 必须被子类实现（除非子类也是抽象类）



#### 2.为什么要使用 abstract？

主要目的是**代码设计和规范**：

**①. 定义规范框架**

```java
abstract class Animal {
    // 所有动物都会叫，但叫声不同
    public abstract void makeSound();
}
```

告诉子类：“你必须实现 makeSound 方法，但具体怎么叫你自己决定”



**②.代码复用**

抽象类可以包含具体方法，子类可以直接使用

```java
abstract class Animal {
    // 抽象方法
    public abstract void makeSound();
    
    // 具体方法 - 所有动物共享
    public void eat() {
        System.out.println("吃东西");
    }
}
```



**③. 实现多态**

```java
Animal dog = new Dog();  // 抽象类引用指向具体子类
Animal cat = new Cat();
```


------

### Byte

用于表示一个 8 位（1 字节）有符号整数。它的值范围是 -128（-2^7）到 127（2^7 - 1）。

由于 byte 类型占用的空间较小，它通常用于处理大量的数据，如文件读写、网络传输等场景，以节省内存空间。

```java
byte minByte = -128;
byte maxByte = 127
```

---



### 基本数据类型

Java 中的数据类型可分为 2 种：

1）**基本数据类型**:

基本数据类型是 Java 语言操作数据的基础，包括 boolean、char、byte、short、int、long、float 和 double，共 8 种。

2）**引用数据类型**。

除了基本数据类型以外的类型，都是所谓的引用类型。常见的有[数组](https://javabetter.cn/array/array.html)（对，没错，数组是引用类型，后面我们会讲）、class（也就是[类](https://javabetter.cn/oo/object-class.html)），以及[接口](https://javabetter.cn/oo/interface.html)（指向的是实现接口的类的对象）。



![2basicDataType](../studyDoc/assets/javaAsstes/2basicDataType.png)

[变量](https://javabetter.cn/oo/var.html)可以分为局部变量、成员变量、静态变量。

当变量是局部变量的时候，必须得先初始化，否则编译器不允许你使用它。拿 



#### 引用数据类型

基本数据类型在作为成员变量和静态变量的时候有默认值，引用数据类型也有的（学完数组&字符串，以及面向对象编程后会更加清楚，这里先简单过一下）。

[String](https://javabetter.cn/string/immutable.html) 是最典型的引用数据类型.

是不是很好奇，为什么[数组](https://javabetter.cn/array/array.html)和[接口](https://javabetter.cn/oo/interface.html)也是引用数据类型啊？

先来看数组：

```java
int [] arrays = {1,2,3};
System.out.println(arrays);
```

arrays 是一个 int 类型的数组，对吧？打印结果如下所示：

```java
[I@723279cf
```

`[I` 表示数组是 int 类型的，@ 后面是十六进制的 hashCode——这样的打印结果太“人性化”了，一般人表示看不懂！为什么会这样显示呢？查看一下 `java.lang.Object` 类的 `toString()` 方法就明白了。

```java
public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
```



数组虽然没有显式定义成一个类，但它的确是一个对象，继承了祖先类 Object 的所有方法。那为什么数组不单独定义一个类来表示呢？就像字符串 String 类那样呢？

一个合理的解释是 Java 将其隐藏了。假如真的存在一个 Array.java，我们也可以假想它真实的样子，它必须要定义一个容器来存放数组的元素，就像 String 类那样。



### 堆（heap）和栈（stack）

堆是在程序运行时在内存中申请的空间（可理解为动态的过程）；切记，不是在编译时；因此，Java 中的对象就放在这里，这样做的好处就是：

> 当需要一个对象时，只需要通过 new 关键字写一行代码即可，当执行这行代码时，会自动在内存的“堆”区分配空间——这样就很灵活。



栈，能够和处理器（CPU，也就是脑子）直接关联，因此访问速度更快。既然访问速度快，要好好利用啊！Java 就把对象的引用放在栈里。为什么呢？因为引用的使用频率高吗？

>  Java 在编译程序时，必须明确的知道存储在栈里的东西的生命周期，否则就没法释放旧的内存来开辟新的内存空间存放引用——空间就那么大，前浪要把后浪拍死在沙滩上啊。



![3heapAndStack](../studyDoc/assets/javaAssets/3heapAndStack.png)

| **特性**     | **栈（Stack）**                                      | **堆（Heap）**                           |
| ------------ | ---------------------------------------------------- | ---------------------------------------- |
| **存放内容** | 局部变量（基本类型）、方法调用信息、**对象的引用**。 | **所有对象（`new` 出来的）**、数组。     |
| **分配方式** | **自动分配**和释放。由 JVM 自动管理。                | **动态分配**。需要用 `new` 关键字创建。  |
| **生命周期** | 随着方法的调用结束而销毁。                           | 由垃圾回收器（GC）自动回收，生命周期长。 |
| **存取速度** | **快**。操作简单，像堆叠盘子。                       | **慢**。需要通过地址查找，有额外开销。   |
| **空间大小** | 较小，有限制。                                       | 较大，是程序的主要内存区域。             |



### Catch

用于捕获 try 语句中的[异常](https://javabetter.cn/exception/gailan.html)(具体情况后面会讨论)。在 try 块中可能会抛出异常，而在 catch 块中可以捕获这些异常并进行处理。catch 块可以有多个，每个 catch 块可以捕获特定类型的异常。在 catch 块中，可以根据需要进行异常处理，例如输出错误信息、进行日志记录、恢复程序状态等。

------

### 异常

#### 异常定义

异常是指中断程序正常执行的一个不确定的事件。当异常发生时，程序的正常执行流程就会被打断。一般情况下，程序都会有很多条语句，如果没有异常处理机制，前面的语句一旦出现了异常，后面的语句就没办法继续执行了。”

“有了异常处理机制后，程序在发生异常的时候就不会中断，我们可以对异常进行捕获，然后改变程序执行的流程。”



#### Exception  和 Error 的区别:

Error 的出现，意味着程序出现了严重的问题，而这些问题不应该再交给 Java 的异常处理机制来处理，程序应该直接崩溃掉，比如说 OutOfMemoryError，内存溢出了，这就意味着程序在运行时申请的内存大于系统能够提供的内存，导致出现的错误，这种错误的出现，对于程序来说是致命的。

Exception 的出现，意味着程序出现了一些在可控范围内的问题，我们应当采取措施进行挽救。



#### checked和unchecked异常

checked 异常（检查型异常）在源代码里必须显式地捕获或者抛出，否则编译器会提示你进行相应的操作(**编译器要求你必须处理的异常**)；

而 unchecked 异常（非检查型异常）就是所谓的运行时异常，通常是可以通过编码进行规避的，并不需要显式地捕获或者抛出(**编译器不强制你处理的异常**)

思维导图如下:

![1exception and error](F:\学习\studyDoc\assets\javaAssets\1exception and error.png)



首先，Exception 和 Error 都继承了 Throwable 类。换句话说，只有 Throwable 类（或者子类）的对象才能使用 throw 关键字抛出，或者作为 catch 的参数类型。

<span style = "color : orange;">面试中经常问到的一个问题是，NoClassDefFoundError 和 ClassNotFoundException 有什么区别？</span>

它们都是由于系统运行时找不到要加载的类导致的，但是触发的原因不一样。

- NoClassDefFoundError：程序在编译时可以找到所依赖的类，但是在运行时找不到指定的类文件，导致抛出该错误；原因可能是 jar 包缺失或者调用了初始化失败的类。
- ClassNotFoundException：当动态加载 Class 对象的时候找不到对应的类时抛出该异常；原因可能是要加载的类不存在或者类名写错了。

其次，像 IOException、ClassNotFoundException、SQLException 都属于 checked 异常；

像 RuntimeException 以及子类 ArithmeticException、ClassCastException、ArrayIndexOutOfBoundsException、NullPointerException，都属于 unchecked 异常。



注意打印异常堆栈信息的 `printStackTrace()` 方法，该方法会将异常的堆栈信息打印到标准的控制台下，如果是测试环境，这样的写法还 OK，如果是生产环境，这样的写法是不可取的，必须使用日志框架把异常的堆栈信息输出到日志系统中，否则可能没办法跟踪。

翻译过来就是在开发时可以用 `printStackTrace()` 看到错误，但真正上线运行的系统必须用日志记录异常，否则出问题时根本找不到原因。

---



<span style ="color : green" >🚩 争议点:</span>

checked 异常在业界是有争论的，它假设我们捕获了异常，并且针对这种情况作了相应的处理，但有些时候，根本就没法处理。

例如:

```java
try {
    Class.forName("com.example.MyClass");
} catch (ClassNotFoundException e) {
    e.printStackTrace();
}
```

如果真的出现 ClassNotFoundException，还能怎么处理？

难道你能动态去网上下载这个类？

难道你能“修复”classpath？

难道你能自动创建一个类？

难道你能提示用户去重启 JVM？

所以争议点在于checked 异常强迫开发者使用 try-catch，但实际上 catch 了也处理不了

--------------------------------

不过Checked 异常的价值取决于：
该异常是否是“能够被业务逻辑正常处理的可恢复错误”。

换句话说：

- 能靠代码恢复的 → 适合 checked
- 人类/操作系统才能解决 → 不适合 checked

这是关键。



✔ 有些 checked 异常是鸡肋（不能处理还强制你 try-catch）

如：

- ClassNotFoundException
- SQLException（大多数不可恢复）
- InterruptedException（必须 catch 很烦）

这些 catch 了你也干不了什么，只会污染代码。

------

✔ 有些 checked 异常是非常合理的（真的应该处理）

如：

- IOException（网络 + IO 操作）
- FileNotFoundException（可以让用户选择文件）
- SocketException（网络可重试）

这些异常确实是“可恢复”的。



**总之:**

Checked 异常适用于“可以预料并通过代码优雅恢复”的场景。
Unchecked 异常适用于“代码无法恢复、必须中断执行”的场景(这些异常的出现往往是 **程序逻辑错误** 或 **不可恢复的状态**，用 try-catch 捕获没有意义。)





#### try-catch-finally

值得注意的是如果一些代码确定不会抛出异常，就尽量不要把它包裹在 `try` 块里，因为加了异常处理的代码执行起来要比没有加的花费更多的时间。



一个 `try` 块后面可以跟多个 `catch` 块，用来捕获不同类型的异常并做相应的处理，当 try 块中的某一行代码发生异常时，之后的代码就不再执行，而是会跳转到异常对应的 catch 块中执行。

try-catch之后用`finally`用于执行任何必要的清理操作，无论 `try` 块中的代码是否发生异常，它都保证会被执行。



如果一个 try 块后面跟了多个与之关联的 catch 块，那么应该把特定的异常放在前面，通用型的异常放在后面，不然编译器会提示错误。举例来说。





**核心作用：资源的释放**

`finally` 最重要的用途是确保资源被正确释放，例如：

- 关闭文件流（`FileInputStream`, `FileWriter` 等）
- 关闭数据库连接（`Connection`, `Statement` 等）
- 释放网络连接
- 关闭 `Scanner` 对象（就像我们前面讨论的那样）

**执行保证**

`finally` 块的执行机制非常可靠，它会在以下任何情况发生后执行：

1. `try` 块正常执行完毕。
2. `try` 块抛出异常，并被匹配的 `catch` 块捕获。
3. `try` 块抛出异常，但没有被任何 `catch` 块捕获。
4. 在 `try` 或 `catch` 块中执行了 `return`、`break` 或 `continue` 语句（但 `finally` 会在这些跳转发生之前执行）。



使用 finally 块的时候需要遵守这些规则

- finally 块前面必须有 try 块，不要把 finally 块单独拉出来使用。编译器也不允许这样做。
- finally 块不是必选项，有 try 块的时候不一定要有 finally 块。
- 如果 finally 块中的代码可能会发生异常，也应该使用 try-catch 进行包裹。
- 即便是 try 块中执行了 return、break、continue 这些跳转语句，finally 块也会被执行。



---



#### 关于 throw 和 throws

1️⃣ `throw` —— **执行动作**,抛出一个具体的异常对象

**作用**：在方法体内部**抛出一个异常实例**，用于告诉 JVM 或调用者“出现错误了”。

当程序运行到一个逻辑上不应该继续执行的状态时（比如你的除数是 0，或者账户余额是负数），程序员可以使用 `throw` 来中断当前流程，并通知系统/调用者：“**这里出错了！**”

```java
public void divide(int a, int b) {
    if (b == 0) {
        throw new ArithmeticException("除数不能为0"); // 抛出异常对象
    }
    System.out.println(a / b);
}

```

**特点**：

1. `throw` 后面必须跟一个**异常对象**（如 `new ExceptionType()`）
2. 抛出异常后，当前方法会立即停止执行
3. 可以抛 **checked** 或 **unchecked** 异常



 📢 **比喻：**就像一个士兵发现敌人后，**拉响警报**（`throw`），并把记录有敌人位置、数量等信息的报告（异常对象）扔给指挥中心。



2️⃣**`throws` —— 事先声明,声明方法可能抛出的异常类型**

**作用**：告诉方法的调用者“这个方法可能会抛出这些异常，需要处理”。

`throws` 是给方法的**调用者**看的。它告诉调用者：“**我这个方法有风险，可能会抛出这些异常，你调用我的时候需要准备好处理它们。**” 它是 Java 强制要求的一种异常处理机制。

```java
public void readFile(String path) throws IOException {
    FileReader fr = new FileReader(path); // 可能抛出 IOException
}
```

**特点**：

1. 用在**方法声明上**
2. 后面跟异常类型，不是对象
3. 只能用于**checked 异常**，unchecked 异常可不写



⚠️ **比喻：** 就像在危险区域入口立一个**警告牌**（`throws`），告诉所有进入该区域的人：“这里可能会有地雷（异常），请做好处理准备。”



3️⃣ throw vs throws 的核心区别

| 对比点            | throw                | throws                                      |
| ----------------- | -------------------- | ------------------------------------------- |
| 用法位置          | 方法内部             | 方法声明                                    |
| 后面跟什么        | 异常对象实例         | 异常类型                                    |
| 是否终止方法      | 是，抛出异常立即终止 | 否，只是声明给调用者                        |
| checked/unchecked | 都可                 | 主要是 checked 异常，unchecked 可以不用声明 |

可以将 `throw` 理解为**制造**（或确认）问题，而 `throws` 是**报告**（或声明）问题。





## 数据与字符串

### 数组的声明与初始化

先来看第一种：

```java
int[] anArray;
```

再来看第二种：

```java
int anOtherArray[];
```



不同之处就在于中括号的位置，是跟在类型关键字的后面，还是跟在变量的名称的后面。前一种的使用频率更高一些，像 ArrayList 的源码中就用了第一种方式。

同样的，数组的初始化方式也有多种，最常见的是：

```java
int[] anArray = new int[10];
```







## 其他

### `Class`(大写C)和`class`(小写c)的区别

`class` —— Java 的关键字

作用：**定义一个类**

例如：

```java
public class Person {
}
```



`Class` —— Java 中代表“类”的类

每一个类在运行时都会对应一个 Class 对象。

举例：

```
Class<?> c = Person.class;
```

这里的 `Class` 表示 Java 运行时表示类的一种对象。

它可以干什么？

- 获取类名
- 反射创建对象
- 获取方法/字段/构造器
- 获取注解
- 动态加载类



🌟 打比方帮助理解

假设 **class 是蓝图（图纸）**
 而 **Class 是描述这张图纸的对象。**

🌱 `class`（小写）

写下：

```
class Dog { }
```

就像在纸上画了一个狗的模型蓝图。

🌳 `Class`（大写）

当程序运行时，JVM 会将这个蓝图加载成一个对象：

```
Class<Dog>
```

这是描述“Dog 这个类本身”的对象。



**🌟  两者最根本的区别**

| 名称      | 类型                   | 作用                         |
| --------- | ---------------------- | ---------------------------- |
| **class** | 关键字（keyword）      | 定义 Java 类                 |
| **Class** | java.lang 包中的一个类 | 表示“类的结构信息”，用于反射 |