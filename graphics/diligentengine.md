# DiligentEngine

这个跨平台图形API是我在google上搜索如何设计RHI时候偶然发现的，个人认为其代码结构和设计比[BGFX](https://github.com/bkaradzic/bgfx)这个又名的开源底层渲染接口要好。
其实现了不同的平台下的API封装，整体接口风格是按照DX11来的，因此，能够非常好的与Vulkan和DX12这种现代API适配。当然这个套API比较新，应用没有BGFX多。

## 接口组织方式

整个框架以面向接口（抽象基类，纯虚函数）的方式来组织，类似于COM技术。对于不同层次的API（不同渲染后端），类型提升为不同类型的接口实现，
拿其中某个对象来举例，对于通用的**Buffer**类型，以接口**IBuffer**使用。
对OpenGL后端实现一个**IBufferGL**接口类，对Vulkan实现一个**IBufferVk**接口类，来实现对不同API功能进行接口特化。

```cpp

class IObject{
    virtual void QueryInterface(const IID & iid, void ** ppBuffer)=0;
    virtual void AddRef()=0
    virtual void Release()=0;
};

class IBuffer:IObject{

    //
    virtual void SetData(const void * data,size_t size) = 0;
};


class IBufferGL:IBuffer{
    virtual uint32_t GetNativeGLBufferHandle()=0;
};

class IBufferVk:IBuffer{
    virtual VkBuffer GetNativeVkBufferHandle()=0;
};

```

然后使用CRTP的方式对不同后端的接口进行不同的实现

```cpp

template<typenmae Interface>
class BufferBase:public Interface{
    // Maybe there are multi inheriante levels between BufferBase and Interface
};

class BufferVkImpl:public BufferBase<IBufferVk>{
    VkBuffer GetNativeVkBufferHandle()override{return m_vkBuffer;}
};

class BufferGLImpl:public BufferBase<IBufferGL>{
    VkBuffer GetNativeGLBufferHandle()override{return m_handle;}
};

```

### 导出 C-style 接口

## 框架组成

以D3D11的方式封装的，可以照顾到Vulkan和D3D12这种现代API。

这个框架使用方式如下:

```cpp

void Init(){

auto buffer = pDevice->CreateBuffer(vertexData);
auto image = pDevice->CreateTexture(texData);

// configure pipeline
// ...
auto pipelineState = pDevice->CreatePipelineState(pipelineDesc);


pContext->DrawIndexed(0);

}

void Update(){

}
```

## Internels

如何封装这种API? 首先需要明确两个问题，1）我们希望打算如何使用这个API 2)渲染一帧的主流程是什么样子的

先回答第二个问题，第二个问题非常简单：

```cpp
while(true){
    if(Initialized == false){
        Init();
        Initialized = true;
    }
    Update(){
        // Update per frame data
    }

    Render(){
        // Drawcalls
    }

    Submit(){
        // Submit drawcalls
    }
}
```

所以我们的目的就是在```Init()```中初始化数据，在 ```Update()```里更新每帧的数据，在```Render()```中提交绘制指令。

对于DiligentEngine，一般的渲染流程为

```cpp
Init(){
    vertexBuffer = device->CreateBuffer(vertexData);
    indexBuffer = device->CreateBuffer(indexData);
    uniformBuffer = deice->CreateBuffer(uniformData);
    pipelineState = device->CreatePipeline(pipelineDesc);
    shaderResourcesBinding = pipeline->CreateShaderResourcesBinding(uniformDesc);
}

Update(){
    uniformBuffer->SetData(uniformData);
}

Render(){
    context->SetIndexBuffer(indexBuffer);
    context->SetVertexBufffer(vertexBuffer);
    context->SetPipelineState(pipelineState);
}

```

![mapping1](./res/vkmapping1.drawio.svg)

### OpenGL Backend

由于OpenGL的Contexnt式全局隐式的，所以Device和 Context的实现就比较简单了，基本上Device的实现只是记录信息字段或者glGetIntegerv glGetString这种查询api，Context对应的函数如果有直接的OpenGL实现就直接调用。在OpenGL中没有显式的PipelineState这个对象，所以在这里的实现就只是记录下来接口传过来的管线状态。

### Vulkan Backend

对于Vulkan来说，重点关注**手动同步**，**布局转换**以及**内存管理**。因为这些东西正是一个方便使用的接口所掩盖的。

#### RenderPass


### D3D11 Backend


### D3D12 Backend