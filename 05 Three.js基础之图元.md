# 05 Three.js基础之图元

## 图元(Primitives)介绍

Primitive 这个单词在百度翻译里的解释是：原始的、远古的

Primitive 的复数即为 Primitives。

**所谓 图元 就是 Three.js 内置的一些基础 3D 形状，例如 立方体、球体、圆锥体等。**

**有些文章或教程，包括 Three.js 官方文档，都是将 图元 称呼为 几何体。**

**但是在本文中，我们依然先使用 图元 这个称呼。**

> 请注意，内置的图元并不一定都是 3 维体，也可以是 2 维的，例如 平面圆。

例如之前写的 HelloThreejs 示例中，就使用 BoxGeometry 来创建立方体。

> 虽然我们一直称呼为 立方体，但实际在 Three.js 中称呼其为 盒子(box)

> 在本文后面的一些文章中，也会将图元称呼为几何体。



<br>

<br>

---

> 以下内容更新于 2021.07.20

之前在写这篇文章的时候，还没有学习过图形学，所以对于一些名词的概念解释都是想当然，甚至是胡说八道，胡言乱语。

<br>

#### 顶点、图元、片元、图像 他们之间的递进关系

**顶点**：就是在 3D 世界中某一个具体的点，即点的位置(x,y,z)。除了位置信息，还可能包括 点的颜色或其他信息。

> 请注意，这些顶点位置都是相对的，依次是 局部位置、全局位置、镜头位置等等。
>
> 在管线渲染流程中，顶点处理模块的作用就是负责将顶点进行坐标转换。



<br>

**图元**：由若干个顶点构成的一组数据，用于构建或描述某种 二维或三维物体。

> 图元 中的 “元” 字可以理解为 “原始”的原，也就是说使用最少的点来描述一个物体的空间信息。
>
> 只有 1 个顶点依然可以是 图元，它只能表示某一个 点。例如 自动驾驶中扫描周围环境得到的 3D 点云数据就是由 一个一个小点 组成的。
>
> 如果是 2 个顶点，则可以表示出是一个 线段，同时 2 个点也可以表示出一个长方体。
>
> > 2 个顶点信息就可以表述出 1 个长方体？
> > 没错的，你可以想象成 这 2 个点分别是长方体的 斜对角线上的 2 个点，例如在 three.js 中 包装盒 Box3 就只有 2 个点的信息：坐标最大的点、坐标最小的点
>
> 3个顶点，则可以表示出一个 三角形，同时 3 个点也可以表示出一个圆。
>
> > 至于为什么 3 个顶点 可以表示出一个圆，你可以自己搜索或脑补。

> 请注意：图元依然为一堆顶点数据，而不是图像数据。

> 再次补充：假设一个物体有一部分不在显示范围之内，那么 webgl 会通过 裁切体(由镜头视椎体决定的) 对物体进行裁切，只将需要渲染的部分进行渲染，而裁切得到的内容则会重新计算，得到一个新的图元。



<br>

**片元**：包含图像颜色、位置、深度的信息数据。你可以把片元简单理解为 “未完全加工完成的图像数据”。

> 在 3D 图形管线渲染的流程中，经过裁切处理模块和图元组建处理模块之后，下一步经过光栅化处理模块，会将需要渲染的图元由一堆顶点数据转化为一堆图像数据。

> 请注意：片元已经不再是顶点数据，而是图像数据了，只不过这些图像数据是为完全加工完成，可以最终显示在屏幕上的图像数据。



<br>

**图像**：由 片元 经过片元处理模块，得到的最终图像数据。就是 3D 渲染输出到屏幕上的显示结果。

> 片元数据经过处理，用来更新缓存帧 上的像素，最终 缓存帧 上的结果就是最终渲染出的图像。

> 请注意：图像是由一个个像素构成。



<br>

以上内容为 图形学 中的相关知识，但是在本文中讲解的 “Three.js 中内置的图元” 是 Three.js 为了帮助我们快速创建一些常见物体所提供的 JS 类。

所以一定要理解清楚，本文讲解的 图元 和实际图形学中的图元 是有差异。

> 再次重申一遍：本文讲解的 图元 实际上是 JS 的类，帮助我们快速创建某些形状的 顶点数据。
>
> 一组相关的顶点数据才是图形学中的图元。



<br>

> 以上内容更新于 2021.07.20

---





<br>

## 3D模型的补充说明

内置的图元，都是一些基础的形状，相对简单，但也可以组合成相对复杂的 3D 场景。

**但对于绝大多数 3D 应用来说，通常流程是：**

1. 在专业的 3D 软件 例如 May、Blender、C4D 中创建模型
2. 将创建好的模型导出成模型文件，文件格式为 .obj 或 .gltf
3. Three.js 加载模型文件，然后开始后续操作



我们先不讨论如何导出或加载模型，那些会在后续操作中讲解。

此刻还是回归到默认的 图元 学习中。



## 图元的种类

### 图元汇总

| 图元种类(按英文首字母排序)   | 图元构造函数                                     |
| ---------------------------- | ------------------------------------------------ |
| 盒子(Box)                    | BoxBufferGeometry、BoxGeometry                   |
| 平面圆(Circle)               | CircleBufferGeometry、CircleGeometry             |
| 锥形(Cone)                   | ConeBufferGeometry、ConeGeometry                 |
| 圆柱(Cylinder)               | CylinderBufferGeometry、CylinderGeometry         |
| 十二面体(Dodecahedron)       | DodecahedronBufferGeometry、DodecahedronGeometry |
| 受挤压的2D形状(Extrude)      | ExtrudeBufferGeometry、ExtrudeGeometry           |
| 二十面体(Icosahedron)        | IcosahedronBufferGeometry、IcosahedronGeometry   |
| 由线旋转形成的形状(Lathe)    | LatheBufferGeometry、LatheGeometry               |
| 八面体(Octahedron)           | OctahedronBufferGeometry、OctahedronGeometry     |
| 由函数生成的形状(Parametric) | ParametricBufferGeometry、ParametriceGeometry    |
| 2D平面矩形(Plane)            | PlaneBufferGeometry、PlaneGeometry               |
| 多面体(Polyhedron)           | PolyhedronBufferGeometry、PolyhedronGeometry     |
| 环形/孔形(Ring)              | RingBufferGeometry、RingGeometry                 |
| 2D形状(Shape)                | ShapeBufferGeometry、ShapeGeometry               |
| 球体(Sphere)                 | SphereBufferGeometry、SphereGeometry             |
| 四面体(Tetrahedron)          | TetrahedronBufferGeometry、TetrahedronGeometry   |
| 3D文字(Text)                 | TextBufferGeometry、TextGeometry                 |
| 环形体(Torus)                | TorusBufferGeometry、TorusGeometry               |
| 环形结(TorusKnot)            | TorusKnotBufferGeometry、TorusKnotGeometry       |
| 管道/管状(Tube)              | TubeBufferGeometry、TubeGeometry                 |
| 几何体的所有边缘(Edges)      | EdgesGeometry                                    |
| 线框图(Wireframe)            | WireframeGeometry                                |

一共有 22 种内置的图元。

> 上面表格中关于图元的中文名字，有些是我根据含义自己编的，我已经尽量靠近英文原意。  
> 不同文章或教程可能对同一图元的称呼略微不同。



**不要被上面那么多图元吓到**，事实上他们并不复杂，并且多数情况下我们也用不到。

当需要用到了，只需要去查阅 Three.js 文档即可。



### BufferGeometry 与 Geometry 的区别

从上面的图元表格中不难发现，除了 Edges、WireframeGeometry 以外，其他图元的构造函数都是成对出现的。

**虽然 EdgesGeometry、WireframeGeometry 名字中并未出现 “Buffer”，但和其他所有包含 “Buffer” 字样的图元一样，他们都继承于 BufferGeometry。**

| 差异之处                 | BufferGeometry | Geometry                                                     |
| ------------------------ | -------------- | ------------------------------------------------------------ |
| 运算、渲染所消耗的性能   | 快             | 慢                                                           |
| GPU渲染                  | 支持           | 不支持，<br />需要 Three.js 内部转化为 BufferGeometry 后才支持 |
| 修改灵活度、可自定义程度 | 不高           | 高                                                           |
| 添加新顶点               | 不支持         | 支持                                                         |

**简单来说就是：**

* BufferGeometry 可自定义地方比较少，但性能高
* Geometry 可自定义地方比较多，但性能低一些

> 所有的 Geometry 对象最终都会被 Three.js 转化为 BufferGeometry 对象，然后再进行渲染。



图元理论上的知识就先讲到这里，在下一节中，会编写一些图元示例。
