---
title: 细窥Golang类型系统
mylink: 细窥Golang类型系统
date: 2021-12-20 21:09:47
tags:
	- Golang
	- 好文翻译
categories:
---

本文是一篇翻译，源于与好友的一次关于 Golang 底层类型判定的讨论。



找到了这篇好文，奈何中文翻译欠佳（内容缺、词不达意），就看了原版。



看完就忍不住翻译下来，**逐字斟酌，并补充了许多内容**，标示「译者注」。——by 爱飞的鸟



<!--more-->

以下正文：



**副标题：配合示例详解 Go 语言类型系统**



我们先从一个基本问题开始吧！  

## 为什么我们需要类型系统？

在回答这个问题之前，我们先看看平日里我们不需要接触的编程语言的原始而抽象的底层形态。  

**让我们贴近机器数的底层表示吧！** 

![二进制的0和1](https://bird-notes.oss-cn-hangzhou.aliyuncs.com/img/binary0_1.jpeg)

上面是二进制的 0 和 1，也即机器所能理解的数据形式。但这对我们来说有意义吗？直到看到下图，我才知道（有人是黑客帝国粉吗？）

![matrix](https://bird-notes.oss-cn-hangzhou.aliyuncs.com/img/matrix.gif)

我们进一步抽象这些二进制数字。看下面这段汇编：

![assembly](https://bird-notes.oss-cn-hangzhou.aliyuncs.com/img/assembly.png)

你可能希望 R1、R2、R3 是整数，它们的类型其实在汇编语言层面无法确定。没有什么能阻止 R1、R2、R3成为任意类型，它们只是存在于寄存器内部的一堆 0 和 1。即使没有意义，加法运算也会将这三个数以按位累加，然后存储运算结果。



所以，「类型」的概念，源于更高级语言的更深层次的抽象，例如 `C`, `Go`, `Java`, `Python` 和 `TypeScript`，类型是语言自身的特性。



所以有些语言喜欢在运行时做类型检查，而有些喜欢在编译阶段做类型检查。



> 译者注：即动态语言（解释型）与静态语言（编译型）的区别。Go是门静态强类型、编译型语言，同时因为 `interface` 的存在，让 Go 兼具动态性的优点。

### 所以，类型到底是什么？

各个编程语言中的类型的概念不尽相同，类型能以多种不同的方式表达，但它们还是有不少共通之处的。



1. 类型是一组值
2. 在这组值上可以施加操作。如，整数类型可以施加 `+` 与 `-`，字符串类型可以拼接、空串检查等。
3. 类型需要在 运行时 `runtime` 或编译期 `compile` 进行检查，确保数据的整体性，以及使得编译器按照开发者的意图来正确解析数据。

> 译者注：这里解释一下第三点。所有的数据，存储上的表现都是，占据了一段连续内存空间的值。  
>
> 
>
> 以C语言和Go语言的结构体举例，什么是「数据的整体性」？  
>
> 我们来想想编译器如何能保证取出的某个结构体是完整的，而不是漏了或多了某个属性字段？很简单，每个结构体类型的长度是固定的，编译器只要在内存上取出连续的该结构体类型所占字节数的内存，就能保证取出的结构体是完整的。
>
>   
>
> 什么是「使得编译器按照开发者的意图来正确解析数据」？
>
> 其实上面的例子已经隐含这个解析问题了，我们再举个更具体的例子。  
>
> 当取出了内存中的一段连续的内存后，怎么解析出内部的一个个字段属性呢？也很简单，按照字段的定义顺序，按字段类型的长度，依次解析。例如，结构体的第一个字段是 int32 类型，那么就取这段内存的前4个字节，解析成第一个字段（Go与C的结构体的起始地址都等于第一个字段的地址，这里应该还有内存对齐问题，有兴趣的读者可深入研究）



**所以一个编程语言的类型系统，明确了某类型下的哪些操作是合法的。**



类型检查的目的是为了确保运算和操作只施加在正确的类型之上，并且程序必须遵循编程语言所定义的类型系统。类型检查可能是在编译期执行的，也可能是在「运行时」执行的。通过类型检查，可以使得数据以我们期望的形式被解释。因为在机器码执行阶段，不会有任何检查，全是二进制的 0 和 1。机器会无脑执行施加在二进制数据上的任何操作。



类型系统用来在二进制层面（原文 `bit patterns`）强制执行预期代码解释。例如，它保证了整数不会被施加任何非整数操作，从而得到无意义的结果。



一个类型系统由以下几部分组成：



1. **基础类型**——包含在该编程语言中，可用于以该编程语言编写的任何程序。Go 拥有许多基础类型，int8 , uint8 ( byte ), int16 , uint16 , int32 ( rune ), uint32。

2. **类型构造器**——编程语言提供的定义新类型的途径。例如，定义一个 `T` 的指针类型 `*T`。

   > （译者注：许多人认为Go的指针类型不是新类型，但其实任何类型的指针类型都是一个全新的类型，数组和切片也类似。如 `[]int` 和 `[3]int` 就是两种新类型。理解了这点，对底层数据类型的判断会有很大的帮助）

3. **类型推断**——编译器可以推断变量或函数的类型，而不需要显式声明它。Go 拥有 *单向类型推理* (*Uni-directional type inference*)。

4. **类型兼容**——

   1. 下面哪个赋值是被类型系统所允许的？`a int; b int8; a = b; `
   2. 如何定义两个类型是否相等？——在[Go的可赋值判断](https://go.dev/ref/spec#Assignability)中主要由类型是否可以互换引用决定。我们等会会详细探讨。

## Go 语言的类型系统

Go 有很多基础的规则来控制类型系统，这里我们只挑些重要的一起来看看。



但是，在抛下一切概念前，不如先看看我给你们准备的几个小例在子，它覆盖了一些 Go 类型系统的基础概念。我会在讲解重要概念的时候带你吃透这些例子。



先看看这些代码片段吧，哪些能成功通过编译，哪些不能，为什么？

![type_system_figure1](https://bird-notes.oss-cn-hangzhou.aliyuncs.com/img/type_system_figure1.png)

![type_system_figure2](https://bird-notes.oss-cn-hangzhou.aliyuncs.com/img/type_system_figure2.png)

我希望你能写下自己的答案和理由，这样在最后我们就能一起探讨。

## 有名类型（Named Types）

> 译者注：也可被译作 「命名类型」，又可被叫做「定义类型」（Defined Types）



诸如`int`, `int64`, `float32`, `string` 和 `bool` 等类型是预先定义（pre-declare）的。**所有的预先定义的布尔型、数字型，字符串类型都是有名类型。**

> 译者注：
>
> 内置字符串类型： string 
> 内置布尔类型： bool 
> 内置数值类型：
>
> * int8、uint8（ byte）、int16、uint16、int32（ rune）、uint32、int64、uint64
> * float32、float64
> * complex64、complex128



同样的，任何通过「类型声明 `type declaration`」创建的新类型，也是有名类型。如：

```go
var i int // named type
type myInt int // named type
var b bool // named type
```



> 译者注：
>
> 1. 第一个和第三个是 Go 语言预定义类型
> 2. 第二个用了类型定义，类型声明分为「类型定义」和「类型别名」两种。 `type myInt = int` 是类型别名。



**有名类型与定义类型，与其他类型都是永远不相等的**。

## 无名类型（Unnamed Types）

> 译者注：也可被译作「未命名类型」，「非定义类型」，非定义类型一定是复合类型



**复合（组合）类型**——数组，结构体，指针，函数，接口，切片，映射和 channel ——都是无名类型。

```go
[]string // unnamed type
map[string]string // unnamed type
[10]int // unnamed type
```

上面的「类型字面量」（类型字面量是一个用来表示混合值的概念）仅仅描述了复合类型是如何被组织的，类型本身并没有名字。



## 底层类型

每个类型 `T` 都拥有底层类型。  



1. 如果 `T` 是预定义的布尔、数字、字符串或类型字面量，它的底层类型就是它本身。

2. 否则，`T` 的底层类型是其在类型声明时所引用的类型的底层类型。（译者注：即，在一个类型声明中，新声明的类型和原类型共享底层类型）

![underlying_types](https://bird-notes.oss-cn-hangzhou.aliyuncs.com/img/underlying_types.png)

让我们逐行分析：



* 3 和 8：`string` 是底层预定义类型，所以底层类型是 `string` 本身
* 5 和 7：类型声明最右侧的是两个「类型字面量」，所以底层类型分别是 `map[string]int` 和 `*N` 。注意：类型字面量也是无名类型。
* 4、6 、10：它们的底层类型与类型声明时所引用的那个类型相同。译者注：
  * 4：与 `A` 的底层类型相同，即 `B` 的底层类型是 `string`
  * 6：与 `M` 的底层类型相同，即 `N` 的底层类型是 `map[string]int`
  * 10：与 `T` 的底层类型相同，即 `U` 的底层类型是 `map[S]int`



让我们再看看第 9 行的 `type T map[S]int`。



`S` 的底层类型是 `string`，所以，难道 `T` 的底层类型不应该是 `map[string]int` 吗？为什么是 `map[S]int` ？



因为我们在谈论 `map[S]int` 的底层无名类型时，在遇到第一个无名类型时就停止追溯（译注：`map[S]int`就是我们在递归寻找 `T` 的底层类型时遇到的第一个无名类型，立即停止）。或者正如 Go 语言规范上写的，如果 `T` 是类型字面量，那么它的类型就是它本身。



你可能会好奇，为什么我这么重视「无名类型」、「有名类型（定义类型）」和「底层类型」。因为它们在 Go 语言规范中扮演着重要的角色，使得我们的讨论能够继续。它能帮助我们理解，为什么上面贴的那些代码，有些长得几乎一样，但甚至过不了编译。



## 可赋值性

关于一个值 `x` 是否可以赋值给一个 类型为 `T` 的变量，满足以下规则之一就可赋值（[译自Go语言可赋值性规则](https://go.dev/ref/spec#Assignability)）：

* x 的类型 与 T 相同。

* x 的类型 V 和 类型 T 的底层类型相同，且 V 和 T 至少有一个不是 「定义类型（有名类型）」。

* T  是接口类型，且 x 实现了 T 接口。

* x 是双向 channel 值 ，T 是 channel 类型。x 的类型 V 和 T 拥有相同的元素类型，并且 V 和 T 至少有一个不是「定义类型」。

  > 译者注： `ch = make(chan int)`，其中 `int` 就是「元素类型」

* x 是预定义的 `nil` 值，T 是一个指针，函数，切片，映射，channel 或 接口类型。

* x 是无类型的可被 T 类型表示的常量值

  > 译者注：如， `var num int = 1` 。`x` 为 无类型常量 `1`，可被 `int` 表示。



虽然这些条件都是自解释的，我们还是一起来看看其中的一条吧。



规则：赋值时，两边的底层类型需要相同，并且至少要有一个不是「有名类型」。

> 译者注：为什么这么多规则都强调了，至少有一个不是「有名类型」？读者可在看到「类型一致性」章节时留个心眼。



我们再来看看前文的 Figure 4 和 Figure 5 所示代码。



```go
// Figure 5
package main

type aInt int

func main() {
	var i int = 10
	var ai aInt = 100
	i = ai
	printAiType(i)
}

func printAiType(ai aInt) {
	print(ai)
}
```

上述代码不能通过编译，报错如下：

```
8:4: cannot use ai (type aInt) as type int in assignment
9:13: cannot use i (type int) as type aInt in argument to printAiType
```

原因：`i` 是有名类型 `int`， `ai` 是有名类型 `aInt`，虽然它们的底层类型都是 `int`，但仍不能赋值。

```go
// Firgure 4
package main

type MyMap map[int]int

func main() {
	m := make(map[int]int)
	var mMap MyMap
	mMap = m
	printMyMapType(mMap)
	print(m)
}

func printMyMapType(mMap MyMap) {
	print(mMap)
}
```

代码片段 4 能通过编译。因为 `m` 是无名类型 `map[int]int`，`mMap` 的底层类型和类型声明原类型 `MyMap` 的底层类型相同，都是 `map[int]int`， 即 `m` 和 `mMap` 的底层类型相同。



## 类型转换

一个非 常量的值 `x` 能被转换成类型 `T` ，当满足以下任何条件之一：[译自 Go 语言类型转换规则](https://go.dev/ref/spec#Conversions)



* x 可赋值给 T

* 忽略结构体 tag， x 的类型和 T 的类型拥有相同的底层类型。

* 忽略结构体 tag，x 的类型和 T 都是无名指针类型，并且他们的指针基类型拥有相同的底层类型。

  > 译者注：Go 语言中，改变结构体成员的相对位置，会得到一个新的结构体类型。

* x 的类型和 T 都是整数或浮点数指针类型。

* x 的类型和 T 都是复数类型（complex）。

* x 的是整数切片或字节（byte）切片或 runes 切片，T 是字符串类型。

* x 是字符串， T 是 byte 或 runes 切片。

* x 是切片，T 是指向数组的指针，并且切片和数组的基类型拥有相同的底层类型。



看看 Figure 3。

```go
// figure 3
package main

type Meter int64

type Centimeter int32

func main() {
	var cm Centimeter = 1000
	var m Meter
	m = Meter(cm)
	print(m)
	cm = Centimeter(m)
	print(cm)
}
```

上述代码可编译通过，因为 `Meter` 和 `Centimeter` 的底层类型都是整型，且底层类型能相互转换。



在我们看 Figure1 和 Figure2 前，我们不妨一起研究下 Go 语言类型系统的另一个更为基础的规范。



## 类型一致性

**两种类型要么相同，要么不同**。



在通常的类型系统中，有两个标准方法来判断两个类型是否可视为同一类型。**名字相同** 和 **结构相同**。



**名字相同** 是非常直观的：两个类型当且仅当类型名字相同时才相同。



所以，在 Go 语言里面，一个已定义的类型（有名类型），永远和其他类型不相同。因为两个有名类型的名字不可能相同。



**结构相同**：两个类型当且仅当它们拥有相同的「结构（structure）」时才相同。



在 Go 语言中，当两个类型结构相同且不是有名类型时，类型才相同。



所以，甚至 Go 语言中预定义的 有名类型/定义类型，例如，`int` 和 `int64` 也不是相同类型。并且，Go 语言接口类型的可赋值性也取决于 [结构类型系统（Structural type system）](https://en.wikipedia.org/wiki/Structural_type_system) 。 **Go 语言没有 Duck Type**。



(下面是[原作者义愤填膺的推特](https://twitter.com/in_aanand/status/1072476894308773888?ref_src=twsrc^tfw|twcamp^tweetembed|twterm^1072476894308773888|twgr^|twcon^s1_&ref_url=https%3A%2F%2Fcdn.embedly.com%2Fwidgets%2Fmedia.html%3Ftype%3Dtext2Fhtmlkey%3Da19fcc184b9711e1b4764040d3dc5c07schema%3Dtwitterurl%3Dhttps3A%2F%2Ftwitter.com%2Fin_aanand%2Fstatus%2F1072476894308773888image%3Dhttps3A%2F%2Fi.embed.ly%2F1%2Fimage3Furl3Dhttps253A252F252Fpbs.twimg.com252Fprofile_images252F1097738091530342400252FFS0a8_YX_400x400.png26key3Da19fcc184b9711e1b4764040d3dc5c07))  译者保留意见。

![type_system_twitter](https://bird-notes.oss-cn-hangzhou.aliyuncs.com/img/type_system_twitter.png)



下面我们来看看结构体的转换：



规则：忽略结构体 tag，x 的类型和 T 拥有相同的底层类型。



```go
// figure 2
package main

type Meter struct {
	value int64
}

type Centimeter struct {
	value int32
}

func main() {
	cm := Centimeter{
		value: 1000,
	}

	var m Meter
	m = Meter(cm)
	print(m.value)
	cm = Centimeter(m)
	print(cm.value)
}
```

注意规则：**相同的底层类型**。  

因为结构体域 `Meter.Value` 的底层类型是 `int64`， 而 `Centimeter.Value` 的底层类型是 `int32`，它们不是同一类型，因为「有名类型（定义类型）」永远与其他类型不同。



再来看看代码段 1

```go
// figure 1
package main

type Meter struct {
	value int64
}

type Centimeter struct {
	value int64
}

func main() {
	cm := Centimeter{
		value: 1000,
	}

	var m Meter
	m = Meter(cm)
	print(m.value)
	cm = Centimeter(m)
	print(cm.value)
}
```

 `Meter.Value` 的底层类型和 `Centimeter.Value` 的底层类型都是 `int64`，所以类型转换能通过编译。



原作者：我希望这篇文章能在 Go 型系统上为您提供一些新的见解，而我在写这篇文章的时候也获得了新的洞见。



> 译者：俺也一样！翻译一遍和简单读一遍收获完全不一样。



## 参考文献

1. 原文（需科学上网）：[A closer look at Golang type system | by Ankur Anand | Medium](https://medium.com/@ankur_anand/a-closer-look-at-go-golang-type-system-3058a51d1615)
2. [golang101——第14章 go 类型系统概述](https://github.com/golang101/golang101)