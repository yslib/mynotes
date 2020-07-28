# GLSL in Vulkan
---

Vulkan只支持SPIR-V,一种JIT执行的中间语言。但是现有的很多高级着色语言都可以通过工具翻译成SPIR-V。有些驱动支持GLSL扩展，可以直接在Vulkan中
使用GLSL，但这是不推荐使用的非主流用法。

一般情况下，转换为SPIR-V的GLSL也和OpenGL里的GLSL有些不同。在这里主要记录一下不同的地方。

1. 所有的非透明对象必须放在uniform block里。

2. 
```
gl_VertexID 改为 gl_VertexIndex
gl_InstanceID 改为 gl_InstanceIndex
```
2. 所有的变量声明都需要布局操作符。在OpenGL里也推荐使用布局操作符，所有行为都需显式指定，这也是Vulkan的特点之一。


>这里有一篇讲如何将OpenGL 移植到Vulkan 上的实践。就算不使用Vulkan,在OpenGL里也应该这样使用。
>https://on-demand.gputechconf.com/gtc/2016/events/vulkanday/Migrating_from_OpenGL_to_Vulkan.pdf

