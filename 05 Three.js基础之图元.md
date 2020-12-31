# 05 Three.js基础之图元

## 图元(Primitives)介绍

Primitive 这个单词在百度翻译里的解释是：原始的、远古的

Primitive 的复数即为 Primitives。

**所谓 图元 就是 Three.js 内置的一些基础 3D 形状，例如 立方体、球体、圆锥体等。**

**有些文章或教程，包括 Three.js 官方文档，都是将 图元 称呼为 几何体。**

**但是在本文中，我们依然使用 图元 这个称呼。**

> 请注意，内置的图元并不一定都是 3 维体，也可以是 2 维的，例如 平面圆。

例如之前写的 HelloThreejs 示例中，就使用 BoxGeometry 来创建立方体。

> 虽然我们一直称呼为 立方体，但实际在 Three.js 中称呼其为 盒子(box)



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