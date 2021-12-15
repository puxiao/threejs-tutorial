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

**关于图元的额外补充：**

实际上在 opengl 、webgl 的概念中，图元分为 2 种：

1. 几何图元：使用顶点、线段、三角形、曲线等等 用于描述物体 “几何轮廓” 。

   > 几何图元可以进行空间转换，例如平移，旋转，缩放等操作

2. 图像图元：图像图元又被称为 光栅图元。使用像素阵列 用于直观储存 “图片信息”。

   > 通过描述就应该知道，实际上所谓的 图像图元就是材质中的纹理贴图。

   > 图像图元不可进行空间转换



<br>

几何图元 经过变换、投影、光栅化后，到达片元操作环节的。

图像图元(也就是纹理)是直接到达片元操作环节的。

最终在片元操作环节，几何图元 + 图像图元，最终合成得到物体图像。

> 当然还需要其他操作，例如光线反射等



<br>

而实际中，我们通常不会使用 “图像图元” 这个名词，而是使用 “纹理”。

所以在本文或者一些常见的教程中，“图元” 往往都是指 “几何图元”。



<br>

**片元**：包含图像颜色、位置、深度的信息数据。你可以把片元简单理解为 “未完全加工完成的图像数据”。

> 在 3D 图形管线渲染的流程中，经过裁切处理模块和图元组装模块之后，下一步经过光栅化处理模块，会将需要渲染的图元由一堆顶点数据转化为一堆图像数据。

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



<br>

> 以下内容更新于 2021.11.27

**特别补充说明：内置的图元实际上也是变化多端的！**

为什么这么说呢？

例如：圆柱(Cylinder)，字面上它是用于创建圆柱体的，但是实际上认真阅读官方文档你会发现是这样描述它的构造函数的

> CylinderGeometry 官方文档：https://threejs.org/docs/index.html#api/zh/geometries/CylinderGeometry

```
CylinderGeometry(radiusTop : Float, radiusBottom : Float, height : Float, radialSegments : Integer, heightSegments : Integer, openEnded : Boolean, thetaStart : Float, thetaLength : Float)
radiusTop — 圆柱的顶部半径，默认值是1。
radiusBottom — 圆柱的底部半径，默认值是1。
height — 圆柱的高度，默认值是1。
radialSegments — 圆柱侧面周围的分段数，默认为8。
heightSegments — 圆柱侧面沿着其高度的分段数，默认值为1。
openEnded — 一个Boolean值，指明该圆锥的底面是开放的还是封顶的。默认值为false，即其底面默认是封顶的。
thetaStart — 第一个分段的起始角度，默认为0。（three o'clock position）
thetaLength — 圆柱底面圆扇区的中心角，通常被称为“θ”（西塔）。默认值是2*Pi，这使其成为一个完整的圆柱。
```

请注意最后的 2 个参数：

1. thetaStart(默认值为0) 
2. thetaLength(默认值为2*Pi)

也就是说，你不修改这 2 个默认值，**那么默认创建出的是一个完整的圆柱体**，但是假设你修改了这 2 个值，比如 将 thetaLength 修改成 0.3*Pi (54°)，那么最终将创建出一个 夹角为 54° 的**扇形**(体)。

如果感兴趣，可以看一下我发布的这个项目，由数据生成3D饼图：https://github.com/puxiao/pie-3d

> 提醒：最好你在看完本系列教程后(不仅是本小节)，再去看上面提到的 pie-3d 。

通过上面对 CylinderGeometry 的描述，我们可以知道 Three.js 默认自带的图元实际上是可以产生很多变化的，得到的不一定仅仅是图元的 "字面" 物体。

> 以上内容更新于 2021.11.27



<br>

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



<br>

> 以下内容更新于 2021.11.27

**上面关于 BufferGeometry 和 Geometry 的区别这段话已经过时了**，因为在较新的 Three.js 版本中已经将 Geometry 从核心类中移除。

目前你接触到的都应该只有 BufferGeometry。

#### 一些心里话：

首先非常抱歉得说一句：一年前在写本系列文章时，我是一个对图形学一无所知，对 Three.js 好奇但又非常小白的人，我是一边学习一边写下本系列文章的。

**所以本系列教程绝对不是好的教程——假设 Three.js 是一座大山的话，而我是站在山脚下向你讲述上山道路的那个人，但我自己也未曾上过这座山。**

随着我对图形学、webgl、Three.js、Canvas 的一些认知提升，我深深觉得想要写出好教程，一定要站在更高的维度，拥有更高的视野才可以更好向别人指明方向，写出好教程。

但是本教程对于那些完全小白，完全对 Three.js 一无所知的人，多少还是有些帮助的(尽管我指明的道路并不是最佳道路)，**很感谢那些 Star 本教程的人**。

即使看完全部的本教程，那么最多你也仅仅算是学会了个皮毛，简单入门而已，真正复杂难的是 图形学 中的一些知识点，例如 向量，矩阵，齐次坐标，点乘，叉乘，球极坐标，当然最复杂的莫过于 自定义渲染器(shader)。

关于3D 技术栈，虽然不够严谨，但是大体上可以这样表述：**图形学(CG) > OpenGL > OpenGL ES 2.0 > WebGL > Three.js**

所以 Three.js 仅仅是 web 3D 最基础，表层的知识技术栈，想要深入学习，你会发现这是一条几乎不到头的道路，学秃。

> 图形学就是那种 从入门到放弃 的知识体系。
>
> 但是别灰心，我们实际上并不会真正需要那么深入高深的，学会 Three.js 可以做一些基础的 网页 3D 还是会比一般前端要显得厉害很多。

<br>

#### BufferGeometry的重要知识点：position、normal、uv

> 先普及个基础知识：在 3D 中 Vector3 既可以表示一个 三维坐标，也可以表示一个三维方向。

一个完整的 BufferGeometry 是由若干个 点(Vector3) 构成的：

> 上面提到的 点 准确说应该是 3维 点坐标，对应的是 Vector3 ：https://threejs.org/docs/index.html#api/zh/math/Vector3

下面的知识实际上是针对 图形学 和 OpenGL 的。

1. position：坐标(每个坐标就是一个 vector3，由 3 个数字组成)，所有的坐标就是组成该 BufferGeometry 的所有 点 的信息(对于底层的 BufferGeometry 而言 3维点的 (x,y,z) 坐标是分开存储值的)。

   > 这就是 Three.js 针对 webgl 进行的封装，实际上我们平时更多时候都使用的是 Vector3，而不是具体的 3 个值。

2. normal：法线(每个法线就是一个 vector3，由 3 个数字组成)，用于存储每个 3D 坐标点的朝向，用于计算 反光。

3. uv：纹理映射坐标(每个uv就是一个 vector2，由 2 个数字组成)，用于存储每个 3D 坐标点对应渲染纹理时对应的 位置点信息，用于计算 贴图。

   > 对于纹理而言，它都是 二维的平面，因此 uv 的值对应的是 Vector2，由 x,y 2 个坐标值组成，且每个值的取值范围都是 0 - 1。
   >
   > 你可以简单把 0 - 1 理解成  0% - 100%，对应的是一个百分比的值。

通过上面的讲述，我们大致可以作出以下结论，如果我们自定义一个 BufferGeometry，那么：

> 对于初学者而言几乎不需要、也做不到 可以 自定义 BufferGeometry 这一步，我这里只是超前提一下。

1. 假设这个 BufferGeometry 不需要考虑 反光 和 纹理贴图，那么它只需要拥有(设置) positon 就可以了。

   > this.setAttribute('position', new BufferAttribute(this._vertices, 3))

2. 假设这个 BufferGeometry 需要考虑反光，但不需要考虑纹理贴图，那么它需要设置 postion 和 normal。

   > this.setAttribute('position', new BufferAttribute(this._vertices, 3))
   >
   > this.setAttribute('normal', new BufferAttribute(this._normals, 3))

3. 假设这个 BufferGeometry 需要考虑反光和纹理贴图，那么它的 postion 、normal、uv 都需要设置。

   > this.setAttribute('position', new BufferAttribute(this._vertices, 3))
   >
   > this.setAttribute('normal', new BufferAttribute(this._normals, 3))
   >
   > this.setAttribute('uv', new BufferAttribute(this._uvs, 2))

4. **特别强调，上面提到的 position 是一个 BufferGeomerty 所有点信息的集合，它并不是 Mesh(网格，3D物体) 的 位置信息。**

如果你理解不了我说的这段话，完全没有关系，忽略这段我补充的知识点，我也是学习  Three.js 快 1 年后才明白的。对于现在的你而言不理解是正常的。

忽略我上面的这段话，继续本教程后面的学习吧。

> 以上内容更新于 2021.11.27



图元理论上的知识就先讲到这里，在下一节中，会编写一些图元示例。
