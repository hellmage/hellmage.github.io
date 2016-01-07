---
layout: post
title:  "Myers通用Diff算法的实现"
date:   2016-01-07 17:54:28
categories: myers diff
---

Myers Diff算法是现在的各种diff算法的基础。只要你还用版本控制系统，你就在天天用。我花了一点时间终于大概看懂了是怎么回事儿，赶紧实现了一下加深印象。然后再在这里记一笔，免得过几天自己的代码也不认得了。

原论文: [An O(ND) Difference Algorithm and Its Variations](http://www.xmailserver.org/diff2.pdf)

我觉得最有帮助的一篇文章：[Investigating Myers' diff algorithm: Part 1 of 2](http://www.codeproject.com/Articles/42279/Investigating-Myers-diff-algorithm-Part-1-of-2)

我的代码：[Functional & Imperative Implementation of Myers Diff Algorithm](https://gist.github.com/hellmage/77ad87cb62821f1b6371)

这个算法最核心的概念就是那张由两个字符串画成的表格，d和k这两个变量都是帮助你理解一步一步是怎么走的。设横轴（x轴）的字符串为A，纵轴（y轴）的为B，那么得到的路径就是从A变化为B的patch：

- 对角线只会出现在相同字符交叉点左上的格子
- 从表格的左上角开始，只能向右或者向下移动
- 沿着x轴向右移动一步的意义是删掉A中的一个字符
- 沿着y轴向下移动一步的意义是插入B中的一个字符
- 沿着对角线移动一步的意义是不需要删掉或者插入A/B中的任何字符，因为交叉点上的字符是相同的

所以为了用最快最简洁的方式从A变成B，算法的目标就是找到含有对角线尽量多的路线，或者说横向以及纵向移动步数之和最少的路线。d这个变量的意义就是路线之中横向和纵向的移动步数之和。因为延对角线移动不算步数，所以d也可以表示整个路径的步数。k这个变量其实只是对于原论文中的伪代码实现有意义。在用scala这种函数式的方法实现的时候，k就消失了。按照规则，每在这张表格上移动一步，d是肯定会加一的，所以只需要关心移动的方向，d在实现的时候也不会存在。

CodeProject上的那篇文章的d/k表格最形象的解释了应该如何在表格上移动。用广度优先（BFS）实现最好，可以节省很多不必要的搜索。另外实现的时候还可以通过下面两点优化：

- 在当前d所对应的所有点中，合并重复的
- 记录在之前的d之中访问过的所有的点，抛弃当前d中已经访问过的点

我本来想用FP的方式实现的，结果发现……好难……能够找到路径，但是要完全记录下来真是麻烦到爆，干脆就用传统的方式实现吧。验证也不难，就是把产生的patch应用到A上，看看能不能产生B。

看看产生的输出：

```
          patch: =6 -\n =14 +Ok =1 -BadRequest( =5 +pa -o +rse -bj =2 -error +[] =1 - -> error =2 -)
original string: case L\neft(error) => (BadRequest(Json.obj("error" -> error)))
 patched string: case Left(error) => Ok(Json.parse("[]"))
  target string: case Left(error) => Ok(Json.parse("[]"))
```

Patch其实是一个```List[String]```，每个元素是一个单独的行为。行为有三种：

- ```"="```表示没有变化，之后是一个数字，表示有连续多少个字符没有变化
- ```"+"```表示需要插入，之后是一串需要插入的字符
- ```"-"```表示需要删除，之后是一串需要删除的字符。其实这里字符本身并不重要，只需要数量就可以了。这是可以优化的地方

Patch list的顺序是重要的，因为它们的应用是顺序的。仔细看一下上面例子之中的patch就会注意到，把他们用空格连接起来成为一个字符串表示其实是不行的，因为要插入或者删除的字符串之中都有可能有空格。其实用任何分隔符来连接它们都是不好的，最好是表示成一个json格式的字符串，像这样（注意倒数第三个元素）：

```json
[
  "=6",
  "-\n",
  "=14",
  "+Ok",
  "=1",
  "-BadRequest(",
  "=5",
  "+pa",
  "-o",
  "+rse",
  "-bj",
  "=2",
  "-error",
  "+[]",
  "=1",
  "- -> error",
  "=2",
  "-)"
]
```
