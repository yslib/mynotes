

# Vulkan 基本概念


##### 设计参考
- [Designing a Modern Cross-Platform Low-Level Graphics Library](https://www.gamasutra.com/blogs/EgorYusov/20171130/310274/Designing_a_Modern_CrossPlatform_LowLevel_Graphics_Library.php "Designing a Modern Cross-Platform Low-Level Graphics Library")
- [Diligent Graphics](https://github.com/DiligentGraphics/DiligentEngine)

#### Features
- 单机多卡
- 多机多卡
- 支持OpenGL Vulkan等后端。

#### 关于抽象Graphics API 如何设计

初步打算采用面向接口的API封装。
对于一个应用来说，创建一个全局实例。在这个实例上创建物理设备，在物理设备上创建逻辑设备。这个和Vulkan的机制很像。
所有关于渲染的对象的创建，都要通过逻辑设备作为上下文参数来创建。
比如对于纹理和缓冲:

```

ITextureCreateParams params;
//...
// Setup params
//...
Ref<ITexture> tex = m_device->CreateTexture(params);
// Use tex

```

对于OpenGL来说，Context就相当于一个逻辑设备。

首先需要对vulkan做一个简单的封装，封装出物理设备 和 逻辑设备作为类，然后添加相应的成员函数，对应vulkan相应的关于物理设备和逻辑设备的那些C-API（其实官方是有Vulkan C++ API 的封装的。但是这里为了学习vulkan, 还是从最基本的api写起。

物理设备对应的api较多，但是有些简单的不用封装，用的时候直接拿出native handle然后直接调用api就行了。是否封装对应的api 取决于这个api是否涉及到了vulkan native handle，并且这个vulkan native handle 是否被封装。原则就是简单封装了的vulkan 对象不能在使用过程中涉及到native vulkan object。否则这个封装意义不大。

首先查询内存类型以及
图像的最大尺寸
vkGetPhysicalDeviceFeatures() 



#### 关于 Vulkan

在Vulkan中，函数的Create 和Enumerate的函数是不一样的。Create 创建的示例需要销毁。枚举的对象不需要。
比如，vulkan实例需要销毁，而物理设备是枚举出来的。逻辑设备也是创建出来，需要销毁。


因为vulkan有一个内存分配器，所以在设计api的时候也要考虑这个。对于OpenGL来说，这一项自动忽略。

vulkan 的图形管线是静态的，对于一个渲染流程，管线是需要预先建立好的。（用哪些shader，以及管线状态）
并且对于一个管线来说，是没有默认状态的。视口大小，混合方法，混合函数以及各种测试都要显式提供。

物理设备对应对列簇，逻辑设备对应队列
物理设备不需要创建，是从实例当中枚举出来的。所以物理设备不许销毁。
队列是从设备中创建出来的。

# 内存
查询是否支持跟硬件相关的特性，都要通过物理设备来查询。

物理设备相关的特性：格式，队列簇，表面

# 分配器
# 缓冲区
vkCreateBuffer()
可以创建不同类型的buffer，有Index Buffer, Vertex Buffer, Uniform Buffer,Storage Buffer, Uniform Texel Buffer, Storage Texel Buffer。
纹理冲区是格式化的。

注意sharingMode

# 格式和支持
vkGetPhysicalDeviceFormatProperties()
关于图像，需要更加详细的格式支持查询。
vkGetPhysicalDeviceImageFormatProperties()
 
# 图像
vkCreateImage()
CreateFlags: 是否稀疏图像，是否可以创建多个视图等，立方体纹理。
UsageFlags: 可采样，可写入，颜色附件，深度和模板附件，临时附件，输入附件
layout:这个概念比较复杂。后面详细讲解。创建的时候只能是UNDEFINED 或者 PREINITIALIZED

# 视图

BufferView: 
vkCreateBufferView()
vkDestroyBufferView()
BufferViewCreateFlags:
所对应的Buffer 必须是 Storage Texel Buffer或 Uniform Texel Buffer

ImageView:
vkCreateImageView()
vkDestroyImageView()

# 内存
vkAllocateMemory()
需要指定大小和内存类型，用内存类型索引指定。
vkFreeMemory()

# 内存绑定到资源上

绑定到资源之前，首先需要查询要求。
vkGetBufferMemoryRequirements()
vkGetImageMemoryRequirements()
它们返回这个资源所需要的内存以及
对齐，还有内存类型。
下面是绑定内存
vkBindBufferMemory()
vkBindImageMemory()
绑定不能改变

# 着色器资源

vulkan中着色器的资源绑定有些复杂，主要是概念有些多。
其中涉及到 **描述符集(VkDescriptorSet)**，**描述符集布局(VkDescriptorSetLayout)**，**管线布局(VkPipelineLayout)**，**描述符池(VkDescirptorPool)**以及**描述符集布局绑定(VkDescritorSetLayoutBinding)**这5个概念。

VkDescriptorSetLayout 的 createInfo里指定VkDescirptorSetLayoutBinding。这个VkDescriptorSetLayoutBinding 对应着色器里的变量，比如uniform、Buffer或者 sampler之类的。VkPipelineLayout的createInfo里指定VkDescriptorSetLayout。VkPipelineLayout用在创建管线的时候。
创建完VkPipeline的时候VkPipeline就可以释放了。依次的依赖关系也都可以这样。
上面这是创建布局，即应用程序告诉着色器有哪些需要绑定的资源。下面简单介绍如何把资源绑定到布局上。
绑定资源需要用到**描述符池(VkDescriptorPool)**。 创建描述符池vkCreateDescriptorPool需要指定每种描述符集(VkDescirptorSet)的数量。
描述符集需要通过vkAllocateDsecriptorSets指定描述符池来分配，分都是 allocInfo需要指定对应的描述符集布局(VkDescriptorSetLayout)。

以上介绍的都是着色器使用资源所涉及到的概念之间的关系。

接下来介绍把具体的资源绑定到描述符集上的过程。
需要通过vkUpdateDescriptorSets命令。


# 执行vulkan指令需要命令缓冲区

首先说明两个创建缓冲池的参数
VK_COMMAND_POOL_CREATE_TRANSIENT_BIT  以及
VK_COMMAND_POOL_CREATE_RESET_COMMOND_BUFFER_BIT

前一个参数表明分配出来的缓冲区使用时间很短，使用完就退回缓冲池。这样的指令有数据传输指令。
第二个参数表明缓冲区可以通过重置指令而重用，否则必须重置分配这个缓冲区的缓冲池才能重用。适合这种缓冲区的指令有频繁使用的绘制指令。

创建缓冲池需要指令队列族索引。

我们之所以使用Vulkan，就是因为它的多线程的特性。因此，缓冲池这一块是重点。所以我们需要精心的管理缓冲池和缓冲区对象。

# 内存屏障
在vulkan中，所有屏障都必须显式给出。一个管线屏障由内存屏障，缓冲区屏障以及图像屏障等。

void vkCmdPipelineBarrier()

# 缓冲区的填充和清除

vkCmdFillBuffer
vkCmdUpdateBuffer() 这个函数只能更新buffer 很小的一部分。并且是一个“同步”执行函数。这个函数会把数据指针的内容复制到缓冲区中。所以即使加入到缓冲区之后没执行也可以不再确保数据指针的有效性。

# 图像的填充和清除

vkCmdClearColorImage()
vkCmdClearDepthStencilImage()

向图像中复制数据比较复杂。
vkCmdCopyBufferToImage() <===> vkCmdCopyImageToBuffer()
vkCmdCopyImage()
vkCmdBlitImage()

明天计划：
写完与CommandBuffer和 Pool相关的部分

# 栅栏

栅栏是一种同步元语，可以复用。 主要用在host 和 device之间的同步
所以首先需要封装一个简单的栅栏池，如果需要栅栏，我们就从栅栏池中拿出来一个，用完之后放回去。这样可以减小频繁创建栅栏的开销。放回去的时候检查一下栅栏的状态是不是用过了，所以栅栏池里需要包含一个vulkan设备，然后一个双端队列存放栅栏实例。简单来说，栅栏池两个接口。一个是GetFence，如果栅栏池里有可用的栅栏，就拿一个。否则创建一个。另外一个接口是PutFence，用完之后放回去，检查一下栅栏的状态。放到队列尾部。

所封装的图形接口的Fence 是对应一个命令队列的。而不是一次提交。也就是说封装的这个栅栏对象应该是个栅栏池的概念。比如向队列中提交100次命令缓冲。我们到底需要多少个栅栏？答案是从1-100个不等。所以我们需要一个更高层次的“栅栏”对象，即一个“栅栏”是一个栅栏池。

# 命令队列
因为命令队列不是接口的一部分，不应该暴露给引擎使用者，只是一个内部概念。它的主要作用执行命令缓冲。同时应该暴露一个栅栏对象。当命令队列执行完毕之后给这个栅栏对象发信号。


# 命令列表
如果要设计多线程的图形接口，就必须暴露一个指令列表，其扮演着执行绘制指令的角色。但是这个命令列表没有任何实现，只能作为一个句柄。

