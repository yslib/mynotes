# 拉普拉斯算子 & 拉普拉斯矩阵
**拉普拉斯矩阵就是图上的拉普拉斯算子，或者说是离散的拉普拉斯算子**

拉普拉斯矩阵：半正定->可以写成二次型。 实对称阵->可以用正交矩阵进行相似对角化



## 拉普拉斯算子
梯度的散度
从拉普拉斯算子上看，算子计算的是这一点的扰动增益，即到周围邻域的距离平方和。可类比于方差。

二阶导数 拉普拉斯算子


可以拿热传导方程来举例

GCN


CNN

只有最后是全连接层
CNN 的影响是局部的，参数共享，局部连接性


## 卷积
利用卷积定理和快速离散傅里叶变换可以加速卷积的计算



## 傅里叶级数
可以将任意周期函数写成三角函数的级数

如果我们以向量的角度来看，傅里叶级数的基是无穷维的，且傅里叶级数的基是三角函数和一个常数。

**无穷维的两个向量点积在形式上可以类比作是两个函数乘积的积分**, 因为积分就是无限累加。

$v_1 w_1 + v_2 w_2 + ... + v_n w_n + ... = vw$

$v(x)w(x) = \int_{a}_{b} v(x)w(x)dx$

## GCN: Graph Convolution Network:

Given a graph $G = (V, E)$

- Feature Matrix: $X$: a $N \times D$ matrix, which represents $N$ nodes and its feature, a vector of $D$ dimensions.

- Adjacent Matrix: $N \times N$, a symetric matrix.

- Degree Matrix: $D$: $N \times N$, a diagnal matrix.


### **Feature Extraction**:

Like the filter in CNN by summizing and averaging, in GCN:

$agg(X_i) = \sum_j{A_{ij}X_j}$

for all nodes in the graph $G$, we can derive

$agg(X) = AX$

The procedure is called as **Aggregation**. (Think about in row space of X)

But only adjacent nodes are considered above, if we want to considered the current node itself, just add the node into it:

$agg(X_i) = \sum_j{A_{ij}X_j} + X_j$

Then

$agg(X) = AX+X = (A+I)X$

Denotes $A+I$ as $\bar{A}$


#### **Think in another way**

We could consider the feature represented by the difference between the adjacent nodes and itself

$agg(X_i) = \sum_j{A_{ij}(X_i-X_j)} = D_{ii}X_i - \sum_jA_{ij}X_j$

for all nodes in graph $G$:

$agg(X) = DX - AX = (D-A)X$

where
 $D-A$ is **Laplace Matrix**

