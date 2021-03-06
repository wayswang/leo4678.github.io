---
layout: post
category: "ml"
title: "逻辑/线性回归/GLM"
tags: [回归问题]
urlcolor: blue
---

目录

<!-- TOC -->

- [1. 逻辑回归](#1-逻辑回归)
	- [1.1 逻辑回归原理](#11-逻辑回归原理)
	- [1.2 逻辑回归损失函数](#12-逻辑回归损失函数)
	- [1.3 逻辑回归优化器](#13-逻辑回归优化器)
	- [1.4 逻辑回归的优缺点](#14-逻辑回归的优缺点)
- [2. 逻辑回归与线性回归区别](#2-逻辑回归与线性回归区别)
- [3. 广义线性模型](#3-广义线性模型)

<!-- /TOC -->

## 1. 逻辑回归

### 1.1 逻辑回归原理

逻辑回归首先假定问题域满足Bernoulli分布，模型设计如下：

公式1->线性回归：

$$f(x)=\omega_{0}x_{0}+\omega_{1}x_{1}+...+\omega_{n}x_{n}+bias=\sum_{i}^{n}\omega_{i}x_{i}+bias$$

也可表达为：

$$f(x)=\Theta ^{T}\cdot x,\Theta = [\omega _{0},\omega _{1},...,\omega _{n}]$$

这里\\(\Theta\\)为模型参数，是一个向量，x为输入的特征向量

上述公式被用来解决满足高斯分布的回归问题。对于分类问题，需要在上述公式的基础上设计阈值，然后比较上述公式的输出和阈值大小，从而进行分类。一般情况下，使用公式2作为阈值分类时，存在两个问题，一是其曲线不够光滑，存在突变，导致某一点不连续，在数学处理时不方便，二是阈值如何选取？ 假设t0=10，现在有一个样本进来，最后计算出来的值为10.01，你说这个样本分类应该是为1还是0呢？

公式2：

$$\varepsilon (t)=\left\{\begin{matrix}
1(t>t_{0})\\ 
0(t<t_{0})\\
\end{matrix}\right.$$

为了解决阈值函数存在的上边两个问题，这里引入**sigmoid函数来代替阈值分类**。在公式1的基础上添加sigmoid函数即构建了逻辑回归的基本表达形式

公式3->逻辑回归：

$$h_{\Theta }(x)=sigmoid(\Theta ^{T}\cdot x)$$

$$sigmoid(x)=\frac{1}{1+e^{-x}}$$

选择[sigmoid函数](https://baike.baidu.com/item/Sigmoid%E5%87%BD%E6%95%B0/7981407)来代替阈值分类的原因是它有两个优点：

+ 它的输入范围是\\(-\infty \rightarrow +\infty \\)，值域为(0,1)，正好满足概率分布为(0,1)的要求。用概率去描述分类器，自然比单纯的某个阈值要方便很多
+ 它是一个单调上升的函数，具有良好的连续性，不存在不连续点

根据样本求解最优参数的推导过程如下图所示：

<html>
<br/>

<img src='/assets/lr梯度求解公式推导.png' style='max-height: 800px; max-width:600px'/>
<br/>

</html>

### 1.2 逻辑回归损失函数

对数损失函数(logarithmic loss function)或对数似然损失函数(log-likehood loss function)

$$L(Y, P(Y|X))=-logP(Y|X)$$

如果损失函数越小，表明模型越好

逻辑回归的对数似然损失函数公式如下：

$$cost(h_{\Theta }(x), y)=\begin{cases}
 -log(h_{\Theta }(x))& \text{ if } y=1 \\ 
 -log(1-h_{\Theta }(x))& \text{ if } y=0 
\end{cases}$$

将上式合并表达：

$$cost(h_{\Theta }(x), y)=-y_{i}log(h_{\Theta }(x))-(1-y_{i})log(1-h_{\Theta }(x))$$

全样本的损失函数可以表达为：

$$cost(h_{\Theta }(x), y)=\sum_{i=1}^{m}-y_{i}log(h_{\Theta }(x))-(1-y_{i})log(1-h_{\Theta }(x))$$

### 1.3 逻辑回归优化器

可使用批量梯度求解，但由于计算量太高，一般使用随机梯度下降，计算示例代码如下：

[参考](https://blog.csdn.net/StayFoolish_Fan/article/details/80314941)

```python
# coding=utf-8
from numpy import *
import matplotlib.pyplot as plt

# Sigmoid激活函数，这里也给了tanh激活函数
def sigmoid(X):
    return 1.0/(1+exp(-X))
    #return 2*1.0/(1+exp(-2*X))-1  # tanh(x):mean value is 0

# 用于显示后面梯度上升训练过程中的weights
def trainingProcessDisplay(weights):
    fig = plt.figure()
    n = len(weights)
    x = range(n)
    ax1 = fig.add_subplot(311)
    ax1.plot(x,weights[:,0])
    plt.ylabel('w0')
    ax = fig.add_subplot(312)
    ax.plot(x,weights[:,1])
    plt.ylabel('w1')
    ax = fig.add_subplot(313)
    ax.plot(x,weights[:,2])
    plt.ylabel('w2')
    plt.xlabel('Iterations')
    plt.show()

# 梯度上升，每次更新都是计算出所有样本之间的误差（数据量很大时计算很耗时）
def gradAscent(data,labels, alpha = 0.001, numIter=500):
    dataMatrix = mat(data)
    labelMat = mat(labels).transpose()
    m,n = shape(dataMatrix)
    weights = ones((n,1))
    for k in range(numIter):
        h = sigmoid(dataMatrix*weights)       # 激活输出
        error = labelMat - h                  # 误差计算，上图中的蓝色框公式
        weights = weights + alpha*dataMatrix.transpose()*error    # 权值更新，这里使用批量数据
    return array(weights)

# 梯度上升，每次使用单个样本进行更新，error为标量
def stocGradAscent0(data,labels, alpha = 0.001, numIter=500):
    m,n = shape(data)
    weights = ones(n)
    for j in range(numIter):
        for i in range(m):
            h = sigmoid(sum(data[i]*weights))
            error = labels[i]-h
            weights = weights + alpha*error*data[i]
    return weights

# 随机梯度上升，每次迭代过程中顺序不一样，同样也是单样本更新
def stocGradAscent1(data,labels, numIter=500):
    m,n = shape(data)
    weights = ones(n)
    allWeights = ones((numIter,n))
    for j in range(numIter):
        allWeights[j] = weights
        dataIndex =  range(m)
        random.shuffle(dataIndex)
        for i in range(m):
            alpha = 4/(1.0+i+j) + 0.0001
            index = dataIndex[i]
            h = sigmoid(sum(data[index]*weights))
            error = labels[index] - h
            weights = weights + alpha *error*data[index]
    trainingProcessDisplay(allWeights)
    return weights

# 结果可视化：在这里回归出来的是直线
def resultVisualization(data,labels,weights):
    n = shape(data)[0]
    x1 = [];y1=[];x2=[];y2=[]
    for i in range(n):
        if int(labels[i])==1:
            x1.append(data[i][1]);y1.append(data[i][2])
        else:
            x2.append(data[i][1]);y2.append(data[i][2])
    fig = plt.figure()
    ax = fig.add_subplot(111)
    ax.scatter(x1,y1,s=30,c='red',marker='s')
    ax.scatter(x2,y2,s=30,c='green')
    x = arange(-5.0,5.0,0.1)
    # div = weights[2]
    # if weights[2]==0: div = 0.00001
    y = (-weights[0]-weights[1]*x)/weights[2]
    ax.plot(x,y)
    plt.xlabel('X1');plt.ylabel('X2')
    plt.show()

# 测试
def testLR():
    data,labels = loadData("data.txt")
    dataArr = array(data)
    #weights = gradAscent(dataArr,labels)
    #weights = stocGradAscent0(dataArr,labels)
    weights = stocGradAscent1(dataArr,labels)
    #print weights
    resultVisualization(dataArr,labels,weights)

if __name__ == "__main__":
    testLR()
```

### 1.4 逻辑回归的优缺点

[参考](https://zhuanlan.zhihu.com/p/54197906](https://zhuanlan.zhihu.com/p/54197906)

优点：

+ 预测结果是界于0和1之间的概率
+ 可以适用于连续性和离散型自变量
+ 模型简单
+ 可解释性强
+ 易扩展为在线学习算法
+ 解决过拟合方法很多，如：L1、L2正则

缺点：

+ 处理不好特征之间相关，对自变量多重共线性较为敏感，例如两个高度相关自变量同时放入模型，可能导致较弱的一个自变量回归符号不符合预期，符号被扭转。​需要利用因子分析或者变量聚类分析等手段来选择代表性的自变量，以减少候选变量之间的相关性
+ 容易欠拟合，分类精度可能不高
+ 特征空间很大时，性能不好

## 2. 逻辑回归与线性回归区别

+ 线性回归要求自变量服从正太分布，逻辑回归要求自变量满足伯努利分布

线性回归:

$$y|x;θ\sim Ν(µ, σ^2)$$

逻辑回归：

$$y|x;θ\sim Bernoulli(\phi)$$

+ 线性回归要求因变量是连续性数值变量，而逻辑回归要求因变量是分类型变量
+ 线性回归要求自变量和因变量呈线性关系，而逻辑回归不要求自变量和因变量呈线性关系（因为逻辑回归在输出阶段使用了激活函数）
+ 线性回归分析因变量与自变量关系，而逻辑回归分析因变量取某个值的概率和自变量的关系

总之，线性回归和逻辑回归有很多相似之处，最大的区别就是因变量不同。它们同属于一个家族，即广义线性模型（GLM），因变量如果是连续的，就是多重线性回归，如果是离线的，就是逻辑回归（逻辑回归的因变量可以是二分类也可以是多分类）

[参考](https://zhuanlan.zhihu.com/p/31233767)

## 3. 广义线性模型

广义线性模型(GLM/Generalized Linear Model)符合指数分布的一般模型可以转换成广义线性模型，广义线性模型即可构建因变量和自变量之间的关系。GLM常用的分布有三种：

+ 高斯分布对应的线性回归算法
+ 伯努利分布对应的逻辑回归算法（二分类）
+ 伯努利分布对应的多分类算法（多分类）

[参考](https://zhuanlan.zhihu.com/p/22876460)

