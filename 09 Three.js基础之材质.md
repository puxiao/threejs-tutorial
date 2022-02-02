# 09 Three.js基础之材质

材质(material) 即 线段属性或物体表面的一些颜色、贴图、光亮程度、反光特性、粗糙度等属性。

按照用途，所以材质大体上可以划分为：

1. 点材质(应用来点、粒子上)
2. 线性材质(应用在线段或虚线上)
3. 基础材质(应用在面上的各种材质)
4. 特殊用途的材质(例如阴影)
5. 自定义材质

> 上面的分类划分，仅仅是我个人观点，事实上并没有明确的种类划分规定。



#### 材质基础

| 材质名称 | 解释说明       |
| -------- | -------------- |
| Material | 所有材质的父类 |



#### 点材质

| 材质名称       | 解释说明         |
| -------------- | ---------------- |
| PointsMaterial | 点材质(粒子材质) |



#### 线性材质

| 材质名称           | 解释说明                                 |
| ------------------ | ---------------------------------------- |
| LineBasicMaterial  | 线段材质(颜色、宽度、断点、连接点等属性) |
| LineDashedMaterial | 虚线材质                                 |



#### 基础材质(针对”面“)

| 材质名称             | 解释说明                                           |
| -------------------- | -------------------------------------------------- |
| MeshBasicMaterial    | 最基础的材质，不反射光，仅显示材质本身颜色         |
| MeshLambertMaterial  | 仅顶点处反射光                                     |
| MeshMatcapMaterial   | 自带光效(明暗)的材质                               |
| MeshPhongMaterial    | 任何点都反射光，拥有光泽度                         |
| MeshToonMaterial     | 卡通着色                                           |
| MeshStandardMaterial | 除光泽度外，还有粗糙度和金属度                     |
| MeshPhysicalMaterial | 除光泽度、粗糙度、金属度外，还有清漆度和清漆粗糙度 |



#### 特殊用途材质

| 材质名称             | 解释说明             |
| -------------------- | -------------------- |
| ShadowMaterial       | 阴影材质             |
| MeshDistanceMaterial | 另外一种阴影投射材质 |
| MeshDeptMaterial     | 远近距离深度着色材质 |
| MeshNormalMaterial   | 网格法向量材质       |
| SpriteMaterial       | 精灵材质/雪碧材质    |



#### 自定义材质

| 材质名称          | 解释说明       |
| ----------------- | -------------- |
| ShaderMaterial    | 着色器材质     |
| RawShaderMaterial | 原始着色器材质 |



注意：本文只是大体上讲解一些 Three.js 中的各个材质特性，具体每个材质的详细参数和用法，需要自己查阅 Three.js 官方文档：https://threejs.org/docs/index.html#api/zh/materials/Material



## 点材质：PointsMaterial

#### PointsMaterial：点材质/粒子材质

用来创建 粒子 材质。



## 线性材质：LineBasicMaterial、LineDashedMaterial

#### LineBasicMaterial：基础的线段材质

用来创建 线段 的材质，属性包括：颜色、宽度、断点、连接点等。



#### LineDashedMaterial：虚线材质

LineDashedMaterial 继承于 LineBasicMaterial，用来绘制虚线。



## 基础材质讲解说明

Three.js 中基础材质类型，我们先从 MeshPhongMaterial 说起。

#### 为什么要先讲 MeshPhongMaterial？

因为 MeshPhongMaterial 是使用最频繁，且处于特殊位置的材质。

MeshPhongMaterial 可以作为其他材质的参考对象：

1. 比 MeshPhongMaterial 简单的有 MeshBasicMaterial、MeshLambertMaterial
2. 和 MeshPhongMaterial 相似的有 MeshToonMaterial
3. 比 MeshPhongMaterial 复杂的有 MeshStandardMaterial、MeshPhysicalMaterial



## MeshPhongMaterial

### Phong光照模型：

Phong光照模型是最简单、最基础的光照模型，该模型只考虑物体对直线光的反射作用，不考虑物体之间的漫反射光(环境光)。



### Phong的假设前提：

1. 物体通常被设置为不透明
2. 物体表面反射率相同



### Phone的简单用法：

**新建一个 MeshPhongMaterial**

```
const material = new Three.MeshPhongMaterial({
  color:0xFF0000,
  flatShading:true
})
```

或者

```
const material = new Three.MeshPhoneMaterial()
material.color.set(0xFF0000)
material.flatShading = true
```



**设置颜色的N种方式：**

```
new Three.MeshPhoneMaterial({color:0xFF0000})
new Three.MeshPhoneMaterial({color:'red'})
new Three.MeshPhoneMaterial({color:'#F00'})
new Three.MeshPhoneMaterial({color:'rgb(255,0,0)'})
new Three.MeshPhoneMaterial({color:'hsl(0,100%,50%)'})
```

修改颜色：

```
material.color.set(0xFF0000)
material.color.set('red')
material.color.set('#F00')
material.color.set('rgb(255,0,0)')
material.color.set('hsl(0,100%,50%)')

material.color.setHSL(0,1,0.5)
material.color.setRGB(1,0,0)
```



### Phong的属性：flatShading(平面着色)

**特别强调：flatShading 属性并不是由 MeshPhongMaterial 定义的，而是有父类 Material 定义的。**

只不过由于 flatShading 比较重要，因此这里特别讲解一下该属性的作用。

flatShading 的值为 布尔值，该值指 是否启用 平面着色 模式。

> 默认值为 false，即不启用 平面着色 模式。



#### 补充说明 3 大着色模式：

**什么叫 着色？**

在三维图形学中，“着色” 的含义为：根据光照条件重建 物体各表面明暗效果的过程，就叫着色。

这个 "着色" 过程中，就牵扯到不同的着色算法，也就是不同的着色模式。



#### 最常见的 3 种着色算法(模式)：

**Flat Shading：平面着色**

根据每个三角形的法线计算着色效果，每个面只计算一次，也就是说相同的面采用同一个计算结果。

这种模式对于 立方体 来说会减小计算量，因为立方体每个面都是平整且唯一的。



**Gouraud Shading：逐顶着色**

针对每个顶点计算，而后对每个顶点的结果颜色进行线性插值得到片源的颜色。



**Phong Shading：补色渲染**

对每个三角形的每个片元进行着色计算。所以 Phong Shading 又被称为 **逐片元着色**

由于颜色是按片元着色的，得到的结果比 逐顶着色(Gouraud Shading) 要更加细腻，尤其是用于光亮表面效果更加真实。

Phong 并不是传统的英文单词，而是 生活在美国的越南籍科学家 Bui Tuong Phong  (裴祥风) 的名字。

所以 Phong Shading 又被称为 冯氏着色。


<br>

> 以下内容更新于 2022.02.02

在图形学中有一个被应用非常广泛的简单光照模型：冯氏光照模型

> 注：这里的 简单 是指计算量非常小，但却可以模拟出简单的高光和漫反射。


<br>

**冯氏光照模型、冯氏着色法 简介：**

**裴祥风** (1942-1975)，出生于越南，1973 年在美国 尤他大学 取得博士学位，并发明了冯氏光照模型和冯氏着色法，被广大 CG 界采用。


冯氏光照模型主要有 3 个分量组成：

1. 环境光照(Ambient Lighting)：物体几乎永远不会是完全黑暗的，环境光照一般是一个常量。

2. 漫反射光照(Diffuse Lighting)：模拟光源对物体的方向性影响，越是正对着光源的地方越亮。

   > 通过计算物体平面某个点的法向量与该点与光源的单位向量进行乘积，得到该点的亮度。
   >
   > 当这两个向量相互重叠时该点最亮(亮度值为1)，当这两个向量成九十度则最暗(亮度值为0)
   >
   > 向量、法线、乘积(也称内积) 这些都是 线性代数 中的词语，属于图形学中需要掌握的基础知识。如果学会了基础的 Three.js 后一定要去学习图形学，否则以后也做不出什么好的应用。

3. 镜面光照(Specular Lighting)：模拟有光泽物体上面出现的亮点。

最终物体呈现的样子就是以上 3 种光照结果直接叠加后的样子。



<br>

冯氏着色法(Phong Shading)：每个片元(fragment)或者每个点计算一次光照，点的法向量是通过顶点的法向量插值得到的。冯氏着色更加接近真实，当然计算开销也大。

与冯氏着色法相对的有：平面着色法(Flat Shading)、高洛德着色法(Gouraud Shading)



> 以上内容更新于 2022.02.02


<br>


> Phong 光照反射模型 也被称为 冯氏反射模型

**特别补充：**

1. MeshPhongMaterial 中的 Phong 是指 Phong 光照反射模型
2. Phong Shading 中的 Phong 只指 补色渲染
3. Phong光照反射模型、Phong补色渲染 这 2 个理论都是由 科学家 Bui Tuong Phong 提出的，所以也都以他的名字命名。

**目前来说，补色渲染/逐片元着色，也就是 Phong Sharding 是最好、最复杂的着色方式。**



### Phong的属性：emissive(发光颜色)

color 指材质的基本颜色，而 emissive 指材质的发光色。

注意：若将材质的 color 设置为黑色、emissive 设置为 某色 (例如 紫色)，那么此时材质呈现出的是 emissive 颜色。



### Phong的属性：shininess(光泽度)

shininess 的值为数字。

1. 最小值可设置为 0，即无光泽度，此时呈现出的效果和 Lambert 相同
2. 默认值为 30
3. 该值越大，光泽度越高，呈现的效果越接近 高清玻璃或钢琴烤漆的那种光泽感

注意：若将材质的 color 设置为黑色、emissive 设置为 某色(例如 紫色)、光泽度 设置为 0，那么此时材质呈现出的是 emissive 颜色，且无光泽度。



## MeshBasicMaterial、MeshLambertMaterial

当我们对 MeshPhongMaterial 一些特性有所了解后，通过对 光感反射 特性的对比，可以学习了解到 MeshBasicMaterial 和 MeshLambertMaterial。



#### 几种材质的光感对比：

* **MeshBasicMaterial：不反射任何光，仅显示材质本身颜色**

* **MeshLambertMaterial：仅顶点处反射光**

* **MeshPhongMaterial：任何地方都可反射光**

  > Lambert 虽然顶点也可以反光，但是相对 Phong 而言，Lambert 整体反光度极其微小、不明显



#### 性能提示：

关于颜色，以下 3 种情况所呈现出的最终效果完全相同：

1. 基于 MeshBasicMaterial，color 设置为紫色
2. 基于 MeshLambertMaterial，color 设置为黑色，emissive 设置为 紫色
3. 基于 MeshPhongMaterial，color 设置为黑色，emissive 设置为 紫色，shininess 设置为 0

以上 3 种设置下，最终所呈现出的颜色效果完全相同：都是紫色且无光泽。

但是，从渲染性能上来讲，从上往下 所需要的性能越来越高，因此假设材质不需要  Phone 反光模式 或 镜面高光(光泽)的情况下，应优先选择 较低性能 的材质。



## MeshToonMaterial

MeshToonMaterial 和 MeshPhongMaterial 类似但又不同。

MeshToonMaterial 并不会像 MeshPhongMaterial 那样使用平滑着色，而是使用渐变贴图( X 乘 1 的纹理) 来决定如何着色。

> 注意："渐变贴图( X 乘 1 的纹理) " 这句话我是从参考教程中看到的，我并没理解这句话的含义。

并不是说必须要设置纹理图片，若不设置则会采用默认的渐变策略：

1. 前 70% 区域 亮度为 70%
2. 后 30% 区域亮度为 100%

最终呈现出的效果，看起来特别像卡通动画的风格。所以 **MeshToonMaterial 又被称为 卡通网格材质**。

> 卡通动画通常大面积为纯色，只在底部增加深色的颜色，以此来表现出立体效果。



**补充一点：**

网上很多教程在讲解 MeshToonMaterial 时会提到：

`“MeshToonMaterial 是 MeshPhongMaterial 的扩展”`

但是我自己通过 MeshToonMaterial.d.ts 源码查询，并未发现 MeshToonMaterial 是继承于 MeshPhongMaterial 的，所以我认为这句话并不正确。



## MeshMatcapMaterial

一种自带光效(明暗)的材质。



## MeshStandardMaterial、MeshPhysicalMaterial

Phong 材质有一个属性 shininess(光泽度)，而 **MeshStandardMaterial** 有 2 个相对应的属性：

1. roughness：粗糙度，取值范围为 0 - 1，即 0 为粗糙度最低，此时表现出的光泽度最高
2. metalness：金属度，取值范围为 0 - 1，即 0 为非金属、金属度最高为 1

在 粗糙度和金属度 共同的作用下，呈现出 更加细腻、可控 的光泽度。



**MeshPhysicalMaterial** 继承于 MeshStandardMaterial ，新增加 2 个属性：

1. clearcoat：添加(应用)透明涂层的程度，取值范围为 0 - 1
2. clearCoatRoughness：透明涂层的粗糙度，取值范围为 0 - 1

额外添加的透明涂层，在装修上有一个专业的名词：**清漆**(又名 凡立水)

**清漆的含义为：**用透明涂料涂抹在物体表面，形成光滑薄膜，由于是透明的所以原有物体表面的纹理依然清晰可见不受影响。

> 清漆 会让物体呈现出更加光泽的效果。

**因此：**

1. clearcoat 可以翻译成：添加 清漆 的程度
2. clearCoatRoughness 翻译成：清漆粗糙度



## 基础材质小总结

从各种材质渲染所需性能，也就是渲染所需时间的快慢排序，依次是：

**MeshBasicMaterial > MeshLambertMaterial > MeshPhongMaterial > MeshStandardMaterial > MeshPhysicalMaterial**

上面排序中，越靠后的材质所呈现出的 细节 越多、真实感越强。



## 特殊材质：ShadowMaterial、MeshDistanceMaterial、MeshDepthMaterial、MeshNormalMaterial

#### ShadowMaterial：阴影类型的材质

我们目前还从未在示例中使用过 ShadowMaterial 材质，ShadowMaterial  是用来创建 阴影 的。



#### MeshDistanceMaterial：另外一种阴影投射材质

相对于 ShadowMaterial 的另外一种阴影投射材质，可以确保内部不透明部分不投射阴影。



#### MeshDepthMaterial：以像素的深度来着色的材质

**所谓 “像素的深度” 是指物体距离 镜头(摄像机) 的远近距离。**不同的距离决定不同的着色效果。

当物体距离镜头越近时会呈现白色、当距离越远时会呈现黑色。

> 你可以想象成在黑夜中去看远方的发光物体，越近的物体所发出的光眼睛看到的越多(显得物体越亮)，越远的物体所发出的光越暗，直至完全消失在黑暗中。

> 创建镜头的时候，会有 2 个 参数：near(最近距离)、far(最远距离)



#### MeshNormalMaterial：网格法向量材质

该材质是根据 三角面 的 法向量 方向的不同，从而赋予不同的颜色。

当物体旋转的时候由于各个面的法向量不断发生变化，物体的颜色也是不断发生变化。



#### SpriteMaterial：精灵材质/雪碧材质

精灵材质，也叫 雪碧材质。

大体来说，就是在场景中，可以加载图片，并且将图片当做纹理贴图，使用在材质上。



## 自定义材质：ShaderMaterial、RawShaderMaterial

#### ShaderMaterial：使用 Three.js 着色器制作自定义材质

#### RawShaderMaterial：完全自定义着色器所创建的自定义材质



**特殊材质、自定义材质具体的用法，此刻都不必深究，道路漫漫，时间还长，以后再慢慢研究。**



## 材质通用、常用的2个属性：flatShading、side

#### flatShading：是否平面着色

默认值为 false，即使用 渐变过渡着色。

若设置值为 true，则使用平面着色。

> 若启用平面着色，会让物体看起来更像是多面体，而不是光滑体。

> 本文在讲解 MeshPhongMaterial 的时候已经提到过此属性。



#### side：显示三角形的哪侧边(面)

默认值为 Three.FrontSide，即 只显示(渲染) 前面一侧的面。

若设置值为 Three.BackSide，则 只显示(渲染) 里面一侧的面。

> 对于绝大多数 图元 来说，通常 内部是不可见的，例如 球体或立方体的内部 你是看不见的，只能看见外面。side 通常是针对平面或非实体对象才有效果，例如 一个平面圆形，则背对 镜头的那一面即内面，在物体旋转过程中是可以看到内测那一面的。

若设置值为 Three.DoubleSide，则 两侧(外面和里面) 都将被显示(渲染)。

> 对于实体物体对象(非平面物体) 设置值为 Three.BackSide 或 Three.DoubleSide 都是无意义的。



## 材质不常用的1个属性：needsUpdate

#### 第1种情况：材质种类发生了重大变化

**针对 面 的材质**，之前已经提到过，大致分文 3 个类别：基础材质、特殊用途材质、自定义材质

在实际项目中，通常情况下我们并不会将某个物体的材质进行 3 大类别之间的转换。

例如我们不太会将某个 物体的材质 由某种基础材质突然变更为 阴影材质。

尽管实际中发生几率非常小，但万一要发生了呢？



#### 第2种情况：材质种类没变，但设置发生了变化

若材质在被使用过后，发生了以下 2 种设置变化：

1. flatShading 属性值的改变
2. 添加或删除 纹理(texture)
   1. 从不使用纹理变为使用纹理
   2. 从使用纹理变为不使用纹理
   3. 纹理的变更是允许的，并不属于 “添加或删除纹理” 的范畴中



#### 设置 needsUpdate 属性

**当上述 2 种情况发生后，此时就需要设置 needsUpdate 属性：**

```
material.needsUpdate = true
```

明确告知 Three.js 材质发生了重大变化，请使用新的材质重新渲染。

> 更换新的材质并重新渲染，这个过程将消耗比较多的计算性能。



**补充说明：**

在官方教程中，讲解 needsUpdate 属性时还有一句话：

`在从纹理过渡到无纹理的情况下，通常最好使用1x1像素的白色纹理。`

由于目前还没有学习过纹理，所以我暂时没理解这句话具体的含义是什么。



关于 材质 的一些基础知识，本文已经讲完。

**具体的每个材质都需要阅读官方文档，以及经过大量的练习才能掌握。**

同一个材质在不同光照、纹理的作用下，可能呈现出的效果相差很大。

下一节，学习 纹理(Texture)。

