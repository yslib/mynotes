# 6.s081 XV6

在做6.s081课程实验时不明白的问题先记录下来：


1. 整体的中断流程是什么样子的，用户态的中断和内核态的中断的细节是什么。

2. 对于用户态来说，合法的地址范围是什么，栈的起始位置、0地址，以及进程大小之间的关系是什么？

3. 在xv6中， proc结构体下面的trapframe是做什么的？与context有什么区别？

4. 调度器，多和调度

5. Buffer cache 扮演这一个什么样的角色



## 文件系统

### 基础概念

- sector: 

这个通常指的是磁盘的最小读取单位

- block:

操作系统（文件系统）最小的操作单位，一般一个block可以包含多个sector

其实这两个名字有时候也混用，通常是都用做**block**。但无论如何，其核心在于，对于磁盘来讲，是有一个最小的访问单元的。
即区别于对内存的地址访问，对于磁盘访问，对block按顺序编址，一个**block**通常512kb。

所以，从程序的角度来说，磁盘可以看作一个粒度比内存大的一个数组。而文件系统就是构建在这个数组上的一个数据结构。

### Buffer Cache

文件系统建立在磁盘和内存之间的一块缓存上。任何对文件的读写都要经过这层缓存，缓存以block为单位，双向链表做LRU。
也就是读写磁盘的api是建立在buffer cache之上的。

以下是bio.c文件中的主要api

|API|Parameter|Description|
|:----:|:----:|:----:|
|bget|(uint dev,uint blockno)|合适的策略寻找一个缓存，引用计数+1|
|bread|(uint dev,uint blockno)|同步磁盘数据到内存，并返回缓存对应的地址|
|bwrite|struct buf *|同步内存数据到磁盘|
|brelse|struct buf *|引用计数-1|

上面这些API提供了完备的磁盘读写逻辑，这就是缓存层。

请求一个磁盘block时，先在buffer cache中查找，

key 为(dev,blockno), bget,

如果没找到，就用LRU替换一块。
与
### inode

inode 相当于文件的metadata数据结构。其中的一个问题是，inode的个数是固定的，所以对于文件系统来说，文件个数是有上限的。

这里我们以open系统调用的执行过程来举例，其实open就是打开或创建inode的过程，其中是建立在获取buffer cache的api的基础之上的。
比如```open``` --> ```create``` --> ```ialloc``` --> ```bget``` 

inode也有一层缓存结构，在```fs.c```中，对inode的操作与对block的操作的api几乎是对应的，多个inode存储在一个block中。

|API|Parameter|Description|
|:----:|:----:|:----:|
|iget|(uint dev,uint blockno)|合适的策略寻找一个缓存，引用计数+1|
|bread|(uint dev,uint blockno)|同步磁盘数据到内存，并返回缓存对应的地址|
|bwrite|struct buf *|同步内存数据到磁盘|
|brelse|struct buf *|引用计数-1|

这里需要说明的是，inode缓存和block buffer缓存是独立的。**缓存(Cache)**的特性是透明的，一般使用带有缓存的机制时，我们只要考虑read/write逻辑，然后在这两个下面设计对于逻辑来讲是不透明的缓存。

因此，上文所表达的意思是，对于block，我们在```bread/bwrite```时的缓存和```iget\iput```的缓存是不相关的。虽然iget/iput的真实读取过程是通过bread/bwrite从它们存在于内存的缓存中复制到了iget\iput存在于内存中的缓冲中。看似好像从一块内存复制到另外一块内存似乎是多余的，并且iget/iput是建立在bread/bwrite之上的。但因为iget/iput 和bread/bwrite是两个无关(独立)的操作，所以缓存也必须无关（独立）。

#### 文件目录在inode当中的组织方式 (Directory layer)

我们知道，inode是一个文件的metadata, inode和文件之间是平坦的对应关系，但是树形的目录结构和以inode为核心的文件系统是如何映射的呢？



### logging 

文件系统中的日志主要是用来进行文件的完整性保证和恢复的，相当于在磁盘上的缓存（备份）。
我们不期望时时刻刻都能把我们的数据保存完好，比起不丢失任何数据，数据的完整性更重要。而文件系统中的日志就是以维护文件的完整性为目的的。

比如完整的一个记录操作需要两步办到，在第一步完成之后发生了崩溃，当重启之后会发现数据有问题（这个问题当然不只是数据内容上的不完整，而是这个操作没完成导致影响整个文件系统）。
