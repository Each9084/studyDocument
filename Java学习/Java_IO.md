# 知识体系

写在前面,按照传输方式对 IO 进行了一个简单的分类

<img src="../assets/javaAssets/16.IO.png" width="55%" style="display: block; margin: 0 auto;">

Java 的设计者考虑得比较多，所以 IO 给人一种很乱的感觉，现在来梳理一下。

## 00、初识 Java IO

IO，即in和out，也就是输入和输出，指应用程序和外部设备之间的数据传递，常见的外部设备包括文件、管道、网络连接。

Java 中是通过流处理IO 的，那么什么是流？

流（Stream），是一个抽象的概念，是指一连串的数据（字符或字节），是以先进先出的方式发送信息的通道。

当程序需要读取数据的时候，就会开启一个通向数据源的流，这个数据源可以是文件，内存，或是网络连接。类似的，当程序需要写入数据的时候，就会开启一个通向目的地的流。这时候你就可以想象数据好像在这其中“流”动一样。

一般来说关于流的特性有下面几点：

- 先进先出：最先写入输出流的数据最先被输入流读取到。
- 顺序存取：可以一个接一个地往流中写入一串字节，读出时也将按写入顺序读取一串字节，不能随机访问中间的数据。（`RandomAccessFile`除外）
- 只读或只写：每个流只能是输入流或输出流的一种，不能同时具备两个功能，输入流只能进行读操作，对输出流只能进行写操作。在一个数据传输通道中，如果既要写入数据，又要读取数据，则要分别提供两个流。

### 01、传输方式划分

就按照那副思维导图来说吧。

传输方式有两种，字节和字符，那首先得搞明白字节和字符有什么区别，对吧？

字节（byte）是计算机中用来表示存储容量的一个计量单位，通常情况下，一个字节有 8 位（bit）。

字符（char）可以是计算机中使用的字母、数字、和符号，比如说 $A 1 $ 这些。

通常来说，一个字母或者一个字符占用一个字节，一个汉字占用两个字节。

<img src="../assets/javaAssets/16.shangtou-02.png" width=60%>

具体还要看字符编码，比如说在 UTF-8 编码下，一个英文字母（不分大小写）为一个字节，一个中文汉字为三个字节；在 Unicode 编码中，一个英文字母为一个字节，一个中文汉字为两个字节。

明白了字节与字符的区别，再来看字节流和字符流就会轻松多了。

字节流用来处理二进制文件，比如说图片啊、MP3 啊、视频啊。

字符流用来处理文本文件，文本文件可以看作是一种特殊的二进制文件，只不过经过了编码，便于人们阅读。

换句话说就是，字节流可以处理一切文件，而字符流只能处理文本。

看着很多,其实核心就是：

- 抽象类：

  - 字节流：`InputStream`、`OutputStream`
  - 字符流：`Reader`、`Writer`

- 核心方法：

  - 读取数据 → `read()`

  - 写出数据 → `write()`

永远记得一点**IO 类库再复杂，本质就是“读”和“写”两件事”**。



Java IO 中有两种流：

| 类型   | 类                         | 单位         | 作用                                     |
| ------ | -------------------------- | ------------ | ---------------------------------------- |
| 字节流 | InputStream / OutputStream | byte（字节） | 处理所有二进制数据，比如图片、视频、音频 |
| 字符流 | Reader / Writer            | char（字符） | 处理文本数据，比如字符串、文本文件       |

`Stream` → “流” → **字节流**

`Reader/Writer` → “读写文本” → **字符流**



**`InputStream` 类**

- `int read()`：读取数据

  > 你可能好奇，读取一个字节明明只要 8 位，为什么要用 32 位的 `int` 来接收？
  >
  > **原因:**
  >
  > `byte` 的范围是 -128 到 127。如果读到末尾，我们需要一个特殊值来表示 **“没了”**（通常是 -1）。
  >
  > 如果返回 `byte`，-1 可能是数据本身，也可能是结束符，这就搞混了。
  >
  > 所以用 `int`，用 **0~255** 表示正常字节，用 **-1** 表示结束。

- `int read(byte b[], int off, int len)`：从第 off 位置开始读，读取 `len` 长度的字节，然后放入数组 b 中

- `long skip(long n)`：跳过指定个数的字节

- `int available()`：返回可读的字节数

- `void close()`：关闭流，释放资源

**`OutputStream` 类**

- `void write(int b)`： 写入一个字节，虽然参数是一个 `int` 类型，但只有低 8 位才会写入，高 24 位会舍弃（这块后面再讲）

  > 这就像是把一头大象（32位 int）塞进一个小冰箱（8位存储单位）。
  >
  > Java 只取最右边的那一截（低 8 位），剩下的直接扔掉。所以你在传参时，要确保这个 `int` 的值在 0~255 之间，否则数据会“变形”。

- `void write(byte b[], int off, int len)`： 将数组 b 中的从 off 位置开始，长度为 `len` 的字节写入

- `void flush()`： 强制刷新，将缓冲区的数据写入

  > 为了快，数据通常先存在内存的“缓冲区”里。如果不 `flush`，数据可能一直憋在内存里，没真正进硬盘。

- `void close()`：关闭流

  > IO 资源是有限的。如果不关，文件就会被锁死，或者内存慢慢被耗尽。
  >
  > **职业习惯**：现在的 Java 流行使用 `try-with-resources` 语法，让程序自动帮你关。

**`Reader` 类**

- `int read()`：读取单个字符
- `int read(char cbuf[], int off, int len)`：从第 off 位置开始读，读取 `len` 长度的字符，然后放入数组 b 中
- `long skip(long n)`：跳过指定个数的字符
- `int ready()`：是否可以读了
- `void close()`：关闭流，释放资源

**`Writer` 类**

- `void write(int c)`： 写入一个字符
- `void write( char cbuf[], int off, int len)`： 将数组 `cbuf` 中的从 off 位置开始，长度为 `len` 的字符写入
- `void flush()`： 强制刷新，将缓冲区的数据写入
- `void close()`：关闭流

理解了上面这些方法，基本上 IO 的灵魂也就全部掌握了。

字节流和字符流的区别：

- 字节流一般用来处理图像、视频、音频、PPT、Word等类型的文件。字符流一般用于处理纯文本类型的文件，如TXT文件等，但不能处理图像视频等非文本文件。用一句话说就是：字节流可以处理一切文件，而字符流只能处理纯文本文件。
- 字节流本身没有缓冲区，缓冲字节流相对于字节流，效率提升非常高。而字符流本身就带有缓冲区，缓冲字符流相对于字符流效率提升就不是那么大了。

以写文件为例，我们查看字符流的源码，发现确实有利用到缓冲区：

```java
// 声明一个 char 类型的数组，用于写入输出流
private char[] writeBuffer;

// 定义 writeBuffer 数组的大小，必须 >= 1
private static final int WRITE_BUFFER_SIZE = 1024;

// 写入给定字符串中的一部分到输出流中
public void write(String str, int off, int len) throws IOException {
    // 使用 synchronized 关键字同步代码块，确保线程安全
    synchronized (lock) {
        char cbuf[];
        // 如果 len <= WRITE_BUFFER_SIZE，则使用 writeBuffer 数组进行写入
        if (len <= WRITE_BUFFER_SIZE) {
            // 如果 writeBuffer 为 null，则创建一个大小为 WRITE_BUFFER_SIZE 的新 char 数组
            if (writeBuffer == null) {
                writeBuffer = new char[WRITE_BUFFER_SIZE];
            }
            cbuf = writeBuffer;
        } else {    // 如果 len > WRITE_BUFFER_SIZE，则不永久分配非常大的缓冲区
            // 创建一个大小为 len 的新 char 数组
            cbuf = new char[len];
        }
        // 将 str 中的一部分（从 off 开始，长度为 len）拷贝到 cbuf 数组中
        // 这一行是关键！它不是用 for 循环一个个复制，
        // 而是调用底层系统级的拷贝（类似 System.arraycopy），速度极快。
        str.getChars(off, (off + len), cbuf, 0);
        // 将 cbuf 数组中的数据写入输出流中
        write(cbuf, 0, len);
    }
}
```

> 这段代码的核心逻辑在于：**如何处理你要写入的那串字符？**
>
> #### 情况 A：小批量货物（$\le 1024$ 字符）
>
> 如果这次要写的字符串很短（不超过 1024 个字符），程序会选择 **“复用旧纸箱”**。
>
> - 它会检查 `writeBuffer`（那个成员变量）。如果还没买（为 null），就买一个固定的 1024 大小的箱子。
> - **好处**：你写一万次短字符串，它都只占用那一个 1024 的空间。这极大地减轻了 **GC（垃圾回收）** 的压力，不用频繁地扔旧箱子、造新箱子。
>
> #### 情况 B：大批量货物（$> 1024$ 字符）
>
> 如果要写的字符串太长了，程序会选择 **“定制临时大箱子”**。
>
> - 它直接 `new char[len]`。
> - **原因**：如果为了偶尔一次的大数据去永久分配一个巨型 `writeBuffer`，那这个大箱子会一直霸占内存不释放（常驻内存），非常浪费。所以对于大件，咱们“用完即焚”。

这段代码是 Java IO 类库中的 `OutputStreamWriter` 类的 write 方法，可以看到缓冲区的大小是 1024 个 char。1024 字符（2KB 内存）是一个经过大量实践得出的“甜点位”。

我们再以文件的字符流和字节流来做一下对比，代码差别很小。

```java
// 字节流
try (FileInputStream fis = new FileInputStream("input.txt");
     FileOutputStream fos = new FileOutputStream("output.txt")) {
    byte[] buffer = new byte[1024];
    int len;
    while ((len = fis.read(buffer)) != -1) {
        fos.write(buffer, 0, len);
    }
} catch (IOException e) {
    e.printStackTrace();
}

// 字符流
try (FileReader fr = new FileReader("input.txt");
     FileWriter fw = new FileWriter("output.txt")) {
    char[] buffer = new char[1024];
    int len;
    while ((len = fr.read(buffer)) != -1) {
        fw.write(buffer, 0, len);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

核心结构：`try-with-resources`

你会发现这两段代码的 `try` 后面都跟着一对圆括号 `()`。这是 Java 7 引入的“自动关窗”语法。

- **以前**：你必须在 `finally` 块里手动写 `fis.close()`，还要防备 `close` 时又抛异常。
- **现在**：只要括号里的类实现了 `AutoCloseable` 接口，不管代码是正常结束还是中途崩溃，Java 都会**百分之百保证**帮你把流关掉。这解决了“资源泄漏”的头号难题。

这两段代码对比

| **特性**     | **第一段：字节流 (FileInputStream)**             | **第二段：字符流 (FileReader)**                              |
| ------------ | ------------------------------------------------ | ------------------------------------------------------------ |
| **搬运单位** | **`byte` (8位)**                                 | **`char` (16位)**                                            |
| **操作对象** | **一切文件**（图片、视频、exe）                  | **仅限纯文本**（txt, html, json）                            |
| **编码处理** | 原始搬运，不转码。                               | 会根据系统或指定的编码（如 UTF-8）进行**解码**。             |
| **风险**     | 搬运文本时如果遇到多字节字符，可能会在中间断开。 | 搬运非文本（如图片）时，会尝试按字符解码，导致**文件损坏**。 |



#### 实战演示

在`src/assets/`路径下创建test.txt文件,内容为:你好，Java IO!

以及随便的一个jpg图片picture.jpg

创建`src/assets/output`文件夹

##### 1)字节流

```java
public class TextCharLab {
    public static void main(String[] args) {
        try(
                //FileReader：打开通往 test.txt 的“读信通道”。
                FileReader fr = new FileReader("src/assets/test.txt");
                //FileWriter：打开通往 copy_test.txt 的“写信通道”。如果这个文件不存在，它会自动帮你创建一个。
        FileWriter fw = new FileWriter("src/assets/output/copy_test.txt")){
        int data;

        while((data = fr.read())!=-1){
            System.out.print((char)data);
            fw.write(data);
        }
            System.out.println("\n文本复制成功！检查 copy_test.txt，中文依然完美。");
        }catch (IOException e){
            e.printStackTrace();
        }

    }
}
```

在普通代码里，`try` 后面直接跟大括号 `{}`。但在 IO 操作中，你看到的这种 `try (声明资源)` 叫做 **“带资源的 try 语句” (Try-with-resources)**。

IO 流（文件、网络、数据库连接）是非常占用系统资源的。用完必须关掉。

- **老办法**：你得在 `finally` 块里手动写 `fr.close()`。如果忘了写，文件就会被一直锁死，程序跑久了内存就爆了。
- **这种写法（新办法）**：圆括号里声明的 `fr` 和 `fw` 就像进了“自动关窗模式”。**只要代码离开了 `try` 的范围（无论是正常跑完，还是中途报错炸了），Java 都会保证百分之百帮你调用 `close()` 方法。**

你可以理解为一个保险丝业务,而且很适合字符流



**`fr.read()`**：每次从文件读**一个字符**。

**为什么返回 `int`？**：因为当读到文件末尾时，它需要返回 `-1` 来告诉你“没货了”。

**`(char)data`**：这是强转。如果不转，`System.out` 打印出来的就是字符对应的数字编号（ASCII/Unicode 码）。

**`fw.write(data)`**：读到一个，立刻往新文件写一个。

**`IOException`**：IO 操作是“高危动作”。如果文件被删了、硬盘满了、或者你没有读取权限，程序都会抛出这个异常。

**`e.printStackTrace()`**：打印出错误在代码的哪一行，方便你修 Bug。

这段代码采用的是 **“单挑模式”**：读一个字，写一个字。

- **优点**：代码极其直观，适合新手理解原理。
- **缺点**：**慢**。如果你复制一个 10MB 的文本，这种写法就像用勺子去掏空一个游泳池，效率很低，因为每读写一次都要跟硬盘打交道。



所以接下来我们可以改进一下,通常我们会用 **数组缓冲区**

```java
char[] buffer = new char[1024]; // 准备一个能装 1024 个字符的“大铲子”
int len;
while ((len = fr.read(buffer)) != -1) {
    fw.write(buffer, 0, len); // 一次写一铲子
}
```

关于改进也有两个版本分别是自动挡的`BufferedReader,BufferedWriter`以及手动挡的“字符数组”作为缓冲区

**自动挡:**

```java
package JavaBase.IoExample.assets;

import java.io.*;

public class BufferedTextLab {
    public static void main(String[] args) {
        // 【套娃模式】：用 BufferedReader 包裹 FileReader
        try (BufferedReader br = new BufferedReader(new FileReader("src/assets/test.txt"));
             BufferedWriter bw = new BufferedWriter(new FileWriter("src/assets/output/test_buffer.txt"))) {
            
            String line; // 这次我们按“行”收割
            
            // readLine() 是 BufferedReader 的神技，一次读一整行，没货了返回 null
            while ((line = br.readLine()) != null) {
                bw.write(line);      // 写入这一行
                bw.newLine();        // 【关键】写入一个换行符，因为 readLine() 不会带换行符
            }
            
            System.out.println("使用 BufferedReader 复制成功！速度提升了 N 倍。");
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

手动挡:

```java
public class TextCharBufferLab {
    public static void main(String[] args) {
        try (FileReader fr = new FileReader("src/assets/test.txt");
             FileWriter fw = new FileWriter("src/assets/output/test_buffer.txt")) {

            char[] buffer = new char[1024];
            int len;
            while ((len = fr.read(buffer))!=-1){
                fw.write(buffer,0,len);
            }
            System.out.println("使用 BufferedReader 复制成功！速度提升了 N 倍。");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

##### 2)字符流

```
public class ImageByteLab {
    public static void main(String[] args) {
        try(FileInputStream fi = new FileInputStream("src/assets/picture.jpg");
            FileOutputStream fo = new FileOutputStream("src/assets/output/copy_picture.jpg")
        ){
            byte[] buffer = new byte[1024];
            int len;

            while((len = fi.read(buffer))!=-1){
                fo.write(buffer,0,len);
            }

            System.out.println("图片拷贝成功！请检查项目根目录下的 copy_cat.jpg");

        }catch (Exception e){
            e.printStackTrace();
        }
    }
}

```

你可能会觉得：这不就是把 `char` 改成 `byte` 吗？

没错！因为在操作系统的眼里，**万物皆字节**。

- **文本文件**：是字节。
- **图片文件**：是字节。
- **你的表情包、视频、甚至是 `.class` 编译文件**：通通都是字节。

**字节流（`InputStream`/`OutputStream`）** 是 IO 界的“老祖宗”。它是直接对着文件的二进制 0 和 1 进行操作的，不经过任何大脑加工（不处理编码），所以它能保证图片在搬运过程中一个比特位都不会错。







### 02、操作对象划分

仔细想一下,IO IO，不就是输入输出（Input/Output）嘛：

- Input：将外部的数据读入内存，比如说把文件从硬盘读取到内存，从网络读取数据到内存等等
- Output：将内存中的数据写入到外部，比如说把数据从内存写入到文件，把数据从内存输出到网络等等。

所有的程序，在执行的时候，都是在内存上进行的，一旦关机，内存中的数据就没了，那如果想要持久化，就需要把内存中的数据输出到外部，比如说文件。

文件操作算是 IO 中最典型的操作了，也是最频繁的操作。那其实你可以换个角度来思考，比如说按照 IO 的操作对象来思考，IO 就可以分类为：文件、数组、管道、基本数据类型、缓冲、打印、对象序列化/反序列化，以及转换等。

<img src="../assets/javaAssets/16.shangtou-03.png" width=40%>

#### **1）文件**

文件流也就是直接操作文件的流，可以细分为字节流（FileInputStream 和 FileOuputStream）和字符流（FileReader 和 FileWriter）。

FileInputStream 的例子：