---
layout: post
title: Games 101 Lecture Notes-Rasterization
tags: notes
date: 2023-03-26
---

# Notes-5/6-Rasterization

## Finishing up Viewing

- aspect ratio = width/height (nearest)
- field-of-view (fovY)  (vertical or horizontal)  广角镜头
    
    与 l r b t 转换：  
    
    $$
    \tan {\frac{fovY}{2}} = \frac{t}{|n|}  \\ aspect = \frac {r}{t}
    $$
    
- after Model Viewl Projection?  canonical cube to screen
    - define screen:  an array of pixels, size of the array: resolution
    - raster == screen in German
    - pixel: RGB  坐标 (x,y)  integers  (0,0)→(width-1, height-1)
        - centered at (x+0.5, y+0.5)
            
            ![Untitled](Untitled%206.png)
            
        - 与z无关  [-1, 1]^2 → [0, width] * [0, height]  视口变换 Viewport transform
            - 非常早期的一些应用： CRT  晶体管电视 （隔行扫描）
            - LCD 液晶显示器 Liquid Crystal Display  两层光栅 旋光性
            - LED 显示器
            - 墨水屏 Electrophoretic Display

## Rasterization

- triangle represented   很多优点： planar, well-defined interior & barycentric coordinates for interpolation
- 判断三角形（的中心点）是否覆盖了像素
    - simple: sampling 利用像素中心对整个屏幕采样 （遍历也是一种采样）
    - 如何判断在内部： 3 cross products
    - 通过bounding box 剪枝

# Rasterization 2

- sampling
    
    jaggies moire patterns  (space)  wagon wheel effect (time)
    
    aliasing: high signal frequency, low sample frequency
    
    - how to deal with it?
        - pre-filter to blur images and then sample
        - why can’t reverse (sample then filter) anti-aliasing?     不满足交换律
- 滤波
    - 经过fft的频谱图： 中间低频外侧高频。十字线？叠加后的边缘高频
    - high/low pass filter   边界：高频
    - filtering = convolution = averaging
        
        spatial convolution ==  multiplication in frequency
        
- 采样在频谱上？
    - f(x） * 冲激函数  == (时域的卷积)   == repeating frequency contents
    
    ![Untitled](Untitled%207.png)
    
    - 因此 aliasing 就是在复制的过程中发生了混叠
    
    ![Untitled](Untitled%208.png)
    
    - 如何反走样？
        - increase sampling rate (受制于分辨率)
        - 减少原来信号中的高频信号： 先卷积（即滤波）再采样
            - 怎样模糊？ 先计算像素格被覆盖区域的百分比 ——超采样(MSAA 多重采样抗锯齿)
            
            ![Untitled](Untitled%209.png)
            
            - 但是提高了计算量
        - 还有其他的抗锯齿方式
            - FXAA (fast approximate AA)   采样后替换边缘
            - TAA (Temporal AA) 相邻的两帧使用MSAA中不同的小采样点
            - Super resolution ( 低分辨率到高分辨率之后细节缺失 ） DLSS 猜