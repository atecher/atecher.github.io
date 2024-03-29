---
title: 浅谈算法和数据结构之二叉查找树
date: 2016-12-11 19:00:10
categories: "Data Structures"
tags: 
     - Data Structures
     - Tree
---

无序链表在插入的时候具有较高的灵活性，而有序数组在查找时具有较高的效率，本文介绍的二叉查找树(Binary Search Tree，BST)这一数据结构综合了以上两种数据结构的优点。

二叉查找树具有很高的灵活性，对其优化可以生成平衡二叉树，红黑树等高效的查找和插入数据结构，后文会一一介绍。

<!-- more -->

### 定义

二叉查找树（Binary Search Tree），也称有序二叉树（ordered binary tree）,排序二叉树（sorted binary tree），是指一棵空树或者具有下列性质的二叉树：

1. 若任意节点的左子树不空，则左子树上所有结点的值均小于它的根结点的值；
2. 若任意节点的右子树不空，则右子树上所有结点的值均大于它的根结点的值；
3. 任意节点的左、右子树也分别为二叉查找树。
4. 没有键值相等的节点（no duplicate nodes）。

如下图，这个是普通的二叉树：

![binary tree][1]

在此基础上，加上节点之间的大小关系，就是二叉查找树：

![binary search tree][2]


### 实现

在实现中，我们需要定义一个内部类Node，它包含两个分别指向左右节点的Node，一个用于排序的Key，以及该节点包含的值Value，还有一个记录该节点及所有子节点个数的值Number。

```csharp
public class BinarySearchTreeSymbolTable<TKey, TValue> : SymbolTables<TKey, TValue> where TKey : IComparable<TKey>, IEquatable<TValue>
{
    private Node root;
    private class Node
    {
        public Node Left { get; set; }
        public Node Right { get; set; }
        public int Number { get; set; }
        public TKey Key { get; set; }
        public TValue Value { get; set; }

        public Node(TKey key, TValue value, int number)
        {
            this.Key = key;
            this.Value = value;
            this.Number = number;
        }
    }
...
}
```

#### 查找

查找操作和二分查找类似，将key和节点的key比较，如果小于，那么就在Left Node节点查找,如果大于，则在Right Node节点查找，如果相等，直接返回Value。

![SearchhitAndSearchMissinBST][3]

该方法实现有迭代和递归两种。

递归的方式实现如下：

```csharp
public override TValue Get(TKey key)
{
    TValue result = default(TValue);
    Node node = root;
    while (node != null)
    {

        if (key.CompareTo(node.Key) > 0)
        {
            node = node.Right;
        }
        else if (key.CompareTo(node.Key) < 0)
        {
            node = node.Left;
        }
        else
        {
            result = node.Value;
            break;
        }
    }
    return result;
}
```

迭代的如下：

```java
public TValue Get(TKey key)
{
    return GetValue(root, key);
}

private TValue GetValue(Node root, TKey key)
{
    if (root == null) return default(TValue);
    int cmp = key.CompareTo(root.Key);
    if (cmp > 0) return GetValue(root.Right, key);
    else if (cmp < 0) return GetValue(root.Left, key);
    else return root.Value;
}
```

#### 插入

插入和查找类似，首先查找有没有和key相同的，如果有，更新；如果没有找到，那么创建新的节点。并更新每个节点的Number值，代码实现如下：

```java
public override void Put(TKey key, TValue value)
{
    root = Put(root, key, value);
}

private Node Put(Node x, TKey key, TValue value)
{
    //如果节点为空，则创建新的节点，并返回
    //否则比较根据大小判断是左节点还是右节点，然后继续查找左子树还是右子树
    //同时更新节点的Number的值
    if (x == null) return new Node(key, value, 1);
    int cmp = key.CompareTo(x.Key);
    if (cmp < 0) x.Left = Put(x.Left, key, value);
    else if (cmp > 0) x.Right = Put(x.Right, key, value);
    else x.Value = value;
    x.Number = Size(x.Left) + Size(x.Right) + 1;
    return x;
}

private int Size(Node node)
{
    if (node == null) return 0;
    else return node.Number;
}

```
  插入操作图示如下：
  
![insert into BST][4]

下面是插入动画效果：

![insert into BST][5]

随机插入形成树的动画如下，可以看到，插入的时候树还是能够保持近似平衡状态：

![Insert keys random order in BST][6]

#### 最大最小值

如下图可以看出，二叉查找树的最大最小值是有规律的：

![the max and min item in bst][7]

从图中可以看出，二叉查找树中，最左和最右节点即为最小值和最大值，所以我们只需迭代调用即可。

```java
public override TKey GetMax()
{
    TKey maxItem = default(TKey);
    Node s = root;
    while (s.Right != null)
    {
        s = s.Right;
    }
    maxItem = s.Key;
    return maxItem;
}

public override TKey GetMin()
{
    TKey minItem = default(TKey);
    Node s = root;
    while (s.Left != null)
    {
        s = s.Left;
    }
    minItem = s.Key;
    return minItem;
}
```

以下是递归的版本：

```java
public TKey GetMaxRecursive()
{
    return GetMaxRecursive(root);
}

private TKey GetMaxRecursive(Node root)
{
    if (root.Right == null) return root.Key;
    return GetMaxRecursive(root.Right);
}

public TKey GetMinRecursive()
{
    return GetMinRecursive(root);
}

private TKey GetMinRecursive(Node root)
{
    if (root.Left == null) return root.Key;
    return GetMinRecursive(root.Left);
}
```

#### Floor和Ceiling

查找Floor(key)的值就是所有<=key的最大值，相反查找Ceiling的值就是所有>=key的最小值，下图是Floor函数的查找示意图：

![floor  function in BST][8]

以查找Floor为例，我们首先将key和root元素比较，如果key比root的key小，则floor值一定在左子树上；如果比root的key大，则有可能在右子树上，当且仅当其右子树有一个节点的key值要小于等于该key；如果和root的key相等，则floor值就是key。根据以上分析，Floor方法的代码如下，Ceiling方法的代码类似，只需要把符号换一下即可：

```java
public TKey Floor(TKey key)
{
    Node x = Floor(root, key);
    if (x != null) return x.Key;
    else return default(TKey);
}

private Node Floor(Node x, TKey key)
{
    if (x == null) return null;
    int cmp = key.CompareTo(x.Key);
    if (cmp == 0) return x;
    if (cmp < 0) return Floor(x.Left, key);
    else
    {
        Node right = Floor(x.Right, key);
        if (right == null) return x;
        else return right;
    }
}
```

#### 删除

删除元素操作在二叉树的操作中应该是比较复杂的。首先来看下比较简单的删除最大最小值得方法。

以删除最小值为例，我们首先找到最小值，及最左边左子树为空的节点，然后返回其右子树作为新的左子树。操作示意图如下：

![delete minimun in BST][9]

代码实现如下：
```java
public void DelMin()
{
    root = DelMin(root);
}

private Node DelMin(Node root)
{
    if (root.Left == null) return root.Right;
    root.Left = DelMin(root.Left);
    root.Number = Size(root.Left) + Size(root.Right) + 1;
    return root;
}
```
删除最大值也是类似。

现在来分析一般情况，假定我们要删除指定key的某一个节点。这个问题的难点在于：删除最大最小值的操作，删除的节点只有1个子节点或者没有子节点，这样比较简单。但是如果删除任意节点，就有可能出现删除的节点有0个，1 个，2个子节点的情况，现在来逐一分析。

**当删除的节点没有子节点时**，直接将该父节点指向该节点的link设置为null。

![delete node which has 0 childrens][10]

**当删除的节点只有1个子节点时**，将该自己点替换为要删除的节点即可。

![delete node which has 1 childrens][11]

**当删除的节点有2个子节点时**，问题就变复杂了。

假设我们删除的节点t具有两个子节点。因为t具有右子节点，所以我们需要找到其右子节点中的最小节点，替换t节点的位置。这里有四个步骤：

1. 保存带删除的节点到临时变量t
2. 将t的右节点的最小节点min(t.right)保存到临时节点x
3. 将x的右节点设置为deleteMin(t.right)，该右节点是删除后，所有比x.key最大的节点。
4. 将x的做节点设置为t的左节点。

整个过程如下图：

![delete node which has 2 childrens][12]

对应代码如下：

```java
public void Delete(TKey key)
{
    root =Delete(root, key);
        
}

private Node Delete(Node x, TKey key)
{
    int cmp = key.CompareTo(x.Key);
    if (cmp > 0) x.Right = Delete(x.Right, key);
    else if (cmp < 0) x.Left = Delete(x.Left, key);
    else
    {
        if (x.Left == null) return x.Right;
        else if (x.Right == null) return x.Left;
        else
        {
            Node t = x;
            x = GetMinNode(t.Right);
            x.Right = DelMin(t.Right);
            x.Left = t.Left;
        }
    }
    x.Number = Size(x.Left) + Size(x.Right) + 1;
    return x;
}

private Node GetMinNode(Node x)
{
    if (x.Left == null) return x;
    else return GetMinNode(x.Left); 
}
```

以上二叉查找树的删除节点的算法不是完美的，因为随着删除的进行，二叉树会变得不太平衡，下面是动画演示。

![delete node in BST][13]

### 三 分析

二叉查找树的运行时间和树的形状有关，树的形状又和插入元素的顺序有关。在最好的情况下，节点完全平衡，从根节点到最底层叶子节点只有lgN个节点。在最差的情况下，根节点到最底层叶子节点会有N各节点。在一般情况下，树的形状和最好的情况接近。

![BST Tree shape][14]


在分析二叉查找树的时候，我们通常会假设插入的元素顺序是随机的。对BST的分析类似与快速排序中的查找：

![BST and quick sort partition][15]

BST中位于顶部的元素就是快速排序中的第一个划分的元素，该元素左边的元素全部小于该元素，右边的元素均大于该元素。

对于N个不同元素，随机插入的二叉查找树来说，其平均查找/插入的时间复杂度大约为2lnN，这个和快速排序的分析一样，具体的证明方法不再赘述，参照快速排序。

 

### 四 总结

有了前篇文章 二分查找的分析，对二叉查找树的理解应该比较容易。下面是二叉查找树的时间复杂度：

![analysis of binary search tree][16]

它和二分查找一样，插入和查找的时间复杂度均为lgN，但是在最坏的情况下仍然会有N的时间复杂度。原因在于插入和删除元素的时候，树没有保持平衡。我们追求的是在最坏的情况下仍然有较好的时间复杂度，这就是后面要讲的平衡查找树的内容了。下文首先讲解平衡查找树的最简单的一种：2-3查找树。

希望本文对您了解二叉查找树有所帮助。

>本文系转载文章，原作者为yangecnu，原文链接:[请点此处][17]。



  [1]: //qn.atecher.com/mts/20180417/3852386529018880
  [2]: //qn.atecher.com/mts/20180417/3852386531558400
  [3]: //qn.atecher.com/mts/20180417/3852386535556096
  [4]: //qn.atecher.com/mts/20180417/3852386541241344
  [5]: //qn.atecher.com/mts/20180417/3852386541077504
  [6]: //qn.atecher.com/mts/20180417/3852393206400000
  [7]: //qn.atecher.com/mts/20180417/3852386544337920
  [8]: //qn.atecher.com/mts/20180417/3852386547647488
  [9]: //qn.atecher.com/mts/20180417/3852386547663872
  [10]: //qn.atecher.com/mts/20180417/3852386549728256
  [11]: //qn.atecher.com/mts/20180417/3852386552349696
  [12]: //qn.atecher.com/mts/20180417/3852386510390272
  [13]: //qn.atecher.com/mts/20180417/3852393206531072
  [14]: //qn.atecher.com/mts/20180417/3852386510504960
  [15]: //qn.atecher.com/mts/20180417/3852386513814528
  [16]: //qn.atecher.com/mts/20180417/3852386522203136
  [17]: http://www.cnblogs.com/yangecnu/p/Introduce-Binary-Search-Tree.html
  
  