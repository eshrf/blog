---
title: 论文笔记：Selective Search for Object Recognition
date: 2017-05-04 22:18:01
tags: [论文, 计算机视觉]
categories: [计算机视觉]
mathjax: true
---

与 Selective Search 初次见面是在著名的物体检测论文 `Rich feature hierarchies for accurate object detection and semantic segmentation` ，因此，这篇论文算是阅读 R-CNN 的准备。

这篇论文的标题虽然也提到了 Object Recognition ，但就创新点而言，其实在 Selective Search 。所以，这里只简单介绍 Selective Search 的思想和算法过程，对于 Object Recognition 则不再赘述。

### 什么是 Selective Search

Selective Search，说的简单点，就是从图片中找出物体可能存在的区域。

![result](/images/2017-5-4/result.png)

上面这幅宇航员的图片中，那些红色的框就是 Selective Search 找出来的可能存在物体的区域。

<!--more-->

在进一步探讨它的原理之前，我们分析一下，如何判别哪些 region 属于一个物体？

![image seg](/images/2017-5-4/image seg.png)

作者在论文中用以上四幅图，分别描述了四种可能的情况：

1. 图 a ，物体之间可能存在层级关系，比如：碗里有个勺；
2. 图 b，我们可以用颜色来分开两只猫，却没法用纹理来区分；
3. 图 c，我们可以用纹理来区分变色龙，却没法用颜色来区分；
4. 图 d，轮胎是车的一部分，不是因为它们颜色相近、纹理相近，而是因为轮胎包含在车上。

所以，我们没法用单一的特征来定位物体，需要综合考虑多种策略，这一点是 Selective Search 精要所在。

### 需要考虑的问题

在学习 Selective Search 算法之前，我曾在计算机视觉课上学到过关于物体（主要是人脸）检测的方法。通常来说，最常规也是最简单粗暴的方法，就是用不同尺寸的矩形框，一行一行地扫描整张图像，通过提取矩形框内的特征判断是否是待检测物体。这种方法的复杂度极高，所以又被称为 **exhaustive search**。在人脸识别中，由于使用了 Haar 特征，因此可以借助 **Paul Viola** 和 **Michael Jones** 两位大牛提出的积分图，使检测在常规时间内完成。但并不是每种特征都适用于积分图，尤其在神经网络中，积分图这种动态规划的思路就没什么作用了。

针对传统方法的不足，Selective Search 从三个角度提出了改进：

1. 我们没法事先得知物体的大小，在传统方法中需要用不同尺寸的矩形框检测物体，防止遗漏。而 Selective Search 采用了一种具备层次结构的算法来解决这个问题；
2. 检测的时间复杂度可能会很高。Selective Search 遵循简单即是美的原则，只负责快速地生成可能是物体的区域，而不做具体的检测；
3. 另外，结合上一节提出的，采用多种先验知识来对各个区域进行简单的判别，避免一些无用的搜索，提高速度和精度。

### 算法框架

![algorithm](/images/2017-5-4/algorithm.png)

论文中给出的这个算法框架还是很详细的，这里再简单翻译一下。

+ 输入：彩色图片。
+ 输出：物体可能的位置，实际上是很多的矩形坐标。
+ 首先，我们使用这篇[论文](http://cs.brown.edu/~pff/segment/)的方法将图片初始化为很多小区域 $R={r_i, …, r_n}$。由于我们的重点是 Selective Search，因此我直接将该论文的算法当成一个黑盒子。
+ 初始化一个相似集合为空集： $S=\varnothing$。
+ 计算所有相邻区域之间的相似度（相似度函数之后会重点分析），放入集合 S 中，集合 S 保存的其实是一个**区域对**以及它们之间的相似度。
+ 找出 S 中相似度最高的区域对，将它们合并，并从 S 中删除与它们相关的所有相似度和区域对。重新计算这个新区域与周围区域的相似度，放入集合 S 中，并将这个新合并的区域放入集合 R 中。重复这个步骤直到 S 为空。
+ 从 R 中找出所有区域的 bounding box（即包围该区域的最小矩形框），这些 box 就是物体可能的区域。

另外，为了提高速度，新合并区域的 feature 可以通过之前的两个区域获得，而不必重新遍历新区域的像素点进行计算。这个 feature 会被用于计算相似度。

### 相似度计算方法

相似度计算方法将直接影响合并区域的顺序，进而影响到检测结果的好坏。

论文中比较了八种颜色空间的特点，在实际操作中，只选择一个颜色空间（比如：RGB 空间）进行计算。

正如一开始提出的那样，我们需要综合多种信息来判断。作者将相似度度量公式分为四个子公式，称为**互补相似度测量(Complementary Similarity Measures)** 。这四个子公式的值都被归一化到区间 [0, 1] 内。

#### 1. 颜色相似度$s_{color}\ (r_i, r_j)$

正如本文一开始提到的，颜色是一个很重要的区分物体的因素。论文中将每个 region 的像素按不同颜色通道统计成直方图，其中，每个颜色通道的直方图为 25 bins （比如，对于 0 ～ 255 的颜色通道来说，就每隔 9(255/25=9) 个数值统计像素数量）。这样，三个通道可以得到一个 75 维的直方图向量 $C_i={c_{i}^{1}, …, c_{i}^{n}}$，其中 n = 75。之后，我们用 **L1 范数**（绝对值之和）对直方图进行归一化。由直方图我们就可以计算两个区域的颜色相似度：
$$
s_{color}(r_i, r_j) =\sum_{k=1}^{n}{min(c_{i}^{k}, c_{j}^{k})}
$$
这个相似度其实就是计算两个区域直方图的交集。

这个颜色直方图可以在合并区域的时候，很方便地传递给下一级区域。即它们合并后的区域的直方图向量为：$$C_t=\frac{size(r_i)*C_i+size(r_j)*C_j}{size(r_i)+size(r_j)}$$，其中 $size(r_i)$ 表示区域 $r_i$ 的面积，合并后的区域为 $size(r_t)=size(r_i)+size(r_j)$。

#### 2. 纹理相似度$s_{texture}\ (r_i, r_j)$

另一个需要考虑的因素是纹理，即图像的梯度信息。

论文中对纹理的计算采用了 SIFT-like 特征，该特征借鉴了 SIFT 的计算思路，对每个颜色通道的像素点，沿周围 8 个方向计算高斯一阶导数($\sigma = 1$)，每个方向统计一个直方图（bin = 10），这样，一个颜色通道统计得到的直方图向量为 80 维，三个通道就是 240 维：$T_i={t_i^{(1)}, …, t_i^{(n)}}$，其中 n = 240。注意这个直方图要用 **L1 范数**归一化。然后，我们按照颜色相似度的计算思路计算两个区域的纹理相似度：
$$
s_{texture}(r_i, r_j) =\sum_{k=1}^{n}{min(t_{i}^{k}, t_{j}^{k})}
$$
同理，合并区域后，纹理直方图可以很方便地传递到下一级区域，计算方法和颜色直方图的一模一样。

#### 3. 尺寸相似度$s_{size}\ (r_i, r_j)$

在合并区域的时候，论文优先考虑小区域的合并，这种做法可以在一定程度上保证每次合并的区域面积都比较相似，防止大区域对小区域的逐步蚕食。这么做的理由也很简单，我们要均匀地在图片的每个角落生成不同尺寸的区域，作用相当于 **exhaustive search** 中用不同尺寸的矩形扫描图片。具体的相似度计算公式为：
$$
s_{size}(r_i, r_j)=1-\frac{size(r_i) + size(r_j)}{size(im)}
$$
其中，$size(im)$ 表示原图片的像素数量。

#### 4. 填充相似度$s_{fill}(r_i, r_j)$

填充相似度主要用来测量两个区域之间 fit 的程度，个人觉得这一点是要解决文章最开始提出的物体之间的包含关系（比如：轮胎包含在汽车上）。在给出填充相似度的公式前，我们需要定义一个矩形区域 $BB_{ij}$，它表示包含 $r_i$ 和 $r_j$ 的最小的 bounding box。基于此，我们给出相似度计算公式为：
$$
s_{fill}(r_i, r_j)=1-\frac{size(BB_{ij})-size(r_i)-size(r_j)}{size(im)}
$$
为了高效地计算 $BB_{ij}$，我们可以在计算每个 region 的时候，都保存它们的 bounding box 的位置，这样，$BB_{ij}$ 就可以很快地由两个区域的 bounding box 推出来。

#### 5. 相似度计算公式

综合上面四个子公式，我们可以得到计算相似度的最终公式：
$$
s(r_i, r_j) = a_1 s_{color}(r_i, r_j) +a_2s_{texture}(r_i, r_j) \\\\ +a_3s_{size}(r_i, r_j)+a_4s_{fill}(r_i, r_j)
$$
其中，$a_i$ 的取值为 0 或 1，表示某个相似度是否被采纳。

### Combining Locations

前面我们基本完成了 Selective Search 的流程，从图片中提取出了物体可能的位置。现在，我们想完善最后一个问题，那就是给这些位置排个序。因为提取出来的矩形框数量巨大，而用户可能只需要其中的几个，这个时候我们就很有必要对这些矩形框赋予优先级，按照优先级高低返回给用户。原文中作者称这一步为 **Combining Locations**，我找不出合适的翻译，就姑且保留英文原文。

这个排序的方法也很简单。作者先给各个 region 一个序号，前面说了，Selective Search 是一个逐步合并的层级结构，因此，我们将覆盖整个区域的 region 的序号标记为 1，合成这个区域的两个子区域的序号为 2，以此类推。但如果仅按序号排序，会存在一个漏洞，那就是区域面积大的会排在前面，为了避免这个漏洞，作者又在每个序号前乘上一个随机数 $RND \in [0, 1]$，通过这个新计算出来的数值，按从小到大的顺序得出 region 最终的排序结果。

### 参考

+ [Selective Search for Object Recognition(阅读)](http://blog.csdn.net/langb2014/article/details/52575507)
+ [Efficient Graph-Based Image Segmentation](http://cs.brown.edu/~pff/segment/)