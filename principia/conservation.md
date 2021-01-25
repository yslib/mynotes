# 守恒定律是什么

我们从纯数学的角度来重新审视中学学过的能量守恒和动量守恒,然后我们再通过同样的角度得出角动量守恒。

能量守恒，动量守恒和机械能守恒都是运动积分。 为了明白什么是运动积分，需要弄清楚下面两个问题。
1. **动能定理** 和 **动量定理**与**牛顿第二定律**的关系
2. **动能定理** 和**动量守恒** 以及 **动量定理** 和**动量守恒**的关系。


### 前置知识：

0. Nabla算子

$$
\nabla
$$

1. 标量场$\Psi(\vec{r})$的**梯度**是**向量**:
   $$\nabla\Psi = \frac{\partial{\Psi}}{\partial{x}}\vec{i} + \frac{\partial{\Psi}}{\partial{y}}\vec{j}+ \frac{\partial{\Psi}}{\partial{z}}\vec{k}$$

2. 矢量场$\vec{A}(\vec{r})$ 的**散度**和**旋度**是**标量**:
   散度：

   $$
   \nabla \cdot \vec{A}=\frac{\partial{A_x}}{\partial{x}} + \frac{\partial{A_y}}{\partial{y}}+ \frac{\partial{A_z}}{\partial{z}}
   $$

   旋度：

   $$
   \left|\begin{array}{cccc}
   1& 2 &3    \\
   1& 2 &3    \\
   1& 2 &3
   \end{array}\right|
   $$


## 能量守恒

$$
\vec{F} = m\ddot{\vec{r}} \Rightarrow \vec{F} = m \frac{d\vec{v}}{dt}
$$

如果我们定义一个量
$$T = \frac{1}{2}mv^2 $$

那么就有
$$
\vec{F}d\vec{r} = m\frac{d\vec{v}}{dt}d\vec{r} = d(\frac{1}{2}mv^2)
$$

得到动能定理
$$\vec{F}d\vec{r} = dT$$

其积分形式

$$
T_2-T_1 = \int_{\vec{r_1}}^{\vec{r_2}}\vec{F}d\vec{r}
$$

**如果$\vec{F} = 0$，运动前后动能不变**.

## 动量守恒

$$
\vec{F} = m\ddot{\vec{r}} \Rightarrow \vec{F} = m \frac{d\vec{v}}{dt}
$$

如果我们定义一个量$\vec{p} = m\vec{v}$

就得到
$$
\vec{F} = \frac{d\vec{p}}{dt}
$$

首先我们需要明白，动量定力和牛顿第二定律$\vec{F} = m\ddot{\vec{r}}$ 都是二阶微分方程，也就是他们的次数是一样的，从形式上来讲他们是等价的。实际上，牛顿最开始提出来的第二定律其实就是这个我们今天所熟悉的**动量定理**$\vec{F} = \frac{d\vec{p}}{dt}$

然后我们把这个动量定理对**时间**积分，得到
$$
\vec{p_2} - \vec{p_1} = \int_{t_1}^{t_2}\vec{F}dt
$$

**如果$\vec{F} = 0$，运动前后动量不变**, 因为和牛顿第二定律等价，所以也可以从牛顿第二定律的角度来解释，**如果合外力为0,那么速度不变**。
当然以上都是显而易见的结论，但重点在下面:

也就是说
$$
\vec{F} = 0 \Rightarrow \vec{p} = \vec{C}
$$
其中的这个 **$\vec{p}= \vec{C}$** 是一个**守恒量**。
我们知道，$\vec{p}$里面的速度是$\vec{r}$的一阶导数，因此，如果我们以计算出系统的位置$\vec{r}$为目标，那么这个守恒量$\vec{p} = \vec{C}$就是一个一阶微分方程，相比于牛顿第二定律或者说动量定理，已经降了一阶了。


## 动量矩守恒（角动量守恒）

### 矩
位置的矢量和矢量的叉积
### 力矩
位置矢量和力
$$
\vec{M} = \vec{r}\times\vec{F}
$$