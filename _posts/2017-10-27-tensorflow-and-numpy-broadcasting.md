---
layout: post
permalink: /tensorflow-and-numpy-broadcasting/
title: TensorFlow 和 NumPy 的 Broadcasting 机制
date: 2017-10-27T13:21:12
lang: zh-CN
---

TensorFlow 采用 NumPy 的 Broadcasting  机制，来处理不同形状的 Tensor 之间的算术运算，来节省内存、提高计算效率。

NumPy 数组运算通常是逐元素（element-by-element ）计算，因此要求两个数组的**形状必须相同**：
```
>>> a = np.array([1.0, 2.0, 3.0])
>>> b = np.array([2.0, 2.0, 2.0])
>>> a * b
array([ 2.,  4.,  6.])
```
NumPy 的 Broadcasting  机制解除了这种限制，在两个数组的**形状**满足**某种条件**的情况下，不同形状的数组之间仍可以进行算术运算。最简单的就是数组乘以一个标量：
```
>>> a = np.array([1.0, 2.0, 3.0])
>>> b = 2.0
>>> a * b
array([ 2.,  4.,  6.])
```
结果和第一个 `b` 是数组的例子相同。可以认为标量 `b` 被拉伸成了和 `a` 相同形状的数组，拉伸后数组每个元素的值为先前标量值的复制，这样形式上和第一种例子相同，因此结果当然一样。但这只是**理论**上的，复制操作并不会真正进行，只是在计算时使用标量的值罢了。因此，**第二个例子效率更高，因为节省了内存**。

## Broadcasting 规则 
当两个数组进行算术运算时，**NumPy 从后向前，逐元素比较两个数组的形状**。当逐个比较的元素值满足以下条件时，认为满足 Broadcasting 的条件：
1. **相等**
2. **其中一个是1**

当不满足时，会抛出 `ValueError: frames are not aligne` 异常。算术运算的结果的形状的每一元素，是两个数组形状逐元素比较时的最大值。

而且，两个数组可以有不同的维度。比如一个 ` 256x256x3` 的数组储存 RGB 值，如果对每个颜色通道进行不同的放缩，我们可以乘以一个一维、形状为 `(3, )` 的数组。因为是**从后向前**比较，因此 `3 == 3`，符合 Broadcasting 规则 。
```
Image  (3d array): 256 x 256 x 3
Scale  (1d array):             3
Result (3d array): 256 x 256 x 3
```


当其中一个是 `1` 时，就会被“拉伸”成和另一个相同大小，即“复制”（没有真正复制）元素值来 Match 另一个，如：
```
A      (4d array):  8 x 1 x 6 x 1
B      (3d array):      7 x 1 x 5
Result (4d array):  8 x 7 x 6 x 5
```

更多的例子：
```
A      (2d array):  5 x 4
B      (1d array):      1
Result (2d array):  5 x 4

A      (2d array):  5 x 4
B      (1d array):      4
Result (2d array):  5 x 4

A      (3d array):  15 x 3 x 5
B      (3d array):  15 x 1 x 5
Result (3d array):  15 x 3 x 5

A      (3d array):  15 x 3 x 5
B      (2d array):       3 x 5
Result (3d array):  15 x 3 x 5

A      (3d array):  15 x 3 x 5
B      (2d array):       3 x 1
Result (3d array):  15 x 3 x 5
```

一些反例（不满足 Broadcasting 规则 ）：
```
A      (1d array):  3
B      (1d array):  4 # trailing dimensions do not match

A      (2d array):      2 x 1
B      (3d array):  8 x 4 x 3 # second from last dimensions mismatched
```

实践：

```
>>> x = np.arange(4)
>>> xx = x.reshape(4,1)
>>> y = np.ones(5)
>>> z = np.ones((3,4))

>>> x.shape
(4,)

>>> y.shape
(5,)

>>> x + y
<type 'exceptions.ValueError'>: shape mismatch: objects cannot be broadcast to a single shape

>>> xx.shape
(4, 1)

>>> y.shape
(5,)

>>> (xx + y).shape
(4, 5)

>>> xx + y
array([[ 1.,  1.,  1.,  1.,  1.],
       [ 2.,  2.,  2.,  2.,  2.],
       [ 3.,  3.,  3.,  3.,  3.],
       [ 4.,  4.,  4.,  4.,  4.]])

>>> x.shape
(4,)

>>> z.shape
(3, 4)

>>> (x + z).shape
(3, 4)

>>> x + z
array([[ 1.,  2.,  3.,  4.],
       [ 1.,  2.,  3.,  4.],
       [ 1.,  2.,  3.,  4.]])
```
再例如：
```
>>> a = np.array([0.0, 10.0, 20.0, 30.0])
>>> b = np.array([1.0, 2.0, 3.0])
>>> a[:, np.newaxis] + b
array([[  1.,   2.,   3.],
       [ 11.,  12.,  13.],
       [ 21.,  22.,  23.],
       [ 31.,  32.,  33.]])
```
`newaxis` 操作为数组 `a` 插入一维，变成二维 `4x1` 数组，因此 `4x1` 的数组加 `(3, )` 的数组，结果为 `4x3` 的数组。

## 参考
1. https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html
