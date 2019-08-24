

# VisualMan 



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


#### 关于 Vulkan

在Vulkan中，函数的Create 和Enumerate的函数是不一样的。Create 创建的示例需要销毁。枚举的对象不需要。
比如，vulkan实例需要销毁，而物理设备是枚举出来的。逻辑设备也是创建出来，需要销毁。


因为vulkan有一个内存分配器，所以在设计api的时候也要考虑这个。对于OpenGL来说，这一项自动忽略。

vulkan 的图形管线是静态的，对于一个渲染流程，管线是需要预先建立好的。（用哪些shader，以及管线状态）
并且对于一个管线来说，是没有默认状态的。视口大小，混合方法，混合函数以及各种测试都要显式提供。
