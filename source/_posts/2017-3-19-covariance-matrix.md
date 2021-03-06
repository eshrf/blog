---
title: 协方差矩阵
date: 2017-03-19 09:41:39
tags: [线性代数, 概率统计]
categories: 线性代数
mathjax: true

---

### 概念
协方差（Covariance）在概率论和统计学中用于衡量两个变量的总体误差。而方差是协方差的一种特殊情况，即当两个变量是相同的情况。
这个解释摘自维基百科，看起来很是抽象，不好理解。其实简单来讲，协方差就是衡量两个变量相关性的变量。当协方差为正时，两个变量呈正相关关系（同增同减）；当协方差为负时，两个变量呈负相关关系（一增一减）。
而协方差矩阵，只是将所有变量的协方差关系用矩阵的形式表现出来而已。通过矩阵这一工具，可以更方便地进行数学运算。
<!--more-->

### 数学定义

回想概率统计里面关于方差的数学定义：
$$
Var(X)=\frac{\sum_{i=1}^n{(x_i-\overline x)(x_i-\overline x)}}{n-1}
$$
协方差的数学定义异曲同工：
$$
Cov(X,Y)=\frac{\sum_{i=1}^n{(x_i-\overline x)(y_i-\overline y)}}{n-1}
$$
这里的 $X$，$Y$ 表示两个变量空间。用机器学习的话讲，就是样本有 $x$ 和 $y$ 两种特征，而 $X$ 就是包含所有样本的 $x$ 特征的集合，$Y$ 就是包含所有样本的 $y$ 特征的集合。

### 协方差矩阵
#### 两个变量的协方差矩阵
有了上面的数学定义后，我们可以来讨论协方差矩阵了。当然，协方差本身就能够处理二维问题，两个变量的协方差矩阵并没有实际意义，不过为了方便后面多维的推广，我们还是从二维开始。

用一个例子来解释会更加形象。

假设我们有 4 个样本，每个样本都有两个变量，也就是两个特征，它们表示如下：
$x_1=(1,2)$，$x_2=(3,6)$，$x_3=(4,2)$，$x_4=(5,2)$

用一个矩阵表示为：

$$
Z=\begin{bmatrix}
1 & 2 \\
3 & 6 \\
4 & 2 \\
5 & 2
\end{bmatrix}
$$

现在，我们用两个变量空间 $X$，$Y$ 来表示这两个特征：

$$
X=\begin{bmatrix} 1 \\ 3 \\ 4 \\ 5 \end{bmatrix},  \ \ \    Y=\begin{bmatrix} 2 \\ 6 \\ 2 \\ 2 \end{bmatrix}
$$
由于协方差反应的是两个变量之间的相关性，因此，协方差矩阵表示的是所有变量之间两两相关的关系，具体来讲，一个包含两个特征的矩阵，其协方差矩阵应该有 $2 \times 2$ 大小：

$$
Cov(Z)=\begin{bmatrix} Cov(X,X) & Cov(X,Y) \\ Cov(Y,X) & Cov(Y,Y) \end{bmatrix}
$$
接下来，就来逐一计算 $Cov(Z)$ 的值。
首先，我们需要先计算出 $X$，$Y$ 两个特征空间的平均值：$\overline x=3.25$，$\overline y=3$。
然后，根据协方差的数学定义，计算协方差矩阵的每个元素：

$Cov(X,X)=\frac{(1-3.25)^2+(3-3.25)^2+(4-3.25)^2+(5-3.25)^2}{4-1}=2.9167$

$Cov(X,Y)=\frac{(1-3.25)(2-3)+(3-3.25)(6-3)+(4-3.25)(2-3)+(5-3.25)(2-3)}{4-1}=-0.3333$

$Cov(Y,X)=\frac{(2-3)(1-3.25)+(6-3)(3-3.25)+(2-3)(4-3.25)+(2-3)(5-3.25)}{4-1}=-0.3333$

$Cov(Y,X)=\frac{(2-3)^2+(6-3)^2+(2-3)^2+(2-3)^2}{4-1}=4$

所以协方差矩阵 $Cov(Z)=\begin{bmatrix} 2.9167 & -0.3333 \\ -0.3333 & 4.000 \end{bmatrix}$

好了，虽然这只是一个二维特征的例子，但我们已经可以从中总结出协方差矩阵 $\Sigma$ 的「计算套路」：

$\Sigma_{ij}=\frac{(样本矩阵第i列-第i列均值)^T(样本矩阵第j列-第j列均值)}{样本数-1}$

这里所说的样本矩阵可以参考上面例子中的 $Z$。

#### 多个变量的协方差矩阵
接下来，就用上面推出的计算协方差矩阵的「普世规律」。
假设我们有三个样本：
$x_1=(1,2,3,4)^T$， $x_2=(3,4,1,2)^T$， $x_3=(2,3,1,4)^T$。
同理我们将它们表示成样本矩阵：

$Z=\begin{bmatrix} 1 & 2 & 3 & 4 \\ 3 & 4 & 1 & 2 \\ 2 & 3 & 1 & 4  \end{bmatrix}$

按照上面给出的计算套路，我们需要先计算出矩阵每一列的均值，从左到右分别为：2、3、1.67、3.33。
然后按照上面讲到的公式，计算矩阵每个元素的值，对了，三个变量的协方差矩阵，大小为 $3 \times 3$ ：

$\Sigma_{11}=\frac{(第1列-第1列的均值)^T*(第1列-第1列的均值)}{样本数-1}  \\=\frac{(-1,1,0)^T*(-1,1,0)}{2}=1$

（后面的依此类推......）

#### 独立变量的协方差

以上的讨论都是针对一般情况进行计算的，毕竟变量互相独立的情况较少。

不过，如果两个变量 $X$, $Y$ 独立，那么它们的协方差 $Cov(X,Y) = 0$。简要证明如下（简单起见，假设变量是离散的）：

由于 $X, Y$ 独立，所以它们的概率密度函数满足：$p(x,y)=p_x(x)p_y(y)$。

求出期望：
$$
\begin{eqnarray} E(XY) & = &\sum_x \sum_y {x*y*p(x,y)} \notag \\
& = &\sum_x \sum_y x*y*p_x(x)*p_y(y) \notag \\
& = &\sum_x{x*p_x(x)}\sum_y{y*p_y(y)} \notag \\
& = &E(X)E(Y) \notag
\end{eqnarray}
$$
利用协方差的另一个公式：$Cov(X,Y)=E(X,Y)-E(X)E(Y)$，可以推出，当 $X, Y$ 相互独立时，$Cov(X, Y)=0$。

这时，协方差矩阵就变成一个对角矩阵了：
$$
Cov(Z)=\begin{bmatrix} Cov(X,X) & 0\\ 0 & Cov(Y,Y) \end{bmatrix}
$$

### 协方差矩阵的作用

虽然我们已经知道协方差矩阵的计算方法了，但还有一个更重要的问题：协方差矩阵有什么作用？作为一种数学工具，协方差矩阵经常被用来计算特征之间的某种联系。在机器学习的论文中，协方差矩阵的出现概率还是很高的，用于降维的主成分分析法（PCA）就用到了协方差矩阵。另外，由于协方差矩阵是一个对称矩阵，因此它包含了很多很有用的性质，这也导致它受青睐的程度较高。

### 参考
+ [详解协方差与协方差矩阵](http://blog.csdn.net/ybdesire/article/details/6270328)
+ [浅谈协方差矩阵](http://pinkyjie.com/2010/08/31/covariance/)
+ [协方差](https://zh.wikipedia.org/wiki/%E5%8D%8F%E6%96%B9%E5%B7%AE)




