# 08 Three.js基础之场景

再次回顾一下 Three.js 3 大 核心要素：场景、镜头、渲染器

**本文主要将 Three.js 中的 场景，但是请注意，本文讲的场景实际上是指 场景图(scene graph)，而不是单指 我们之前示例代码中用到过的 场景 Three.Scene。**

但是请注意，本文讲的场景实际上是指 场景图(scene graph)，而不是单指 我们之前示例代码中用到过的 场景 Three.Scene。



## 场景图(scene graph)的概念解释

### 场景与场景图的关系：

**SceneGraph 准确的翻译应该是叫：场景图，但是本文中，我有时依然倔强得把他叫做 "场景"。**

**但无论我怎么称呼它，请你记得：场景(Three.Scene) 只是 场景图 中的一种。**



### 场景图的数据结构：

抛开 Three.js 不谈，我们先看一下在数据结构中，树与图 的概念区分。

**树：一种 分层 数据的抽象模型**

> 呈现出的是像大树枝一样的结构，根据结构特征还可以划分为 二叉树、红黑树、大顶树、小顶树等等

**图：网络结构的抽象模型，是一组由边连接的节点**

> 呈现出的是像蜘蛛网、道路网、航班线路一样的结构



回到 Three.js 中。

**请务必记得：**

1. 场景图 中的 图，并非数据结构中的图
2. **场景图的数据结构并非 图，而是 树**



**补充一下：**

在有一些教程示例代码中，当循环遍历 场景 中物体对象时，你或许会看到这样的代码：

```
他使用的是：xxx.forEach((node) => { node ....}) 

而不是：xxx.forEach((itme) => { item ...})
```

尽管无论数组元素变量名是叫 node 还是 item，实际上效果是相同的，但是**他为什么会用 node 这个单词呢？**

答：因为**场景图的数据结构是树，而场景上的物体对象实际就是树结构中的一个节点**，节点对应的单词就是 node。





### 场景图(空间)的含义：

**在 Three.js 中，场景即空间，而 空间 包含以下几种情况**：

1. 由  Scene 创建的普通场景、普通场景中还可以添加雾(Fog、FogExp2)从而变成具有雾化效果的场景

   > 无论哪种场景下，都可以添加 Object3D、Mesh

   > Scene 场景下，距离镜头越远的物体看上去越小，但清晰度不变  
   > 包含 雾(Fog、FogExp2) 场景下，距离镜头越远的物体不光看上去越小，同时被雾气环绕

   > 对于现阶段的我们来说，目前主要以使用 Scene 场景为主，Fog、FogExp2 会在以后学习和使用

2. 由 Object3D 创建的 空白空间

   > 可以添加 Mesh

3. 由 Mesh 创建的 具体的物体所在的网格空间

   > 可以添加其他的 Mesh

> 理论上，Object3D 和 Mesh 是可以互相添加，互相嵌套的，最终会构成一个复杂的空间体系



> 请注意，为了避免 “场景图” 这 3 个字过于绕口，以及为了方便理解，在下面文字中，我会将 场景图 称呼为 场景或空间

### 场景的几个概念

### 概念1：一个局部的相对空间，即为一个场景

例如太阳系就是一个空间(场景)



### 概念2：一个空间(场景) 又可能是由 几个子空间(场景) 组合而成

太阳系由 8 大行星构成

行星除了本身之外还包卫星，例如地球和月球

地球上又包含陆地和海洋

陆地上又包含中国，中国包含你此刻所处的空间   



### 概念3：表面上添加某场景，但实际上执行的是合并场景

例如 sceneA.add(sceneB)，表面上看 sceneA 添加了 sceneB，sceneB 称为了 sceneA 的子场景，但事实上根本并不是这样！

**什么？这岂不是和 概念 2 完全相悖？**

**没错！确实是即合并又互相独立。**

**所谓独立：sceneB 中的元素(物体、灯光)的坐标位置继续保持独立**

**所谓合并：sceneB中的元素(物体、灯光)被复制添加到其他场景中，例如 场景B 中的灯光会影响 场景C**



**举一个很容易犯错的例子：**

假设有 环境灯光 lightB、lightC，和 场景 sceneA、sceneB、sceneC

```
sceneB.add(lightB) //场景B 中添加 灯光B
sceneC.add(lightC) //场景C 中添加 灯光C

sceneA.add(sceneB) //场景A 中添加 场景B
sceneA.add(sceneC) //场景A 中添加 场景C

renderer.render(sceneA,camera) //使用场景渲染器，将 场景A 渲染出来
```

**你可能以为 灯光B 只在 场景B 中起作用、灯光C 只在 场景C 中起作用。**

**但事实根本不是这样，上面代码渲染过后，你会发现：场景B 和 场景C 中，分别都受到 环境灯光B 和 环境灯光C。**

**因为环境灯光是全局的、环境灯光在场景中无处不在、会影响场景中全部的物体。**

**假设不是环境灯光，而是普通的平行灯光，事实上依然会影响(照耀)到其他 “子场景”上的物体，只不过可能因为距离设定原因，不会像全局环境光那样影响明显。**



**为什么会这样？**

我们查看一下 scene.add() 函数源码：

> 注意：Scene 继承于 Object3D，所以 scene.add() 方法实际上是由 Object3D 定义的。

```
add: function (object) {

    if (arguments.length > 1) {
        for (let i = 0; i < arguments.length; i++) {
            this.add(arguments[i]);
        }
        return this;
    }

    if (object === this) {
        console.error("THREE.Object3D.add: object can't be added as a child of itself.", object);
        return this;
    }

    if ((object && object.isObject3D)) {
        if (object.parent !== null) {
            object.parent.remove(object);
        }

        object.parent = this;
        this.children.push(object);

        object.dispatchEvent(_addedEvent);

    } else {
        console.error("THREE.Object3D.add: object not an instance of THREE.Object3D.", object);
    }
    return this;
}
```

**源码分析：**

1. if (object.parent !== null) { object.parent.remove(object);  } //如果元素(物体、灯光)拥有父级，则将该元素从父级中删除
2. object.parent = this; //将元素(物体、灯光)的父级指向 this(自己)
3. this.children.push(object); //将元素(物体、灯光)添加到自己场景中的 children 中

经过以上 3 步操作，**add() 函数实现了 将 子场景元素拆散、合并到自己(最外层场景、顶场景)中**。



**假设我就希望有若干个“子场景”，子场景中的灯光(哪怕是环境光)是独立，不会影响其他 子场景的，怎么实现？**

答：只能声明多个 渲染器(WebGLRenderer)，每个渲染器渲染一个场景(Scene)、每个场景内添加一种光源。

> 提前预告：在后续讲解 灯光 那一章节中，就会运用到这个知识点。



### 概念4：一个子空间(场景)只需要关注和他最紧密相关的空间即可

假设你此刻在家里，那么你的相对空间就只针对家里即可，尽管你此刻所处的地球正在自转，你无需关心这个事情。

月球也可能只关心它是否围着地球转，而不需要关心他在太阳系中的运动轨迹



#### 概念4引申出来的另外一个概念：通过空间嵌套来改变原有的相对状态

* **一个 空间A 嵌套进入另外一个 空间B，此时 空间A 将会拥有 空间B 的一些属性，例如 空间A 会随着 空间B 一起缩放**
* **两个子空间 A和B 都嵌套进另外一个空间 C，此时 空间A、空间B 相对独立且共存**



#### 举例说明1：修改文字对象的旋转中心点

默认情况下，Three.js 中创建的 TextBufferGeometry 对象旋转点位于左侧。

为了让 文字对象 看上去以 中心位置 为中心点旋转，那么可以这样操作：

1. 通过 new Object3D() 创建 空间A

2. 通过 new Mesh( new TextBufferGeometry({ ... } ), createMaterial() ) 创建文字对象

3. 修改文字的中心点

   ```
   geometry.computeBoundingBox()
   geometry.boundingBox?.getCenter(mesh.position).multiplyScalar(-1)
   ```

4. 将 文字对象(网格) 添加到 空间A 中，同时将 空间A 添加到场景中

经过这样操作过后，即可将 文字对象 文字对象的中心点改为中间。



#### 举例说明2：创建月球与地球的相对空间

太阳和地球构成一个相对空间、地球与月亮也构成一个相对空间。

假设我们现在的目标是创建 月球与地球的相对空间，那么可以这样操作：

1. 创建地球对象 A、月球对象 B

   > “地球对象”，更加精准的描述应该是：地球对应的网格，也就是 “地球本身的空间”
   >
   > 为了不让月球和地球重叠在一起，通常情况下会给 月球对象 B 设置 .position.x = xx，好让地球和月球之间存在一定的距离

2. 通过 new Object3D() 创建空间 C

3. 将 A、B 都添加到 C 中

4. 将 C 添加到主场景中

经过这样操作后，主场景中包含 C，而 C 包含 A、B，至此形成了一个 地球和月球 共同存在的空间。



### 场景(空间)的最常见操作

1. 将 空间A 加入到 空间B：B.add(A)
2. 设置空间 A 在空间B 中的位置：A.position.x = xxx



## 场景的示例：太阳、地球、月亮

#### 我们模拟出以下场景：

1. 月球自转的同时，围绕地球旋转
2. 地球自转的同时，围绕太阳旋转
3. 太阳仅自转，位置不变



> 本文的重点在于讲解 场景 的概念，若对代码中某些 方法或属性的使用 不太能够理解也没有关系，将来会慢慢学习到。

#### 代码文件说明：

1. 我们将在 src/components/hello-scene/ 目录下创建 index.stx 作为本次演示主文件。

2. 与以往代码不同，这次我们将创建 太阳、地球、月亮、以及 光源 的过程迁移到另外一个单独的文件中 ，好让我们在  useEffect 中的代码更加清爽一些。

   对应的文件为 src/components/hello-scene/create-something.ts



#### 代码核心说明：

1. 我们将创建一个球体，让太阳、地球、月亮都由这个球体创建而来，只不过每个球体网格在材质(颜色)、大小方面不同。

2. 我们将创建 3 个相对空间：

   1. 月球相对地球的轨道空间

      > 这个空间中只有月球，因为设置了偏差(poisition.x = 2)，所以月球会做圆形轨道运动

   2. 地球(含月球)相对太阳的轨道空间

      > 这个空间中有地球(含月球)，同样因为设置了偏差(position.x = 10)，所以会整体做圆形轨道运动

   3. 太阳与地球轨道构成的相对空间 

      > 这个空间包含太阳、地球(含月球)



#### 补充说明：

1. 为了让我们更加容易看到 球体 的自转，所以无论是太阳还是地球或月亮，外形都设置成一个 六边形的球体。

2. 我们只是为了演示 相对空间 的使用，所以 太阳、月亮、地球 的尺寸、自转频率、位置关系等是随意设置的值，并不是真实中的大小比例。

   > 科普一下：实际中，太阳直径是地球直径的 109 倍、地球直径是月球直径的 4 倍



#### 具体的代码：

#### create-something.js

```
import { Mesh, MeshPhongMaterial, Object3D, PointLight, SphereBufferGeometry } from "three"

//创建一个球体
const sphere = new SphereBufferGeometry(1, 6, 6) //球体为6边形，目的是为了方便我们观察到他在自转

//创建太阳
const sunMaterial = new MeshPhongMaterial({ emissive: 0xFFFF00 })
const sunMesh = new Mesh(sphere, sunMaterial)
sunMesh.scale.set(4, 4, 4) //将球体尺寸放大 4 倍

//创建地球
const earthMaterial = new MeshPhongMaterial({ color: 0x2233FF, emissive: 0x112244 })
const earthMesh = new Mesh(sphere, earthMaterial)

//创建月球
const moonMaterial = new MeshPhongMaterial({ color: 0x888888, emissive: 0x222222 })
const moonMesh = new Mesh(sphere, moonMaterial)
moonMesh.scale.set(0.5, 0.5, 0.5) //将球体尺寸缩小 0.5 倍


//创建一个 3D 空间，用来容纳月球，相当于月球轨迹空间
export const moonOribit = new Object3D()
moonOribit.position.x = 2
moonOribit.add(moonMesh)

//创建一个 3D 空间，用来容纳地球，相当于地球轨迹空间
export const earthOrbit = new Object3D()
earthOrbit.position.x = 10
earthOrbit.add(earthMesh)
earthOrbit.add(moonOribit)

//创建一个 3D 空间，用来容纳太阳和地球(含月球)
export const solarSystem = new Object3D()
solarSystem.add(sunMesh)
solarSystem.add(earthOrbit)

//创建点光源
export const pointLight = new PointLight(0xFFFFFF, 3)

export default {}
```



#### index.tsx

```
import { useRef, useEffect } from 'react'
import * as Three from 'three'
import { solarSystem, earthOrbit, moonOribit, pointLight } from '@/components/hello-scene/create-something'

import './index.scss'

const nodeArr = [solarSystem, earthOrbit, moonOribit] //太阳、地球、月亮对应的网格

const HelloScene = () => {

    const canvasRef = useRef<HTMLCanvasElement>(null)
    const rendererRef = useRef<Three.WebGLRenderer | null>(null)
    const cameraRef = useRef<Three.PerspectiveCamera | null>(null)
    const sceneRef = useRef<Three.Scene | null>(null)

    useEffect(() => {

        //创建渲染器
        const renderer = new Three.WebGLRenderer({ canvas: canvasRef.current as HTMLCanvasElement })
        rendererRef.current = renderer

        //创建镜头
        const camera = new Three.PerspectiveCamera(40, 2, 0.1, 1000)
        camera.position.set(0, 50, 0)
        camera.up.set(0, 0, 1)
        camera.lookAt(0, 0, 0)
        cameraRef.current = camera

        //创建场景
        const scene = new Three.Scene()
        scene.background = new Three.Color(0x111111)
        sceneRef.current = scene

        //将太阳系、灯光添加到场景中
        scene.add(solarSystem)
        scene.add(pointLight)

        //创建循环渲染的动画
        const render = (time: number) => {
            time = time * 0.001
            nodeArr.forEach((item) => {
                item.rotation.y = time
            })
            renderer.render(scene, camera)
            window.requestAnimationFrame(render)
        }
        window.requestAnimationFrame(render)

        //添加窗口尺寸变化的监听
        const resizeHandle = () => {
            const canvas = renderer.domElement
            camera.aspect = canvas.clientWidth / canvas.clientHeight
            camera.updateProjectionMatrix()
            renderer.setSize(canvas.clientWidth, canvas.clientHeight, false)
        }
        resizeHandle()
        window.addEventListener('resize', resizeHandle)

        return () => {
            window.removeEventListener('resize', resizeHandle)
        }
    }, [canvasRef])

    return (
        <canvas ref={canvasRef} className='full-screen' />
    )
}

export default HelloScene
```



#### 上述代码共同构建出的空间体系：

1. 主场景 Scene 包含 太阳系
2. 太阳系：太阳系本身 + 太阳  + 地球系(含月球系)
3. 地球系：地球系本身 + 地球 + 月球系
4. 月球系：月球系本身 + 月球

**每一个空间体系都是相互独立运作，但在他们共同作用下，构成了一个复杂的空间体系。**



> 思考题：如何实现一辆简单的，有 4 个滚动轮子的汽车？



## 补充一个类：AxesHelper

在传统的 3D 制作软件中，都会直观的显示出 X、Y、Z 网格线，帮助我们比较直观的查看 物体所在网格的位置。

在 Three.js 中，可以通过给空间网格添加 AxesHeler 实例来让渲染的时候，显示出 XYZ 网格。



**具体用法：请将以下代码，添加到本文的示例代码中**

```
useEffect(() => {

        ...

        //显示轴线
        nodeArr.forEach((item) => {
            const axes = new Three.AxesHelper()
            const material = axes.material as Three.Material
            material.depthTest = false
            axes.renderOrder = 1 // renderOrder 的该值默认为 0，这里设置为 1 ，目的是为了提高优先级，避免被物体本身给遮盖住
            item.add(axes)
        })
        
        ...
        
}, [canvasRef])
```



关于 Three.js 中 场景、空间 的概念和基本用法，先讲解到这里。在后续稍微复杂点的项目中，都会有大量 空间 相互嵌套 的使用需求。

**空间的相互嵌套才构建出了复杂的 3D 场景。**

学习到本篇，是否有些心累的？感觉贴出来的示例代码越来越长，越来越复杂了？ 打起精神，继续加油吧。



下一节，开始讲一下 决定物体外观被渲染成什么样子的 “材质” 。