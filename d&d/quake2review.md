# Quake 2 Review
---

本项目改编自[Yamagi Quake2](https://github.com/yquake2/yquake2)，基于GPL License

关于 Quake2 的代码资料来源于[Quake2 Source Code Review](https://fabiensanglard.net/quake2/index.php)


## History of Quake
Quake I:

- Original Quake: For Dos

- VQuake:

- GLQuake:

- WinQuake:


## Yamagi Quake2和原版的区别

由于官方原版代码工程基于VC6.0，目前来看有点跟不上时代，况且能完全无阻碍运行VC6.0的环境也不太好折腾了。
并且跨平台方面可能做的有些问题，并且有部分的汇编代码，对阅读代码不利。
Yamagi Quake2 用CMake组织工程，并且进行了跨平台化，一键无阻碍编译，并且去掉了汇编代码。增加了
OpenGL3.x和Vulkan的支持，是原生的64bit程序，可以无阻碍地运行在现代硬件之上。

- 修改了部分数据结构和 API的名字，比如原版用Refresh来表示渲染器，可能当初Renderer的概念还没成型。模块的名字虽然没有改，但是内部API的名字已经改了
比如原版VID_LoadRefresh改成了VID_LoadRenderer。相比于原版，renderer的导出函数增加了。

## 软渲染流程

为了高效，软渲染其做了两个妥协：
1. 只支持白色光源，不支持彩色光源，这就为查表实现纹理着色提供了可能。
2. 没有线性滤波的纹理

降低纹理的颜色位宽，是为了降低CPU读写内存的开销。软渲染器的通病是从内存写入到显存的过程开销大，而且频繁发生内存读写，考虑到那时候的
CPU性能，所以颜色位宽不能太大。

纹理是256色的贴图，通过光照贴图，给每个颜色提供一个6bit的梯度(64级)。就可以很容易并且高效地用查表的方式进行着色。

这个软渲染器是一个为BSP场景 + PVS剔除 和基于调色板的纹理和光照贴图的高度特化实现。

### BSP:

### PSV:

### Lightmap:
读入磁盘之后被归一化到[0,64)之间的灰度值。(选择R,G,B中最大的)

### 流程：
  1. 遍历BSP，找到当前viewpoint所在节点。
  2. 查询PSV
  3. 再次遍历BSP，计算光源对每个节点是否有影响，并标记上影响的光源id。
  4. 这时，有了场景多边形的可见性以及光照等动态信息，下面就按照常规来渲染场景。
  5. 遍历BSP，按照从前到后的顺序渲染场景，扫描线光栅化，使用zbuffer。
  6. 渲染实体，根据zbuffer决定可见性。
  7. 绘制透明纹理，
  8. 粒子渲染
  9. 后处理，受伤后视野变红。


  Lightmap和纹理通过查表的方式融合。而不是直接blend

```c
  void RE_RenderFrame(refdef_t *fd)
  {

    R_SetupFrame()
    {
      R_ModPointInLeaf();   // Just a 3D-Plane Equation
    }

    R_MarkLeaves(){
      ...

      if (vis[cluster>>3] & (1<<(cluster&7)))
      {
        node = (mnode_t *)leaf;
        do
        {
          if (node->visframe == r_visframecount)
            break;
          node->visframe = r_visframecount;
          node = node->parent;
        } while (node);
      }
      ...
    }

    R_PushDlights()
    {
    }

    R_EdgeDrawing()
    {
      R_RenderWorld()
      {
        // clip if need

        R_RecursiveWorldNode (front or back)

        // Draw (Generate surface stack)

        R_RecursiveWorldNode (back or front)

      }
      R_DrawBEnittiesOnLists();
      R_ScanEdge();
    }

    R_DrawEntitiesOnList(); // Character animation, etc
    R_DrawParticles();

    R_CalcPalette(); // Post-effect by changed the palette
  }
```
