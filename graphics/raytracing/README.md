# 一个简单的光纤追踪涉及的接口
* 相机
    raycast(int x,int y) -> (vec3 ro,vec3 rd)
* 场景求交
    intersect(vec3 ro,vec3 rd,float t)->(t:float,objectid:int)
* 物体的表面信息
    surface(int objectid, vec3 pos) -> (color:vec3,nor:vec3)
* 计算直接光照
    lighting(vec3 pos, vec3 nor)->(color:vec3)
* 计算和光源的遮挡
    occlude(vec3 pos)->(occ:bool)
* 生成递归光线
    brdf(int objectid, vec3 pos, vec3 nor)->(ro:vec3,rd:vec3)


