# 刚体

## 自由度
任取刚体上三个不共线的点，可以确定刚体运动的自由度。

不加约束的刚体有6个自由度。

- 平动：

运动过程中，连接刚体任意两点的直线在空间中的指向总保持平行

3个自由度

- 定轴转动：

1个自由度

- 定点转动：

3个自由度

- 平面平行运动

3个自由度

## 前置数学知识

### 同一矢量在不同坐标系下的表示

在三维坐标下的单位正交基，有如下关系成立。即**一个在这个坐标系下表示的矢量$\vec{A}$，在所有基底方向上的投影所构成的矢量就是他本身。**
显然这是一个等式。
$$
\vec{A} = \sum_i^3{\vec{A}\vec{e_i}\vec{e_i}}
$$

上面这个公式就可以用来研究坐标系的转换。

### 欧拉定理

刚体的定点转动可以等价为一次定轴转动。
也就是矩阵方程
$$
\lambda\vec{r\prime} = \vec{T}\vec{r}
$$
存在一个 $\lambda = 1$ 的特征值,使得$\vec{r\prime} = \vec{T}\vec{r}$成立。

>对于一个定点转动$\vec{T}$来说，这是一个正交矩阵，正交矩阵存在一个特征值为1的特征向量，这个特征向量就是这个转轴。


我们知道， 对于一个正交矩阵其行列式的值$\left|T\right| = \pm1$。 但是对于这种变换矩阵行列式的值为1。

>$\det{T}=1$是右手坐标系，$\det{T}=-1$是左手坐标系。


### 无限小转动

我们知道，矩阵乘法不满足交换率，也就是定点转动的每一步的顺序会对最后的结果造成影响。但我们有一个必要条件，也就是两个变换无限小的矩阵满足交换率。
$$
T_1 \cdot T_2=\left(I+\epsilon_1\right)\cdot\left(I+\epsilon_2\right) = I + \epsilon_1\epsilon_2 + \epsilon_1 + \epsilon_2
$$
其中$\epsilon_1\epsilon_2$是高阶无穷小，可以去掉。所以$T_1 \cdot T_2$和$T_2 \cdot T_1$相同。

以上只是针对一般的变换，如果是转动，有额外的约束条件即$T$是正交矩阵$T\cdot\bar{T}=I$，
这样结合之前的欧拉定理，我们又可以进一步得出结论,**其转轴为$\vec{n} = \delta_1\vec{e_1}+\delta_2\vec{e_2}+\delta_3\vec{e_3}$** 。因为此时的无穷小定点转动变换的变化量是一个反对称矩阵
$$
T\bar{T} = (T+\epsilon)(\bar{T} + \bar{\epsilon}) = I \Rightarrow T\bar{T} = I +  \epsilon + \bar{\epsilon} \Rightarrow \epsilon + \bar{\epsilon} = 0
$$
从而这个无限小转动矩阵也是反对称矩阵。

有了无限小转动矩阵是一个反对称矩阵的形式，再求一下特征值为1的特征向量。就是 **$\vec{n} = \delta_1\vec{e_1}+\delta_2\vec{e_2}+\delta_3\vec{e_3}$** 。

有了旋转轴，接下来我们探究刚体每一点的**无限小变化$\mathrm{d}{\vec{r}} = \vec{r\prime} - \vec{r}$** 来确定旋转角度，进而引出角速度。
由
$$
\mathrm{d}{\vec{r}} = \vec{r\prime} - \vec{r} = (I + \epsilon)\vec{r} - \vec{r}
$$
可以计算出 **$\mathrm{d}{\vec{r}} = \vec{n}\times\vec{r}$** 。 这个结论非常直观，就是$\vec{r}$在垂直转周上的分量的圆周运动的改变量，可以想想一下那个圆锥。角度的变化量就是角度的模。

然后我们再从几何上考虑，即弧长增量为角度增量和半径的积 **$\left|\mathrm{d}\vec{r}\right| = r_\perp\mathrm{d}\phi$** ，与 **$\mathrm{d}{\vec{r}} = \vec{n}\times\vec{r}$** 两边取模后有相同的形式，那我们就可以认为 **$\mathrm{d}\phi=\left|\vec{n}\right|$** ,即角度的改变量大小为用无限小转动矩阵中的小量表示的旋转轴的模长。 这样,我们就可以把旋转轴表示为($\vec{e_n}$为转轴的方向向量)
$$
\vec{n} = \mathrm{d}{\phi}\cdot\vec{e_n}
$$
转动改变量表示为：
$$
\mathrm{d}{\vec{r}} = \mathrm{d}{\phi}\cdot\vec{e_n}\times\vec{r}
$$
因此，就有**速度**
$$
\vec{v} = \frac{\mathrm{d}\vec{r}}{dt} = \dot{\phi}\vec{e_n} \times \vec{r}
$$
这个就是欧拉公式，当然了，这个是推广到了定点转动下的绕定点转动的速度。不仅仅是简单的定轴转动，虽然他们的形式是一样的，但是含义差别很大。

进而有**加速度**:

$$
\vec{a} = \frac{d\vec{a}}{dt} = \dot{\vec{\omega}} \times \vec{r} + \vec{\omega} \times \left(\vec{\omega}\times\vec{r}\right)
$$

>我没有写出来推导过程是因为现在暂时写不出矩阵的latex表达式，这个直接把矩阵的形式写出来就能推出，或者直接由结论展开成成矩阵形式验证一下。

## 欧拉角

三个自由度，三个旋转角度来表示，三个定轴转动。

### 进动

### 章动

### 自转