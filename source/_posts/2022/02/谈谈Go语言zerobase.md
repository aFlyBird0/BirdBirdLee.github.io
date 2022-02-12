---
title: 谈谈Go语言zerobase
mylink: 谈谈Go语言zerobase
date: 2022-02-12 15:32:25
tags:
    - Golang
categories:
---

挺早就知道 Go 语言的 zerobase，一直没深入过。正巧前天朋友发来一篇文章，于是乎就借着这个契机稍微研究一番。大概分为敲出来跑跑，简单查查源码，拓展这三块。

<!--more-->

# 〇、缘起

因为朋友发我的这篇文章：[[Golang]空结构体引发的大型打脸现场](https://segmentfault.com/a/1190000039315999#/) 聊起了 gozerobase。这里直接截取代码贴出来。

```go
type User struct {
}

func FPrint(u User) {
    fmt.Printf("FPrint %p\n", &u)
}

func main() {
    u := User{}
    FPrint(u)
    fmt.Printf("main: %p\n", &u)
}
// 运行结果
FPrint 0x118eff0
main: 0x118eff0
```

我们都知道，Go 语言的函数参数只有「传值」没有传引用，`u` 理应被拷贝了一份，可为什么输出的地址相等？  

原因是，所有的空结构体，指向的都是同一块内存，叫 zerobase。

于是，我又想到了，我曾经看某篇文章说过，所有的空切片（不为 nil，但长度容量都为 0）的底层数组指针，指向的都是同一块内存。  

那么，会不会，所有的占据内存为0的类型的变量，都指向了同一块 zerobase 呢？

# 一、敲出来跑跑

于是我就敲了这段代码：

```go
func TestZeroBase(t *testing.T) {
    // 结论，Go 的零字节内存都会指向同一地点
    s1 := struct{}{}
    s2 := struct{}{}
    fmt.Printf("s1:%p, s2:%p\n", &s1, &s2)

    array1 := [0]int{}
    array2 := [0]string{}
    fmt.Printf("array1:%p, arry2:%p\n", &array1, &array2)

    slice1 := make([]int, 0)
    slice2 := make([]chan string, 0)
    sliceHead1 := (*reflect.SliceHeader)(unsafe.Pointer(&slice1))
    sliceHead2 := (*reflect.SliceHeader)(unsafe.Pointer(&slice2))
    data1 := *(*[0]int)(unsafe.Pointer(sliceHead1.Data))
    data2 := *(*[0]chan string)(unsafe.Pointer(sliceHead2.Data))
    fmt.Printf("slice1.data:%p, slice2.data:%p\n", &data1, &data2)

    // 输出
    //s1:0xad48e8, s2:0xad48e8
    //array1:0xad48e8, array2:0xad48e8
    //slice1.data:0xad48e8, slice2.data:0xad48e8
}
```

结论显而易见：**Go 的零字节内存都会指向同一地点(zerobase)**

# 二、简单查查源码

这块的源码已经烂大街了，直接贴：

```go
func mallocgc(size uintptr, typ *_type, needzero bool) unsafe.Pointer {
    if gcphase == _GCmarktermination {
        throw("mallocgc called with gcphase == _GCmarktermination")
    }

    if size == 0 {
        return unsafe.Pointer(&zerobase)
    }
    //...
}
```

可以看到，在分配内存的时候，如果 `size` 为 0，直接不分配，并将其指向全局变量 zerobase。

# 三、拓展

Go 语言有这样一个 spec : **Pointers to distinct [zero-size](https://go.dev/ref/spec#Size_and_alignment_guarantees) variables may or may not be equal.**  

即，**指向了零内存对象的不同指针，可能相等也可能不等**。

比较权威的缘由，可以在 [Github 某 issue](https://github.com/golang/go/issues/23440#issuecomment-357476298) 找到，如下：

> This is an intentional language choice to give implementations flexibility in how they handle pointers to zero-sized objects. If every pointer to a zero-sized object were required to be different, then each allocation of a zero-sized object would have to allocate at least one byte. If every pointer to a zero-sized object were required to be the same, it would be different to handle taking the address of a zero-sized field within a larger struct.

我怎么翻译都觉得别扭，就不翻译了，请读者细细品。

现在，来看一段代码：

```go
    s1 := new(struct{})
    s2 := new(struct{})
    println(s1, s2, s1 == s2)

    s3 := new(struct{})
    s4 := new(struct{})
    println(s3, s4, s3 == s4)
    fmt.Println(s3, s4, s3 == s4)

    s5 := struct{}{}
    s6 := struct{}{}
    fmt.Printf("s5:%p, s6:%p, s5==s6?:%v\n", &s5, &s6, &s5 == &s6)

    // 输出
    0xc000063ef7 0xc000063ef7 false
    0x10f5908 0x10f5908 true
    &{} &{} true
    s5:0x10f5908, s6:0x10f5908, s5==s6?:true
```



可以看到，s1==s2 是 false，s3==s4 是 true。他们都是指向了零字节内存的指针，但，有时候地址相等，有时候地址不等。这也就是前面说的，指向了零内存对象的不同指针，可能相等也可能不等。为什么呢？



同时，通过 s5 与 s6 的输出，可以看到，s3, s4, s5, s6 的地址都是 `0x10f5908`，这应该就是 zerobase 了。



# 四、意外之喜

又看到了一篇文章，终于解了上面的惑，我简化如下：

看两段代码：

```go
    // 代码段1
    s1 := struct{}{}
    s2 := struct{}{}
    fmt.Println(&s1 == &s2)
```

代码段1的输出一定是 false。

```go
    // 代码段2
    s1 := struct{}{}
    s2 := struct{}{}
    fmt.Println(&s1 == &s2)
    fmt.Printf("s1:%p, s2:%p\n", &s1, &s2)


```

代码段2的输出的第一行一定是 true

是由于 `fmt.Printf` 导致的，原因如下

> 其实，fmt.Printf 第二个参数，是一个 interface 类型。而 fmt.Printf 的内部实现，使用了反射 reflect，正是由于 reflect 才导致变量从栈向堆内存的逃逸成为可能（注意，并非所有reflect操作都会导致内存逃逸，具体还得看怎么使用reflect的）。我们简单总结为：
> 
> 使用 fmt.Printf 由于其函数第二个参数是接口类型，而函数内部最终实现使用了 reflect 机制，导致变量从栈逃逸到堆内存。



所以纠正一下：**堆上内存分配**调用了 runtime 包的 newobject 函数，而 newobject 函数其实本质上会调用 runtime 包内的 mallocgc 函数，而 mallocgc 函数就是文章最开头讲的 zerobase 那段。



也就是，零内存对象只有在堆内存分配时才会指向 zerobase。



代码段1和代码段2的小对象内存分配，应该原本都是分配栈上的。所以代码段1的 s1 和 s2 未指向 zerobase。



而代码段2因为内存逃逸，对象分配到了堆上，就调用了 mallocgc 函数，指向了 zerobase。



前文第三大段的「拓展」部分贴的代码，原理也是一样的。



所以之前只知其一，不知其二。内存分配这块，还是要好好学学！

# 四、参考文献

- [一个奇怪的golang等值判断问题](https://studygolang.com/articles/19535?fr=sidebar#/)

- [[Golang]空结构体引发的大型打脸现场](https://segmentfault.com/a/1190000039315999#/)

- [spec: Empty struct address comparison returns false even if addresses are equal · Issue #23440 · golang/go](https://github.com/golang/go/issues/23440#/)

- [Golang最细节篇— struct{} 空结构体究竟是啥？](https://jishuin.proginn.com/p/763bfbd35690#/)
