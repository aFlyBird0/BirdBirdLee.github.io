---
title: 浅析Golang之T与*T方法集及receiver
mylink: 浅析Golang之T与*T方法集及receiver
date: 2021-10-06 00:36:38
tags:
	- Golang
categories:
---

到今天为止，差不多正式学 Go 一个月。一直觉得 receiver 的 T 和 *T 的方法集云里雾里，好在通过多次查阅资料和代码实践，终于在表象方面掌握了七七八八。本文涉及 T 与 *T 方法集的关系，编译器自动解引用与取地址，T 与 *T 接口实现等。不一定全对，希望大佬指正错误。本文废话很多，一个概念翻来覆去讲了很多遍，但也是这样，才能使读者从各个角度印证自己的想法是否和我的思想一致，太多博客讲得单一了容易有理解偏差。

<!--more-->

## 一、什么是方法集

首先弄明白什么是方法集，之前看帖子把自己看迷糊了就是因为，把方法集的引出对象搞错了。方法集的主语是变量。也就是说，如果有个变量类型是 T，那么我们可以说，T有什么什么方法集，T的方法集是什么。

再做个简单的阅读理解：方法集就是，方法的集合。T拥有的方法集是所有 receiver 为 T 的方法的集合，也就是说，T 可以调用所有 receiver 为 T 的方法。更进一步，如果这些 receiver 为 T 的方法中的某几个，实现了某个接口需要实现的方法，那么，T 也实现了该接口。

类似的，*T 的方法集，是所有 receiver 为 T 与 *T 的方法的集合，也就是说，\*T 变量可以调用 receiver 为 T 或 *T 的方法（至于为什么，后面会讲到）。有的文章会写到， *T 的方法集包含了 T 的方法集，或者说，T 的方法集是 *T 的子集。意思就是，\*T 能调用 receiver 为 T 和 \*T 的方法，而 T 只能调用 receiver 为 T 的方法。

## 二、什么时候能通过方法调用改变 receiver 的值

方法能不能改 receiver 的值，只取决于方法定义的时候，接收者是不是指针。如果接收者是指针，方法被调用后，改变的值是生效的。详细看图。

`ChangeName` 方法的 receiver 是指针，所以无论是 值类型（以后值类型都用 T 表示），还是指针类型（以后指针类型都用 \*T 表示）调用 `ChangeName` 方法，改动都是生效的。

同理，如果 receiver 是 T，那么无论调用者是 T 还是 *T ，改动都不生效

![img](https://bird-notes.oss-cn-hangzhou.aliyuncs.com/img/%5BQJUH5B6I~%60~X96GWO%5D@IP3.jpg)

## 三、T 与 *T 作为 receiver 的调用法则

现在我们抛弃编译器的语法糖，即把编译器自动取地址的功能抛弃

![img](https://bird-notes.oss-cn-hangzhou.aliyuncs.com/img/LE22YTJCV4%5B%5BW058%60%5B31ZQL.jpg)

说人话就是，如果我定义了一个 T 变量，那么它只能调用 (t T)func() 的方法，也就是只能调用 receiver 为 T 的方法

如果定义了一个 *T 变量，那么它能同时调用 (t *T)func() 和 (t T)func() 的方法。也就是说，*T 类型的方法集包含了 T 和 *T 所有的方法集

那为什么 T 有时候能调用 (t *T)func() 的方法呢？因为，如果 t 可取址，编译器会自动加上&，所以大部分时候，无论接收者是 T 还是 *T，都能互相调

同样，我猜测，*T 能调用 (t T)func() 的，是编译器自动解引用的功劳。类似于，结构体指针引用成员变量，不需要这样：(\*s).name， 只需要 s.name，编译器会自动解引用。

无论如何 *T 一定能调用 (t T)func() 的，因为，解引用永远不会失败。但是取址不一定每次都成功。（例如字面量不能取址）

如果上面没看懂，可以看图：

![image-20211006010905729](https://bird-notes.oss-cn-hangzhou.aliyuncs.com/img/image-20211006010905729.png)

**小结论：**所以其实在一般情况下，**无论 receiver 是 T 还是 *T，无论变量是 T 还是 *T ，都是能两两互相调用的。**

理由如下：

① T 调用 T，*T 调用 *T，理所当然。

②*T 调用 T 时，变量属于指针，receiver 要求是值，那么编译器会自动把指针解引用成值，一定是成功的。

③T 调用 *T 时，变量是值，receiver 要求是指针，**若变量可取址**，编译器会自动取个地址把值变成指针，依旧能调用。

## 四、不同视角看待方法集与 receiver

下面我们从两种视角探讨一下方法集和 receiver 的关系，巩固一下。

上面那部分讲的是，固定变量的类型（T或*T），来看看它能调用什么方法

下面那部分讲的是，固定方法的接收器的类型（T或*T），来看看它能被什么变量调用

其实内容和上面讲的一样，但是主语不一样，能把橙字全部看懂，应该就算绕过来了。

![image-20211006010854888](https://bird-notes.oss-cn-hangzhou.aliyuncs.com/img/image-20211006010854888.png)

## 五、receiver 的 T、*T 与实现接口的关系

先明确一点，什么叫实现了接口：如果某个类型，实现了某个接口需要实现的所有方法，那么它就实现了这个接口。

这是个废话，应用在本文，我们应当关注：

* T类型实现了什么方法，*T类型实现了什么方法？——见实例
* 有哪些方法是隐含实现的？——见面三张图
* **实现接口与前面讲的谁能调用谁的关系是什么：T变量能调用方法A，那么T就实现了方法A**

![image-20211006011630363](https://bird-notes.oss-cn-hangzhou.aliyuncs.com/img/image-20211006011630363.png)

![image-20211006011635989](https://bird-notes.oss-cn-hangzhou.aliyuncs.com/img/image-20211006011635989.png)

![image-20211006011642911](https://bird-notes.oss-cn-hangzhou.aliyuncs.com/img/image-20211006011642911.png)

下面这段又都是**重点**：

前面讲到，若 receiver 是 T，那么，T 与 \*T 类型的变量，都能调用该方法。换句话说，如果有个方法的 receiver 是 T，那么它其实**隐含实现了 receiver 为 \*T 的方法**。对应上图，黄色框的函数是隐含实现的。

而 receiver 为 *T 的方法，只能被 *T 的类型调用。也就没有隐含实现这一说法。

所以导致了，虽然我们只定义了两个方法，但却生成了上图绿色字体的三个方法。

我们再来数一数，接口实现。 `myInterface` 需要实现 `ChangeName` 和 `SayName` 两个方法，看看橙色的箭头，刚好凑齐了两个方法，并且 receiver 都是 *T。所以 \*myStruct 实现了接口。

而 myStruct ，值类型，只有一个 receiver 为 sayMyName 的方法，只实现了一个，也就是没有实现接口。



## 六、深层次原理探讨

本文全都是讲的现象，或者说以一种不确切的方式来记忆 Golang 特性的方法，结果是正确的，但是原因不一定完全准确。

更深层次的源码级的探讨，可以看这个视频 [【Golang】T和\*T的方法集是啥关系？_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1NK4y137PJ?spm_id_from=333.999.0.0)

## 七、参考文献

* [Golang研学：在用好Golang指针类型 - 掘金](https://juejin.cn/post/6844903831365648391)

* [Go 中关于方法的 receiver 的总结_paul的专栏-CSDN博客_go receiver](https://blog.csdn.net/e3002/article/details/109459356)

* [Go的方法集 - 邱明成 - 博客园](https://www.cnblogs.com/qiumingcheng/p/9829781.html)

* [Golang中的方法集问题 - 简书](https://www.jianshu.com/p/d93656cdce0a)

* [goLang 之 type Method Value 和Method Expressions_weixin_34177064的博客-CSDN博客](https://blog.csdn.net/weixin_34177064/article/details/88918754)