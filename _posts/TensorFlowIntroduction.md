---
title: 从 TensorFlow 入门机器学习
date: 2018-02-03 11:27
tags:
	- TensorFlow
	- MachineLearning
	- Python
	- Math
---

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>

> 写在前面：紧跟时代步伐，开始学习机器学习，抱着争取在毕业之前多看看各个方向是什么样子的心态，发现这是一个很有潜力也很有趣的领域（keng）。// 然后就开始补数学了……

<!-- more -->

## 0 TensorFlow 介绍
刚刚入门的小白，理解不深，直接看官方的介绍吧
> GitHub Description: Computation using data flow graphs for scalable machine learning
> 
> [官网](https://www.tensorflow.org/): \\( TensorFlow^{^{TM}} \\)是一个使用数据流图进行数值计算的开源软件库。图中的节点代表数学运算， 而图中的边则代表在这些节点之间传递的多维数组（张量）。

### 0.1 什么是 TensorFlow ?
\\( Tensor \\) 是张量的意思，\\( Flow \\) 是流的意思。所以可以直接看做利用张量组成的数据流图进行计算的一个开源库。

### 0.2 TensorFlow 可以做什么 ?
目前主要是用于机器学习，这样说有点不亲民，笔者理解是可以将数据转化为向量描述并且构建相应的计算流图都是可以使用的。
举个例子吧，虽然不知道恰不恰当。
比如我们在计算 \\( (1 + 2)*3-4 \\) 时，可以构建一个二叉树

![](0_2_01.png)

这棵二叉树的中序遍历就是上面的表达式，也就是说这个表达式可以转化成一个形如二叉树的图，而 \\( TensorFlow \\) 正好可以计算这个图。下面给出代码，看不懂没关系，只要理解代码流程是对图(二叉树)的计算就可以了，下一章会介绍如何使用\\( TensorFlow \\)。

```python
# coding: utf-8
import tensorflow as tf

a, b, c, d = tf.constant(1), tf.constant(2), tf.constant(3),tf.constant(4)
add = tf.add(a,b)
mul = tf.multiply(add, c)
sub = tf.subtract(mul, d)
with tf.Session() as sess:
    print(sess.run(sub))
# output: 
# 5
```

### 0.3 TensorFlow 安装
这里就不做详细介绍了，相信点开这篇文章的你应该有了运行环境。如果没有这里推荐两个网站[英文:官网](https://www.tensorflow.org/install/) 和 [中文:**学院翻译](http://wiki.jikexueyuan.com/project/tensorflow-zh/get_started/os_setup.html)
然后介绍一下我的环境：\\( Anaconda + PyCharm \\)

注意 \\( PyCharm:\ Project\ Interpreter \\) 设置为 \\( Conda\ Environment \\) 才能跑 \\( TensorFlow \\)。如果不会可以多看看网上的教程，能对虚拟环境加深了解。

## 1 初识 TensorFlow
好了，有前面的介绍，你应该有能够使用 \\( TensorFlow \\) 的环境了，下面开始介绍如何编码。

### 1.1 基础语法
> 其实说语法是不准确的，语法就是 \\( Python \\) 的语法(这里使用 Python)，主要是介绍调用这个计算库来实现这个特殊的计算。同样摆上[官网教程](https://www.tensorflow.org/get_started/)

#### 1.1.1 计算单元介绍
可以看到在计算图中，有两个主要的内容是点(叶子和非叶子节点)和线。我的理解是点代表数据，线代表操作。不知道对不对，不过下面就按照这样思路介绍了。
下面开始介绍有哪些常用的“点”：

`常量`

```python
c = tf.constant(2)
```

`变量`

```python
v = tf.Variable(2)
```

`占位符`

```python
p = tf.placeholder(tf.float32)
```

以上代码都是以最小能用原则传的参，感兴趣的可以去看看源码，这里主要是往 \\( Python \\) 语法上拉，先使用起来再以后自己深究为什么要设计成这样的数据结构对计算图是必须的。

接下来就是有哪些“线”：

`四则运算`

```python
add = tf.add(a, b)
sub = tf.subtract(a, b)
mul = tf.multiply(a, b)
div = tf.divide(a, b)
```

其他的就不再介绍了，详情可看 \\( XXX\\_ ops.py \\) 的源码。比如以上的操作定义在 \\( math\\_ ops.py \\)。

#### 1.1.2 计算流程介绍
知道了常见数据和计算方法下面介绍计算流程：

```python
# coding: utf-8
import tensorflow as tf

# Step1: 创建数据
a, b, c, d = tf.constant(1), tf.constant(2), tf.constant(3),tf.constant(4)

# Step2: 构造计算图
add = tf.add(a,b)
mul = tf.multiply(add, c)
sub = tf.subtract(mul, d)

# Step3: 进行计算
with tf.Session() as sess:
    print(sess.run(sub))
```

上面这个例子是一个标准的常量计算过程，你可以试着 `print(a, add)` 看看你创建的是个什么东西，你会发现他是一个 \\( Tensor \\) 而且里面的值是 \\( 0 \\)。可以猜测，这里只打印不计算，看 \\( Tensor \\) 源码：

```python
# ops.py
def __repr__(self):
    return "<tf.Tensor '%s' shape=%s dtype=%s>" % (self.name, self.get_shape(),self._dtype.name)

@property
def name(self):
    """The string name of this tensor."""
    if not self._op.name:
        raise ValueError("Operation was not named: %s" % self._op)
    return "%s:%d" % (self._op.name, self._value_index)
```

学会了计算常量，变量是不是也一样？如果你试过就知道是不一样的，变量需要初始化操作。

```python
v = tf.Variable(2)
with tf.Session() as sess:
    sess.run(tf.global_variables_initializer())
    print(v, sess.run(v))
```

到这里可能会疑问，那变量和常量有什么区别？从字面意思可以知道变量应该是可变的，方便我们在计算过程中随时调整参数，下面通过一段代码介绍如何使用。

```python
v = tf.Variable(2)
# 将 v 的值自乘 2
update = tf.assign(v, tf.multiply(v, tf.constant(2)))
with tf.Session() as sess:
    sess.run(tf.global_variables_initializer())
    for _ in range(4):
        print("-----------------------")
        print "Before : ", sess.run(v)
        sess.run(update)
        print "After : ", sess.run(v)

# output:
# -----------------------
# Before :  2
# After :  4
# -----------------------
# Before :  4
# After :  8
# -----------------------
# Before :  8
# After :  16
# -----------------------
# Before :  16
# After :  32
```

但是如果我们不想每次都设置-更新-计算-更新-计算……而是直接把数据写入计算，那占位符就起作用了。同样举个小例子。

```python
c = tf.constant(2)
# 注意类型一致，这里是 tf.int32
p = tf.placeholder(tf.int32)
mul = tf.multiply(c, p)
with tf.Session() as sess:
    # tmp = 2 相当于上一个例子变量的初始值是 2
    tmp = 2;
    for _ in range(4):
        # 直接填充 feed_dict
        tmp = sess.run(mul, feed_dict={p:tmp})
        print tmp

# output:
# 4
# 8
# 16
# 32
```

下面总结下计算过程：

- 创建数据：可以创建常量、变量和占位符。
- 构建图：通过前面的数据构建一张图。
- 初始化：把变量初始化。
- 计算：必须通过开启一个 Session 来计算图

### 1.2 可视化
\\( TensorFlow \\)提供了一个可视化工具——\\( TensorBoard \\)，下面开始介绍如何使用。

这里对上面二叉树的例子进行可视化处理。

```python
# coding: utf-8
import tensorflow as tf

a, b, c, d = tf.constant(1), tf.constant(2), tf.constant(3),tf.constant(4)
add = tf.add(a,b)
mul = tf.multiply(add, c)
sub = tf.subtract(mul, d)
with tf.Session() as sess:
    writer = tf.summary.FileWriter('./graphs', sess.graph)
    print(sess.run(sub))
writer.close()
```
然后使用命令行到存储 `graphs` 的文件夹下执行 `tensorboard --logdir="./graphs"` 命令，然后按照提示在浏览器中打开 `http://localhost:6006` 如果成功显示 \\( TensorBoard \\) 界面就说明成功了。

![](1_2_01.png)

## 2 利用 TensorFlow 进行机器学习
这里也算是机器学习的入门介绍吧。直接介绍机器学习相关知识可能不太现实，而且笔者也是在学习阶段，所以举一些小例子来体会机器学习的过程吧。

### 2.1 线性回归
这里我们使用最熟悉的线性回归来体会一下机器学习的过程：

#### 2.1.1 准备数据
这里很简单，就是模拟一个线性回归，所以我们直接自己拟定一些数据好预测结果和自己设想的是否一致。

```python
train_X = numpy.asarray([1.1, 1.8, 3.2, 4.7, 5.9, 6.7])
train_Y = numpy.asarray([1.2, 2.1, 3.1, 4.6, 5.5, 6.9])
```

#### 2.1.2 构建模型
我们采用占位符的形式进行计算，在运算时直接导入数据便可。
这里因为我们采用线性回归，所以目标函数是形如 \\( Y = XW + b \\) 的形式的一次函数。也就是说，我们通过给出的点去拟合一条比较符合这些点分布的直线。

```python
X = tf.placeholder(tf.float32)
Y = tf.placeholder(tf.float32)

W = tf.Variable(-1., name="weight")
b = tf.Variable(-1., name="bias")

# linear model 
# activation = X*W + b
activation = tf.add(tf.multiply(X, W), b)
```

#### 2.1.3 参数评估
我们采用每个点给出的纵坐标和线性模型算出的纵坐标的差\\( (activation - Y) \\)的平方和\\( (tf.reduce\\_ sum(tf.pow(activation - Y, 2))) \\)作为损失函数，在训练中采用梯度下降算法尽量使和最小，学利率选择 \\( 0.01 \\)。其中的数学原理这里就不介绍了，以后会写关于机器学习算法的相关文章。
一般选取损失函数和通过某些最优化手段更新权重是这里的一大难点，如果要知道原理，需要学习大量数学基础知识（概率论，线性代数，微积分……）。

```python
learning_rate = 0.01

cost = tf.reduce_sum(tf.pow(activation - Y, 2))
optimizer = tf.train.GradientDescentOptimizer(learning_rate).minimize(cost)
```

#### 2.1.4 训练数据
这里就是取数据，喂给图中的输入节点，然后模型会自己进行优化，可以将数据多次迭代使得拟合函数能够更好的适应这些数据点。

```python
training_epochs = 2000
display_step = 100

with tf.Session() as sess:
    sess.run(tf.global_variables_initializer())
    for epoch in range(training_epochs):
        for (x, y) in zip(train_X, train_Y):
            sess.run(optimizer, feed_dict={X: x, Y: y})
        if epoch % display_step == 0:
            print("Epoch:", '%04d' % (epoch + 1), "cost=",
                  "{:.9f}".format(sess.run(cost, feed_dict={X: train_X, Y: train_Y})), "W=", sess.run(W), "b=",
                  sess.run(b))
    print("Optimization Finished!")
    print("cost=", sess.run(cost, feed_dict={X: train_X, Y: train_Y}), "W=", sess.run(W), "b=", sess.run(b))
```     

#### 2.1.5 可视化
可以直接绘制二维图形看结果。不熟悉的可以参考[Matplotlib 教程](https://liam0205.me/2014/09/11/matplotlib-tutorial-zh-cn/)

```python
    writer = tf.summary.FileWriter('./graphs', sess.graph)

    plt.scatter(train_X, train_Y, color='red', label='Original data')
    plt.plot(train_X, sess.run(W) * train_X + sess.run(b), color='blue', label='Fitted line')
    plt.show()
writer.close()
```
二维图：
![](2_1_5_01.png)
数据流图：
![](2_1_5_02.png)

#### 2.1.6 小结
其实整个过程如果不深究其中的原理，还是很好理解的，无非就是`提供数据-选取拟合函数-构建图-选取损失函数-最优化-训练数据（更新权重）-得出结论`。这个过程符合我们对线性回归这个问题解决的基本思路的预期。当然，笔者认为这只是开始，要想深入，学习必要的数学知识是机器学习的必经之路。

这里可以参考[TensorFlow 入门](https://juejin.im/entry/5904542c570c350058168c76)
整体代码：

```python
# coding: utf-8
from __future__ import print_function
import tensorflow as tf
import numpy
import matplotlib.pyplot as plt

train_X = numpy.asarray([1.1, 1.8, 3.2, 4.7, 5.9, 6.7])
train_Y = numpy.asarray([1.2, 2.1, 3.1, 4.6, 5.5, 6.9])

X = tf.placeholder(tf.float32)
Y = tf.placeholder(tf.float32)

W = tf.Variable(-1., name="weight")
b = tf.Variable(-1., name="bias")

activation = tf.add(tf.multiply(X, W), b)

learning_rate = 0.01

cost = tf.reduce_sum(tf.pow(activation - Y, 2))
optimizer = tf.train.GradientDescentOptimizer(learning_rate).minimize(cost)

training_epochs = 2000
display_step = 100

with tf.Session() as sess:
    sess.run(tf.global_variables_initializer())
    for epoch in range(training_epochs):
        for (x, y) in zip(train_X, train_Y):
            sess.run(optimizer, feed_dict={X: x, Y: y})
        if epoch % display_step == 0:
            print("Epoch:", '%04d' % (epoch + 1), "cost=",
                  "{:.9f}".format(sess.run(cost, feed_dict={X: train_X, Y: train_Y})), "W=", sess.run(W), "b=",
                  sess.run(b))
    print("Optimization Finished!")
    print("cost=", sess.run(cost, feed_dict={X: train_X, Y: train_Y}), "W=", sess.run(W), "b=", sess.run(b))
    
    writer = tf.summary.FileWriter('./graphs', sess.graph)

    plt.scatter(train_X, train_Y, color='red', label='Original data')
    plt.plot(train_X, sess.run(W) * train_X + sess.run(b), color='blue', label='Fitted line')
    plt.show()
writer.close()

# output:
# Epoch: 0001 cost= 0.785177052 W= 1.07263 b= -0.448403
# Epoch: 0101 cost= 0.440001398 W= 1.02555 b= -0.0137608
# Epoch: 0201 cost= 0.437495589 W= 1.02078 b= 0.0176154
# Epoch: 0301 cost= 0.437433660 W= 1.02043 b= 0.0199056
# Epoch: 0401 cost= 0.437430561 W= 1.02041 b= 0.0200727
# Epoch: 0501 cost= 0.437429130 W= 1.0204 b= 0.0200851
# Epoch: 0601 cost= 0.437429696 W= 1.0204 b= 0.0200854
# Epoch: 0701 cost= 0.437429696 W= 1.0204 b= 0.0200854
# Epoch: 0801 cost= 0.437429696 W= 1.0204 b= 0.0200854
# Epoch: 0901 cost= 0.437429696 W= 1.0204 b= 0.0200854
# Epoch: 1001 cost= 0.437429696 W= 1.0204 b= 0.0200854
# Epoch: 1101 cost= 0.437429696 W= 1.0204 b= 0.0200854
# Epoch: 1201 cost= 0.437429696 W= 1.0204 b= 0.0200854
# Epoch: 1301 cost= 0.437429696 W= 1.0204 b= 0.0200854
# Epoch: 1401 cost= 0.437429696 W= 1.0204 b= 0.0200854
# Epoch: 1501 cost= 0.437429696 W= 1.0204 b= 0.0200854
# Epoch: 1601 cost= 0.437429696 W= 1.0204 b= 0.0200854
# Epoch: 1701 cost= 0.437429696 W= 1.0204 b= 0.0200854
# Epoch: 1801 cost= 0.437429696 W= 1.0204 b= 0.0200854
# Epoch: 1901 cost= 0.437429696 W= 1.0204 b= 0.0200854
# Optimization Finished!
# cost= 0.43743 W= 1.0204 b= 0.0200854
# 可以看到迭代次数到 500 次左右数据就稳定了。
```

## 3 总结
其实这只是一个开始，还有好多好多东西要去学习。越来越觉得基础的重要性，不仅仅是计算机基础，数学基础也是同等重要，特别是未来的物联网趋势，可能编码这种专业越来越淡化，只是作为某些专业人员的一种工具/技能使用。马上面临毕业，只能自己慢慢啃这些东西了……

## 4 参考资料
- [0. 官网](https://www.tensorflow.org/)
- [1. 中文:**学院翻译](http://wiki.jikexueyuan.com/project/tensorflow-zh/get_started/os_setup.html)
- [2. Matplotlib 教程](https://liam0205.me/2014/09/11/matplotlib-tutorial-zh-cn/)
- [3. TensorFlow 入门](https://juejin.im/entry/5904542c570c350058168c76)