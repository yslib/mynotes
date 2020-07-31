# 过程式着色(Procedural Shading)

[Procedural GPU Shading Ready for Use][1]大概介绍了过程式着色的概念。在过程式着色中，很重要的一个部分是随机数的生成。[这个仓库][2]实现了大量且高效的随机数生成算法。实际应用中，就可以使用这些写好的代码。接下来的部分主要简要地记录所涉及到的算法。

[The book of shader](https://thebookofshaders.com/)是一个非常好的关于Shader艺术的教程。这里的笔记都属于这个教程的子集。
### 简单的随机函数

在一些着色器中，经常见到这样的hash函数:
```glsl
float rand(float n){return fract(sin(n) * 43758.5453123);}
```
或者：
```glsl
fract(sin(dot(co.xy ,vec2(12.9898,78.233))) * 43758.5453);
```
如果用函数画图工具画一下的话就会发现，这些函数值看起来很像是随机的，通过把周期函数的振幅设置的很大，用fract把函数值取小数部分，就可以得到看起来非常杂乱无章的分布。二维的同样如此。只不过让人迷惑的Magic Number的最早出处确实找不到了。以下是两个讨论这两个数的链接

* [https://stackoverflow.com/questions/12964279/whats-the-origin-of-this-glsl-rand-one-liner]()

* [可能的论文](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.141.3911&rep=rep1&type=pdf)


### 值噪声 （Value Noise）

   在晶格体的顶点随机生成一系列数，然后在采样点进行线性插值。

### 网格噪声 （Cellular Noise）

[A Cellular Texture Basis Function](http://www.rhythmiccanvas.com/research/papers/worley.pdf)

   这篇文章提出了网格噪声，这种噪声就像蜂窝(Cellular)一样，是一个距离场，与KNN有些类似。

### 柏林噪声 (Perlin Noise)

* [文章]()
* [改进](https://mrl.nyu.edu/~perlin/paper445.pdf)

柏林噪声是一种梯度噪声，与值噪声不同。梯度噪声在晶格点上随机生成方向向量作为梯度，然后在连续点上与晶格点的方向和这个晶格点的梯度求点积，然后对与周围所有晶格点得到的点积做插值。这个插值可以是线性插值也可以是6*x^5 - 15x^4 + 10x^3。
这种噪声的特点是在各个方向上平滑，相对于值噪声，在晶格边界上没有明显的分割现象。常用于生成纹理噪声。以及柏林噪声的[改进](https://mrl.nyu.edu/~perlin/paper445.pdf)

### 单形噪声（Simplex Noise）

* [Simplex noise demystified](http://weber.itn.liu.se/~stegu/simplexnoise/simplexnoise.pdf)

单形或单纯形（Simplex）是一个几何术语。N维单纯形是欧几里得空间中 n+1个处在一般位置（仿射无关：所谓仿射无关就是线性无关）的点的凸包。通俗一点讲就是撑起N维空间最少的点。
0维单纯形是点，1维单纯形是线段，2维单纯形是三角形，3维单纯形是四面体。下面是两个简单的结论。
    1. N维单纯形有N+1个点
    2. N!个N维单纯形能构成N维空间的超晶体格。比如3! = 6个 3维单纯形四面体能够组成一个变了形的立方体。

到这里，我们大概可以猜到单形噪声与柏林噪声的不同了。单纯形噪声也是梯度噪声，这里和柏林噪声相同。但是N维柏林噪声的晶体格是超晶体格，而单形噪声的晶体格就是单形。因此在高维噪声中，生成单形噪声的复杂度降低了不少。这里主要有两个改进：

1. 从超晶格体变为单纯形可以让高维空间的复杂度大幅度减小。
2. 柏林噪声最终对梯度影响还是要求插值的，单纯形直接求和，简单不少。

然而在柏林噪声中，我们通过空间中采样点周围整数的顶点（也就是超晶格体的顶点，如三维空间中的正方体）的梯度来计算当前采样点的噪声。但对于单纯形噪声，给了采样点坐标，确定这个采样点所在的单纯形不像对立方体（正超晶格体）确定顶点那样直观和简单。具体过程在上面链接的文章里。

### 柏林噪声和单纯形噪声实现和效率对比
* [Efficient computational noise in GLSL](http://weber.itn.liu.se/~stegu/jgt2012/article.pdf)
* [Supplymentary](http://weber.itn.liu.se/~stegu/jgt2011/supplement.pdf)
* [Benchmark Demo](http://www.itn.liu.se/~stegu/simplexnoise/GLSL-noise-vs-noise.zip)

下面是我的机器上的运行结果:

```output
        GL vendor:     NVIDIA Corporation
        GL renderer:   GeForce GTX 1060 6GB/PCIe/SSE2
        GL version:    4.6.0 NVIDIA 445.87
        Desktop size:  1920 x 1080 pixels
        Noise version: 2011-04-10

        Constant color shading, 8001.8 Msamples/s
        Single texture shading, 7633.6 Msamples/s
        Texture LUT 2D simplex noise, 8065.8 Msamples/s
        Textureless 2D simplex noise, 6108.4 Msamples/s
        Texture LUT 2D classic noise, 6470.4 Msamples/s
        Textureless 2D classic noise, 6487.9 Msamples/s
        Texture LUT 3D simplex noise, 6359.5 Msamples/s
        Textureless 3D simplex noise, 4541.0 Msamples/s
        Texture LUT 3D classic noise, 5576.5 Msamples/s
        Textureless 3D classic noise, 3901.8 Msamples/s
        Texture LUT 4D simplex noise, 4478.7 Msamples/s
        Textureless 4D simplex noise, 2996.7 Msamples/s
        Texture LUT 4D classic noise, 3465.4 Msamples/s
        Textureless 4D classic noise, 2117.6 Msamples/s
```
TextureLess是完全不用查表的方法。Texture LUT是有一些预设的数据数纹理。
可以看到，在3维及其以上的的噪声中，单纯形噪声的效率提升较为明显。二维的情况下柏林噪声就足够了，因为实现起来较为简单。
    



[1]: http://weber.itn.liu.se/~stegu/gpunoise/GPU-proceduralshading.pdf "GPU-proceduralshading"
[2]: https://github.com/ashima/webgl-noise "gl noise"
