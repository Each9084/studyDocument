# 第一章 预备知识

## 一、Python基础

### 1.列表推导式与条件赋值

在生成一个数字序列的时候，在 `Python` 中可以如下写出：

```py
L =[]

def my_func(x):
    return  2*x

for i in range(5):
    L.append(my_func(i))

print(L)
```

事实上可以利用列表推导式进行写法上的简化： `[* for i in *]` 。其中，第一个 `*` 为映射函数，其输入为后面 `i` 指代的内容，第二个 `*` 表示迭代的对象。

```py
[my_func(i) for i in range(5)]
```

列表表达式还支持多层嵌套，如下面的例子中第一个 `for` 为外层循环，第二个为内层循环：

```py
[m+'_'+n for m in ['a','b'] for n in ['c','d']]
#Out[6]: ['a_c', 'a_d', 'b_c', 'b_d']
```

除了列表推导式，另一个实用的语法糖是带有 `if` 选择的条件赋值，其形式为 `value = a if condition else b` ：

```py
value = 'cat' if 2>1 else 'dog'
```

等价于如下的写法：

```py
a,b='cat','dog'
condition = 2>1; #true

if condition:
    print(a)
else:
    print(b)
```

下面举一个例子，截断列表中超过5的元素，即超过5的用5代替，小于5的保留原来的值：

```py
[i if i <=5 else 5 for i in L]
```





### 2. 匿名函数与map方法

有一些函数的定义具有清晰简单的映射关系，例如上面的 `my_func` 函数，这时候可以用匿名函数的方法简洁地表示：

```py
my_func = lambda x : x*2

print(my_func(3))
#6

multi_para_func = lambda a, b: a + b

multi_para_func(1, 2)
#3
```

但上面的用法其实违背了“匿名”的含义，事实上它往往在无需多处调用的场合进行使用，例如上面列表推导式中的例子，用户不关心函数的名字，只关心这种映射的关系：

```py
[(lambda x: 2*x)(i) for i in range(5)]
#[0, 2, 4, 6, 8]
```

对于上述的这种列表推导式的匿名函数映射， `Python` 中提供了 `map` 函数来完成，它返回的是一个 `map` 对象，需要通过 `list` 转为列表：

```py
list(map(lambda x: 2*x, range(5)))
#[0, 2, 4, 6, 8]
```

对于多个输入值的函数映射，可以通过追加迭代对象实现：

```py
list(map(lambda x, y: str(x)+'_'+y, range(5), list('abcde')))
#['0_a', '1_b', '2_c', '3_d', '4_e']
```

### 3. zip对象与enumerate方法

zip函数能够把多个可迭代对象打包成一个元组构成的可迭代对象，它返回了一个 `zip` 对象，通过 `tuple, list` 可以得到相应的打包结果：

```PY
L1, L2, L3 = list('abc'), list('def'), list('hij')

list(zip(L1, L2, L3))
#[('a', 'd', 'h'), ('b', 'e', 'i'), ('c', 'f', 'j')]

tuple(zip(L1, L2, L3))
#(('a', 'd', 'h'), ('b', 'e', 'i'), ('c', 'f', 'j'))
```

> 通过tuple后就不可变了
>
> | **特性**     | **list(zip(...))**                               | **tuple(zip(...))**                               |
> | ------------ | ------------------------------------------------ | ------------------------------------------------- |
> | **可变性**   | **可变**。你可以 `append` 或修改其中的某个包裹。 | **不可变**。一旦生成，结构和内容都锁死了。        |
> | **性能**     | 略慢。因为要预留空间支持增删改。                 | **更快**。因为它是静态的，Python 内存分配更高效。 |
> | **安全感**   | 适合作为中间处理步骤。                           | 适合作为“最终结果”或配置信息。                    |
> | **作为 Key** | **不可以**作为字典的 Key。                       | **可以**。因为它不可变（Hashable）。              |

往往会在循环迭代的时候使用到 `zip` 函数：

```py
for i, j, k in zip(L1, L2, L3):
    print(i, j, k)
#a d h
#b e i
#c f j
```

`enumerate` 是一种特殊的打包，它可以在迭代时绑定迭代元素的遍历序号：

```py
L = list('abcd')

for index, value in enumerate(L):
    print(index, value)
#0 a
#1 b
#2 c
#3 d
```

> 注意这里的index并不是关键字,而是enum内部维护着一个count可以自动匹配索引,所以会出现0 a这种情况

用 `zip` 对象也能够简单地实现这个功能：

```py
for index, value in zip(range(len(L)), L):
    print(index, value)
#0 a
#1 b
#2 c
#3 d
```

当需要对两个列表建立字典映射时，可以利用 `zip` 对象：

```py
dict(zip(L1, L2))

{'a': 'd', 'b': 'e', 'c': 'f'}
```

既然有了压缩函数，那么 `Python` 也提供了 `*` 操作符和 `zip` 联合使用来进行解压操作：

```py
zipped = list(zip(L1, L2, L3))

zipped
#[('a', 'd', 'h'), ('b', 'e', 'i'), ('c', 'f', 'j')]

list(zip(*zipped)) # 三个元组分别对应原来的列表
#[('a', 'b', 'c'), ('d', 'e', 'f'), ('h', 'i', 'j')]
```



## 二、`Numpy`基础
