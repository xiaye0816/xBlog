---
title: Majority Vote 算法
date: 2019-03-31 16:53:54
description: Find the majority element in a list of values.
tags: [algorithm,majority-vote]
categories: algorithm
---

## 问题描述
给定一个无序list，判断是否有存在某个数的出现次数超过一半。如果存在的话，返回该数。

## 解决思路一
最简单直接的解决方案，是直接对list进行排序。然后，取排序后list中位的数。

由于出现超过一半的数，排序后，无论在list中怎么存放，都会覆盖到中位。
所以如果存在的话，那一定就是该数。

另外，因为可能不存在任何一个数超过一半，所以我们取完该数后，需要再循环一遍list，统计该数的出现次数是否确实超过一半。

#### 伪代码
```python
sort(list)
num = list[length / 2]

count = count(list, num)

if count > length / 2
    return num
else
    return -1
```
完整代码：[SolutionOne.java](https://github.com/xiaye0816/BlogCode/blob/master/src/main/algorithm/majorityvote/solution/SolutionOne.java)

#### 复杂度
因为需要排序，所以复杂度为O (n * logn)

## 解决思路二
直接排序所消耗的性能比较高。

如果不排序的话，还有一种思路，那就是使用位运算。

思路是：如果存在某个数，出现次数超过一半。
那么该数的每一个bit，在整个list中的全部数里，对应bit位上的出现次数，也一定是超过一半的。

基于该思路，我们只需计算出list中，每个bit位的majority是0还是1，最后就能组合出真正的majority。
同样，由于list中该数可能不存在，所以我们需要再循环一遍，确定计算出的数是否出现次数超过一半。

#### 伪代码
假设是4字节32位的int类型：
```python
num = 0

for i in range(32)
    bit_sum = 0

    for item in list
       bit_sum += (item >> i) & 1

    if bit_sum > length / 2
        bit = 1
    else
        bit = 0

    num |= bit << i

count = count(list, num)

if count > length / 2
     return num
else
     return -1
```
完整代码：[SolutionTwo.java](https://github.com/xiaye0816/BlogCode/blob/master/src/main/algorithm/majorityvote/solution/SolutionTwo.java)

#### 复杂度
O (n)，系数32

## 解决思路三
上面的解决方案，已经达到了O(n)的效率了。但是32次循环，看着有点别扭，会不会有更优化的方案呢？
答案是有的。

不知道大家小时候有没有玩过一种叫做"抽老鳖"纸牌游戏。玩法如下：
>首先从一副牌里面随机抽走一张拿掉不用，没有人知道这张牌是什么，这张牌也就是"老鳖"。
>然后洗牌，每个发一堆牌，数量基本差不多就行。
>游戏开始前，如果每个人手里的牌，有两两重复的，就直接把两张都拿掉，扔回牌堆不用。每个人手里剩下的则全是不重复的。
>游戏开始，每个人从下家手里的牌，随机抽过来一张，如果自己手里有重复的，则把两张重复的扔掉，否则就拿在自己手里。
>依次往复，由于有一张单牌"老鳖"对应的牌在大家手里，所以这一张牌一定无法抵消掉，最后别人手里都没牌了，这张牌在谁手里，谁就输了。

为什么提到这种纸牌游戏呢，因为最终的解决方案，跟这个游戏的玩法类似。
把无序list看成是一副纸牌，只不过区别在于，两两丢掉的，不再是相同的"纸牌"，而是每次丢掉不同的一对。

**假如存在某个数超过一半，那么当我们每次丢掉两两不同的一对，那么最后能留下来的，一定就是该数。**

基于以上原理，引出了本次算法的主角：[Majority Vote Algorithm](https://en.wikipedia.org/wiki/Boyer%E2%80%93Moore_majority_vote_algorithm)

#### 伪代码

```python
num = 0
count = 0

for item in list
    if count == 0
        num = item

    if item == num
        count++
    else
        count--

count = count(list, num)

if count > length / 2
     return num
else
     return -1
```
完整代码：[SolutionThree.java](https://github.com/xiaye0816/BlogCode/blob/master/src/main/algorithm/majorityvote/solution/SolutionThree.java)

#### 复杂度
O (n)


## 延伸
假设问题变为出现次数超过1/3、1/4，那么该算法是否就无法使用了呢？
答案是可以的，我们只需要将两两抵消，变成每三个不同的抵消、每四个不同的抵消即可。

所以，记录投票详情的num和count，需要多几对。然后每N个不一样的，抵消一下即可。

#### 伪代码

以出现次数超过1/3的数为例：

```python
num1 = 0
count1 = 0

num2 = 0
count2 = 0

for item in list
    if count1 == 0
        num1 = item
    else if count2 == 0
        num2 = item

    if item == num1
        count1++
    else if item == num2
        count2++
    else
        count1--
        count2--

count1 = count(list, num1)
if count1 > length / 3
     return num1

count2 = count(list, num2)
if count2 > length / 3
     return num2

return -1
```
完整代码：[SolutionExtended.java](https://github.com/xiaye0816/BlogCode/blob/master/src/main/algorithm/majorityvote/solution/SolutionExtended.java)

#### 复杂度
O (n)

## 总结
对于此类，在无序list中，寻找出现次数超过一定数量的问题，可以采用该投票算法，在一次循环中，直接拿到最终结果。

另外，如果原问题是1/3、1/4这种场景，需要注意，同时符合的答案可能有多个。此时需要处理每一个投票结果。