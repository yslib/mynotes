# GLSL in Vulkan

Vulkan只支持SPIR-V,一种JIT执行的中间语言。但是现有的很多高级着色语言都可以通过工具翻译成SPIR-V。有些驱动支持GLSL扩展，可以直接在Vulkan中
使用GLSL，但这是不推荐使用的非主流用法。

一般情况下，转换为SPIR-V的GLSL也和OpenGL里的GLSL有些不同。在这里主要记录一下不同的地方。


所有的非透明对象必须放在uniform block里。

gl_VertexID -> gl_VertexIndex
gl_InstanceID -> gl_InstanceIndex

所有的变量声明都需要布局操作符。

>这里有一篇讲如何将OpenGL 移植到Vulkan 上的实践。
>https://on-demand.gputechconf.com/gtc/2016/events/vulkanday/Migrating_from_OpenGL_to_Vulkan.pdf
