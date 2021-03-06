## 视图矩阵
在描述相机的时候，我们需要三个参数，即相机在世界坐标中的位置以及右方向和上方向。可以通过叉积确定前方向。这三个方向就是相机坐标系在世界坐标系中的基。所以很容易写出相机坐标系到世界坐标系的变换矩阵。这个矩阵的逆就是视图矩阵。

## 法线贴图和切线空间($TBN$)
  法线贴图一般用来取代粗糙的顶点的法线，这个法线即使经过插值之后也是非常粗糙的，没有细节表现。

  一般给的法线贴图也是定义在切线空间中的(顶点的法线当然是在世界坐标系中的，这也就是为什么我们只用顶点法线的时候不涉及这个复杂的变换)，因为这样可以做到描述切线独立于物体的空间属性。把切线空间比变换到世界坐标系中的方法同视图矩阵。首先我们需要知道$T,B,N$在世界空间中的向量是什么。如果我们知道了$u,v$，也知道了三角形的顶点位置，就可以得到$T,B$，然后就可以得到$N$。**纹理坐标(u,v,0)就是定义在切线空间中的。**$u$是$T$的向量坐标，$v$是$B$(副切线)的向量坐标，而$N$(法线向量)的向量坐标是$0$。

  当给定当前点所对应的三角面片的时候，可以通过列一元二次方程来求解$T,B$。即纹理坐标作为$T,B$基的坐标的线性组合可以表示对应的三角面片的两个边（已知）。然后可以求出$T,B$。

  * 注意： 对于在模型空间中的$TBN$（也就是用没有经过模型变换的顶点计算出的$TBN$）需要用**法线矩阵**进行变换。

上面是理论的部分。下面讨论一下实现上的一些优化。
  1. 在一般的应用中，模型都会给定顶点的法向量，因为$TBN$作为正交基相互垂直，所以我们可以只计算$T$向量，$B$向量通过$T$和$N$计算得到。
  2. 在绘制流水线中，我们为每个面片计算$TBN$,顶点的$TBN$来自所在面片的$TBN$的平均，然后作为逐顶点属性传给着色器。(和平均顶点法向量相同)。但是这里会出一个小问题，那就是平均后的$TBN$不再是正交矩阵了，也就是相互不垂直。对于这种情况，应该现在顶点着色器中进行史密斯特正交化，比如先把$T$与$N$正交化，然后再算$B$。
  3. 我们一般**在顶点着色器里把所有光照需要的参数通过$TBN$变换到切线空间中，然后通过光栅化插值，最后在片段着色器中采样了法线贴图后直接在切线空间进行着色**。而不是**在片段着色器里把采样的法向量通过计算好的$TBN$变换到光照参数所在的空间(一般为世界坐标系)中进行着色**。因为后者每个片段需要一次变换，而前者只是每个顶点需要需要一次变换和计算，然后通过光栅化插值求出每个片段的光照参数，再在片段着色器中采样法线后直接计算显然要比后者省下很多空间变换（硬件插值要比运行着色器代码快很多）。

  ### 视差贴图
视差贴图的原理是在片段着色器中**改变了实际采样的纹理的位置，并没有改变顶点**。只是**改变了着色**，所以**空间感并不真实**。
![视差贴图原理](https://learnopengl-cn.github.io/img/05/05/parallax_mapping_scaled_height.png)
上图为视察贴图的原理，简单的说就是把片段A的着色变为对应的B点下方片段对应的着色，也就是在**视线方向**往回移动A片段对应的**位移贴图中的长度**，在**切线空间**的平面上找到A片段**纹理坐标的偏移(offset)**。
视差贴图的计算也用到了切线空间。位移贴图应**该配合法线**贴图来使用，毕竟模拟了平面高度的变化也应该配合相应的法线，否则看起来不真实。


## 裁剪空间以及透视除法

说的简单一些，裁剪空间就是透视投影的平截头体对应的空间或者平行投影对应的长方体的空间，其坐标范围就是这个头体的范围。所谓的透视除法
就是归一化，把齐次坐标变成了NDC坐标。至于这个坐标的范围是什么是由投影矩阵决定的。D3D-like 和 OpenGL-like 不太一样。

## 投影纹理映射(Projective texture mapping)


应用：把一张纹理像投影仪一样投射到三维场景中。

原理：原理也是非常简单的。设想一个投影仪的平截头体的前界面对应要投影的纹理，以及这个平截头体对应的MVP矩阵。
把场景中的顶点通过这个矩阵的变换，得到的在裁剪空间中的顶点坐标归一化（透视除法）就可以作为这个纹理的纹理
坐标进行纹理采样。(说明图可以在任意搜索引擎上找到)

```glsl
// vertex shader
gl_Position = camera_MVP * vertex;
texCoord = projector_MVP * vertex;

```
```glsl
// fragment shader
texCoord.xy = texCoord.xy / texCoord.w (projective divide)
gl_FragColor = texture(texImage,texCoord)
```

在Unity的着色器里与之有关的一个内置函数是```tex2Dproj(tex,i.uvproj)```，也就是说这个函数只接受没有经过透视除法的齐次坐标。
这个函数相当于```tex2D(tex,i.uv/i.w)```。为什么这里的tex2Dproj 要这样设计呢？因为有很多内置函数的返回值是齐次坐标。
这里用```float4 ComputeScreenPos(clipPos)``` 和 ```float4 ComputeGrabScreenPos(clipPos)``` 作为例子。这两个函数以裁剪坐标作为参数，
返回的是对应的归一化的屏幕坐标与```clipPos.w``` 的积（实现看源码），也就是说，如果想要在采样纹理，还需要
除以```clipPos.w```。比如如果想实现一个物体对扭曲效果，则着色器的输出大概为：

```ShaderLab
  return tex2Dproj(bgTex,ComputeGrabScreenPos(objectDistorted.clipPos));
```

