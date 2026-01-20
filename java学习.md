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



<img src="assets/javaAssets/2basicDataType.png" width="65%">



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





#### 拓展:包装类

在 Java 中，数据被分成了两大家族：**基本数据类型**（Primitive Types）和**包装类**（Wrapper Classes）。

简单来说，包装类就是将基本数据类型“包装”起来，使其具有**对象**的特征。

| **基本类型**                      | **对应的包装类**                          |
| --------------------------------- | ----------------------------------------- |
| `byte` / `short` / `int` / `long` | `Byte` / `Short` / **`Integer`** / `Long` |
| `float` / `double`                | `Float` / `Double`                        |
| `char`                            | **`Character`**                           |
| `boolean`                         | `Boolean`                                 |

**1.为什么需要设计包装类?**

既然已经有了性能极高的“基本数据类型”，为什么还要费劲搞出一套“包装类”呢？核心原因有三个：

**① 为了让基本类型“面向对象”**

Java 的口号是“万物皆对象”，但基本类型（`int`, `double` 等）打破了这个规则。

- **集合框架不支持基本类型：** 比如你常用的 `ArrayList`、`HashMap`。它们只能存储对象（`Object`）。如果你想存一串数字，你不能写 `ArrayList<int>`，必须写 `ArrayList<Integer>`。
- **泛型限制：** 泛型（`T`）只能代表引用类型，不支持基本类型。



**② 提供了更多实用的工具方法**

基本类型只是一个单纯的“数字”，而包装类像是一个“工具箱”。

- **类型转换：** 比如把字符串转成数字 `Integer.parseInt("123")`。
- **进制转换：** `Integer.toHexString(255)` 转成十六进制。
- **范围查询：** `Integer.MAX_VALUE` 告诉你 `int` 最大能存多少。



**③ 处理 `null` 的需求（非常关键）**

- **基本类型：** 必须有一个值（比如 `int` 默认是 $0$）。
- **包装类：** 作为对象，它可以是 `null`。
- **应用场景：** 在数据库里，一个“年龄”字段可能是空的。如果你用 `int` 接收，它会变成 $0$（误导）；如果你用 `Integer`，它就是 `null`，准确表达了“未知”。



**为什么基本类型不需要 `new`？**

基本数据类型（如 `int a = 10;`）是 Java 为了**性能**而保留的特例。

- **存储位置：** 基本类型的值直接存储在**栈（Stack）**中。
- **分配效率：** 栈内存的分配是极快的。当你声明 `int a = 10` 时，JVM 只是在栈里划出 4 个字节的空间，把数字 `10` 丢进去。
- **不是对象：** 它们没有方法（你不能写 `a.toString()`），也没有复杂的类结构。因此，它们不需要通过 `new` 在堆中创建复杂的对象结构。



**为什么包装类可以（且曾经必须）用 `new`？**

包装类本质上是一个普通的 **Java 类**。既然是类，它就遵循对象的规则：

- **存储位置：** 对象存储在**堆（Heap）**中。
- **创建方式：** 在 Java 的规则里，创建一个存放在堆上的对象，标准做法就是使用 `new` 关键字（调用构造函数）。
- **功能强大：** 包装类提供了很多基本类型没有的功能。
  - **集合支持：** Java 的集合（如 `ArrayList`）只能存对象，不能存 `int`。所以你必须用 `ArrayList<Integer>`。
  - **包含方法：** 你可以调用 `Integer.parseInt("123")` 等内置方法。
  - **可以为 null：** 基本类型 `int` 必须有值（默认 0），但包装类 `Integer` 可以是 `null`，这在数据库开发中非常有用（表示“无数据”）。



**为什么现在不推荐 `new` 了？**

正如之前看到的报错，现在 Java 强烈建议不要写 `new Integer(10)`，而是写 `Integer.valueOf(10)` 或直接 `Integer i = 10`。

这是因为：

1. **`new` 每次都强制开辟新内存：** 即使两个 `new Integer(10)` 值一样，它们也是堆上两个不同的垃圾。
2. **缓存池优化：** `Integer.valueOf(10)` 会优先从缓存池拿现成的对象（-128 到 127 之间）。这比 `new` 快得多，也省内存`Integer.valueOf(300)` 缓存池里没有，会自动 `new` 



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



### 运算符

#### 位运算符

![4.calculaterCharacter](assets/javaAssets/4.calculaterCharacter.png)



```java
int a = 60, b = 13;
System.out.println("a 的二进制：" + Integer.toBinaryString(a)); // 111100
System.out.println("b 的二进制：" + Integer.toBinaryString(b)); // 1101

int c = a & b;
System.out.println("a & b：" + c + "，二进制是：" + Integer.toBinaryString(c));

c = a | b;
System.out.println("a | b：" + c + "，二进制是：" + Integer.toBinaryString(c));

c = a ^ b;
System.out.println("a ^ b：" + c + "，二进制是：" + Integer.toBinaryString(c));

c = ~a;
System.out.println("~a：" + c + "，二进制是：" + Integer.toBinaryString(c));

c = a << 2;
System.out.println("a << 2：" + c + "，二进制是：" + Integer.toBinaryString(c));

c = a >> 2;
System.out.println("a >> 2：" + c + "，二进制是：" + Integer.toBinaryString(c));

c = a >>> 2;
System.out.println("a >>> 2：" + c + "，二进制是：" + Integer.toBinaryString(c));
```



#### 前自增和后自增

虽然在代码中 `i++` 和 `++i` 看起来只是位置不同，但在 **JVM 内部的执行过程**（也就是在**栈**中的操作顺序）是有本质区别的。

简单来说：

- **前自增 (`++i`)**：**先加 1，后赋值/使用**。（先改变自己，再参与运算）
- **后自增 (`i++`)**：**先赋值/使用，后加 1**。（先参与运算，后改变自己）



从内存中**局部变量表**和**操作数栈**的角度来看看它们发生了什么：

**前自增 (`++i`)**

1. 变量直接在局部变量表中加 $1$。
2. 将加 $1$ 后的新值压入操作数栈。
3. **结果：** 你拿到的是增加后的值。

**后自增 (`i++`)**

1. 先将变量当前的旧值压入操作数栈。
2. 变量在局部变量表中加 $1$。
3. **结果：** 你在当前表达式中拿到的还是旧值，但变量本身已经偷偷变大了。



```java
public static void main(String[] args) {
    // 案例 1：前自增
    int a = 10;
    int b = ++a; 
    System.out.println("a = " + a); // a = 11
    System.out.println("b = " + b); // b = 11 (b 拿到了加完之后的值)

    // 案例 2：后自增
    int x = 10;
    int y = x++;
    System.out.println("x = " + x); // x = 11
    System.out.println("y = " + y); // y = 10 (y 拿到了加之前的旧值)
}
```





#### &|与&&||

**位运算符（&, |）** 和 **短路逻辑运算符（&&, ||）**

**1. 核心区别：短路效应 (Short-Circuit)**

这是它们最重要的区别。

**&& 和 || （短路运算符）**

它们非常“聪明”。如果左边的表达式已经能决定最终结果，它们就**不再计算右边的表达式**。

- **`&&` (短路与)：** 如果左边为 `false`，结果必定为 `false`，所以**右边不再执行**。
- **`||` (短路或)：** 如果左边为 `true`，结果必定为 `true`，所以**右边不再执行**。

**& 和 | （逻辑/位运算符）**

它们比较“死板”。无论左边的表达式结果是什么，**两边的表达式都会被完整计算**。



**2. 为什么要用短路运算符？（安全性）**

在 Java 开发中，短路运算符 `&&` 经常被用来防止 **空指针异常 (NullPointerException)**。

**代码对比：**

Java

```java
String s = null;

// ✅ 使用 && (安全)
if (s != null && s.length() > 0) {
    // 因为 s != null 为 false，&& 直接跳过右边，程序不会报错
}

// ❌ 使用 & (程序崩溃)
if (s != null & s.length() > 0) {
    // 即使 s != null 为 false，它依然会去计算 s.length()
    // 此时会对 null 调用方法，直接抛出 NullPointerException！
}
```



**3. 它们还可以做位运算**

这是 `&` 和 `|` 的另一副面孔：当操作数不是 `boolean` 而是 `int` 等数字时，它们变成了**位运算符**。

- **`&` (按位与)：** 只有两个二进制位都是 1，结果才是 1。
- **`|` (按位或)：** 只要有一个二进制位是 1，结果就是 1。

**例子：**

```java
int a = 5; // 二进制: 0101
int b = 3; // 二进制: 0011

int resultAnd = a & b; // 结果: 0001 (即 1)
int resultOr = a | b;  // 结果: 0111 (即 7)
```

*注意：`&&` 和 `||` 只能用于 `boolean` 类型，不能用于数字的位运算。*



**4. 总结对比表**

| **运算符** | **名称**  | **适用类型**      | **是否短路**      | **常用场景**                   |
| ---------- | --------- | ----------------- | ----------------- | ------------------------------ |
| **`&&`**   | 短路与    | 仅 `boolean`      | **是**            | 条件判断（最常用，安全高效）   |
| **`&`**    | 逻辑/位与 | `boolean` 或 数字 | 否                | 位运算、必须执行两边逻辑的情况 |
| **`        |           | `**               | 短路或            | 仅 `boolean`                   |
| **`        | `**       | 逻辑/位或         | `boolean` 或 数字 | 否                             |



**建议**

在写业务逻辑（比如 `if` 语句）时，**请始终优先使用 `&&` 和 `||`**。这不仅能提高运行效率，还能避免很多潜在的逻辑错误。



### 流程控制语句

#### switch 语句

switch 语句用来判断变量与多个值之间的相等性。变量的类型可以是：

- byte、short、char、int：基本整数类型。
- String：[字符串](https://javabetter.cn/string/immutable.html)类型。
- 枚举类型：自定义的[枚举](https://javabetter.cn/basic-extra-meal/enum.html)类型。
- 包装类：如 Byte、Short、Character、Integer。

来看一下 switch 语句的格式：

```java
switch(变量) {    
case 可选值1:    
 // 可选值1匹配后执行的代码;    
 break;  // 该关键字是可选项
case 可选值2:    
 // 可选值2匹配后执行的代码;    
 break;  // 该关键字是可选项
......    
    
default: // 该关键字是可选项     
 // 所有可选值都不匹配后执行的代码 
}
```

- 变量可以有 1 个或者 N 个值。
- 值类型必须和变量类型是一致的，并且值是确定的。
- 值必须是唯一的，不能重复，否则编译会出错。
- break 关键字是可选的，如果没有，则执行下一个 case，如果有，则跳出 switch 语句。
- default 关键字也是可选的。



示例

```java
int age = 20;
switch (age) {
    case 20 :
        System.out.println("上学");
        break;
    case 24 :
        System.out.println("苏州工作");
        break;
    case 30 :
        System.out.println("洛阳工作");
        break;
    default:
        System.out.println("未知");
        break; // 可省略
}
```



当两个值要执行的代码相同时，可以把要执行的代码写在下一个 case 语句中，而上一个 case 语句中什么也没有，这种写法在 Java 中被称为 **"Case Fall-through"（case 穿透）**。来看一下示例：

```java
String name = "沉默王二";
switch (name) {
        //......
case "沉默王二":
case "沉默王三":
    System.out.println("乒乓球爱好者");
    break;
```

- 当 `name` 是 `"沉默王二"` 时：程序找到了第一个入口。由于这一行后面没有任何代码，它会**直接“掉”到**下一行。

- 进入 `"沉默王三"` 的地盘：程序继续执行，直到它看到了 `System.out.println` 和最重要的 `break`。

- **结果**：无论是王二还是王三，最终执行的都是同一段逻辑。



这种“堆叠” case 的写法非常实用，主要优点是：

- **减少重复代码**：当多个条件对应同一个结果时（比如：周一到周五都输出“工作日”），你不需要写五次同样的 `println`。
- **逻辑清晰**：一眼就能看出哪些分类是属于同一组的。



 **现代 Java 的新姿势 (Java 12+)**

如果觉得这种堆叠方式还是有点啰嗦，现代 Java 提供了一种更简洁的 **Switch Expression（开关表达式）**，使用 `->` 符号，效果完全一样但更直观：

```java
// 这是 Java 12 之后推荐的写法
switch (name) {
    case "詹姆斯" -> System.out.println("篮球运动员");
    case "穆里尼奥" -> System.out.println("足球教练");
    case "沉默王二", "沉默王三" -> System.out.println("乒乓球爱好者"); // 直接逗号隔开
    default -> throw new IllegalArgumentException("名字没有匹配项");
}
```

*这种写法不需要写 `break`，因为它默认就是不穿透的。*



#### 枚举(enumerate)

```java
public class SwitchEnumDemo {
    public enum PlayerTypes {
        TENNIS,
        FOOTBALL,
        BASKETBALL,
        UNKNOWN
    }

    public static void main(String[] args) {
        System.out.println(createPlayer(PlayerTypes.BASKETBALL));
    }

    private static String createPlayer(PlayerTypes playerType) {
        switch (playerType) {
            case TENNIS:
                return "网球运动员费德勒";
            case FOOTBALL:
                return "足球运动员C罗";
            case BASKETBALL:
                return "篮球运动员詹姆斯";
            case UNKNOWN:
                throw new IllegalArgumentException("未知");
            default:
                throw new IllegalArgumentException(
                        "运动员类型: " + playerType);

        }
    }
}
```

但 switch 不支持 long、float、double 类型，这是因为：

- long 是 64 位整数，不在 switch 一开始设计的范围内（32 位的 int 在大多数情况下就够用了）。
- float 和 double 是浮点数，浮点数的比较不如整数简单和直接，存在精度误差。





#### For循环

**1.for-each**

```java
for(元素类型 元素 : 数组或集合){  
// 要执行的代码
}
```

例如:

```java
String [] strs = {"neil","leo","Nio"};

        for(String str : strs){
            System.out.println(str);
        }
```



**2.无限 for 循环**

```java
for(;;){
    System.out.println("xxx");
}
```

一旦运行起来，就停不下来了，除非强制停止。



#### continue

当我们需要在 for 循环或者 （do）while 循环中立即跳转到下一个循环时，就可以使用 continue 关键字，通常用于跳过指定条件下的循环体，如果循环是嵌套的，仅跳过当前循环。

来个示例：

```java
for (int i = 1; i <= 10; i++) {
    if (i == 5) {
        // 使用 continue 关键字
        continue;// 5 将会被跳过
    }
    System.out.println(i);
}
```







## 数组与字符串

### 数组

#### 数组的声明与初始化

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

上面这行代码中使用了 new 关键字，这就意味着数组的确是一个对象，只有对象的创建才会用到 new 关键字，[基本数据类型](https://javabetter.cn/basic-grammar/basic-data-type.html)是不用的（基本数据的包装类型是可以 new 的，包装类型就是对象）。然后，我们需要在方括号中指定数组的长度。

这时候，数组中的每个元素都会被初始化为默认值，int 类型的就为 0，Object 类型的就为 null。 不同数据类型的默认值不同，可以参照[之前的文章](https://javabetter.cn/basic-grammar/basic-data-type.html)。

另外，还可以使用大括号的方式，直接初始化数组中的元素：

```java
int anOtherArray[] = new int[] {1, 2, 3, 4, 5};
```

这时候，数组的元素分别是 1、2、3、4、5，索引依次是 0、1、2、3、4，长度是 5。

#### 数组的常用操作

过索引来访问数组的元素

```java
anArray[0] = 10;
```

变量名，加上中括号，加上元素的索引，就可以访问到数组，通过“=”操作符可以对元素进行赋值。

如果索引的值超出了数组的界限，就会抛出 `ArrayIndexOutOfBoundException`。由于数组的索引是从 0 开始，所以最大索引为 `length - 1`，不要使用超出这个范围内的索引访问数组，否则就会抛出数组越界的异常了。

比如说你声明了一个大小为 10 的数组，你用索引 10 来访问数组，就会抛出这个异常。因为数组的索引是从 0 开始的，所以数组的最后一个元素的索引是 `length - 1`，也就是 9。

当数组的元素非常多的时候，逐个访问数组就太辛苦了，所以需要通过遍历的方式。

第一种，使用 for 循环：

```java
int anOtherArray[] = new int[] {1, 2, 3, 4, 5};
for (int i = 0; i < anOtherArray.length; i++) {
    System.out.println(anOtherArray[i]);
}
```

通过 length 属性获取到数组的长度，然后从 0 开始遍历，就得到了数组的所有元素。

第二种，使用 for-each 循环：

```java
int anOtherArray[] = new int[] {1, 2, 3, 4, 5};

for (int i:anOtherArray){
    System.out.println(i);
}
```

如果不需要关心索引的话（意味着不需要修改数组的某个元素），使用 for-each 遍历更简洁一些。当然，也可以使用 while 和 do-while 循环

#### **可变参数与数组**

在 Java 中，**可变参数（Varargs，全称 Variable Arguments）** 是从 Java 5 开始引入的一个特性。它允许你在调用方法时，传入**任意数量**的参数（甚至是 0 个）。

简单来说，可变参数让你的方法变得更“灵活”，不再被死板的参数个数所限制。

在方法的声明中，在**数据类型**后面加上三个点（`...`）：

```java
public void 方法名(数据类型... 参数名) {
    // 方法体
}
```

假设我们要写一个求和的方法，如果不确定用户会传几个数字进来：

```java
public class VarargsExample {
    public static void main(String[] args) {
        // 可以传 2 个参数
        System.out.println(sum(1, 2));
        
        // 可以传 5 个参数
        System.out.println(sum(10, 20, 30, 40, 50));
        
        // 甚至可以不传参数
        System.out.println(sum()); 
    }

    // int... 代表可以接收 0 到多个 int 类型的参数
    public static int sum(int... numbers) {
        int total = 0;
        // 在方法内部，numbers 被当作【数组】来处理
        for (int n : numbers) {
            total += n;
        }
        return total;
    }
}
```



 **可变参数的本质：数组**

当你看到 `int... numbers` 时，Java 编译器在底层其实偷偷地把它转换成了 `int[] numbers`。

- **调用时**：编译器会自动把你传入的多个参数打包成一个数组。
- **执行时**：方法体内部完全按照操作数组的方式来处理这些参数。



**使用规则（必须遵守）**

虽然可变参数很方便，但为了避免歧义，Java 对它有两条严格的限制：

**① 一个方法只能有一个可变参数**

不能写成 `public void test(int... a, String... b)`，这会导致编译器不知道哪些参数属于 a，哪些属于 b。

**② 可变参数必须是参数列表的最后一个**

如果方法有多个参数，可变参数必须放在最后。

- **❌ 错误写法**：`public void test(int... nums, String name)` （编译器会报错）
- **✅ 正确写法**：`public void test(String name, int... nums)`

**原因**：Java 在匹配参数时是从左往右看的。如果可变参数在前面，它会“吞掉”后面所有的参数，导致后面的 `name` 拿不到值。

------

**5. 可变参数 vs 数组参数**

你可能会问：“那我直接定义一个数组参数 `sum(int[] numbers)` 不行吗？”

区别在于**调用的便利性**：

- **数组参数**：调用时必须手动创建一个数组，如 `sum(new int[]{1, 2, 3})`。
- **可变参数**：直接写 `sum(1, 2, 3)` 即可，代码更简洁、可读性更好。





#### 数组与 List

在 Java 中，数组与 List 关系非常密切。List 封装了很多常用的方法，方便我们对集合进行一些操作，而如果直接操作数组的话，有很多不便，因为数组本身没有提供这些封装好的操作，所以有时候我们需要把数组转成 List。

> List 会在[集合框架](https://javabetter.cn/collection/arraylist.html)一节详细介绍，这里先来个开胃菜，方便大家回头过来复盘。

最原始的方式，就是通过遍历数组的方式，一个个将数组添加到 List 中。

```java
int[] anArray = new int[] {1, 2, 3, 4, 5};

List<Integer> aList = new ArrayList<>();
for (int element : anArray) {
    aList.add(element);
}
```

更优雅的方式是通过 [Arrays 类](https://javabetter.cn/common-tool/arrays.html)（戳链接了解详情）的 `asList()` 方法：

```JAVA
List<Integer> aList = Arrays.asList(anArray);
```

不过需要注意的是，Arrays.asList 的参数需要是 Integer 数组，而 anArray 目前是 int 类型。

可以这样写：

```JAVA
List<Integer> aList1 = Arrays.asList(1, 2, 3, 4, 5);
```

或者换另外一种方式

```JAVA
List<Integer> aList = Arrays.stream(anArray).boxed().collect(Collectors.toList());
```

这又涉及到了 [Java 流](https://javabetter.cn/java8/stream.html)的知识，戳链接了解。

还有一个需要注意的是，Arrays.asList 方法返回的 ArrayList 并不是 `java.util.ArrayList`，它其实是 Arrays 类的一个内部类：

```java
private static class ArrayList<E> extends AbstractList<E>
        implements RandomAccess, java.io.Serializable{}
```

如果需要添加元素或者删除元素的话，需要把它转成 `java.util.ArrayList`。

```java
new ArrayList<>(Arrays.asList(anArray));
```

Java 8 新增了 [Stream 流](https://javabetter.cn/java8/stream.html)的概念，这就意味着我们也可以将数组转成 Stream 进行操作。

```java
String[] anArray = new String[] {"沉默王二", "一枚有趣的程序员", "好好珍重他"};
Stream<String> aStream = Arrays.stream(anArray);
```



#### 数组的排序与查找

如果想对数组进行排序的话，可以使用 Arrays 类提供的 `sort()` 方法。

- 基本数据类型按照升序排列
- 实现了 Comparable 接口的对象按照 `compareTo()` 的排序

来看第一个例子：

```java
int[] anArray = new int[] {5, 2, 1, 4, 8};
Arrays.sort(anArray);
```

排序后的结果如下所示：

```
[1, 2, 4, 5, 8]
```

<span style = "color:red">注意:此时`anArray` 变量指向的那块**堆内存区域**里的数据被彻底挪动了,所以执行了`Arrays.sort`后`anArray`内部就已经是[1,2,4,5,8]了</span>



来看第二个例子：

```sql
String[] yetAnotherArray = new String[] {"A", "E", "Z", "B", "C"};
Arrays.sort(yetAnotherArray, 1, 3,
                Comparator.comparing(String::toString).reversed());
```

只对 1-3 位置上的元素进行反序，所以结果如下所示：

```java
[A, Z, E, B, C]
```



------

<span style="color:teal">知识点</span>

`Arrays.sort()` 不仅能排整个数组，还能排**指定的区间**。

- **参数解析**：`Arrays.sort(数组, fromIndex, toIndex, ...)`
- **你的代码**：`1, 3` 表示从索引 `1` 开始，到索引 `3` **之前**结束。
- **规则**：**左闭右开** $[1, 3)$。也就是说，它只处理索引为 `1` 和 `2` 的元素，索引 `3` 及其之后的元素保持不动。



**`Comparator.comparing` (比较器)**

这是 Java 8 引入的函数式编程写法，用于定义**排序规则**。

- `String::toString`：这是一个**方法引用**(匿名内部类的终极版)。它告诉 Java：“请使用 String 对象的 `toString()` 方法返回的结果来进行比较。”
- 虽然对字符串来说直接比较就行，但这种写法在处理复杂对象（比如 `User::getName`）时非常强大。



**`.reversed()` (逆序排列)**

默认情况下，`Arrays.sort` 是**升序**（A -> Z）。

- 加上 `.reversed()` 后，排序规则反转，变成了**降序**（Z -> A）。

---



有时候，我们需要从数组中查找某个具体的元素，最直接的方式就是通过遍历的方式：

```sql
int[] anArray = new int[] {5, 2, 1, 4, 8};
for (int i = 0; i < anArray.length; i++) {
    if (anArray[i] == 4) {
        System.out.println("找到了 " + i);
        break;
    }
}
```

上例中从数组中查询元素 4，找到后通过 break 关键字退出循环。

如果数组提前进行了排序，就可以使用二分查找法，这样效率就会更高一些。`Arrays.binarySearch()` 方法可供我们使用，它需要传递一个数组，和要查找的元素。

```sql
int[] anArray = new int[] {1, 2, 3, 4, 5};
int index = Arrays.binarySearch(anArray, 4);

//或者也可以
int [] arrayList = {7,5,1,4,2,3,6};
Arrays.sort(arrayList);

int index = Arrays.binarySearch(arrayList,2);
System.out.println(index);
```

“除了一维数组，还有[二维数组](https://javabetter.cn/array/double-array.html)，比如说用二维数组打印一下杨辉三角，我们下一节会讲。”



#### 数组的复制

有时候我们需要将一个数组的值复制到另外一个数组当中，那就会涉及到数组复制的知识点。

在 [String 类](https://javabetter.cn/string/string-source.html)（讲完数组就会讲）中其实会经常遇到数组复制，比如说 `substring()` 方法。

```sql
public String substring(int beginIndex) {
    return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
}
```

如果你要求的子字符串是从索引 `0` 开始的（即截取整个字符串），那么**直接返回当前对象自己 (`this`)**

**`new String(...)`**：它会根据起始位置和长度，**拷贝**出一份新的字符数组，并创建一个**全新的 String 对象**。

**1. 拆解这个构造方法**

那么把`new String(value, beginIndex, subLen)` 逻辑公式化：

- **起始位置**：`beginIndex`
- **结束位置**：`beginIndex + subLen`
- **截取的总长度**：`subLen`

**举个例子：** 假设 `value`（原字符串数组）是 `{'h', 'e', 'l', 'l', 'o'}`，你想截取 `"ell"`。

- `beginIndex` 就是 `1`（指向 'e'）。
- `subLen` 就是 `3`（因为你要拿 'e', 'l', 'l' 这三个字符）。
- 对应的物理结束位置就是 `1 + 3 = 4`（不包含第 4 位的 'o'）。



**2.内存中到底发生了什么？**

当你调用这个构造方法时，JVM 内部会执行一个非常核心的指令：`System.arraycopy`。

我们可以把这个过程可视化：

1. **开辟空间**：在堆（Heap）中分配一块新的、大小刚好为 `subLen` 的内存区域。
2. **数据搬运**：
   - 源头：原数组 `value`。
   - 起点：`beginIndex`。
   - 目的地：新开辟的数组。
   - 搬运量：`subLen`。
3. **独立封装**：这个新的数组被封装进一个新的 `String` 对象中返回。



**3. 为什么要强调“新空间”？**

它是“开辟一个新的空间”。这意味着：

- **修改互不影响**：虽然 `String` 本身不可变，但这种物理隔离确保了即使你通过反射等特殊手段修改了原数组，截取出来的字符串也不会受到影响。
- **内存回收**：原字符串（大数组）和子字符串（小数组）在内存中是**两块独立的内存**。如果原大字符串不再被使用了，垃圾回收器（GC）可以放心地把那块大内存回收掉，而不会因为还剩一个小小的子字符串而无法回收。



<span style="color:red">你可能会问：*“直接让新字符串指向旧数组的一部分，不是更快更省内存吗？”*</span>

这就是**Java 历史上的一个大坑**：

- **旧版 Java (6 以前) 的做法**：就是共享。
  - **后果**：如果你有一个 $1\text{GB}$ 大小的字符串，你只想截取其中的 $2$ 个字符。因为新字符串还引用着那个 $1\text{GB}$ 的大数组，导致这 $1\text{GB}$ 的内存**永远无法被回收**。这叫内存泄漏（Memory Leak）。
- **新版 Java 的做法**：宁愿多花一点点时间去**复制**，也要保证截取出来的字符串是**完全独立**的。
  - **结果**：一旦原大字符串不再使用，它那 $1\text{GB}$ 的内存就能立刻被垃圾回收器（GC）收走。



那么对于数组的复制这里特别看一下;

```sql
public String(char value[], int offset, int count) {
    this.value = Arrays.copyOfRange(value, offset, offset+count);
}
```

其中的 `Arrays.copyOfRange()` 方法就是用来复制数组的，我们在讲 [Arrays 类](https://javabetter.cn/common-tool/arrays.html#_2-copyofrange)的时候就会讲到。

它底层调用的是 `System.arraycopy()` 方法，这个方法是一个 [native 方法](https://javabetter.cn/oo/native-method.html)，它是用 C/C++ 实现的，效率非常高。



System.arraycopy 方法的定义如下所示：

```java
public static native void arraycopy(Object src,  int  srcPos,
                                    Object dest, int destPos,
                                    int length);
```



| **参数**      | **含义**              | **你的代码 (第一次复制)** | **你的代码 (第二次复制)**   |
| ------------- | --------------------- | ------------------------- | --------------------------- |
| **`src`**     | **源数组** (从哪拷)   | `array1`                  | `array2`                    |
| **`srcPos`**  | **源数组起始位置**    | `0`                       | `0`                         |
| **`dest`**    | **目标数组** (拷到哪) | `mergedArray`             | `mergedArray`               |
| **`destPos`** | **目标数组存放起点**  | `0`                       | **`array1.length`** (重要!) |
| **`length`**  | **拷贝多少个元素**    | `array1.length`           | `array2.length`             |



用法如下所示：

```java
int[] array1 = {1, 2, 3};
int[] array2 = {4, 5, 6};

// 创建一个新数组，长度为两个数组长度之和
int[] mergedArray = new int[array1.length + array2.length];

// 复制第一个数组到新数组
System.arraycopy(array1, 0, mergedArray, 0, array1.length);
System.out.println(Arrays.toString(mergedArray));

// 复制第二个数组到新数组
System.arraycopy(array2, 0, mergedArray, array1.length, array2.length);
System.out.println(Arrays.toString(mergedArray));
```

输出结果如下所示：

```java
[1, 2, 3, 0, 0, 0]
[1, 2, 3, 4, 5, 6]
```

当然了，我们也可以使用循环来完成数组的复制：



```java
int[] array1 = {1, 2, 3};
int[] array2 = {4, 5, 6};

// 创建一个新数组，长度为两个数组长度之和
int[] mergedArray = new int[array1.length + array2.length];

// 复制第一个数组到新数组
int index = 0;
for (int element : array1) {
    mergedArray[index++] = element;
}

// 复制第二个数组到新数组
for (int element : array2) {
    mergedArray[index++] = element;
}
```





#### 数组越界

在我们进行数组操作的时候，最容易遇到的一个问题就是数组越界，也就是 ArrayIndexOutOfBoundsException [异常](https://javabetter.cn/exception/gailan.html)。

```java
int[] anArray = new int[] {1, 2, 3, 4, 5};
System.out.println(anArray[5]);
```

上面这段代码就会抛出数组越界的异常，因为数组的索引是从 0 开始的，所以最大索引为 `length - 1`，也就是 4，所以当我们使用 5 作为索引的时候，就会抛出异常。

所以在操作数组之前，一定要注意索引的范围。



### 二维数组

二维数组实际上在开发中用的不多，也很简单，就从一维到二维

#### 1.什么是二维数组

二维数组是一种数据类型，可以存储多行和多列的数据。它由一系列的行和列组成，每个元素都可以通过一个行索引和列索引来访问。例如，一个3行4列的二维数组可以表示为以下形式：

```java
array = [
  [a, b, c, d],
  [e, f, g, h],
  [i, j, k, l]
]
```

在这个例子中，第一行有4个元素，第二行有4个元素，第三行有4个元素，每个元素都有一个行索引和一个列索引。例如，元素 $array[2][3]$ 是第2行第3列的元素，它的值是 g。

使用二维数组可以有效地存储和处理表格数据，如矩阵、图像、地图等等。



#### 2.创建二维数组

要在 Java 中创建二维数组，你必须指定要存储在数组中的数据类型，后跟两个方括号和数组的名称。

语法如下所示：

```java
data_type[][] array_name;
```

让我们看一个代码示例:

```java
int[][] oddNumbers = { {1, 3, 5, 7}, {9, 11, 13, 15} };
```



#### 3.访问二维数组中的元素

我们可以使用两个方括号来访问二维中的元素。

第一个表示我们要从中访问元素的数组，而第二个表示我们要访问的元素索引。

让我们用一个例子来简化上面的解释：

```java
int[][] oddNumbers = { {1, 3, 5, 7}, {9, 11, 13, 15} };

System.out.println(oddNumbers[0][0]);
// 1
```

在上面的示例中，`oddNumbers` 数组中有两个数组——`{1, 3, 5, 7}` 和 `{9, 11, 13, 15}`。

第一个数组——`{1, 3, 5, 7}`——用 0 表示。

第二个数组——`{9, 11, 13, 15}`——用 1 表示。

第一个数组是 0，第二个是 1，第三个是 2，依此类推。

因此，要访问第一个数组中的项目，我们将 0 分配给第一个方括号。由于我们试图访问数组中的第一项，我们将使用它的索引，即 0：`oddNumbers[0][0]`。

让我们进一步分解它。

这是访问元素的代码：`oddNumbers[?][?]`。

我在两个方括号中都加上了问号——随着进展填写它们。

假设我们要访问第二个数组中的元素，我们的代码将如下所示：`oddNumbers[1][?]`。

现在我们要在第二个数组（`{9, 11, 13, 15}`）中尝试访问其中一个元素。就像一维数组一样，每个元素都有一个从零开始的索引。

因此，要访问第三个元素 `13`，我们将其索引号传递给第二个方括号：`oddNumbers[1][2]`。

来看这样一个例子：

```java
int[][] oddNumbers = { {1, 3, 5, 7}, {9, 11, 13, 15}, {17, 19, 21, 23} };
```

我们的目标是访问第三个数组中的 21。访问方式仍然通过问号来表示：`oddNumbers[?][?]`。

我们首先给第一个问号一个指向要访问的特定数组的值。

数组 0 => `{1, 3, 5, 7}`

数组 1 => `{9, 11, 13, 15}`

数组 2 => `{17, 19, 21, 23}`

我们要查找的数字在第三个数组中，所以是：`oddNumbers[2][?]`。

第二个方括号的值将指向要访问的元素。为此，我们必须指定元素的索引。以下是该数组中的索引：

17 => 索引 0

19 => 索引 1

21 => 索引 2

23 => 索引 3

21 的索引为 2，因此我们可以将其添加到第二个方括号：`oddNumbers[2][2]`。当你将其打印到控制台时，将会打印出 21。

代码如下所示:

```java
int[][] oddNumbers = { {1, 3, 5, 7}, {9, 11, 13, 15}, {17, 19, 21, 23} };

System.out.println(oddNumbers[2][2]);
// 21
```

你可以使用嵌套循环，遍历二维数组中的所有项目。这是一个例子：





#### 4.打印杨辉三角(TODO)





### 打印数组

之前提到过，数组也是一个对象，但 Java 中并未明确的定义这样一个类。

因此数组也就没有机会覆盖 `Object.toString()` 方法。如果尝试直接打印数组的话，输出的结果并不是我们预期的结果。

那怎么打印数组呢？



#### **1.Why不能直接打印数组?**

来看这样一个例子:

```JAVA
String [] cmowers = {"沉默","王二","一枚有趣的程序员"};
System.out.println(cmowers);
```

程序打印的结果是：

```sql
[Ljava.lang.String;@3d075dc0
```

`[Ljava.lang.String;` 表示字符串数组的 Class 名，@ 后面的是十六进制的 hashCode——这样的打印结果太“人性化”了，一般人表示看不懂！为什么会这样显示呢？查看一下 `java.lang.Object` 类的 `toString()` 方法就明白了。

```java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

再次证明，数组虽然没有显式定义成一个类，但它的确是一个对象，继承了祖先类 Object 的所有方法。

**解析:**

`getClass()` 和 `hashCode()` 都是 **`java.lang.Object`** 类中定义的实例方法。

- 如果数组只是纯粹的内存块（像 C 语言的数组那样），它根本无法调用 `getClass()`。
- 既然数组可以运行这段代码并返回结果，说明数组**继承**了 `Object`。在 Java 中，**只有类（Class）才能继承，只有对象（Object）才是类的实例**。

`getClass()` 的职责是：返回该对象所属的 **类（Class）**。

如果数组没有类，`getClass()` 应该返回 `null`,但它返回了 `[I`。这说明 JVM 在内存里为数组**动态创建了一个类**，名字就叫 `[I`。



看源码的后半部分：`Integer.toHexString(hashCode())`。

- **基本类型**（如普通的 `int` 变量）是没有哈希码的。
- **对象**之所以有哈希码，是因为在内存中，每个对象的头部（Object Header）都存有身份信息。
- 当你调用数组的 `toString()`，它能算出这一串十六进制数字，说明数组在内存里不仅仅存了数据，还拥有一个**对象头**，用来存储哈希码等信息。



我们可以把这个证明过程写成一个逻辑题：

1. **已知**：`toString()` 是 `Object` 类的方法。
2. **现象**：数组可以成功调用 `toString()` 并输出 `[I@7a81197d`。
3. **推论 A**：数组一定继承自 `Object`（否则无法调用）。
4. **推论 B**：数组一定属于某个类（否则 `getClass().getName()` 拿不到 `[I`）。
5. **推论 C**：数组在内存中是一个带有结构的实体（否则 `hashCode()` 算不出值）。

**结论**：数组是一个由 JVM 动态创建类、并由该类实例化的**标准对象**。



#### **2.Why**数组不单独定义为类?

那为什么数组不单独定义一个类来表示呢？就像字符串 String 类那样呢?

**1. 性能：越接近硬件，速度越快**

`String` 是一个由 `char[]` 或 `byte[]` 包装出来的“高级工具”，而数组是计算机内存的**物理映射**。

- **物理内存**：本质上就是一排连续的格子。
- **Java 数组**：直接对应这一排格子。如果你使用 `new Array<Int>()` 这种普通的类，那么每次访问元素 `get(0)` 都需要经过：**方法调用 -> 栈帧入栈 -> 逻辑处理 -> 栈帧出栈**。
- **特殊设计**：现在的数组语法 `arr[0]` 在编译后会直接变成一条**底层的机器指令**。这种“特权”让数组成为 Java 中速度最快的数据结构。



**2. 泛型与基本类型的矛盾**

如果定义一个 `class Array`，它会面临一个巨大的难题：**基本类型（Primitive Types）怎么办？**

- 在 Java 中，`String`、`ArrayList` 等普通类只能存储**引用类型**（对象）。
- 如果数组是一个普通的类，那么 `int[]` 可能会变成 `Array<Integer>`。这会导致每一个数字都要被包装成一个 `Integer` 对象，产生巨大的内存浪费。
- 通过将数组设计为“特殊的类”，JVM 可以直接创建 `int[]`、`double[]` 这种直接存放原始数字的内存块，而不需要对象包装。



**3. 数组是“所有类的基石”**

这是一个“先有鸡还是先有蛋”的问题。

- **String 类** 的内部是用**数组**实现的。
- **ArrayList 类** 的内部也是用**数组**实现的。
- **JVM 加载类** 的过程也需要**数组**。

以`String`为例:

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```

如果数组本身也是一个用 Java 代码定义的“普通类”，那么在加载这个“数组类”之前，JVM 拿什么来存储这个类的信息呢？将数组设计为由 JVM 直接控制的“特殊对象”，就避免了这种循环依赖，它成为了 Java 世界诞生的**最底层建筑**。



**4. 语法的简洁性**

想象一下这两种写法的区别：

- **普通类设计**： `Array<Integer> arr = new Array<>(5);` `arr.set(0, 10);` `int val = arr.get(0);`
- **特殊设计（现状）**： `int[] arr = new int[5];` `arr[0] = 10;`

`[]` 符号是程序员最熟悉的符号，它能一眼让人看出这是在操作一块连续的内存。



#### 3.Why数组可以存储基本类型

<span style="color:green">既然数组是对象（Object），而对象通常只能持有引用，那为什么它能直接把 `int`, `double` 这种“非对象”的基本类型塞进肚子里呢？</span>

这正是 Java 数组最特殊、也最强大的地方。我们可以从以下三个核心逻辑来理解：



**1. 数组的“容器性质”是由 JVM 定制的**

普通的 Java 类（比如 `ArrayList`）是由 Java 代码定义的。根据 Java 的泛型规则，它们只能处理 `Object` 及其子类。

但**数组不是由 Java 代码定义的，它是 JVM 的“亲儿子”**。 JVM 在设计数组时，直接在底层内存模型上做了两种分类：

1. **引用类型数组 (Reference Arrays)**：内存里存的是一排地址（指针）。
2. **基本类型数组 (Primitive Arrays)**：内存里直接存的是一排原始数值。

当你写 `int[]` 时，JVM 会在内存中开辟一块**连续的、特定大小**的物理空间。

- 如果是 `int[]`，每个格子就是固定的 **32 位（4字节）**。
- 如果是 `double[]`，每个格子就是固定的 **64 位（8字节）**。



**2. 为了绝对的性能：拒绝“包装”**

如果数组不能存储基本类型，会发生什么灾难？

假设你想存 100 万个数字：

- **如果必须存对象**：Java 必须把每个 `int` 包装成 `Integer` 对象。每个 `Integer` 对象除了存数字，还要存对象头（约 12-16 字节）。这会多消耗 3 到 4 倍的内存，且 CPU 读取时还要经过“解引用”的操作，速度极慢。
- **现在的特殊设计**：`int[]` 直接把 100 万个数字紧挨着排在内存里。CPU 访问时，可以通过“基地址 + 偏移量”瞬间定位。

**这就是为什么数组被设计成“特殊的类”：它必须拥有直接操作物理内存的特权，而不受普通类“只能存对象”的规则限制。**



**3. “对象”只是数组的皮囊，“内存块”才是它的内核**

虽然我们说“数组是对象”，但这更多是指它的**行为**（可以调用 `toString`，有 `getClass`）。

但在**存储层面**，它本质上是一个**带有元数据的内存块**。

- **元数据**（对象头）：记录了这个数组是什么类型（`int[]`）、长度是多少。
- **数据区**：紧随其后，直接存放原始数据。

| **特性**     | **数组 (int[])**                | **集合 (ArrayList<Integer>)**          |
| ------------ | ------------------------------- | -------------------------------------- |
| **存储内容** | **直接存原始数值** (1, 2, 3...) | **存对象的引用** (指向 Integer 的地址) |
| **内存布局** | 连续、紧凑                      | 离散（指针指向堆的不同地方）           |
| **访问速度** | 极快（硬件级别）                | 较慢（需要寻址和拆箱）                 |
| **实现者**   | **JVM 虚拟机直接实现**          | Java 开发者编写的类                    |

数组，是 Java 世界里唯一能在大规模处理数据时，避开对象包装、直接触摸原始字节的地方。



#### 4.stream 流打印 Java 数组

“我们来看第一种打印数组的方法，使用时髦一点的[Stream 流](https://javabetter.cn/java8/stream.html)。”

第一种形式：

```java
Arrays.asList(cmowers).stream().forEach(s -> System.out.println(s));
```

第二种形式：

```java
Stream.of(cmowers).forEach(System.out::println);
```

第三种形式：

```java
Arrays.stream(cmowers).forEach(System.out::println);
```



没错，这三种方式都可以轻松胜任本职工作，并且显得有点高大上，毕竟用到了 Stream，以及 [lambda 表达式](https://javabetter.cn/java8/Lambda.html)。

后面会提及 Stream流,Lambda表达式;



#### 5.for 循环打印 Java 数组

“当然了，也可以使用传统的方式，for 循环。甚至 for-each 也行。”

```java
for(int i = 0; i < cmowers.length; i++){
    System.out.println(cmowers[i]);
}

for (String s : cmowers) {
    System.out.println(s);
}
```



#### 6.Arrays 工具类打印 Java 数组

[上一篇](https://javabetter.cn/common-tool/arrays.html)在讲 Arrays 工具类的时候，提到过另外一种方法 `Arrays.toString()` ,有些人认为 `Arrays.toString()` 是打印数组的最佳方式，没有之一。

`Arrays.toString()` 可以将任意类型的数组转成字符串，包括基本类型数组和引用类型数组。该方法有多种重载形式。

使用 `Arrays.toString()` 方法来打印数组很优雅



#### 7.POJO 的打印规约



POJO 指的是那些**极其纯净**的 Java 对象。它不继承任何复杂的框架类（比如 `Servlet` 或 `EntityBean`），也不实现奇怪的接口。

POJO 是用最纯粹的 Java 代码定义的数据模型；而重写 POJO 的 `toString` 方法，则是为了给这堆纯粹的数据穿上一件“可读”的外衣，让 `Arrays.toString()` 这种原生扫描仪能扫出我们看得懂的人话。



一个标准的 POJO 通常长这样：

1. **私有属性**（Private fields）
2. **公共的 Getter/Setter 方法**
3. **一个无参构造函数**

**比喻：** 如果复杂的框架对象是“全副武装的特种兵”，那么 POJO 就是“穿白 T 恤的普通老百姓”。它只负责**存数据**。



**为什么打印 POJO 很重要？**

当有一个 `Student` 类的 POJO 数组时：

```java
Student[] students = { new Student("张三", 20), new Student("李四", 22) };
```

如果你直接用 `Arrays.toString(students)` 打印，结果往往是： `[com.test.Student@7a81197d, com.test.Student@5ca881b5]`

**这是因为 POJO 默认继承了 `Object` 类的 `toString()`。** 正如我们之前讨论的，它只会打印“类名+哈希码”。



#### 如何让 POJO 打印得“漂亮”？

为了能一眼看出 POJO 里的内容（比如名字和年龄），我们通常会在 POJO 类里**重写（Override）** `toString()` 方法。



```java
public class Student {
    private String name;
    private int age;

    // 重写后，打印出来的就是内容而不是地址
    @Override
    public String toString() {
        return "Student{name='" + name + "', age=" + age + "}";
    }
}
```



### 字符串源码解读

字符串的细节特别多，什么[字符串常量池](https://javabetter.cn/string/constant-pool.html)、[字符串不可变性](https://javabetter.cn/string/immutable.html)、[字符串拼接](https://javabetter.cn/string/join.html)、字符串长度限制等等,我们慢慢来说

#### String 类的声明

```JAVA
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
}
```



第一，String 类是 [final](https://javabetter.cn/oo/final.html) 的，意味着它不能被子类[继承](https://javabetter.cn/oo/encapsulation-inheritance-polymorphism.html)。这些知识我们讲面向对象编程的时候都会讲到

第二，String 类实现了 [Serializable 接口](https://javabetter.cn/io/Serializbale.html)，意味着它可以[序列化](https://javabetter.cn/io/serialize.html)

第三，String 类实现了 [Comparable 接口](https://javabetter.cn/basic-extra-meal/comparable-omparator.html)，意味着最好不要用‘==’来[比较两个字符串是否相等](https://javabetter.cn/string/equals.html)，而应该用 `compareTo()` 方法去比较。

因为 == 是用来比较两个对象的地址，这个在讲[字符串比较](https://javabetter.cn/string/equals.html)的时候会详细讲。如果只是说比较字符串内容的话，可以使用 String 类的 equals 方法，源码和注释如下所示：

```java
public boolean equals(Object anObject) {
    // 检查是否是同一个对象的引用，如果是，直接返回 true
    if (this == anObject) {
        return true;
    }
    // 检查 anObject 是否是 String 类的实例
    if (anObject instanceof String) {
        String anotherString = (String) anObject; // 将 anObject 强制转换为 String 类型
        int n = value.length; // 获取当前字符串的长度
        // 检查两个字符串长度是否相等
        if (n == anotherString.value.length) {
            char v1[] = value; // 当前字符串的字符数组
            char v2[] = anotherString.value; // 另一个字符串的字符数组
            int i = 0; // 用于遍历字符数组的索引
            // 遍历比较两个字符串的每个字符
            while (n-- != 0) {
                // 如果在任何位置字符不同，则返回 false
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            // 所有字符都相同，返回 true
            return true;
        }
    }
    // 如果 anObject 不是 String 类型或长度不等，则返回 false
    return false;
}
```

第四，[String 和 StringBuffer、StringBuilder](https://javabetter.cn/string/builder-buffer.html) 一样，都实现了 CharSequence 接口，所以它们仨属于近亲。由于 String 是不可变的，所以遇到[字符串拼接](https://javabetter.cn/string/join.html)的时候就可以考虑一下 String 的另外两个好兄弟，StringBuffer 和 StringBuilder，它俩是可变的。



#### String 底层为什么由 char 数组优化为 byte 数组

```java
private final char value[];
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