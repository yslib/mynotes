# Shader Sampler

## 扭曲（水面，火焰）
扭曲是通过uv扰动实现的，也就是对要扭曲的材质的的采样点做一个偏移，这个便宜来自于一张纹理，然后再让这个纹理做一个动态平移就可以了。

## 菲涅尔
这个其实属于光照的范畴，就是把specular取成1-specular.