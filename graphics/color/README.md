# Color
## 颜色的表示

颜色其实是一个能量分布的积分（单位watt）。比较精确的表示是这样的：

```c++
template<int nSamples>
struct Spectrum
{
    float c[nSamples];
};
```

这一些列系数其实是一个基的系数。比如用高斯函数作为基。这样的话，一个Spectrum示例就可以表示一个复杂的概率密度分布了。

还可以直接把这个数组当作能量分布的采样点，也就是把光谱能量分布$S(λ)$离散化。给定一个具有$n$个（波长，能量）的采样点，构造一个具有$m$个采样点的```Spectrum```。这里面涉及到一些简单的计算。也就是把n个采样点重新平均到$m$个采样点上。



## 采样的颜色表示转换为RGB颜色表示

因为人的眼睛对三种颜色比较敏感，所以可以用三个分量来作为一组基表示颜色。这也是为什么好多颜色空间都用三个分量来表示。
我们知道一个光谱的能量分布为$S(λ)$，

### SPD & SSF

- Spectral Power Distribution: $\Phi(\lambda)$

- Spectral Sensitivity Function: $f(\lambda)$

- $color=\int \Phi(\lambda) f(\lambda)d\lambda$

$blue = \int_\lambda \Phi(\lambda) S(\lambda)d\lambda$

$green=\int_\lambda \Phi(\lambda) M(\lambda)d\lambda$

$red=\int_\lambda \Phi(\lambda) L(\lambda)d\lambda$

### While Balance

The process of removing color casts so that we would perceive as white are rendererd as white in final image






