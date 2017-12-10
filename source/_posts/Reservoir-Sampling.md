title: Algorithms# 001：Reservoir Sampling
date: { { date } }
categories: Algorithms
tags: [算法,抽样]
description:
---
**摘要**：Reservoir Sampling 是一系列的随机算法，其目的在于从包含 n 个项目的集合 S 中选取 k 个样本，其中 n 为一很大或未知的数量，尤其适用于不能把所有 n 个项目都存放到主内存的情况，Reservoir Sampling 使用固定长度的链表，从而保证内存大小的可控化。
<!--more-->

试想一下如何从一个大海中装起一杯水，而整个大海中的所有水分子都有相同的概率被装进这个杯子里，当然你可以把整个大海搅浑后从中舀出一杯，然而问题是我们并没有这么大型的搅拌机，那么我们还能够怎么做？

一个行之有效的办法就是让海水中的每一个水分子一粒一粒都装入装入水杯中，同时由于杯子的容量是有限的，每进入一个水分子就会溢出一个水分子，溢出的水分子将被浪费掉，假设杯中的每一滴水都有相同的概率被挤出杯外，那么当海洋中的每一滴水都进入杯子后，我们就能够获得一杯集成大海精华的海水，而这时需要的只是一个杯子大小的搅拌机
***

# Google 面试题
Google 曾经有一道面试题，十分有趣：

>I have a linked list of numbers of length N. N is very large and I don’t know in advance the exact value of N.
How can I most efficiently write a function that will return k completely random numbers from the list

题目非常简单：有 N 个元素的链表，事先不知道有多长，写一个函数可以高效地从其中取出 k 个随机数。
***
# Introduction

在知道文件行数的情况下，我们可以很容易的用 C 运行库的 rand 函数随机的获得一个行数，从而随机的取出一行，但是，当前的情况是不知道行数，这样如何求呢？我们需要一个概念来帮助我们做出猜想，来使得对每一行取出的概率相等，也即随机，水塘抽样就是这么一个概念,Wikipedia 上对水潭抽样是这么介绍的：

>水塘抽样是一系列的随机算法，其目的在于从包含 n 个项目的集合 S 中选取 k 个样本，其中 n 为一很大或未知的数量，尤其适用于不能把所有 n 个项目都存放到主内存的情况。

![](http://7nar5o.com1.z0.glb.clouddn.com/Reservoir%20Sampling.png)

最常见例子为 Jeffrey Vitter 在其论文 Random Sampling with a Reservoir 中所提及的算法 R。

参照 Dictionary of Algorithms and Data Structures 所载的 O(n)算法，包含以下步骤（假设阵列 S 以0开始标示）：

	1.从 S 中抽取首 k 项放入「水塘」中
	2.對於每一個 S[j]項（j ≥ k）：
	3.隨機產生一個範圍從0到 j 的整數 r
	4.若 r < k 則把水塘中的第 r 項換成 S[j]項

即在0-j 项中生成随机数 r，被挑中的 r 就被丢弃掉，同时后边的队列补充一个 j 进来,因为我们 array R[k]类似一个 reservoir 水库（蓄水池）,所以该算法取名为水塘抽样

>The algorithm creates a "reservoir" array of size k and populates it with the first k items of S. It then iterates through the remaining elements of S until Sis exhausted. At the $i^{th}$ element of S, the algorithm generates a random number j between 1 and i. If j is less than k, the $j^{th}$ element of the reservoir array is replaced with the ith element of S. In effect, for all i, the $i^{th}$ element of S is chosen to be included in the reservoir with probability $k/i$. Similarly, at each iteration the $j^{th}$ element of the reservoir array is chosen to be replaced with probability ${j/k}*{k/i}$, which simplifies to ${j/i}$. It can be shown that when the algorithm has finished executing, each item in S has equal probability (i.e. k/length(S)) of being chosen for the reservoir.

# Java 代码实现

```
final int SAMPLE_COUNT = 10;
int index = 0;

Object[] samples = new Object[SAMPLE_COUNT];
Random rnd = new Random();
for (Object element in stream) {
    if (index < SAMPLE_COUNT) {
        samples[index] = element;
    } else {
        int r = rnd.nextInt(0, index);
        if (r < SAMPLE_COUNT) {
            samples[r] = element;
        }
    }
    index++;
}
```

# 算法证明
每个数在水潭中留下的概率

$$ {1 \over j} \times {j \over {j+1}} \times \dfrac{j+1}{j+2} \times \dfrac{j+2}{j+3} \times {···} \times \dfrac{n-1}{n} = {1 \over k} $$

