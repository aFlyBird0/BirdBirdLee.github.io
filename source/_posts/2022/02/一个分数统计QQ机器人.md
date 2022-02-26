---
title: 一个分数统计QQ机器人
mylink: 一个分数统计QQ机器人
date: 2022-02-26 21:33:33
tags:
	- Go
categories:
---

22考研分数刚出，学校建了个实验室宣传群，我也接过老师的旨意，进去宣传。陆续有考生进群，浙江省未公布排名，所以考生的一大痛点是：知道总体排名情况，提前判断自己是否能进复试。去年我写过一个 python 爬虫分数统计，用起来不是那么舒服。今年我突然想到，直接用 QQ 机器人应该会优雅很多。花了半个下午写了初代版本，后面又迭代了两大版本。没有人推出新的分数排名工具，都在用我的，说明写得还行？哈哈哈。顺便写个文档，方便下届复用。 [Github 传送门](https://github.com/aFlyBird0/bird_qq_bot)。

<!--more-->

## 一、需求分析与实现设计

### 1.1 总体思路

* 统计来源：使用 [MiraiGo](https://github.com/Mrs4s/MiraiGo) 获取群内所有成员的名片信息，从名片信息内提取分数，统计
* 交互方式：通过群内聊天关键词触发，获取与计算分数，发送结果

### 1.2 需求分析

1. 需要统计各个分数段的人数，每10分为一段，并从高到低展示
2. 为了兼容以及可重用，应当在设计初期就支持多群独立统计
3. 虽然都是计算机学院，但是分为计算机学硕，计算机专硕，软件工程学硕等等六个专业，需要独立排名。

### 1.3 实现设计

#### 1.3.1 QQ 机器人 API 相关

以下功能是通过 QQ 机器人的 API 实现的，本文略。

获取群列表、获取某个群所有成员的QQ以及群名片、群消息识别、群消息发送。

#### 1.3.2 多群独立统计分数

很简单，弄个 map 就好，key 是群号，value 是相应的分数排名结构体

#### 1.3.3 各专业独立排名，并方便拓展

已知，大部分考生的群名片都按要求改过。如，计算机学硕的同学，名片中会含有「计学」或「计科」。



我不想写太多过程式的代码，很不优雅，维护也非常麻烦，所以采用了接口的实现方式。



构思是这样的，其实很简单。



如果采用过程式的代码，计算某一个群的各专业分数排名，大概是这样的：

```go
// (输入)群名片列表：[]string
type (
    major = string
    score = int
)
// (输出)各专业分数列表： map[major][]score

// 方法：遍历群名片列表，如果群名片含有「计学」字样，并且能正则出一个三位数，就 append 一下 map["计学"] 的切片。如果含有「计专」，就...；如果...，就...
```



所以，每个专业的其实只有识别规则与专业名字不同，那就将其约定成为一个接口。



每想实现一个新的专业的识别，就定义一个新的实现了该接口的过滤规则，传进去就好。



代码如下：



```go
// ScoreFilter 专业过滤接口
type ScoreFilter interface {
	Name() string                // 专业名字
	Filter(nickname string) bool // 专业匹配规则
}

// GetScoreMap 传入昵称列表和过滤规则，返回每个规则命中的分数列表
func GetScoreMap(nicknames []string, filters ...ScoreFilter) map[ScoreFilter][]int {
	scoreMap := make(map[ScoreFilter][]int)
	for _, nickname := range nicknames {
		for _, filter := range filters {
			if filter.Filter(nickname) {
                // ExtractScore 就是从 string 中尝试匹配一个三位数
				score, ok := ExtractScore(nickname)	
				if ok {
					scoreMap[filter] = append(scoreMap[filter], score)
				}
			}
		}
	}
	// 分数排序
	for _, scores := range scoreMap {
		sort.Slice(scores, func(i, j int) bool {
			return scores[i] > scores[j]
		})
	}
	return scoreMap
}

```



后面输出各专业标头的时候，调用 `ScoreFilter.Name()` 即可。

#### 1.3.4 统计各分数段人数

我实现的是经典的 10 分一段。

存储每个分数段的数据结构是这样的：

```go
type ScoreGroup struct {
	Min    int   // 最小分数
	Max    int   // 最大分数
	scores []int // 分数列表
}

func (s ScoreGroup) Describe() string {
	return fmt.Sprintf("%v - %v", s.Min, s.Max)
}

func (s ScoreGroup) Len() int {
	return len(s.scores)
}
```



分段方法很简单：



```go
// 选用第一个数，抹个位，作为 start
var start int = scores[0] / 10 * 10
// start + 9，作为 end，这样就能实现类型 350-359 的分数区间
var end int = start + 9
// 当前分数段内的所有分数
scoresOneGroup := make([]int, 0)
```

遍历分数切片（已从大到小有序）

* 若当前分数在 [start, end] 内，就加到当前分数段内
* 否则，说明上一个分数段满了。将上一个分数段的所有分数存储到总的分数段结构体内，再根据当前分数调整 start 与 end，清空当前分数段分数，将当前分数段加到新的当前分数段切片内。
* 记得最后一组分数段，也要加到总分数段结构体内。



#### 1.3.5 最终效果

```
考研分数段统计来啦！
计算机学硕(共92个分数)
390 - 399: 1人(累计1)
//...略
330 - 339: 12人(累计39)
320 - 329: 10人(累计49)
//...略
260 - 269: 1人(累计92)
计算机专硕(共357个分数)
390 - 399: 1人(累计1)
380 - 389: 11人(累计12)
370 - 379: 7人(累计19)
//...略
```



## 二、过密分数段分布

随着进群人数的增多，发现某些分数段过于密集，某些10分段内的人数甚至达到了30+人，参考意义不大。所以做了个过密分数段分布。



写这个机器人的时候，我个人愈发觉得，**应该多用复杂的数据结构解决问题，而不是用复杂的算法解决问题。因为复杂的数据结构能简化思考，减少错误发生的可能性，且方便拓展；而复杂的算法，非常容易出错，且难懂，难改。**



所以我干脆换了分数段统计的方法，直接粗暴得统计每个分数的人数，然后再分组。

```go
// ScoreCount 记录每个分数的数量
type ScoreCount struct {
	Score int
	Count int
}

type ScoreGroupCount struct {
	Min    int          // 最小分数
	Max    int          // 最大分数
	Scores []ScoreCount // 分数列表
}
```

定义数据结构的一大好处是，能在其基础上定义方法，将功能模块化拆分。



最后，计算密集分数段只要这样一行链式操作：

```go
// 输入: var scores []int
denseGroups := ScoreGroupCountList(GroupScoresFromCount(CountScores(scores))).FilterByCount(10)

// CountScores(scores) 计算每个分数的人数
// GroupScoresFromCount(...) 从上面的结果中十分一段分组
// ScoreGroupCountList(...) 做一个类型转换
// ScoreGroupCountList(...).FilterByCount(10)	// 过滤出超过10人的分段
```



## 三、输出为网页

之前的输出方式，都是群内发送「:score」或「分数实时排名」，就会计算该群分数信息，并直接发送查询结果到群内。但是因为分数信息实在太多了，导致每次查询分数，都会直接占满两屏，刷屏太影响其他人的聊天。



所以我在寻求另外的展示方式。



之前想过私聊，但这存在需要问题：

* 需要开启群内临时会话，不安全，容易滋生广告
* 消息不共享，增加调用次数，容易被风控。如果在群内，一个人查了其他人都能看到；如果私聊，那么每个人想查都要调用一次，群里700多个人，不仅耗费资源，还容易导致机器人被风控。
* 临时会话问题。如果非好友，向机器人发送的消息属于临时会话，可以获取到来源的群号。如果是好友，消息属于私聊消息，除了存在两种不同消息需要兼容问题，还可能存在私聊消息不知道他来源于哪个群的问题（所以就无法获取其群名片，以及排名。一个可能的解决方案是，遍历机器人所在的所有的群，找到有该人的群，获取其名片）

私聊发送分数统计信息，明显不优雅。



我想到了将查询结果输出为网页的方式：将结果写入到 html 文件中，直接访问网页。



但这又有很多问题，直接修改文件，明显是不优雅的，而且可移植性非常低。



我就在想，有没有这样一个服务，能托管 html，并且支持 api 修改内容？很遗憾的是，找了半天没找到。



灵光一现！



我直接起个 gin web 服务，暴露一个 GET 查询，返回 `c.String()` ，然后反向代理一下，不就相当于一个网页了吗？因为通过网址访问网页，用的就是 `http.GET` ！ 再设置一个 `Query` 参数，通过群号访问各个群的独立排名。



在QQ群端，接到指令，重新排名。拼接 `base_url` 与群号，返回用户直接可点击的网址。



最终效果如下：

![image-20220226225137979](https://bird-notes.oss-cn-hangzhou.aliyuncs.com/img/image-20220226225137979.png)