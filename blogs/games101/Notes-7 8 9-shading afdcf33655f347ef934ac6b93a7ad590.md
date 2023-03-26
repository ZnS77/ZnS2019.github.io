---
layout: post
title: Games 101 Lecture Notes-Shading
tags: notes
date: 2023-03-26
---

# Notes-7/8/9-Shading

## Visibility/occlusion

画家算法： 从远到近 覆盖式的

- 需要从远到近地排序(O(nlogn) for n triangles)
- 当拓扑关系成环时会无法解决

### Z-Buffer

对每一个pixel 维护current min abs(z) value （最近的）和一个 frame buffer（具体的值）

![Untitled](Untitled%2010.png)

- O(n) 因为对每个像素只是在求最小值 而没有排序
- 与三角形的遍历顺序无关（因为深度往往是用浮点数储存的， 相同的概率非常小）
- 在MSAA中 要对每一个超采样点使用z-buffer
- 目前非常普遍的应用
- 无法处理（半）透明物体

## Shading

darkening & coloring.  Def here: applying a material to an object

shading 是局部的 不考虑occlusion

- specular highlights
    - 观察方向接近镜面反射的方向(v close to r)
        - blinn-phong   半程向量(v+l)/norm(v+l))  接近法向量（因为反射向量不太好算）
        - 接近：点积^alpha （cos函数的容忍度太高了） 通常的alpha在100-200
- diffuse reflection
    - 漫反射
    - 需要考虑到入射角的角度带来的能量变化 Lambert’s cosine law
        
        light per unit area is proportional to $cos\theta = I * n$
        
    - 还要考虑到Light Fallof, 点光源的能量在传播更远时球壳面积更大，单位面积下的能量更小
    - 考虑漫反射系数
    
    $$
    L_d = k_d \ (I/r^2) \ max(0, \ n ·l)
    $$
    
- ambient lighting
    - 近似的环境光 Ia 没有方向的要求 是常数
    - 但 可能过于近似 后期全局光照具体说明
    
    ![Untitled](Untitled%2011.png)
    

- 着色频率
    - 应用在每一个平面？ flat shading; one normal vector
    - 顶点后插值？ gouraud shading
    - 每一个像素？phong shading : 在所有的triangle上插值得到normal vector 然后在每一个像素点上计算full shading model
        
        ![Untitled](Untitled%2012.png)
        
    
    取决于具体的模型
    
    - 如何定义顶点的法线： weighted diffuse among all adjacent faces
    - per-pixel normal  vectors:  barycentric interpolation of vertex normals + normalize

## Graphics ( Real-time Rendering Pipeline) 渲染管线

![Untitled](Untitled%2013.png)

也就是Shader() 对每一个像素/verex/…都这么执行一次

着色： 可能在vertex processing中  也可能在fragment processing中（取决于着色频率的不同）

fragment即像素的概念

for fun: shader toy

那么 texture呢？ 也在fragment processing当中

## Barycentric Coordinates

## Texture Mapping 纹理映射

一个物体各个位置上的kd有一定的模式而不是全部相同

textured applied to surface ( parametization)

- 纹理坐标系： uv in [0,1]^2
    
    同一个纹理可以重复多次使用 tiled textures   wang tile 无缝纹理衔接的生成
    
- [u, v] = mapping(x, y )
- Texture Magnification
    - 映射得到的u,v不是整数，如何插值？
        - Nearest
        - Bilinear
        - Bicubic  取周围临近的16个(?)
- hard case: what if texture is too large?
    - 相当于采样频率变小了 会产生摩尔纹
    - 超采样 too expensive
    - what if we don’t map? mipmap!!  可以提前生成(串起来了)  有点像cv中的图像金字塔
        
        额外存储量 仅仅1/3 （二维图像，所以每一个小的是大的四分之一）非常妙的小证明： 复制为三份====》求和一共四份
        
        如何知道在MipMap的哪一层求值： 取本像素和相邻像素在纹理空间上的位置差距（4个）于是可以有此处四边形的边长，得到D的大小（数量级对应即可）
        
        ![Untitled](Untitled%2014.png)
        
    - Point Query vs. (Avg.) Array Query
    - 针对MipMap得到的 不连续层级 可以在两层之间再次插值 Trilinear interpolation
    - MipMap trilinear sampling 在远处有可能overblur 各向异性过滤可以部分解决
        - Anisotropic Filtering
        - 变成了D^2 个mipmap而不是D个
        - 因为原始的MipMap默认映射到texture中的正方形，而各向异性的假设是投射到长条形
        - 仍然存在问题：斜向的长条还是无法解决 → EWA filtering 使用椭圆嵌套的mipmap
- Advance Texure Methods