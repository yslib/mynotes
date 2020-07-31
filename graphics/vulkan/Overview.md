# Vulkan Overview
---

```mermaid
graph BT
linkStyle default interpolate basis
subgraph Vulkan Context
A(VkDevice):::vkobj --> B(VkPhysicalDevice):::vkobj
B --> C(VkInstance):::vkobj
D(VkSurfaceKHR):::khr-->B
D-->C
D-->W{{NativeWindowHandle}}
S(VkSwapchainKHR):::khr-->D
S-->A
end

subgraph vulkan resources 
i(vkimage):::vkobj-.->A
b(vkbuffer):::vkobj-.->A
end

classDef khr fill:#369f;
classDef vkobj fill:#93bfe1
style W fill:#fff0,stroke-dasharray:4
```

## 实例(vkinstance)
---

### 简介

vulkan实例隔离了不同的vulkan环境，在一个应用程序中，可以创建多个实例。但是实例之间的对象不能共享，如内存。（在不涉及扩展的情况下）

```mermaid
graph LR
linkStyle default interpolate basis
ci[CreateInfo]
i(VkInstance):::vkobj
o([应用]):::none--查询当前Vulkan支持的扩展-->b([指定扩展]):::field
subgraph 功能
b -->ci:::cistyle
c([指定验证层]):::field-->ci
d([指定Vulkan对象的全局内存分配器]):::field-->ci
z([...]):::field-->ci
end

ci--VkCreateInstance-->i

classDef vkobj fill:#93BFE1
classDef field fill:#FEBF63
classDef cistyle fill:#ADE498
classDef none fill:#7FDBDA

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
---

### 简介

在初始化Vulkan的物理设备时，除了选择需要的物理设别外，还应该获取关于物理设备的一些属性供之后使用

```mermaid
graph LR
linkStyle default interpolate basis
i(vkInstance):::vkobj--VkEnumeratePhysicalDevices-->p(vkPhysicalDevice):::vkobj
p-->a([vkGetPhysicalDeviceFormatProperties]):::get
p-->c([vkGetPhysicalDeviceMemoryProperties]):::get
p-->d([vkGetPhysicalDeviceQueueFamilyProperties]):::get
p-->e([vkGetPhysicalDeviceFeatures]):::get

classDef vkobj fill:#93BFE1
classDef get fill::#282828
classDef none fill:#7FDBDA
```

### 功能

1. 图像格式属性(vkGetPhysicalDeviceFormatProperties)：

  通常，为了完整性，在创建一个需要指定格式的Vulkan对象时，比如创建一个R8B8GBA8格式的纹理，或者格式为24位的深度缓冲，需要查询物理设备是不是支持这种格式。但是一个格式还有附加的属性，比如这个格式是不是支持

2. 设备内存：

  Vulkan的内存类型比较复杂，任何需要设备内存的对象的创建都需要指定内存类型。如Image和Buffer。内存类型有很多，并且每种类型都由某种类型的堆负责创建。
  当使用vkGetPhysicalDeviceMemoryProperty 查询相应物理设备支持的内存时，获取到的内存属性包括两个数组。第一个是支持的内存类型，
  第二个是支持的内存堆。支持的内存类型是由一系列bitflags决定的，并且支持的内存类型里还包括了一个索引，这个索引就是由相应支持分配
  的堆所在数组的索引。
  分配设备内存的时候需要指定内存类型和堆类型

    - 内存类型
       
      内存类型标志位大概有这几种类型：（其余见官方规范手册）

      1. DEVICE_LOCAL_BIT： 设备专用内存，一般是纹理或者顶点缓冲使用的内存

      2. HOST_VISIBLE_BIT： 主机可见内存，表明内从可以被主机端映射，可以在主机端像访问CPU内存一样直接存取

      3. COHERENT_BIT：对于主机可见内存的访问保持一致性，否则需要手动更新内存。

      4. HOST_CACHED_BIT：这种内存会缓存在cpu端，但是主机端的访问可能会慢一些。

      5. LAZILY_ALLOCATED_BIT：延迟分配。

      这几种并不是随意组合的，合法的组合请参照Vulkan规范手册

    - 堆类型

      堆类型标志位大概由这几种类型：（其余见官方规范手册）

      1. DEVICE_LOCAL_BIT: 设备中的堆，一般位于是运行Vulkan的硬件设备，比如GPU。这种就是大多数情况。

      2. MULTI_INSTANCE_BIT: 当一个逻辑设备是由多个物理设备构成时，分配内存的时候会重复分配到每个物理设备中。（用在分布式上？）

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
---

### 简介

  逻辑设备是Vulkan对物理设备的抽象。可以从一个物理设备上创建多个逻辑设备。逻辑设备对象负责vulkan的设备资源分配。
  负责的主要功能有：

### 功能

  1. 开启扩展：

     如果需要窗口显示功能，这里需要开启交换链扩展

  2. 指定设备要用到的队列：

     如果需要绘制流水线功能就指定图形队列，如果要使用计算着色器的功能，就指定计算队列。

## 表面（vkSurfaceKHR, 扩展）
---

### 简介

  表面是展示渲染结果的那个对象，可以不严谨地理解为呈现绘制结果的那个窗口区域。表面不是Vulkan核心的一部分。因为显示窗口不具备跨平台性，所以vulkan没有义务去做这个限制。
  （其实OpenGL也没有）。首先查询物理设备的队列族是不是支持表面。输入参数为物理设备和所使用的队列族索引。
  创建表面需要从实例创建，创建表面除了需要实例，还需要一个显示表面的窗口句柄，这个窗口句柄取决于平台。

### 功能



## 平面(Plane,扩展)
---

### 简介

## 交换链(vkSwapchainKHR, 扩展)
---

### 简介

交换链提供和管理表面中的数据，一般情况下是一个环形的缓冲，一些表面用来显示在窗口上，一些表面用来接受绘制的结果供接下来的展示用。
不严谨地说就是帧缓冲的管理器。为什么不严谨，因为表面的数据不是Vulkan所管理的，而是操作系统。
创建交换链需要指定之前创建的表面以及逻辑设备。因此交换链是被逻辑设备所拥有的。
创建交换链需要一下信息：

## 缓冲(VkBuffer)
---

### 简介

  与OpenGL中glCreateBuffer创建出来的对象等价。需要指名Buffer 的用法。与opengl不同的是，这里创建好的buffer没有内存，需要绑定到另外的内存对象上。

## 图像(VkImage)
---

### 简介
  在Vulkan 里创建图像的一般方式也是先把图像数据放入暂存缓冲，然后从暂存缓冲复制到GPU中。然而与OpenGL不同的是这里的图像需要指定
  图像布局(ImageLayout)。图像布局描述了当前图像所扮演的角色。图像的布局大概分为一下几种：

  1. PRESENT (VK_IMAGE_LAYOUT_PRESENT_SRC_KHR): 用来作为呈现图像，这个布局可以绘制到屏幕上。
