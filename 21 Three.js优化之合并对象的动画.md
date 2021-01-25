# 21 Three.js优化之合并对象的动画

在上一篇中，我们将 19000 个柱状对象合并为 1 个整体，这样优化过后渲染速度性能大幅提高。

但是，我们所加载的是 2010 年 全球男性人口数量统计。

**假设现在我们添加新的需求：**

1. 加载并显示全球女性人口数量
2. 此时，我们程序中 男性与女性 各有 1 份数据
3. 我们需要做的动画就是：当切换 不同性别数据时，柱状物也会随着人口数量不同而发生 高低 变化的动画



**不同性别的人口数据 .asc 文件下载：**

1. 男性：https://threejsfundamentals.org/threejs/resources/data/gpw/gpw_v4_basic_demographic_characteristics_rev10_a000_014mt_2010_cntm_1_deg.asc
2. 女性：https://threejsfundamentals.org/threejs/resources/data/gpw/gpw_v4_basic_demographic_characteristics_rev10_a000_014ft_2010_cntm_1_deg.asc



很显然，之前把所有柱状物都合并成 1 个整体之后，是没有办法单独操作某一个柱状物的。

想实现每一个柱状物高低变化的动画，又该如何实现呢？



#### 实现方式：

1. 通过 设置物体材质的 morphtargets(变形目标) 属性为 true 来改变柱状物形状

   ```
    const material = new THREE.MeshBasicMaterial({
     vertexColors: true,
     morphTargets: true,
   })
   ```

   

2. 通过 Tween.js 来创建改变过程中的动画

   ```
   import { TWEEN } from 'three/examples/jsm/libs/tween.module.min.js'
   
   或者自己去安装 tween.js 的模块
   
   yarn add @tweenjs/tween.js
   //npm i @tweenjs/tween.js --save
   import TWEEN from '@tweenjs/tween.js'
   ```

   > Tween.js 的用法，请参考官网：https://github.com/tweenjs/tween.js

   

3. 通过材质的 Material.onBeforeCompile() 函数来产生改变过程中柱状物颜色的变化

   > Material.onBeforeCompile() 的用法，请参考文档：https://threejs.org/docs/index.html#api/zh/materials/Material.onBeforeCompile

   

#### 具体代码

...

上面仅仅是提供了解决思路，而实际具体的代码复杂程度，超出了我目前的认知。

所以我们暂且搁置，所有能力的同学，可自行查看本文对应的原版教程：

https://threejsfundamentals.org/threejs/lessons/threejs-optimize-lots-of-objects-animated.html，



将来有一天 Three.js 能力提高了，再来弥补上。

下一节，我们学习另外一个可以提升计算性能的方式：使用 WebWorker。