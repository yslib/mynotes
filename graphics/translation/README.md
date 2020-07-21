## 切线空间
在用法线贴图的时候，如果表面的朝向和法线贴图的坐标系不一致的话结果就会有问题。

比如给一张法相贴图，如果不经过处理，那么这个贴图只能描述空间当中朝向特定方向的法线。切线空间就是把这个固定朝向的法线变换到另外一个朝向的矩阵。这个矩阵在实现raytracing的brdf中就用到了。

T(tangent)  切线。向右。

B(bitangent)  副切线。可以通过切线和法线求出来。

## 裁剪空间以及透视除法

说的简单一些，裁剪空间就是透视投影的平截头体对应的空间或者平行投影对应的长方体的空间，其坐标范围就是这个头体的范围。所谓的透视除法
就是归一化，把齐次坐标变成了NDC坐标。至于这个坐标的范围是什么是由投影矩阵决定的。D3D-like 和 OpenGL-like 不太一样。

## 投影纹理映射(Projective texture mapping)
---


应用：把一张纹理像投影仪一样投射到三维场景中。

原理：原理也是非常简单的。设想一个投影仪的平截头体的前界面对应要投影的纹理，以及这个平截头体对应的MVP矩阵。
把场景中的顶点通过这个矩阵的变换，得到的在裁剪空间中的顶点坐标归一化（透视除法）就可以作为这个纹理的纹理
坐标进行纹理采样。(说明图可以在任意搜索引擎上找到)

```
// vertex shader
gl_Position = camera_MVP * vertex;
texCoord = projector_MVP * vertex;

```
```
// fragment shader
texCoord.xy = texCoord.xy / texCoord.w (projective divide)
gl_FragColor = texture(texImage,texCoord)
```

在Unity的着色器里与之有关的一个内置函数是**tex2Dproj(tex,i.uvproj)**，也就是说这个函数只接受没有经过透视除法的齐次坐标。
这个函数相当于**tex2D(tex,i.uv/i.w)**。为什么这里的tex2Dproj 要这样设计呢？因为有很多内置函数的返回值是齐次坐标。
这里用**float4 ComputeScreenPos(clipPos)** 和 **float4 ComputeGrabScreenPos(clipPos)** 作为例子。这两个函数以裁剪坐标作为参数，
返回的是对应的归一化的屏幕坐标与clipPos.w 的积（实现看源码），也就是说，如果想要在采样纹理，还需要
除以clipPos.w。比如如果想实现一个物体对扭曲效果，则着色器的输出大概为：

```
  return tex2Dproj(bgTex,ComputeGrabScreenPos(objectDistorted.clipPos));
```


