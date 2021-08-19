---
layout: post
title: Python 技巧：活用函数对象
subtitle: 如何写一个 switch 语句
author: Pavinberg
tags: [编程, Python]
comments: true
---

Python 作为最近非常热门的语言，网络上有着许多相关教程。这里分享一些网上不常见的、但在我个人使用过程中感受到十分有趣的好用用法：如何灵活运用函数对象这个概念。

## 前情提要

这篇[博客](https://www.cnblogs.com/wisir/p/11072116.html)解释了 Python 中的函数对象的相关概念。简单来说，Python 中函数也是一个对象，具备 id、类型和值。通俗来讲，就是函数和一个普通变量没有区别，所以你可以把一个函数作为参数传递、作为变量赋值等等。

```python
>>> def func():
...     print("Hello, world.")
...
>>> hello = func # 赋值
>>> hello()
Hello, world.
```

## 如何写一个 switch 语句

C/C++、Java 等语言中有 `switch` 分支语句，作为 `if` 分支语句的补充。在更为现代的语言如 Rust、Scala 等语言中有 `match` 语句做分支匹配。但 Python 只有 `if`。例如，在有很多个分支时，通常 Python 会这样写：

```python
if foo == 1:
    run1()
elif foo == 2:
    run2()
elif foo == 3:
    run3()
else:
    runOther()
```

那么有没有类似 `switch` 语句的写法呢？Python 中可以借助字典，配合函数作为对象这一事实完成这个任务：

```python
switch = {
    1: run1,
    2: run2,
    3: run3
}

switch[foo]()
```

注意这段代码并不等价于上面的 `if` 分支写法。这一段代码定义了一个字典名为 `switch`，其字典键是我们判断分支时的值，而字典值是函数（注意不要加括号）。于是 `switch[foo]` 就可以找到相应的函数对象，再使用一对儿括号 `()` 进行函数的调用即可。但是，这里没有处理 `if` 语句中最后一个 `else` 的情况。

对于这种需求，需要使用字典的方法 `dict.get`：

```python
switch = {
    1: run1,
    2: run2,
    3: run3
}

switch.get(foo, runOther)()
```

`get` 方法接收两个参数，第一个是需要查找的字典键，第二个是如果没找到这个字典键时返回一个什么值。于是，如果 `foo` 是 1、2、3 中的一个，`get` 会返回相应的函数 `run1`, `run2`, `run3`，而如果是其它值，会返回 `runOther` 函数。最后依然，使用一对儿括号 `()` 进行函数的调用。这样这段代码与最开始的 `if` 分支语句的逻辑就是完全一样的了。

> 注意因为是字典，键不仅可以是数值，也可以是各种对象，只要可哈希即可。

看完你可能觉得何必这样大费周章，`if` 也没什么不好。的确，你总是可以用 `if` 解决，但是代码会比较混乱。使用这种写法的好处是：

1. 分支逻辑先打包成函数，再通过字典做键值映射，逻辑更为清晰，代码更为美观。

2. 促进对程序逻辑的封装。

3. 有函数式编程的思想，有利于使代码更进一步偏向于函数式。

此外，也可以将一组函数放到一个列表里，通过下标调用或是遍历调用，在一些情况下有非常好的效果。

## 相关内容

提到函数式就顺带提一下 Python 中的三个内置函数 `map`、`reduce` 和 `filter`。这三个函数都是对一个序列进行某种操作。以 `map` 为例，它是对序列中的每个元素都执行一个函数。例如：

```python
def double(x):
    return 2 * x
	
array = [1, 2, 3, 4]

# 对 array 中对每个元素执行 double 函数
result = map(double, array)
# 近似等于
result = []
for val in array:
    result.append(double(val))
```

这种操作显然非常常见，在一开始学编程时会更多地把函数看作一个计算过程。那么函数式的思想之一就是把函数作为一种有逻辑的对象，可以作用于其它对象上。这种方式可以极大简化代码，可读性很高，并且易于进行底层优化。

> 上面的之所以说近似等于，是因为 `map` 真正的返回值是一个迭代器，并不是列表。

更多细节可参考：

英文：[Python Tips: Map, Filter and Reduce](https://book.pythontips.com/en/latest/map_filter.html)

中文：[简书：Python 高阶函数 filter、map、reduce 的使用](https://www.jianshu.com/p/3c43992b8845)

> `map` 和 `reduce` 的组合在工程应用中极其实用，由谷歌最先提出的 `MapReduce` 成为了分布式计算领域的著名技术。大家有兴趣可以自行探索。
