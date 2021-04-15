# 16 Three.js基础之自定义几何体



**重要说明：本文部分内容已经过时**

> .以下内容更新于2021年4月15日



本文写的时候还使用的是 0.124.0版本，但是在 0.125.2 以后（目前最新的是 0.127.0 ），关于 几何体 官方做了重大调整：

1. 官方已经将 Geometry 从核心库中移除，新的位置改为：

   ```
   import { Geometry } from 'three/examples/jsm/deprecated/Geometry'
   ```

   请注意 目录名为 deprecated，这个单词的意思就是：已弃用、不建议使用。

   也就是说官方已经不再建议你使用 Geometry 这个类了，那它的替代者是谁呢？

2. 在 r124 版本的时候，几何体(例如 BoxGeometry) 他们都继承的是 Geometry，但是在新版本中它们继承的是 BufferGeometry。

   也就是说如果你想自定义几何体，现在应该使用的是 BufferGeometry

   > Three.BufferGeometry

   > 换句话说，就是 BufferGeometry 替代了 Geometry

3. Three.Mesh() 函数中的参数也发生了变化
4. 或许还有其他更多地方发生了变化...



<br>

本文在编写的时候，还采用的是 Geometry，所以本文内容过时了。

目前先暂且不做修改，等到以后有时间了再将本文中的代码 Geometry 修改为 BufferGeometry。



<br>

你可以先跳过本章，继续后面章节的学习。

> 以下内容更新于2021年4月15日



<br>

接下来开始本文(已过时)的内容。

____





<br>

本文说的 几何体(geometry)，也就是之前 “05 Three.js基础之图元.md” 中的 图元(primitives)。

> 图元和几何体只是同一个对象(事物)的不同的叫法而已。

自定义几何体也就相当于自定义图元。

> 自定义几何体的这个过程，在传统 3D 软件中被称为 建模



## 自定义几何体(custom geometry)概述

先来一个灵魂拷问。

### 有必要在 Three.js 中自定义几何体吗？

**答：似乎没有必要，因为实际项目中，绝大多数情况下，我们都会通过专业的 3D 软件中来创建物体模型(建模)，而不是通过 Three.js 自定义建模。**

传统专业的 3D 转件包括：

1. Blender：开源免费的 3D 软件
2. Maya：侧重动画渲染的 3D 软件
3. Cinema4D(C4D)：轻量级的 3D 软件
4. 3D Sudio Max
5. ...

> 以上软件中，都有学习成本，相对而言 C4D 更加轻量、更加简单。

当 3D 模型创建好后，我们将模型导出为 gLTF 或 .obj 的文件，然后在 Three.js 中加载并使用它们。

尽管如此，但 Three.js 依然提供了自定义几何体(自定义建模)的方法。

> 可以让我们在不导入 建模文件 的前提下，通过 Three.js 来自定义几何体。

> 所有的传统 3D 软件建模过程都是可视化的，也就是说你可以时时看到物体，并且进行细微调整。而 Three.js 则是通过代码来建模的，整个建模过程是不可见的。

尽管实际中我们可能会很少机会在 Three.js 中自定义几何体，但是学习这方面的知识还是非常有必要的。



### Three.js中如何自定义几何体？

答：在 Three.js  中，一共可以有 2 种方式自定义几何体。

**第1种：继承于 Geometry**

优点：创建和使用的难度小

缺点：渲染启动速度慢、占更多内存



**第2种：继承于 BufferGeometry**

优点：渲染启动速度快、占更少内存

缺点：创建和使用的难度大



**补充说明：**

上面说的 渲染启动速度 慢，是指当场景第一次被渲染、后续修改后重新渲染的速度慢。

并不是说 绘制速度慢。

> 无论选择 Geometry 还是 BufferGeometry，绘制过程速度是相同的，他们的 “快慢” 主要体现在 第一次渲染启动速度 这方面。

当然，你不需花太多精力去理解 慢 的细节，你只需知道 Geometry 相对而言 渲染速度更慢一些即可。



**如何选择？**

答：对于要创建的自定义几何体各个面的三角形总和小于 1000，则优先选择继承于 Geometry。三角形数量超过这个范围的则推荐继承于 BufferGeometry。

事实上以上的选择仅供参考，重点是你实际项目中 是否觉得渲染启动速度、修改响应速度慢，如果慢则可进行优化改进。

> 如果由于客户端硬件配置比较高，感知不到慢或卡顿，那么可以而继续使用 Geometry。



## 自定义几何体示例：HelloCustomGeometry

#### 示例目标

1. 通过自定义几何体，来实现一个 立方体
2. 自定义 立方体 6 个面的颜色
3. 自定义 立方体 8 个顶点的颜色
4. 给 立方体 添加 光照法线，让立方体可以反光



#### 代码思路

**如何自定义一个立方体？**

答：主要分为 3 步

1. 第1步：实例化一个 Three.Geometry

   ```
   const geometry = new Three.Geometry()
   ```

   

2. 第2步：按照立方体相应的坐标，添加 8 个顶点

   ```
   geometry.vertices.push(
       new Three.Vector3(-1, -1, 1), // 1
       new Three.Vector3(1, -1, 1), // 2
       new Three.Vector3(-1, 1, 1), // 3
       new Three.Vector3(1, 1, 1), // 4
       new Three.Vector3(-1, -1, -1), // 5
       new Three.Vector3(1, -1, -1), // 6
       new Three.Vector3(-1, 1, -1), // 7
       new Three.Vector3(1, 1, -1) // 8
   )
   ```

   

3. 第3步：将相邻的 3 个顶点，依次按照逆时针顺序，构建成一个个三角形。

   > 立方体一共有 6 个面，每个面由 2 个三角形构成，因此一共需要构建 12 个三角形

   > 为什么必须是逆时针？这是 Three.js 规定的。

   ```
   geometry.faces.push(
       //前面
       new Three.Face3(0, 3, 2),
       new Three.Face3(0, 1, 3),
       //右面
       new Three.Face3(1, 7, 3),
       new Three.Face3(1, 5, 7),
       //后面
       new Three.Face3(5, 6, 7),
       new Three.Face3(5, 4, 6),
       //左面
       new Three.Face3(4, 2, 6),
       new Three.Face3(4, 0, 2),
       //顶面
       new Three.Face3(2, 7, 6),
       new Three.Face3(2, 3, 7),
       //底面
       new Three.Face3(4, 1, 0),
       new Three.Face3(4, 5, 1)
   )
   ```

   

   至此，就以成功构建出一个立方体的基本骨架。



**如何自定义 6 个面的颜色额？**

```
geometry.faces[0].color = geometry.faces[1].color = new Three.Color('red')
geometry.faces[2].color = geometry.faces[3].color = new Three.Color('yello')
geometry.faces[4].color = geometry.faces[5].color = new Three.Color('green')
geometry.faces[6].color = geometry.faces[7].color = new Three.Color('cyan')
geometry.faces[8].color = geometry.faces[9].color = new Three.Color('blue')
geometry.faces[10].color = geometry.faces[11].color = new Three.Color('magenta')
```



**如何自定义 8 个顶点的颜色？**

```
geometry.faces.forEach((face, index) => {
    face.vertexColors = [
        (new Three.Color()).setHSL(index / 12, 1, 0.5),
        (new Three.Color()).setHSL(index / 12 + 0.1, 1, 0.5),
        (new Three.Color()).setHSL(index / 12 + 0.2, 1, 0.5)
    ]
})
```



**如何开启顶点着色？**

```
const material = new THREE.MeshBasicMaterial({vertexColors: true})
```

```
const material = new Three.MeshPhongMaterial({ vertexColors: true })
```

> vertexColors 默认值为 false，即默认显示材质 color 的颜色。
>
> 如果没有给材质设置 color 值，那么默认颜色值为 白色



特别说明，在 Three.js 之前的版本中，vertexColors 的值并不是 Boolean，而是进行以下设置：

```
//export enum Colors {}
//export const NoColors: Colors;
//export const FaceColors: Colors;
//export const VertexColors: Colors;

const material = new THREE.MeshBasicMaterial({vertexColors: THREE.FaceColors});
```

在目前比较新的版本中，vertexColors 的值改为 Boolean 类型。



**如何添加光照法线？**

答：一共有 3 种方式

1. 给每一个 face 设置 normal 属性值

   ```
   face.normal = new Three.Vector3(...)
   ```

2. 通过 vertexNormals 属性来设置

   ```
   face.vertexNormals = {
     new Three.Vector3(...),
     new Three.Vector3(...),
     ...
     new Three.Vector3(...)
   }
   ```

3. 通过 computeFaceNormals() 和 computeVertexNormals() 这 2 个方法自动帮我们计算出光照法线。

   但是对于立方体而言，只执行 computeFaceNormals() 方法即可。

   ```
   geometry.computeFaceNormals()
   //geometry.computeVertexNormals() // 这个方法并不适用于立方体
   ```

> 第 3 种 方法最为简便，也比较常用。
>
> 本示例就采用第 3 种方式。



**补充说明：computeVertexNormals() 为什么不适用于立方体？**

答：因为 computeVertexNormals() 会从每个顶点共享的所有面的法线中计算得出法线，这样的发现会让立方体的顶点看上去更像一个球体。

> 如果你执行了 computeVertexNormals()，并不会报错，仅仅是立方体顶点处看似更加圆润，像球一样。



#### 代码示例：

```
import { useEffect, useRef } from 'react'
import * as Three from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

import './index.scss'

const HelloCustomGeometry = () => {
    const canvasRef = useRef<HTMLCanvasElement>(null)

    useEffect(() => {

        if (canvasRef.current === null) {
            return
        }

        const renderer = new Three.WebGLRenderer({ canvas: canvasRef.current })
        const scene = new Three.Scene()

        const camera = new Three.PerspectiveCamera(45, 2, 0.1, 100)
        camera.position.z = 8

        const light = new Three.DirectionalLight(0xFFFFFF, 1)
        light.position.set(2, 2, 4)
        scene.add(light)

        const helper = new Three.DirectionalLightHelper(light)
        scene.add(helper)

        const controls = new OrbitControls(camera, canvasRef.current)
        controls.update()

        //自定义一个立方体几何体
        const geometry = new Three.Geometry()
        geometry.vertices.push(
            new Three.Vector3(-1, -1, 1), // 1
            new Three.Vector3(1, -1, 1), // 2
            new Three.Vector3(-1, 1, 1), // 3
            new Three.Vector3(1, 1, 1), // 4
            new Three.Vector3(-1, -1, -1), // 5
            new Three.Vector3(1, -1, -1), // 6
            new Three.Vector3(-1, 1, -1), // 7
            new Three.Vector3(1, 1, -1) // 8
        )
        geometry.faces.push(
            //前面
            new Three.Face3(0, 3, 2),
            new Three.Face3(0, 1, 3),
            //右面
            new Three.Face3(1, 7, 3),
            new Three.Face3(1, 5, 7),
            //后面
            new Three.Face3(5, 6, 7),
            new Three.Face3(5, 4, 6),
            //左面
            new Three.Face3(4, 2, 6),
            new Three.Face3(4, 0, 2),
            //顶面
            new Three.Face3(2, 7, 6),
            new Three.Face3(2, 3, 7),
            //底面
            new Three.Face3(4, 1, 0),
            new Three.Face3(4, 5, 1)
        )

        geometry.faces[0].color = geometry.faces[1].color = new Three.Color('red')
        geometry.faces[2].color = geometry.faces[3].color = new Three.Color('yello')
        geometry.faces[4].color = geometry.faces[5].color = new Three.Color('green')
        geometry.faces[6].color = geometry.faces[7].color = new Three.Color('cyan')
        geometry.faces[8].color = geometry.faces[9].color = new Three.Color('blue')
        geometry.faces[10].color = geometry.faces[11].color = new Three.Color('magenta')

        geometry.faces.forEach((face, index) => {
            face.vertexColors = [
                (new Three.Color()).setHSL(index / 12, 1, 0.5),
                (new Three.Color()).setHSL(index / 12 + 0.1, 1, 0.5),
                (new Three.Color()).setHSL(index / 12 + 0.2, 1, 0.5)
            ]
        })

        geometry.computeFaceNormals()
        //geometry.computeVertexNormals() //对于立方体而言，无需执行此方法

        //const material = new Three.MeshBasicMaterial({ color: 'red' })
        const material = new Three.MeshPhongMaterial({ vertexColors: true })
        //const material = new Three.MeshPhongMaterial({ color: 'red' })
        const cube = new Three.Mesh(geometry, material)
        scene.add(cube)

        const render = (time: number) => {
            cube.rotation.x = cube.rotation.y = time * 0.001
            renderer.render(scene, camera)
            window.requestAnimationFrame(render)
        }
        window.requestAnimationFrame(render)

        const handleResize = () => {
            if (canvasRef.current === null) {
                return
            }

            const width = canvasRef.current.clientWidth
            const height = canvasRef.current.clientHeight
            camera.aspect = width / height
            camera.updateProjectionMatrix()
            renderer.setSize(width, height, false)
        }
        handleResize()
        window.addEventListener('resize', handleResize)

        return () => {
            window.removeEventListener('resize', handleResize)
        }
    }, [canvasRef])

    return (
        <canvas ref={canvasRef} className='full-screen' />
    )
}

export default HelloCustomGeometry
```

实际运行后，就会看到一个 炫彩的立方体。



#### 本文小结：

通过自定义一个立方体的示例，可以看出，尽管是一个很简单的立方体，可我们都需要非常复杂的空间坐标计算配置，因此还是本文开头那段话：

1. 如非必要，不要在 Three.js 中自定义几何体。
2. 使用传统的 3D 软件建模，更香。



#### 补充说明：

本系列教程，实际上是我一边学习 https://threejsfundamentals.org/threejs/lessons/ ，一边使用 React + TypeScript + 自己的语言和理解 重写一遍的。



本文对应的英文教程为：https://threejsfundamentals.org/threejs/lessons/threejs-custom-geometry.html

在原版的英文教程中，还有另外一个 通过一张图片来获得 纹理坐标(UV)，进而生成一张地图的例子。

我个人感觉没有必要去这么深入学习自定义几何体，所以本文略过这个示例。



除此之外，官方还有单独一篇，使用 BufferGeometry 来自定义几何体的教程：

https://threejsfundamentals.org/threejs/lessons/threejs-custom-buffergeometry.html

我认为现阶段，没有必要如此这般的深入去学习自定义几何体，因为暂停这部分的学习。

> 如果你还有精力，可以去学习一下。



## Three.js基础知识总结

通过前面一系列的学习，我们终于将 Three.js 基础知识学习完成。

回顾一下我们都学习了哪些知识点：

1. Three.js 简介、项目初始化、入门示例
2. 图元、3D文字、场景、材质、纹理、灯光、镜头、阴影、雾、离屏渲染、自定义几何体
3. 辅助对象(灯光辅助对象 LightHelper 、镜头辅助对象 XxxxCameraHelper、坐标轴辅助对象 AxesHelper)、镜头轨道控制类(OrbitControls)



真心不容易，给自己一朵小红花！

...

我们本系列教程整体的规划是：

1. 基础篇 (✓)
2. 技巧篇 (x)
3. 优化篇 (x)
4. 解决方案 (x)
5. WebVR (x)
6. 实例篇 (x)

目前我们已经学习了基础篇，对 Three.js 已经有了足够的基础知识掌握，后面的学习都是建立在这些基础知识之上的。



**接下来，进入技巧篇——按需渲染。**



#### 稍等，再啰嗦几句：

我们后续的讲解文章中，将加快进度，不再像 基础篇 这样如此细致，甚至是啰嗦。

因此，我希望你不看教程示例代码，而是自己独立敲出示例代码。如果做不到，那么你先不要着急进入下一篇，而是应该再回过头，反复阅读，反复敲几遍代码。

> 在做(动手敲代码)的过程中学习，而不是只看不动手。