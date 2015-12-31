---
layout: post
title:  "前缀字符串计算的性能比较"
date:   2015-12-31 19:23:28
categories: scala prefix performance
---

今天在研究diff算法的时候看到Neil Fraser的一个实验表明，用Binary Search来寻找两个字符串的common prefix/suffix竟然比线性寻找的算法更快。他还给了一个[实验结果](https://neil.fraser.name/news/2007/10/09/)，的确蛮出乎人意料的。我手贱就赶紧用scala验证了一下。

------
Gist: https://gist.github.com/hellmage/cd581084dc26c90838cd

{% highlight ruby %}
import java.lang.System

object BinVsLin {

  def commonPrefixBin(t1: String, t2: String): Int = {
    if (t1.isEmpty && t2.isEmpty)
      0
    else if (t1.isEmpty || t2.isEmpty)
      0
    else if ((t1.length == 1 && t1 != t2.take(1)) || (t2.length == 1 && t2 != t1.take(1)))
      0
    else {
      val mid = if (t1.length > t2.length) t2.length / 2 else t1.length / 2
      if (t1.take(mid) == t2.take(mid))
        mid + commonPrefixBin(t1.drop(mid), t2.drop(mid))
      else
        mid + commonPrefixBin(t1.take(mid), t2.take(mid))
    }
  }

  def commonPrefixLin(t1: String, t2: String): Int = {
    if (t1.isEmpty && t2.isEmpty)
      0
    else if (t1.isEmpty || t2.isEmpty)
      0
    else if (t1.take(1) == t2.take(1))
      1 + commonPrefixLin(t1.drop(1), t2.drop(1))
    else
      0
  }

  def measure(t1: String, t2: String, n: Int)(f: (String, String) => Int): Long = {
    val start = System.currentTimeMillis()
    (1 to n).foreach(_ => f(t1, t2))
    val end = System.currentTimeMillis()
    end - start
  }

  def main(args: Array[String]): Unit = {
    val t1 = "The common prefix and suffix algorithms in my Diff, Match and Patch library uses a binary search of substrings to locate the divergent point. I claim this to be O(log n) instead of O(n). In the year since this was first published I've been taking a lot of heat from programmers who look at the algorithm and realize what the substring operation is doing at the processor level. They write back, or burst out laughing when they realize that my 'optimization' actually evaluates to O(n log n). I patiently explain that high-level languages don't work that way, but the skeptics remain unconvinced."
    val t2 = "The common prefix and suffix algorithms in my diff, Match and Patch library uses a binary search of substrings to locate the divergent point. I claim this to be O(log n) instead of O(n). In previous year since this was first published I've been taking a lot of heat from programmers who look at the algorithms and realize what the substring operation is doing at the processor level. They write back, call me up, or burst our laughing when they realized that my 'optimization' actually evaluates to O(n log n). I patiently explain that high-level languages don't work that way, but the skeptics remain unconvinced."
    List(1, 10, 100, 1000, 10000, 100000, 1000000).foreach { n =>
      val bin = measure(t1, t2, n)(commonPrefixBin)
      val lin = measure(t1, t2, n)(commonPrefixLin)
      println(s"$n, $bin, $lin, ${100 * bin / lin.toFloat}%")
    }
  }

}
{% endhighlight scala %}

实验结果（单位ms，从左至右依次是循环次数，二分搜索耗时，线性搜索耗时和百分比）：

> 1, 0, 1, 0.0%
>
> 10, 0, 2, 0.0%
>
> 100, 2, 7, 28.571428%
>
> 1000, 5, 40, 12.5%
>
> 10000, 40, 127, 31.496063%
>
> 100000, 99, 1015, 9.753695%
>
> 1000000, 758, 9959, 7.611206%

果然！
