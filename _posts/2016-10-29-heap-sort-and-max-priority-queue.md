---
layout: post
permalink: /heap-sort-and-max-priority-queue/
title: 堆排序以及最大优先队列
date: 2016-10-29T15:58:22
lang: zh-CN
---


堆排序（heapsort）是一种比较快速的排序方式，它的时间复杂度为*`O(nlgn)`*，而且堆排序具有`空间原址性`：即任何时候只需要有限（常数个）的空间来存储临时数据。而且堆排序还被应用在构造优先级队列中，本文将会用Java实现一个最大堆，并利用最大堆实现优先级队列。
## 最大堆的性质
1. 是一棵近似的完全二叉树，除了最底层，其它是全满，且从左向右填充。
2. 树的每个节点对应数组一个元素，根节点对应数组下标`0`元素。
3. 对于下标`i`，它的父节点下标为`(i + 1) / 2 - 1`，左孩子节点下标为`i * 2 + 1`，右孩子节点下标为`i * 2 + 2`。
4. 最大堆中，每个结点都必须大于等于左右孩子节点。

![file](https://static.lufficc.com/image/48ed0eaff594d6907d66d97fcf4adb56.png)

![file](https://static.lufficc.com/image/f9a755efd9abad9f259f555c1fd365c3.png)

## 堆排序

#### 1.维护最大堆

为了建立一个最大堆，我们需要设计一个算法来维护最大堆的性质。对于节点`i`，我们假设它们的左右子树都是最大堆，而节点`i`因为小于左右孩子而违背了最大堆的性质，因此我们需要对节点`i`进行`逐级下降`，从而使得以节点`i`为根结点的子树满足最大堆的性质。

```
private void maxHeapify(int[] a, int i, int length) {
    int l = left(i);
    int r = right(i);
    int max;
		if (l < length && a[l] > a[i]) {
        max = l;
    } else {
        max = i;
    }
    if (r < length && a[r] > a[max]) {
        max = r;
    }
    if (max != i) {
        swap(a, i, max);
        maxHeapify(a, max, length);
    }
  }
```
上述代码的解释是：
对于节点`i`（即数组下标`i`）,首先获取它的左右子孩子下标，分别存储到`l`，`r`变量中，其中`left`和`right`函数可以根据最大堆的性质得到，如下：
```
    private int left(int i) {
        return i * 2 + 1;
    }

    private int right(int i) {
        return i * 2 + 2;
    }
```

变量length是要维护的数组的长度，它的值`<=`数组的真实长度。临时变量`max`用来存储`节点i`，`左孩子`和`右孩子`这三者最大者的下标。因此，首先比较`左孩子a[l]`是否大于`节点i`，是的话将左孩子下标`l`赋值给`max`，否则，当前最大者仍为节点`i`，并把`i`的值赋给`max`。然后当前最大者和`右孩子`比较，小于`右孩子`的话就把下标`r`赋值给`max`。到目前为止，我们找到了`节点i`，`左孩子`和`右孩子`这三者之中的最大者的下标`max`，如果`节点i`本来就是最大的，那么就不需要维护了，所以函数结束。否则的话，有孩子节点大于`节点i`，因此我们需要将`节点i`和较大的那个孩子进行交换，即`swap(a, i, max)`，`swap`函数如下：
```
    private void swap(int[] a, int i, int j) {
        int tmp = a[i];
        a[i] = a[j];
        a[j] = tmp;
    }
```

交换之后，节点`max`的值变成了较小的原来节点`i`的值，因此，以节点`max`为根节点的子树可能违背最大堆的性质。注意到此时问题和一开始一模一样，只是节点`i`变成了节点`max`，因此递归调用`maxHeapify(a, max, length)`即可。

如果把第一张图的节点`1`的值改为`1`，这时从节点`1`开始调用`maxHeapify(a, 1, a.length`，整个流程如下：

![file](https://static.lufficc.com/image/5335225c6577f7107c4ebf6b4c1c9909.png)

![file](https://static.lufficc.com/image/4c5e9d7f835b938be62c2f086842b42b.png)

![file](https://static.lufficc.com/image/1ccfee5037ae69b3beb56d4564d2a3f9.png)

可以看到，在经历两次递归调用，逐级下沉，原来因为`1`节点变小，而被打破的最大堆，又重新维护完成。

#### 2.建立最大堆

我们采用自底向上的方法来建立一个最大堆。通过观察发现，下标为`a.length / 2` 到`a.length - 1`的节点均为叶子节点，没有子孩子。**这意味着，它们已经是一个最大堆，只是仅有一个元素**。因此，我们只需要自底向上，对其他节点调用`maxHeapify`来维护最大堆即可：
```
    private void buildMaxHeap(int a[]) {
        for (int i = a.length / 2 + 1; i >= 0; i--) {
            maxHeapify(a, i);
        }
    }
```
注意，循环变量从`a.length / 2 + 1`开始，因为`a.length / 2` 到`a.length - 1`的节点满足最大堆，即`a.length / 2 + 1`的左右子孩子（如果有的话）是最大堆，符合使用`maxHeapify`调用条件。如果我们对数组`[10, 8, 11, 8, 14, 9, 4, 1, 17]`进行建立最大堆，那么输出数组为`[17, 14, 11, 8, 10, 9, 4, 1, 8]`:
![file](https://static.lufficc.com/image/041c010526c57459c279f27ef360a380.png)

![file](https://static.lufficc.com/image/dac3dfac3f5413863611cedc0e78f97f.gif)

#### 3.堆排序

在第二步建立完成最大堆之后，注意到整个数组的** 最大元素 **总是在最大堆的第一个，即`a[0]`。**这时，如果我们拿走第一个元素，放到数组的最后一个位置，然后对第一个元素进行维护最大堆`maxHeapify`，如此循环进行，便可完成整个数组的排序**。
```
    public void heapSort(int[] a) {
        buildMaxHeap(a);//首先建立最大堆，完成后第一个元素为最大值
        int length = a.length;
        for (int i = a.length - 1; i >= 1; i--) {
            swap(a, i, 0);//将第一个最大的元素移到后面，并且在maxHeapify的过程中通过减小length忽略它
            length--;
            maxHeapify(a, 0, length);
        }
    }
```
这样我们就完成了对数组的从小到大排序。

## 最大优先队列
堆排序是一个优秀的算法，在操作系统中可以利用最大堆实现最大优先队列来实现共享计算机系统的作业调度。最大优先队列记录各个作业之间的相对优先级，当某个作业中断后选出具有最高优先级的队列来执行。最大优先级队列应该支持如下操作：
1. `maximum()`：返回堆的最大值。
2. `extractMax()`：返回堆的最大值并从堆中删除。
3. `heapIncreaseKey(i, key)`：将下标为`i`的元素增大为`key`。
4. `maxHeapInsert(key)`：将元素`key`插入到堆中。


`maximum()`最简单，直接返回第一个元素：
```
    public int maximum() {
        return a[0];
    }
```

`extractMax`也很简单，取出第一个后，只需把最后一个元素放到第一个，然后对第一个元素进行维护最大堆即可：
```
public int extractMax() {
    int max = a[0];
    a[0] = a[a.length - 1];
    int[] newA = new int[a.length - 1];
    System.arraycopy(a, 0, newA, 0, newA.length);
    maxHeapify(newA, 0, newA.length);
    return max;
}
```

![file](https://static.lufficc.com/image/fe4d61ff68ef634a93e8bf5e94b4f9b6.gif)

`heapIncreaseKey(i, key)`会增大下标为`i`的元素为`key`。首先将`a[i]`的值更新为`key`，因为增大的`a[i]`关键字可能会违背最大堆的性质，因此我们需要对`a[i]`进行`逐级上升`。即将当前元素逐级与父节点比较，如果大于父节点，则与父节点进行交换，一直到当前元素小于父节点为止：
```
public void heapIncreaseKey(int i, int key) {
    if (key > a[i]) {
        a[i] = key;
        while (i > 0 && a[parent(i)] < a[i]) {  //逐级上升
            swap(a, parent(i), i);
            i = parent(i);
        }
    } else {
        throw new IllegalArgumentException("key is too small");
    }
}
		
private int parent(int i) {
    return (i + 1) / 2 - 1;
}
```

`maxHeapInsert(key)`十分简单，因为它等价于数组长度增加一，然后最后一个元素设置为`－∞`，然后把它增大为`key`的操作：
```
public void maxHeapInsert(int key) {
    int[] newA = new int[a.length + 1];
    System.arraycopy(a, 0, newA, 0, a.length);
    newA[newA.length - 1] = Integer.MIN_VALUE;
    this.a = newA;
    heapIncreaseKey(a.length - 1, key);
}
```
![file](https://static.lufficc.com/image/345b28f8cf1854653526f4b935021959.gif)

在一个包含n个元素的堆中，所有优先级队列的操作都可以在`O(lgn)`的时间内完成。
源代码可以在这里找到：[lufficc/Algorithm](https://github.com/lufficc/Algorithm/tree/master/src/com/lufficc/algorithm/Sort/HeapSort)


*文章来自自己的理解以及算法导论*。
