---
title: NumPy 官方快速入门教程(译)
date: 2018-02-04 11:27
tags:
	- MachineLearning
	- Python
	- Math
	- NumPy
---

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>

> 写在前面：本来是学习下 \\( NumPy \\)，看到官网的[入门教程](https://docs.scipy.org/doc/numpy-dev/user/quickstart.html)想跟着实验一下，怕不常用，而我这人健忘，所以记录下来。索性就照着翻译一下，同样可以提升自己的阅读和写作能力，需要的可以存一下。这里是基于 \\( NumPy v1.13.dev0\ Manual \\) 翻译的。截止时间\\( 2018/02/04 \\)
<!-- more -->

## 快速入门教程

### 1 准备工作
在你浏览这个指导之前，你应该懂一点 \\( Python \\) 。如果你想回顾一下可以看[Python tutorial](https://docs.python.org/3/tutorial/)。如果你想跟着教程写代码，必须安装一些软件，请参考[http://scipy.org/install.html](http://scipy.org/install.html)

### 2 基础知识
\\( NumPy \\) 的主要操作对象是同类型的多维数组。它是一个由正整数元组索引，元素类型相同的表（通常元素是数字）。在 \\( NumPy \\) 维度被称为 `axes`, `axes` 的数量称为 `rank`。

例如，在 \\( 3D \\) 空间的一个点 \\( [1, 2, 1] \\) 是一个 `rank = 1` 的数组，因为它只有一个 `axes`。这个 `axes` 的长度是 \\( 3 \\)。在下面这个例子中，数组 `rank = 2` （它是 \\( 2  \\)维的）。第一维（`axes`）长度是 \\( 2 \\)，第二位长度是 \\( 3 \\)

```
[[ 1., 0., 0. ],
 [ 0., 1., 2. ]]
```

\\( NumPy \\) 的数组类是 `ndarray`。也可以叫做 `array`。说到这里，`numpy.array` 和标准 \( Python \\) 库中的 `array.array` 是不一样的，它只能处理一维的数组和提供更少的功能。`ndarray` 对象的一些重要属性如下：

##### ndarray.ndim
> 数组的 `axes` （维数）数值大小。在 \\( Python \\) 中维数的大小可以参考 `rank`

##### ndarray.shape
> 数组的维数，这是由每个维度的大小组成的一个元组。对于一个 \\( n \\) 行 \\( m \\) 列的矩阵。`shape` 是 `(n, m)`。由 `shape` 元组的长度得出 `rank` 或者维数 `ndim`。

##### ndarray.size
> 数组元素的个数总和，这等于 `shape` 元组数字的乘积。





































