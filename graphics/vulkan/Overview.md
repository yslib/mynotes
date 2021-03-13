# Vulkan Overview

Explicitly,Statically,Seperately

```mermaid
graph BT
linkStyle default interpolate basis;
subgraph g0[Vulkan]
subgraph g1[Vulkan Context]

A(VkDevice):::vkobj --> B(VkPhysicalDevice):::vkobj
B --> C(VkInstance):::vkobj
D(VkSurfaceKHR):::khr--Present support-->B
D-->C
D-->W{{NativeWindowHandle}}
S(VkSwapchainKHR):::khr-->D
S-->A
end

subgraph g2[Resources]
i(vkimage):::vkobj-.->A
b(vkbuffer):::vkobj-.->A
end
end

classDef khr fill:#98971a,stroke-width:0px;
classDef vkobj fill:#d79921,stroke-width:0px;
style g1 fill:#928374,stroke-width:0px,color:#ebdbb2;
style g2 fill:#928374,stroke-width:0px,color:#ebdbb2;
style g0 fill:#665c54,stroke-width:0px,color:#ebdbb2;
style W fill:#4585ff55,stroke-dasharray:4


```

**Explicit, Explicit, Explicit**

## 实例(vkinstance)
### 简介
vulkan实例隔离了不同的vulkan环境，在一个应用程序中，可以创建多个实例。但是实例之间的对象不能共享，如内存。（在不涉及扩展的情况下）

```mermaid
graph LR
subgraph g0[vkInstance]
linkStyle default interpolate basis
ci[CreateInfo]
i(VkInstance):::vkobj
o([应用]):::none--查询当前Vulkan支持的扩展-->b([指定扩展]):::field
subgraph g1[功能]
b -->ci:::cistyle
c([指定验证层]):::field-->ci
d([指定Vulkan对象的全局内存分配器]):::field-->ci
z([...]):::field-->ci
end

end

ci--VkCreateInstance-->i

classDef field fill:#689d6a,stroke-width:0px;
classDef vkobj fill:#d79921,stroke-width:0px;
classDef cistyle fill:#b16286,stroke-width:0px;
style o fill:#fe8019,stroke-width:0px;
style g1 fill:#928374,stroke-width:0px,color:#ebdbb2;
style g2 fill:#928374,stroke-width:0px,color:#ebdbb2;
style g0 fill:#665c54,stroke-width:0px,color:#ebdbb2;


```


### 功能

1. 枚举当前环境支持的扩展:


2. 是否启用扩展：

  真正启用扩展的地方是在创建逻辑设备的时候，但是是否允许开启扩展的地方是在创建实例时。

3. 是否开启验证层：

  包括调试回调。调试回调也属于扩展。

4. 指定全局内存分配器：

  Vulkan提供了一个全局内存分配器回调，让用户可以接管Vulkan对象的所需要使用主机端的内存的分配。

## 物理设备(vkPhysicalDevice)
### 简介
在初始化Vulkan的物理设备时，除了选择需要的物理设别外，还应该获取关于物理设备的一些属性供之后使用

```mermaid
graph LR
subgraph g0[VkPhysicalDevice]
linkStyle default interpolate basis
i(vkInstance):::vkobj--VkEnumeratePhysicalDevices-->p(vkPhysicalDevice):::vkobj
subgraph g1[物理设备的Get 查询]
p-->a([vkGetPhysicalDeviceFormatProperties]):::get
p-->c([vkGetPhysicalDeviceMemoryProperties]):::get
p-->d([vkGetPhysicalDeviceQueueFamilyProperties]):::get
p-->e([vkGetPhysicalDeviceFeatures]):::get
p-->f([vkGetPhysicalDeviceXXXKHR]):::get2
end
end

classDef field fill:#689d6a,stroke-width:0px;
classDef vkobj fill:#d79921,stroke-width:0px;
classDef cistyle fill:#b16286,stroke-width:0px;
classDef get fill:#458588,stroke-width:0px;
classDef get2 fill:#fb4934,stroke-width:0px
style g1 fill:#928374,stroke-width:0px,color:#ebdbb2;
style g0 fill:#665c54,stroke-width:0px,color:#ebdbb2;
```

### 功能
1. 图像格式属性(vkGetPhysicalDeviceFormatProperties)：

  图像格式属性的描述结构体为：

  通常，为了完整性，在创建一个需要指定格式的Vulkan对象时，比如创建一个R8B8GBA8格式的纹理，或者格式为24位的深度缓冲，需要查询物理设备是不是支持这种格式。但是一个格式还有附加的属性，比如这个格式是不是支持

2. 设备内存：



3. 支持的队列族：

  同样也用索引来标识，创建逻辑设备时需要指定。队列也是不同于之前传统API的新特性。

4. 物理设备属性(VkPhysicalDeviceProperties via vkGetPhysicalDeviceProperties)

  物理设备属性字段中有一个设备限制（VkPhysicalDeviceLimits）字段，这个字段给出了当前物理设备支持的各种属性的极限或者说是最大值。
  其中比较基础的有以下几项：
    
    - 最大采样数

    - 

5. 物理设备特性(VkPhysicalDeviceFeatures via vkGetPhysicalDeviceFeatures)

  注意物理属性做区分，这个字段当中都是Bool变量，描述了此物理设备是否支持某一个特性。在之后创建逻辑设备中，要指定一些特性，这些特性必须是物理设备支持的。


## 逻辑设备(vkDevice)
### 简介

  逻辑设备是Vulkan对物理设备的抽象。可以从一个物理设备上创建多个逻辑设备。逻辑设备对象负责vulkan的设备资源分配。
  负责的主要功能有：

### 功能

  1. 开启扩展：

     如果需要窗口显示功能，这里需要开启交换链扩展

  2. 指定设备要用到的队列：

     如果需要绘制流水线功能就指定图形队列，如果要使用计算着色器的功能，就指定计算队列。

## 表面（vkSurfaceKHR, 扩展）

### 简介

  表面是展示渲染结果的那个窗口对象，可以不严谨地理解为呈现绘制结果的区域。表面不是Vulkan核心的一部分。因为显示窗口不具备跨平台特性。

### 功能

  表面并且和表面相关的API带有**KHR**后缀。

  - 表面的创建需要依赖与实例(VkInstance)。
  - 表面是否被支持取决于于物理设备(VkPhysicalDevice)(vkGetPhysicalDeviceSurfaceSupportKHR)。可见物理设备章节的说明图中红色的部分。
  - 表面的创建需要依赖于一个平台相关的窗口句柄。

  以Windows上的平台为例，下面是创建表面所需要的信息

```cpp
// Provided by VK_KHR_win32_surface
typedef struct VkWin32SurfaceCreateInfoKHR {
  VkStructureType sType;
  const void* pNext;
  VkWin32SurfaceCreateFlagsKHR flags;
  HINSTANCE hinstance;
  HWND hwnd;            // win32 native window handle
} VkWin32SurfaceCreateInfoKHR;

```
创建函数,需要用到Vulkan实例

```cpp
// Provided by VK_KHR_win32_surface
VkResult vkCreateWin32SurfaceKHR(
  VkInstance instance,                 // vulkan instance is needed
  const VkWin32SurfaceCreateInfoKHR* pCreateInfo,
  const VkAllocationCallbacks* pAllocator,
  VkSurfaceKHR* pSurface);

```

从接口来看，判断设备是否支持交换链之后，在窗口系统下，我们就可以直接窗前表面（VkObject）对象了。


## 平面(Plane,扩展)
### 简介
  
  与平面平行的概念是窗口句柄。根据Vulkan规范，这个扩展由VK_KHR_display提供。这个对象用来代替之前创建表面(VkSurface)用的平台相关的窗口句柄来创建一个表面。也就是说，这个扩展可以实现在没有窗口系统上直接把结果绘制到显示器上的功能。（目前为止，没见过使用这个扩展的任何Demo，我自己也没考察过）
  
  可以通过与之前创建表面的信息结构体做对比会发现，通过这种方法创建表面不许要窗口句柄。而是一个Plane，这个Plane对象直接从物理设备获得支持。

  ```cpp
typedef struct VkDisplaySurfaceCreateInfoKHR {
  VkStructureType sType;
  const void* pNext;
  VkDisplaySurfaceCreateFlagsKHR flags;
  VkDisplayModeKHR displayMode;
  uint32_t planeIndex;
  uint32_t planeStackIndex;
  VkSurfaceTransformFlagBitsKHR transform;
  float globalAlpha;
  VkDisplayPlaneAlphaFlagBitsKHR alphaMode;
  VkExtent2D imageExtent;
} VkDisplaySurfaceCreateInfoKHR;
// no need for native window handle
  ```

 注意与之前的```vkCreateWin32SurfaceKHR``` 对比
  ```cpp
// Provided by VK_KHR_display
VkResult vkCreateDisplayPlaneSurfaceKHR(
  VkInstance instance,
  const VkDisplaySurfaceCreateInfoKHR* pCreateInfo, // replace somthing like VkWin32SurfaceCreateInfoKHR that must be supported by window system
  const VkAllocationCallbacks* pAllocator,
  VkSurfaceKHR* pSurface);
```

## 交换链(vkSwapchainKHR, 扩展)

### 简介

交换链提供和管理表面中的绘制结果数据，一般情况下是一个环形的缓冲，一些表面用来显示在窗口上，一些表面用来接受绘制的结果供接下来的展示用。相当于帧缓冲的管理器，把绘制完成的数据呈现到表面上。
创建交换链需要指定之前创建的表面以及逻辑设备。因此交换链是被逻辑设备所拥有的。
交换链由**VK_KHR_swapchain**扩展提供，因此如果需要交换链，创建逻辑设备时需要启用这个扩展。

创建交换链时，至少有三个属性是有必要检查的。(或者说是必要的)

  - 交换链的图像个数（缓冲个数），图像大小(```vkGetPhysicalDeviceSurfaceCapabilitiesKHR```)
  - 支持的表面格式(```vkGetPhysicalDeviceSurfaceFormatsKHR```)
  - 呈现模式(立即刷新，三缓冲等)(```vkGetPhysicalDeviceSurfacePresentModesKHR```)

```cpp
// Provided by VK_KHR_swapchain
typedef struct VkSwapchainCreateInfoKHR {
  VkStructureType sType;
  const void* pNext;
  VkSwapchainCreateFlagsKHR flags;
  VkSurfaceKHR surface;               // create before
  uint32_t minImageCount;             // check
  VkFormat imageFormat;               // check
  VkColorSpaceKHR imageColorSpace;    // check
  VkExtent2D imageExtent;             // check
  uint32_t imageArrayLayers;
  VkImageUsageFlags imageUsage;
  VkSharingMode imageSharingMode;
  uint32_t queueFamilyIndexCount;
  const uint32_t* pQueueFamilyIndices;
  VkSurfaceTransformFlagBitsKHR preTransform;
  VkCompositeAlphaFlagBitsKHR compositeAlpha;
  VkPresentModeKHR presentMode;         // check
  VkBool32 clipped;
  VkSwapchainKHR oldSwapchain;
} VkSwapchainCreateInfoKHR;
```

```cpp
VkResult vkCreateSwapchainKHR(
  VkDevice device,
  const VkSwapchainCreateInfoKHR* pCreateInfo,
  const VkAllocationCallbacks* pAllocator,
  VkSwapchainKHR* pSwapchain);
```

## 如何使用 Vulkan
前面所涉及的实例、物理设备逻辑设备、表面、交换链都是使用Vulkan之前必须要做的基础性工作，不涉及任何渲染的操作。我们把渲染一帧大概三部分
```cpp

ConfigureGraphicsAPI(){
  // 对于Vulkan来说，枚举物理设备，创建物理设备，逻辑设备交换链等
  // 创建窗口
}
while(true){
  if(not Init){
    Initialize(){
      // 数据初始化工作，载入模型，建立渲染管线状态等
      vertexBuffer = device->CreateBuffer(USAGE_VERTEX_BUFFER);
      vertexBuffer->SetData(vertexData)
      uniform = device->CreateBuffer(USAGE_SHADER_UNIFORM);
      vertexBuffer->SetData(uniformData)

      pipelineState = device->CreatePipelineState(pipelineDesc);
    }
  }
  Update(){
    // 更新每帧数据
    context->UpdateBuffer()
  }
  Render(){
    // 提交绘制指令
    context->SetPipelineState(pip)
  }
}
```

我们之前所涉及的仅仅是在```ConfigGraphicsAPI()```中的需要进行的工作。完成了前几个章节的工作之后，我们就需要开始使用Vulkan来绘制东西了。但是由于Vulkan的复杂性，如果不清楚接下来要做什么，就会给接下来的学习带来困难。首先我们要明确Vulkan和OpenGL的最大的不同是什么。其中一个最明显的区别就是指令的显式执行。

例如在OpenGL中，如果我们创建一个Buffer并且给Buffer传数据(对于纹理对象大体也是如此)，基本上只需要两个主要的API：
```cpp

// 其他工作
glCreateBuffer(&bufferHandle);  // 创建缓冲对象
glNamedBufferSubData(bufferHandle,vertexData) // 给缓冲对象传数据
// 使用缓冲对象
```
然而在Vulkan中，要想完成同样的操作，需要更多的工作:
- 创建Buffer对象的时候，我们需要给Buffer分配内存，这就需要提前创建一个Memory对象。然后从Memory中给Buffer分配内存，这样这个Buffer才能是一个完整可用的Buffer。

上面的这个工作大致对应```glCreateBuffer```。

接下来，要给Buffer数据，
- 则需要建立一个命令缓冲池(**VkCommandPool**)

- 从中分配命令缓冲(**VkCommandBuffer**)

- 向命令缓冲中提交从SrcBuffer到DstBuffer的复制命令(**VkCmdCopyBuffer**)

- 然后把这个命令缓冲提交到我们之前创建的图形队列中。(**VkQueueSubmit**)

因为我们最终使用的顶点数据是以设备内存形式存在的，对CPU不可见。所以我们还需要需要构造一个暂存缓冲(Staging Buffer)作为可以直接传输数据的Buffer(其内存类型是CPU可见的内存类型，这种Buffer我们只要用memcpy就可以往里面写数据),然后通过GPU指令，把Staging复制到真正的顶点缓冲中。


## 管线状态
### 简介
在Vulkan当中，管线的所有状态被抽象成了一个对象，不再像OpenGL那样，是一个全局的隐式状态， 每个状态可以随时更改。而Vulkan需要在使用之前指定好所有的固定状态，
然后创建管线状态对象(VkPipelineState)，这个管线状态需要绑定到一个渲染通道的子通道(VkSubpass of VkRenderPass)上使用。

```dot
digraph g {
rankdir=TB
graph [
bgcolor="#665c54"
style="filled"
];
node [
shape="record"
fontsize = "16"
style = "filled"
gradientangle=90
];
edge [];
subgraph cluster0
{
label="PipelineStateObject"
"node0"[ label= "Shader|PrimitiveType|Viewport|Scissor|Uniforms|Attributus|BlendState|DepthStencil|Rasterization|SampleState" ]
}
subgraph cluster1{
  label="DynamicStates"
  "node1"[label="...|LineWidth|..."]
}
}
```
### 组成
1. #### 着色器模块
着色器相对来说是一个独立的模块。这个模块作为渲染管线状态的一部分，是创建管线必须的参数。Vulkan核心功能中的着色器只支持SPIR-V字节码，NVDIA扩展支持GLSL。但是我们在编写着色器时一般都用高级的着色语言比如GLSL,HLSL。有很多工具可以把GLSL,HSLS先编译成字节码然后供Vulkan。[glslang](https://github.com/KhronosGroup/glslang)是Khronos官方维护的将GLSL（同时也支持HLSL）转换成AST的前端，以及将AST转换成SPIR-V的后端。通过这个除了可以实现把高级的着色语言转换成SPIR-V用来给Vulkan使用，还可以用来在不同的高级着色语言之间互转。除此之外，还有很多基于这个工具的二次封装的工具，更加易用。

```dot

digraph g {
rankdir=LR
graph [
bgcolor="#665c54"
style="filled"
];
node [
shape="record"
fontsize = "16"
style="filled"
gradientangle=90
];
edge [];
subgraph cluster0{
label="Shader"
"node0"[label="... | VkShaderModule* |..."]
}

}
```
>[!TIP|style:flat]
>当管线创建完成之后，着色器模块对象就可以被销毁了。

2. #### 渲染通道

![renderpass](./res/renderpass.drawio.svg)


## 资源：缓冲(VkBuffer) 和 图像(VkImage)
### 简介
  需要指名Buffer 的用法。与OpenGL不同的是，这里创建好的buffer没有内存，需要绑定到另外的内存对象上。
  Buffer通常用来存储线性的机构化或非结构化的数据。
  相比于OpenGL，Vulkan中的Buffer设计的更为一般化和清晰，OpenGL由于沉重的历史包袱，各种版本的Buffer的API非常混乱。
  从缓冲对象和图像对象我们可以观察到，Vulkan的概念反而更容易理解。下面的表格是关于缓冲对象的基本功能对应的API在Vulkan 和不同OpenGL版本之间的对比。

  |功能|Vulkan|OpenGL(DSA)|OpenGL(non-DSA)|OpenGL(legacy)|
  |------|------|----|----|----|
  |创建缓冲对象|vkCreateBuffer|glCreateBuffers|glGenBuffers+glBindBuffer|----|
  |分配缓冲存储空间|vkBindBufferMemory|glNamedBufferStorage|glBufferData|---|
  |暂存缓冲(host visible)数据传输|memcpy+vkMapMemory|glMapNamedBufferRange|glMapBufferRange|---|
  |设备缓冲(device local)数据传输|暂存缓冲+vkCmdCopyBuffer(*) | glNamedBufferData|glBufferData|---|
  |创建图像对象|vkCreateImage|glCreateTextures|glGenTextures+glBindTexture|glGenTextures+glBindTexture|
  |分配图像存储空间|vkBindBufferMemory|glTextureStorage{x}D|glTexStorage{x}D|glTexImage{x}D|
  |设备本地图像数据传输1|暂存缓冲+vkCmdCopyImage(*)|glTextureSubImage{x}D|glTexSubImage{x}D|glTexImage{x}D|
  |设备本地图像数据传输2|暂存缓冲+vkCmdCopyImage(*)|glMapNamedBufferRange+glReadPixel(PBO)|glMapBufferRange+glReadPixel|---|

  这个表格基本上展示了Vulkan 的缓冲和图像的基本功能的API。但是图像的使用在Vulkan中更加复杂，因为Vulkan的特点之一，显式同步会在图像这里得到体现。具体来说就是图像的布局之间的转换需要自己编写代码进行转换。

  资源这一部分属于各个Graphics API的核心内容，由于Vulkan的特性，这一部分更加复杂。在这里不做过多的说明。

  - [Modern OpenGL Functions](https://github.com/fendevel/Guide-to-Modern-OpenGL-Functions)



## 资源视图：缓冲视图（VkBufferView）和 图像视图（VkImageView）
### 简介

资源视图是对资源的重新解释，并且赋予了这个资源更加具体的属性。尤其是对于图像来说，图像资源本身的信息并不多，如果要使用图像资源，还应该赋予更加具体的解释，这样才能实现资源的复用。

## 内存(VkMemory)
### 简介
Vulkan的内存是Vulkan的重点特性之一。Vulkan中的资源对象与对应的内存是分离的。资源在使用之前，需要根据用途来绑定到不同的类型的内存对象。

Vulkan的内存属性比较复杂，任何需要设备内存的对象的创建都需要指定内存类型。如Image和Buffer。内存类型有很多，并且每种类型都由某种类型的堆负责创建。
当使用```vkGetPhysicalDeviceMemoryProperty``` 查询相应物理设备支持的内存时，获取到的内存属性包括两个数组。第一个是支持的内存类型，
第二个是支持的内存堆。支持的内存类型是由一系列bitflags决定的，并且支持的内存类型里还包括了一个索引，这个索引就是由相应支持分配的堆所在数组的索引。

总体来说，Vulkan的设备内存属性用**堆类型**和**内存类型**两个维度来描述。从设计上来说，这两个维度是正交的。实际上考虑到实现，这两个维度并不是完全独立的。关于内存这一块，
下面会有详细的介绍，这里只是简要的引出内存属性这个概念。
分配设备内存的时候需要指定**内存类型**和**堆类型**

  - 内存类型
      
    内存类型标志位大概有这几种类型：（其余见官方规范手册）

    1. **DEVICE_LOCAL_BIT**： 设备专用内存，一般是纹理或者顶点缓冲使用的内存

    2. **HOST_VISIBLE_BIT**： 主机可见内存，表明内从可以被主机端映射，可以在主机端像访问CPU内存一样直接存取

    3. **COHERENT_BIT**：对于主机可见内存的访问保持一致性，否则需要手动更新内存。

    4. **HOST_CACHED_BIT**：这种内存会缓存在cpu端，但是主机端的访问可能会慢一些。

    5. **LAZILY_ALLOCATED_BIT**：延迟分配。

    这几种并不是随意组合的，合法的组合请参照Vulkan规范手册

  - 堆类型

    堆类型标志位大概由这几种类型：（其余见官方规范手册）

    1. **DEVICE_LOCAL_BIT**: 设备中的堆，一般位于是运行Vulkan的硬件设备，比如GPU。这种就是大多数情况。

    2. **MULTI_INSTANCE_BIT**: 当一个逻辑设备是由多个物理设备构成时，分配内存的时候会重复分配到每个物理设备中。（用在分布式上？）


Vulkan本身的内存分配次数有限制，鼓励分配大块内存作为内存池，然后在这个基础上进行二次分配。然后把资源绑定在分配的内存区间段上。所以，如果编写一个基于Vulkan的通用RHI，需要自己实现一个高效的内存分配器。

![Memory](./res/memory.drawio.svg)


## 资源绑定
### 简介

  ```glsl
  layout(location = 0) in vec3 pos;
  layout(location = 1) in vec3 color;
  layout(location = 2) in vec2 texCoord;

  layout(binding = 0) uniform MVP{
    mat4 model;
    mat4 view;
    mat4 proj;
  }mvp;

  layout(binding = 1) uniform Sampler2D tex;

  ```
  资源绑定就是如何为在渲染管线中的上述着色器代码中的变量指定数据。
  局部变量对应逐顶点属性(Attribute)，共享变量对应统一变量(Uniform)。至于属性(Attribute) 和统一变量(Uniform)只是glsl中的修饰符。不同API名字虽然不同，
  但对应的都是这两个概念。

  
  |Vulkan(OpenGL)|D3D12|说明|
  |------|-----|----|
  |UBO(Uniform Buffer Object)|Constant Buffer|全局统一变量|
  |Texel Buffer |typed buffer | 纹素缓冲|
  |SSBO|UAV buffer(Unordered Access View)|通用缓冲 |
  |Image|UAV texture | 可写的图像类型|

  在Vulkan 的API中，着色器局部变量也就是Attribute，用属性(Attribute)来描述。着色器共享变量(Uniform)信息用描述符(Descriptor)来描述。区分这两个概念有助于理清繁琐的Vulkan API。


### 功能

  描述符集布局（以下简称描述符布局）是描述都有哪些Uniform变量的对象，创建描述符布局需要一个CreateInfo（VkDescriptorSetLayoutCreateInfo）,其中一个数组字段包含了每个Uniform变量的信息（VkDescriptorSetLayoutBinding）。

  注意，描述符集和描述符布局的区别是，描述符**布局**并没有和具体的数据关联，而**集**却关联了数据。例如，布局只是说这个集有一个uniform buffer 和 一个 uniform sampler。
  而在这个布局的基础上通过池分配出来的集需要关联哪一个uniform buffer和uniform sampler数据。

  + 描述符集布局绑定（单个变量的信息对象）
  + 描述符集布局（单个变量信息对象的集和）
  + 描述符池: 创建这个池需要指定将要从这个池中分配多少个描述符以及多少个描述符集。分配方式为描述符个数以最大描述符集个数的一个划分。
  + 描述符集（根据布局通过池分配出来的带有真正数据的对象）
    描述符集属于管线资源，在绘制指令的时候进行绑定。绑定的这个描述符集要和

  如果直接从API翻译，这几个概念对应的中文很拗口。他们几个之间的关系如下图

![B](./res/resbind.drawio.svg)

  顶点属性的buffer 也是要通过绘制指令绑定到管线。顶点属性一般指定：
  + 对应的顶点缓冲数组的索引（Buffer Index）
  + 指定着色其中的绑定点（Binding）
  + 步长（Stride）
  + 偏移 (Offset)
  + 格式（Format） （在OpenGL中，这一步通过分量格式和分量个数表示）
  

在vulkan中，上面5个属性的指定被分配在了两个结构体中。分别是```VkVertexInputBindingDescription``` 和 ```VkVertexInputAttributeDescription```其中前者指定了绑定索引(binding),这个也就是在后续进行绘制指令提交时指定的缓冲区数组的索引。以及步长(stride),即每一个被解释的元素的大小。后者指定了绑定的着色器中的变量（location）。这两个描述结构体是的参数正交地描述了顶点属性的配置。


  + 下图为Vulkan和OpenGL 中进行逐顶点属性绑定API的对比表格

|Vulkan|OpenGL(DSA version)|OpenGL(non-DSA)|
|------|------|------|
|VkVertexInputBindingDescription|glVertexArrayVertexBuffer|glVertexAttribPointer|
|VkVertexInputAttributeDescription|glVertexAttribFormat|

## GPU指令和指令缓冲区
### 简介
在Vulkan当中，```vkCmd*```类型的GPU指令（比如数据传输，管线绑定，渲染通道操作，以及Drawcall）都是通过提交到指令缓冲(VkCommandBuffer)中异步执行的。
指令缓冲需要从指令池中分配，指令池的创建需要指定队列族的索引。我们在创建逻辑设备的时候指定了这个逻辑设备所需要的队列族。因此创建指令池指定的队列族要是创建指令池的逻辑设备所包含队列族的子集。

下面是一些常用的GPU指令，这些指令的上下文参数就是指令缓冲。调用每个指令的时候需要指定一个指令缓冲。当最终提交命令缓冲时，调用VkQueueSubmit提交相应的指令缓冲，指令开始真正地执行。


|Vulkan|Render Pass Scope| Supported Queue Types|Pipeline Type|Level|
|:----|:-----|:-----|:-----|:-----|
|vkCmdCopyImage|Outside|Transfer Graphics Compute|Transfer|Primary Secondary|
|vkCmdCopyBuffer|Outside|Transfer Graphics Compute|Transfer|Primary Secondary|
|vkCmdCopyBufferToImage|Outside|Transfer Graphics Compute|Transfer|Primary Secondary|
|vkCmdVertexBuffer|Both|Graphics||Primary Secondary|
|vkCmdDraw|Inside|Graphics|Graphics|Primary Secondary|
|vkCmdDrawIndexed|Inside|Graphics|Graphics|Primary Secondary|
|vkCmdDrawIndirected|Inside|Graphics|Graphics|Primary Secondary|
|vkCmdBindPipeline|Both|Graphics Compute||Primary Secondary|
|vkCmdBeingRenderPass|Outside|Graphics|Graphics|Primary|
|vkCmdEndRenderPass|Inside|Graphics|Graphics|Primary|
|vkCmdNextSubpass|Inside|Grahpics|Graphics|Primary|
|vkCmdPipelineBarrier|Both|Transfer Graphics Compute||Primary Secondary|
|**vkQueueSubmit**|||||
|**vkQueuePresentKHR**|

执行指令的具体流程如下:

在开始执行指令之前需要做以下操作
- 创建指令池
- 从指令池中分配相应的指令缓冲，并指定指令缓冲的级别
  
  指令缓冲级别表明这个指令缓冲对应的指令是否可以作为另外一个指令缓冲的子指令序列。也就是我们可以把一个标记为Secondary的指令缓冲作为标记为Primary指令缓冲的子指令序列(通过```vkBeginCommandBuffer```来指定)。这样的好处是，我们可以把一些列常用的指令预定义为一个Secondary指令缓冲，然后作为一个指令序列复用。

执行指令时：
所有```vkCmd*```类型的往指令缓冲中提交指令的API需要在```vkBeginCommandBuffer``` 和 ```vkEndCommandBuffer```之间调用，用指令缓冲对象作为上下文参数。
```vkBeginCommandBuffer```为指令缓冲指定了一个很重要的参数，即指定这个指令缓冲的具体用法。其次，这个指令起到了重置指令缓冲的作用。
- VK_COMMAND_BUFFER_USAGE_ONE_TIME_SUBMIT_BIT: 提交后就被用来记录新的指令。适合存储一次性的指令序列，比如数据传输。
- VK_COMMAND_BUFFER_USAGE_RENDER_PASS_CONTINUE_BIT:在渲染通道内使用的辅助指令缓冲。
- VK_COMMAND_BUFFER_USAGE_SIMULTANEOUS_USE_BIT：在等待执行的时候，仍然可以提交指令。

![Command](./res/cmd.drawio.svg)



## 资源同步
### 简介
GPU的工作模式相对于CPU端来说是异步的，也就是当我们提交了指令缓冲的时候，API立即返回，并且执行。同时，我们CPU端也做了很多往GPU进行数据传输或者读写GPU中数据的工作。由于是异步的工作流，如果没有任何同步措施，我们无法保证GPU是否已经读取到我们传输的最新数据，或者CPU读取到GPU已经处理好的数据。因此，为了正确完成渲染工作，我们还需要让CPU和GPU之间在资源访问这一部分进行先后的配合，这就是同步。
  

除此之外，Vulkan为我们提供了一个非常重要的同步方法，那就是指令缓冲。指令缓冲中的指令是保证按提交顺序执行的，这也是指令缓冲的意义。但是提交的多个指令缓冲之间，以及每个指令缓冲中的GPU指令和CPU端的操作之间的
逻辑上的依赖关系，是Vulkan不保证的。因为GPU要通过异步的方式实现高并发来达到最高的效率。


上一章介绍了Vulkan中的GPU指令的执行过程，由于Vulkan是一个完全显式的API，它的资源同步自然也需要应用程序来控制。Vulkan提供了三种不同粒度的同步方法。

- 栅栏(VkFence)
  栅栏是同步GPU和CPU之间的一种方式。一般在```vkQueueSubmit```当中指定栅栏对象，CPU端负责查询栅栏状态，判断所提交的执行序列是否执行完成。是一种指令缓冲级别的同步对象。粒度较大。见而言之，栅栏就是CPU端去查询这个对象状态，然后判断伴随这个对象提交的一系列指令是否执行完成，并且栅栏可以以等待的方式判断栅栏的状态（挂起，线程切换），因此需要操作系统的支持。等待单一的栅栏和直接使用```vkQueueWaitIdle```本质上是一样的。

  |Vulkan|Render Pass Scope| Supported Queue Types|Pipeline Type|Level|
  |:----|:-----|:-----|:-----|:-----|
  |vkQueueSubmit|||||
  |vkQueuePresentKHR|||||
- 事件(VkEvent)

  事件是细粒度的同步方法。它可以同步一个指令缓冲内部之间的指令。因此，它除了在主机端可以查询事件状态之外，还可以在GPU端查询时间状态。同时，设备端是不能以等待的方式查询事件的状态的，只能自旋查询。
  相反，事件在设备端以挂起的方式等待事件的完成。

  |Vulkan|Render Pass Scope| Supported Queue Types|Pipeline Type|Level|
  |:----|:-----|:-----|:-----|:-----|
  |vkCmdSetEvent|Outside|Graphics Compute||Primary Secondary|
  |vkCmdResetEvent|Outside|Graphics Compute||Primary Secondary|

  
- 信号量(VkSemophore)
  信号量不能显式的设置或等待。它同步不同缓冲区之间的资源。使用```vkQueueSubmit```时，指定了需要等待信号和通知信号。所提交的指令还中只有在指定的等待信号被通知时执行，执行完成后通知指定的通知信号。
  典型的应用是向不同的队列中提交指令缓冲，并且指令缓冲之间有逻辑上的先后顺序。比如绘制一些东西的时候需要依赖计算管线的结果，那么就向创建自计算队列的指令缓冲里提计算作指令，向创建自图形队列的指令缓冲里提交绘制指令。然后设置好决定先后依赖关系的信号量。然后一次性用```vkQueueSubmit```提交。

  |Vulkan|Render Pass Scope| Supported Queue Types|Pipeline Type|Level|
  |:----|:-----|:-----|:-----|:-----|
  |vkQueueSubmit|||||
  |vkQueuePresentKHR|||||


下表列出了三种同步原语的对比。

||栅栏(VkFence)|事件(VkEvent)|信号量(VkSemophore)|
|:-----|:-----|:-----|:-----|
|粒度|指令缓冲和主机端|指令缓冲之内|指令缓冲之间|
|是否涉及主机端同步|是|是|否|


图示：

![Sync](./res/sync.drawio.svg)