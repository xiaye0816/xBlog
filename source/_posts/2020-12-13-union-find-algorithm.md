---
title: 朋友圈问题的解法
date: 2020-12-13 18:04:21
description: 针对不交集的合并和查询问题的几种解决办法，包括bitmap、无向图深度优先搜索、并查集等
categories: algorithm
tags: [algorithm,union-find,bitmap,dfs]
---
## 问题引入
班上有N个学生，通过一个二维数组，标记每两个学生之间是否是朋友，值为1代表是朋友，值为0代表不是朋友。
朋友的关系可以传递，如果已知 A 是 B 的朋友，B 是 C 的朋友，那么我们可以认为 A 也是 C 的朋友。
所谓的朋友圈，是指所有朋友的集合。

给出一个代表朋友关系的N*N数组，计算班上朋友圈的数量。

## 示例
** 输入：**
```
[
  [1,1,0],
  [1,1,0],
  [0,0,1]
]
```
** 输出：**
2
** 解析：**
已知学生 0 和学生 1 互为朋友，他们在一个朋友圈。第2个学生自己在一个朋友圈。所以返回 2。

## 解法1：暴力破解
通过set存储朋友圈关系，set内的元素就是学生的序号。
每当出现一个新的朋友圈set时，判断新的朋友圈和旧朋友圈是否有重叠。有重叠的话，就将多个重叠的集合融合成一个集合。

** 完整代码 **
[BruteForceSolution.java](https://github.com/xiaye0816/BlogCode/blob/master/src/main/algorithm/disjointset/solution/bruteforce/BruteForceSolution.java)

** 复杂度 **
O (N^4)

** 总结 **
复杂度过高，没有实际应用价值。

## 解法2：基于BitMap的解法
思路是使用一个bitmap存储单个朋友圈的状态，bitmap的下标是学生序号，值为1代表该朋友圈包含该学生，值为0代表不包含。
通过两个bitmap的与操作(And)，判断两个朋友圈是否存在重叠。
通过两个bitmap的或操作(Or)，实现两个朋友圈的融合。

** 完整代码 **
[BitMapSolutionOne.java](https://github.com/xiaye0816/BlogCode/blob/master/src/main/algorithm/disjointset/solution/bitmap/BitMapSolutionOne.java)

** 复杂度 **
O (N^3)

** 总结 **
该解法巧妙的用bitmap存储了状态，并且用位运算，高效的实现了集合的重叠判断和融合操作。
缺点是当集合的成员基数较大，或者基本自身的数量很多时，会占用过大空间，或者bitmap出现溢出问题。（单个long类型最大支持8字节64个成员）

## 解法3：基于BitMap的解法 - 优化了溢出问题
通过long整型存储状态，限制了能最大支持的成员数量上限，超出上限的话，bitmap自身会出现数值溢出问题。
这里可以优化成通过BitInteger存储。

** 完整代码 **
[BitMapSolutionTwo.java](https://github.com/xiaye0816/BlogCode/blob/master/src/main/algorithm/disjointset/solution/bitmap/BitMapSolutionTwo.java)

** 复杂度 **
O (k * N^3), k = 成员基数 / 32，BitInteger内部会将移位操作拆成每4个字节一次

** 总结 **
使用bitmap的解法，是一个很新颖的思路。
但是要注意，如果在成员基数很大、朋友圈数量很大的场景下强行使用，性能和空间反而可能表现比较差。

## 解法4：基于无向图深度优先搜索的解法
整个朋友圈关系可以看做是一个无向图，图中每个节点是一个学生，两个节点之间相连，代表两个学生是朋友关系。
给定的二维数组关系中，每个值为1的记录，在无向图中代表两个节点相连。如下图：
![](https://tva1.sinaimg.cn/large/0081Kckwly1glmf8jlb8xj31480pcwih.jpg)

基于这样的思路，我们可以做一次深度优先遍历搜索。基于每个学生的维度，搜索该学生的朋友以及朋友的朋友，依次类推，遍历标记当前朋友圈包含的所有学生。

当每次新检索到一个未被标记的学生时，则代表当前是一个新的朋友圈，朋友圈数量+1即可。

当遍历完成后，就能得到朋友圈数量。

** 完整代码 **
[DfsSolution.java](https://github.com/xiaye0816/BlogCode/blob/master/src/main/algorithm/disjointset/solution/dfs/DfsSolution.java)

** 复杂度 **
O (N^2)

## 解法5：基于并查集的解法
并查集是一种基于“有根树”的树形数据结构，提供了查找和合并两个基础操作。

> 查找：查找一个节点的根节点
> 合并：合并两个根节点成一棵树（合并后根节点相同）

通过并查集记录朋友圈状态，一颗并查集树下的所有节点同属一个朋友圈。

通过数组实现并查集数据结构，节点的每个元素的索引就是学生的序号，节点的值是父节点的索引，值为-1代表根节点。同一个根节点下的所有节点都同属一个树。

![](https://tva1.sinaimg.cn/large/0081Kckwly1glmg69t5j5j30mc0b0myj.jpg)

初始状态下，每个学生都是根节点。然后遍历每个关系，通过并查集的查找操作找到根节点，判断是否属于同一个树，如果不是的话，执行并查集的合并操作，将两个树合并。

最终，根节点的数量就是朋友圈的数量。

** 完整代码 **
[UnionFindSolutionOne.java](https://github.com/xiaye0816/BlogCode/blob/master/src/main/algorithm/disjointset/solution/unionfind/UnionFindSolutionOne.java)

** 复杂度 **
O (N^3)

** 总结 **
未经优化的并查集，可能会在极端情况下，多次合并后树的深度过深，导致性能退化比较严重的情况。（类似二叉树退化成链表）

## 解法6：基于并查集的解法 - 按秩合并优化和路径压缩优化
基于解法5，为了防止深度过深，可以有两个思路的优化：

** 优化1：按秩合并 **
在执行并查集的合并操作时，总是让小树合并到大树上，这样合并后整体的深度不会增加。为了实现这个优化，我们需要用一个额外的数组，记录节点的深度。

** 优化2：路径压缩 **
在执行并查集的查找操作时，顺便将当前节点所在的树扁平化，即查找过程中，将每个节点的父节点都指向根节点，这样整体的深度就压缩成了1。

** 完整代码 **
[UnionFindSolutionTwo.java](https://github.com/xiaye0816/BlogCode/blob/master/src/main/algorithm/disjointset/solution/unionfind/UnionFindSolutionTwo.java)

** 复杂度 **
O (k * N^2)，k < 5

下图引用自并查集[wikipedia](https://zh.wikipedia.org/wiki/%E5%B9%B6%E6%9F%A5%E9%9B%86)：
![](https://tva1.sinaimg.cn/large/0081Kckwly1glmggsh59oj31xa0dc42v.jpg)

** 总结 **
使用了按秩合并优化和路径压缩优化后，并查集的整体平均性能提升了一个数量级。

## 写在最后 ##
以上就是朋友圈、群覆盖等不交集类问题的几种常规解法。

对于朋友圈问题，由于需要遍历整个朋友圈的二维数组关系，所以基础的N^2的复杂度是必不可少的。
在此之上，对于深度优先搜索和并查集的解法，都可以在不再提升复杂度数量级的情况下，解决问题。

最后，了解多种解法的目的，更主要是开阔自己的视野和思路，在实际场景中，就可以根据具体的情况运用最合适的方案。