
# Distance Function 距离场()函数 (By iquilezles: https://iquilezles.org/)
---
在过程式着色中，使用距离场（符号）函数可以非常方便的建立一些简单的几何模型。

1. ### 圆形
```
float sdCircle( vec2 p, float r )
{
    return length(p) - r;
}
```

2. ### 长方形

```
   float sdBox( in vec2 p, in vec2 b )
   {
        vec2 d = abs(p)-b;
        return length(max(d,0.0)) + min(max(d.x,d.y),0.0);
   }
``` 

这个就是一个点到长方形的最近距离公式。分四种情况写一下就明白了。

3. ### 任意正多边形

```
```

4. ### 圆角化

在距离场函数中减去一个半径。相当于做了扩展。

5. ### 环化

对距离长函数取绝对值，减去一个半径。

6. 