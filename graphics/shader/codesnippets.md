# 常用的GLSL代码片段

## 伪随机
```glsl
float hash(vec2 st)
{
       return fract(sin(dot(st.xy, vec2(12.9898,78.233))) * 43758.5453123);
}
```

## 颜色转换
### HSV 与 RGB互转 
```glsl
vec3 hsv2rgb( in vec3 c )
{
    vec3 rgb = clamp( abs(mod(c.x*6.0+vec3(0.0,4.0,2.0),6.0)-3.0)-1.0, 0.0, 1.0 );
	return c.z * mix( vec3(1.0), rgb, c.y);
}
// Smooth HSV to RGB conversion 
vec3 hsv2rgb_smooth( in vec3 c )
{
    vec3 rgb = clamp( abs(mod(c.x*6.0+vec3(0.0,4.0,2.0),6.0)-3.0)-1.0, 0.0, 1.0 );
	rgb = rgb*rgb*(3.0-2.0*rgb); // cubic smoothing	
	return c.z * mix( vec3(1.0), rgb, c.y);
}
```
From https://www.shadertoy.com/view/MsS3Wc by iq


## 过程式着色器
### 基本图形
1. #### 正方形
```
float box(vec2 st,vec2 size) // st in [0,1]^2, size in [0,1]^2
{
    size = (1.0 - size)/2.0;
    vec2 m = smoothstep(size,size+0.001/*boarder*/,st);
    m *= smoothstep(size,size+0.001,1.0-st);
    return m.x*m.y;
}
```

### 基本操作
1. #### 平铺
```
vec2 tiling(vec2 st,float factor){
    st *= factor;
    return fract(st);
} // from the book of shader
```

2. #### 砖块式平铺
```
vec2 brickTiling(vec2 st,float factor){
    st *= factor;
    if(mod(floor(st.y),2.0) == 1.0){st.x += 0.5;} // or st.x += step(1., mod(_st.y,2.0)) * 0.5;
    return fract(st);
} // from The book of Shader
```

### 变换
1. #### 旋转
```
mat2 rotate2d(float _angle){
    return mat2(cos(_angle),-sin(_angle),
                sin(_angle),cos(_angle));
} //from The book of Shader
```

2. #### 缩放
```
mat2 scale(vec2 _scale){
    return mat2(_scale.x,0.0,
                0.0,_scale.y);
}// from The book of Shader
```

# 例子
## 移动的砖块
```
float box(vec2 st,vec2 size){
    size = (1.0-size)/2.0;
    vec2 m = smoothstep(size,size+0.001,st);
    m *= smoothstep(size,size+0.001,1.0-st);
    return m.x*m.y;
}
vec2 bricks(vec2 st,float factor){
    st *= factor;
    float tt = iTime;
    float t = fract(tt);
    if(mod(floor(tt),2.0) == 1.0){
        if(mod(floor(st.y),2.0) == 1.0){
            st.x += t;
        }else{
            st.x -= t;
        }
    }else{
        if(mod(floor(st.x),2.0) == 1.0){
            st.y += t;
        }else{
            st.y -= t;}
    }
    return fract(st);
}
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = fragCoord/iResolution.xy;
    vec3 col =vec3(box(bricks(uv,5.0),vec2(0.9)));
    fragColor = vec4(col,1.0);
}
```

## 简单的FBM
```
float hash(vec2 st)
{
       return fract(sin(dot(st.xy,
                         vec2(12.9898,78.233)))*
        43758.5453123);
}

float noise2d(vec2 st){
    vec2 i00 = floor(st);
    vec2 f = fract(st);
    vec2 i01 = i00 + vec2(0.,1.);
    vec2 i10 = i00 + vec2(1.,0.);
    vec2 i11 = i00 + vec2(1.,1.);
    float a = hash(i00);
    float b = hash(i10);
    float c = hash(i01);
    float d = hash(i11);
    return mix(mix(a,b,smoothstep(0.,1.0,f.x)),mix(c,d,smoothstep(0.,1.,f.x)),smoothstep(0.,1.,f.y));
}

float fbm(vec2 st){
    float A = 1.0;
    float f = 0.5;
    float v = 0.0;
    for(int i = 0;i<5;i++){
        v += f * noise2d(A * st);
        A *= 2.;
        f *=.5;
    }
    return v;
}

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = fragCoord/iResolution.xy;
    vec3 col = vec3(fbm(5.*uv));
    fragColor = vec4(col,1.0);
}
```
```
float hash(vec2 st)
{
       return fract(sin(dot(st.xy,
                         vec2(12.9898,78.233)))*
        43758.5453123);
}

float noise2d(vec2 st){
    vec2 i00 = floor(st);
    vec2 f = fract(st);
    vec2 i01 = i00 + vec2(0.,1.);
    vec2 i10 = i00 + vec2(1.,0.);
    vec2 i11 = i00 + vec2(1.,1.);
    float a = hash(i00);
    float b = hash(i10);
    float c = hash(i01);
    float d = hash(i11);
    return mix(mix(a,b,smoothstep(0.,1.0,f.x)),mix(c,d,smoothstep(0.,1.,f.x)),smoothstep(0.,1.,f.y));
}

float fbm(vec2 st){
    float A = 1.0;
    float f = 0.5;
    float v = 0.0;
    for(int i = 0;i<5;i++){
        v += f * noise2d(A * st);
        A *= 2.;
        f *=.5;
    }
    return v;
}

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 st = fragCoord/iResolution.xy;
    vec2 v = vec2(fbm(3.*st),fbm(iTime*0.13+st));
    float a = fbm(3.*st);
    float b = fbm(st + vec2(fbm(st*9.0),fbm(st*7.0*iTime)) + iTime*0.15);
    float c = fbm(st + 10.*vec2(fbm(3.0*st),3.0*fbm(st*5.0)) + iTime);
    float d = fbm(st + 1.*vec2(fbm(5.0*st),fbm(st)) + iTime*0.35);
    vec3 color = vec3(0.0);
    color = mix(vec3(0.1,0.419608,0.267),vec3(0.666667,0.8,0.4),clamp(smoothstep(0.,1.,a),0.,1.)); // cubic
    color = mix(color,vec3(0.,0.,0.324),clamp(smoothstep(0.,1.,length(v)+b),0.,1.)); // length
    color = mix(color,vec3(0.4,0.32,0.081),clamp(a*a,0.,1.));  //linear
    color = mix(color,vec3(.8,0.4,0.8),clamp(c*c,0.,1.)); 
    color = mix(color,vec3(0.2,0.073,0.6),clamp(d,0.,1.)); 
    fragColor = vec4(color,1.0);
}
```
