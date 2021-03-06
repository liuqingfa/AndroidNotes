- [ 漫画：什么是 B-树？](https://mp.weixin.qq.com/s?__biz=MzI1MTIzMzI2MA==&mid=2650561220&idx=1&sn=2a6d8a0290f967027b1d54456f586405&chksm=f1feec47c689655113fa65f7911a1f59bbd994030ad685152b30e53d643049f969eefaa13058&scene=21#wechat_redirect)
- [漫画：什么是 B+ 树？](https://mp.weixin.qq.com/s?__biz=MzI1MTIzMzI2MA==&mid=2650561244&idx=1&sn=df3abafd3aa2f5a3abfe507bfc26982f&chksm=f1feec5fc6896549f89cbb82ee3d8010c63da76814030b285fa29322795de512ccca207064ee&scene=21#wechat_redirect)
- [【经典数据结构】B树与B+树](https://www.cnblogs.com/vincently/p/4526560.html)


-----

### 引子

从算法逻辑上将，二叉查找树的查找速度和比较次数都是最小的，但是必须考虑磁盘IO。

例如一棵树


```
              9
       /             \
      5              13
    /    \         /    \
   2 	  7       11     15
 /  \    /   \   /        \
1    3	6    8  10         12

```

查找10时，需要进行4次IO（9-13-11-10）

所以需要将原本“瘦高”的树变成“矮胖”的树。

### B-树

**千万别读B减数，哈哈哈哈**

**B树是一种多路平衡查找树，它的每个节点最多包含K个孩子，K被称为B数的阶。K的大小取决于磁盘页的大小。**


下面来具体介绍一下B-树（Balance Tree），一个m阶的B树具有如下几个**特征**：

- 1.根结点至少有两个子女。
- 2.每个中间节点都包含k-1个元素和k个孩子，其中 m/2 <= k <= m
- 3.每一个叶子节点都包含k-1个元素，其中 m/2 <= k <= m
- 4.所有的叶子结点都位于同一层。
- 5.每个节点中的元素从小到大排列，节点当中k-1个元素正好是k个孩子包含的元素的值域分划。

![](https://github.com/sparkfengbo/AndroidNotes/blob/master/PictureRes/SJJG/B-%E6%A0%91.png?raw=true)

B树查询中的比较次数不比二叉查找树少，但是相比磁盘速度，内存中的比较耗时几乎可以忽略。

所以只要树的高度足够低，IO次数足够少，就可以提升性能。

**应用**

文件系统以及部分数据库索引，例如非关系型数据库MongoDB

### B+树

B+树是B树的一种变体，**有着比B-树更高的查询性能。**

一个m阶的B+树具有如下几个**特征**：

- 1.有k个子树的中间节点包含有k个元素（B树中是k-1个元素），每个元素不保存数据，只用来索引，所有数据都保存在叶子节点。
- 2.所有的叶子结点中包含了全部元素的信息，及指向含这些元素记录的指针，且叶子结点本身依关键字的大小自小而大顺序链接。
- 3.所有的中间节点元素都同时存在于子节点，在子节点元素中是最大（或最小）元素。

![](https://github.com/sparkfengbo/AndroidNotes/blob/master/PictureRes/SJJG/B+%20%E6%A0%91.png?raw=true)

根节点的最大元素 等同于 整个B+树的最大元素，所有的叶子节点包含了全量元素信息，
每个叶子节点都带有指向下一个节点的指针，形成了一个有序链表。

**B+树的优势在于**

- 1.查询性能好，IO次数少。B+数的中间节点没有卫星数据（中间节点只保存索引，所有卫星数据保存在叶子节点中），所以同样大小的磁盘页可以容纳更多的节点元素。相当于比B-树更矮胖。
- 2.查询性能稳定。B+树的查询必须最终查找到叶子节点，而B-树只要找到匹配元素即可，B树最好的情况就是查找根节点，最坏是查找到叶子节点。
- 3.范围查询简便。B+树的范围查询只要遍历叶子节点的链表，而B-树的范围查询需要进行繁琐的中序遍历。



