# 01 Three.js简介

## Three.js简介概述

**Three.js概述**

Three.js 是基于 WebGL 技术，用于浏览器中开发 3D 交互场景的 JS 引擎。

> 默认 WebGL 只支持简单的 点、线、三角，Three.js 就是在此 WebGL 基础之上，封装出强大且使用起来简单的 JS 3D 类库。

> 目前主流现代浏览器都已支持 WebGL，也意味着支持 Three.js。



**Three.js优缺点**

* Three.js 擅长 WebGL 场景渲染，作为 JS 类库特别原生、灵活、自由度高
* Three.js 不擅长物理碰撞，因此不适合开发 3D 游戏



**先感受几个Three.js示例**

* 3D沙发产品在线预览：http://app.xuanke3d.com/apps/trayton/#/show
* 游乐园可交互场景：http://letsplay.ouigo.com/

* 跟随音乐楼房跳动：http://analysis.4sceners.de/



**Three.js应用场景**

1. 3D数据可视化场景
2. 产品720度在线预览
3. H5/微信小游戏
4. 科技教学3D模型展示
5. 网页VR、网页VR看房



**Three.js相关资料官网**

* Three.js官网：https://threejs.org/
* threejs.org 中文文档：https://threejs.org/docs/index.html#manual/zh/introduction/Creating-a-scene
* threejs.org 官方教程：https://threejsfundamentals.org/threejs/lessons/zh_cn/
* Three.js Github：https://github.com/mrdoob/three.js
* hewebgl.com Three.js基础教程：http://www.hewebgl.com/article/articledir/1
* webgl3d.cn Three.js教程：http://www.webgl3d.cn/Three.js/



## Three.js中的技术名词

### 3大核心关键模块

**场景(scene)**

场景是所有物体的容器。



**相机(camera)**

决定场景中哪些角度的内容会显示出来。

当然你也可以把`相机` 称呼为 `摄像头` 、`镜头`、`摄像机` 等。

> 本系列文章中，绝大多数时候都会使用 “镜头” 这个称呼



**渲染器(renderer)**

将 `相机` 中的内容渲染到浏览器页面中。



### 其他技术关键词

**几何体(Geometry)**

顾名思义，就是几何体，例如 球体、立方体、平面、以及自定义的几何体(汽车、动物、房子、数目等)。

在 Three.js 中，一个几何体的来源有 3 个：

1. Three.js 中内置的一些基本几何体
2. 自己创建自定义的几何体
3. 通过文件加载进来的几何体



**材质(Material)**

几何体的表面属性，包括颜色、光亮程度。

> 光亮程度是指物体表面反射光的能力值。Three.js 内置了不同的材质，不同材质对应不同的光亮程度。
>
> 内置材质 MeshBasicMaterial 是一种不可以反射光的材质，请注意这里说的不可以反射光并不是指该物体向黑洞那样连光都能吸收，而是指无论什么光源以何种角度照射到该物体上，该物体都不显示 “光亮”，而仅仅以材质本身的颜色或纹理来显示。

> 幸亏以前我学些过 C4D，所以对于这些名词和概念不是那么陌生

一个材质可以引用一个或多个纹理。



**纹理(Texture)**

纹理可以简单理解为一种图像或一张图片，用来包裹到几何体表面上。

纹理来源可以是：

1. 通过文件加载进来
2. 在画布上生成
3. 由另外一个场景渲染出



**网格(Mesh)**

一种特定的 几何体和材质 绘制出的一个特定的几何体系。

> 网格包含的内容为：几何体、几何体的材质、几何体的自身网格坐标体系

> 同一个材质和几何体可以被多个网格对象使用。
>
> 一个场景可以同时添加多个网格。



**光源(Light)**

指不同种类的光。



**视椎(frustum)**

透视镜头(PerspectiveCamera)所创造出的一种视觉可见空间。



以上提到的所有关键词和概念，会在后续学习过程中，逐个细致学习掌握。

加油！