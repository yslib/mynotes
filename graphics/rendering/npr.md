# 非真实感渲染(Non-Photorealistic Rendering)

## 描边


## 着色

色阶少，有边界

### 卡通着色

一般根据```NdotL=normalized(dot(norm, lightDir))```来从一维渐变纹理中采样。

如果要模拟高光，还需要```NdotV = normlized(dot(norm,lightDir))```来进行采样。

在卡通着色中，高光效果也是风格化的，多种多样。

### 基于色调的着色(Tone based shading)

与卡通着色不同，色调着色的色阶是连续的。根据```NdotL```在两种色调之间插值。


