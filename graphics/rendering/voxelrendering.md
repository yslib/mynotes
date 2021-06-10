# 体素渲染

最近自己在自娱自乐的写一个体素[存储引擎](https://github.com/yslib/VoxelMan.git)， 在这里记录一下需要用到的相关资料

### 关于体素存储的文章：
1. High Resolution Sparse Voxel DAGs
2. OpenVDB
3. GigaVoxel
4. Optimizing Memory Access on GPUs using Morton Order Indexing
5. **虚拟纹理**


### 和体素有关的项目或工具:
1. MagicaVoxel
2. Goxel
3. PolyVox
4. Cubiquity 2
5. [](http://www.volumesoffun.com/)


### 关于体素渲染
1. raycasting
[](https://medium.com/@calebleak/cube-voxel-rendering-bc5d87c24c3)
2. Cube Rendering: [](https://medium.com/@calebleak/quads-all-the-way-down-simple-voxel-rendering-fea1e4488e26)

### 虚拟纹理

- VT in software
    - Simple in theroy, but hard to implement efficiently.

软件虚拟纹理实现的难点在于，现在的管线似乎不是为频繁更新纹理设计的。因为VT不可避免的频繁（相对）更新纹理，所以这是一个drawback。
虚拟纹理的核心是feekback的过程。这个过程决定了渲染的效率。当然，这个也得是全程软件实现，因此效率千差万别。
当然优化的办法除了动态的确定working set，还可以离线烘焙feekback data。如同PVS一样，不过这个还和具体应用有关。对于科学可视化来的体绘制来说说，这个几乎不能用，因为不能假定可见性。

现在使用的是ray-guided 方式，完全由ray决定。排除实现本身的效率之外，并不清楚这种做法的效率有多高。

就拿PageTable(Address translation)的实现方式来说,hashtable 很直观，但是想要实现一个对GPU管线友好的数据结构还是很困难。
用Texture? 每次都要更新部分纹理。但是其地址转换过程是最快的（直接采样）。但是很浪费空间，基本上虚拟纹理有多大，这个pagetable就有多大（个数上成正比）。

每种方式的实现都千差万别。

- VT in hardware
    不用自己管理映射结构（页表），但是需要自己做缺页处理。一般是在shader里用一个纹理采样，如果采到了就返回纹理数据，采不到就返回一个0之类的。feedback的过程自己处理。
