# 11 Three.js基础之灯光

## 灯光的种类

在场景中添加灯光后，灯光照射在物体上产生明暗、光亮和阴影，从而让物体显得更加立体有光泽。

> 在有些文档或教程中，会把 灯光 称呼为 光源，这只是对 light 这个单词的不同翻译而已  
> 如果我在本系列文章中，有时候使用 “灯光”，有时候使用 “光源”，请勿见怪。

> 向场景中添加灯光 这个行为，在传统 3D 软件制作中通常被称为 “打灯”



**若场景中的物体由非反光材质构成，即使场景中没有任何光源，渲染后依然可以看见该物体。**

> 非反光材质为 MeshBasicMaterial

**若场景中的物体由反光材质构成，假设场景中没有任何光源，渲染出的结果将是一片漆黑，什么物体都看不见。**

> 反光材质为 MeshPhongMaterial 等



### 灯光的种类

**在 Three.js 中，有 6 种基础类型的灯光，他们都继承于 Three.Light 。**

| 灯光类型(都继承于Light) | 灯光名称           | 是否支持阴影 | 是否作用于全局(无处不在) | 是否有照射目标 |
| ----------------------- | ------------------ | ------------ | ------------------------ | -------------- |
| AmbientLight            | 环境光、氛围光     | 否           | 是                       | 无             |
| DirectionalLight        | 平行光             | 是           | 否                       | 有             |
| HemisphereLight         | 半球光源、户外光源 | 否           | 是                       | 无             |
| PointLight              | 点光源             | 是           | 否                       | 有             |
| RectAreaLight           | 矩形面光源         | 否           | 否                       | 无             |
| SpotLight               | 聚光灯光源         | 是           | 否                       | 有             |



**补充说明1：环境光**

有个别文档或教程中，会把 HemisphereLight 也称呼为 “环境光”。  
事实上 AmbientLight 和 HemisphereLight 的作用都是提供环境光，只是 HemisphereLight 的环境光更加真实，当然渲染所需性能也更多。



**补充说明2：是否支持阴影**

**所有的光照射到物体上后，都会产生阴影。**但 这里说的不是 **是否会产生阴影**，而是说 **“是否支持阴影”**。

一共有 3 种光不支持阴影：AmbientLight、HemisphereLight、RectAreaLight

其他种类的光，都支持阴影：DirectionalLight、PointLight、SpotLight

关于 Three.js 中的 阴影 (LightShadown、DirectionalLightShadown、PointLightShadown、SpotLightShadown)，我们会在后面单独开辟一节来讲解。



**关于“阴影”的进一步补充：**

在之前所有的示例中，当场景上有反光物体且有灯光时，物体会产生明暗，但是请注意：

1. 这个“物体显示出的明暗”并不是真正的“阴影”。

   > 在 Three.js 中 有真正的阴影对象，这个会在后面的 “Three.js基础之阴影”  一文中有详细说明。

2. 这个“物体显示出的明暗”并不是完全符合我们日常的“光影明暗”。

   > 这是因为我们目前所有示例都使用的是“简单光照模型”，也就是说光照射在物体上后并不进行漫反射，所以渲染出的“明暗”并不完全自然合理。

3. 默认渲染器并不会渲染阴影、默认支持阴影的灯光也不会投射阴影，若想产生真正的阴影，还需开启阴影渲染和投射。

   > 具体如何开启，也会在后面的 “Three.js基础之阴影”  一文中有详细说明。



**补充说明3：是否作用域全局**

只有环境光(AmbientLight、HemisphereLight)作用于全局，其他光则照耀范围都是有限的。



**补充说明4：什么叫 “是否有照射目标？”** 

答：就是这个光除了光源本身之外，还包含一个 target 属性，并且可以通过设置 target.position 的位置。对于有照射目标的灯光，在场景中不光要添加灯光本身，还可以添加 灯光照射目标。

> 注意是 “可以添加” 灯光目标，而不是说 “必须也要添加” 灯光目标。

请注意上面表格中，关于 “是否产生阴影” 和 “是否有照射目标”，这 2 项 是完全相同的，也就是说，只有包含照射目标的光，才会产生阴影。



**在 Three.js 中，有 3 种 光探针 类型的环境光。**

| 灯光类型                               | 灯光名称   |
| -------------------------------------- | ---------- |
| LightProbe                             | 光探针     |
| AmbientLightProbe(继承于LightProbe)    | 环境光探针 |
| HemisphereLightProbe(继承于LightProbe) | 半球光探针 |

关于 光探针，是另外一套比较复杂的光的算法方式。

根据我网上搜索到的一些信息，大概可以描述为：

**传统的 环境光(AmbientLight、HemisphereLight) 渲染时需要的计算量比较大，对于渲染静止物体来说还可以，但是渲染  运动类型 的物体时所消耗的性能过高，而 光探针类型的环境光(AmbientLightProbe、HemisphereLightProbe) 则更加适合 运动类型 的物体。**



### 光的辅助对象

场景中的光本身是不可见的，为了让我们方便观测光源，Three.js 提供了 光的辅助对象：DirectionalLightHelper、HemisphereLightHelper、PointLightHelper、RectAreaLightHelper、SpotLightHelper

**所谓光的辅助对象，就是在渲染后出现的一些白色细线，这些白色细线指示出光源的位置、大小、以及光发射的方向。**

光的辅助对象用法非常简单，3 步骤：

1. 先创建 光 的实例
2. 将创建好的光实例作为 辅助对象构造函数的参数
3. 场景中添加 辅助对象即可

例如：

```
//创建并设置平行光
const directionalLight = new Three.DirectionalLight(0xFFFFFF, 1)
directionalLight.position.set(0, 10, 0);
directionalLight.target.position.set(-5, 0, 0)

//将平行光添加到场景中
scene.add(directionalLight)
scene.add(directionalLight.target)

//根据平行光实例，创建对应的辅助对象，并将辅助对象添加到场景中
const directionalLightHelper = new Three.DirectionalLightHelper(directionalLight)
scene.add(directionalLightHelper)
```



### 灯光作用与特点

**AmbientLight(环境光、氛围光)：**  

通常仅作为基础光线，一般需要与其他灯光配合使用。不能产生阴影、也无需指定坐标位置，仅需设置颜色和强度。

注意：不支持阴影



**DirectionalLight(平行光)：**

最为经常使用的光源，光纤(光芒)都是平行向着一个方向发射。

经常使用 DirectionalLight 来模拟太阳光照射到某个物体上的光照效果。



**HemisphereLight(半球光源)：**

相对 ambientLight，hemisphereLight 更加真实的模拟自然光源，提供 天空 和 地面 漫反射光线。

一共接收 3 个参数：

* 第 1 个参数：天空光线颜色
* 第 2 个参数：地面反射光颜色
* 第 3 个参数：光的反射强度

注意：不支持阴影



**PointLight(点光源)：**

类似生活中的灯泡，光纤(光芒)没有固定方向，朝着四周散射。

注意：点光源对应的辅助对象 PointLightHelper 只有一个 菱形的光源形状，并没有 光 的发射线条。



**RectAreaLight(矩形面光源)：**

与 DirectionalLight 模拟太阳光不同，RectAreaLight 光源形状为一个矩形，可以模拟出明亮的窗口或矩形照明光源。

注意：

1. 不支持阴影

2. 只有 MeshStandardMaterial 和 MeshPhysicalMaterial 材料才支持 RectAreaLight 光源

3. 按照官方文档描述，场景中必须加入 RectAreaLightUniformsLib.init()

   > 目前我比较疑惑的是，经过试验发现，即使不添加 RectAreaLightUniformsLib.init()，场景依然正常渲染，似乎看不出有任何差别

**特别说明：**

RectAreaLight 对应的 辅助对象 RectAreaLightArea 引入方式和其他 光辅助对象 引入方式不同。

其他光辅助对象都是内置在 three 中的，使用之前无需引入，可以直接使用，例如：

```
const directionalLightHelper = new Three.DirectionalLightHelper(directionalLight)
```

但是 RectAreaLightHelper 在使用前需要引入才可以，引入代码：

```
import { RectAreaLightHelper } from 'three/examples/jsm/helpers/RectAreaLightHelper'
```



**SpotLight(聚光灯)：**

类似生活中的聚光灯效果。



关于每个灯光的具体参数详情、属性用法，请参考官方文档：https://threejs.org/docs/index.html#api/zh/lights/Light



接下来，我们将通过创建一个 HelloLight 的例子，直观的观察不同类型灯光的效果。

> 这个示例中，会用到我们上节学习的 纹理 相关知识。



## 前期准备：OrbitControls讲解、制作纹理图片

### OrbitControls 的简介和用法

本示例中需要用到一个之前从未使用过的类：OrbitControls，先简单介绍一下这个类。

1. orbit 单词的翻译为：轨道
2. controls 单词的翻译为：控制权

OrbitControls 是鼠标镜头轨道控件，可以通过鼠标来配置镜头的运动轨道，例如 缩放、平移、旋转。也就是说在不修改场景的前提下，可以通过鼠标来改变镜头，以便查看不同角度下的场景。

> 在手机端，不是鼠标，而是手指滑动



#### OrbitControls的用法

```
const controls = new OrbitControls(camera, canvas) //创建一个实例
controls.target.set(0, 5, 0) //controls.target 为镜头的坐标系统
//controls.target.set(0, 5, 0) 的意思是：设置原点 Y 轴的坐标(以高出5米的轨道运行)
controls.update() //使控件使用新目标
```

请注意，在上面代码中，OrbitControls 的构造函数中第 2 个参数为DOM 中的 canvas 节点，实际上当添加过 OrbitControls 之后，鼠标在 canvas 上的 拖拽、鼠标滚轴滚动 等操作都会被捕捉到，并且做出相对应的镜头画面切换。

说直白点，我们终于可以通过鼠标对 3D 场景进行不同角度，距离的切换操作了。



**OrbitControls的change事件：**

无论是通过 鼠标或键盘 来修改镜头轨道 都会触发 OrbitControls 的 change 事件。

我们可以通过添加 事件监听 来捕获该事件：

```
const handleChange = () => { ... }

const controls = new OrbitControls(camera, canvasRef.current)
controls.addEventListener('change',handleChange)
```

对于目前的我们来说，是没有必要使用该事件的，在后续的 Three.js 技巧篇 中，我们才会运用到 change 事件。



除此之外，我们还可以设置 禁止缩放、禁止旋转、禁止右键拖拽、设置可旋转角度范围等等一系列配置，具体的可查阅官方文档：https://threejs.org/docs/#examples/zh/controls/OrbitControls



**特别注意：**

OrbitControls 并不是包含在 three 根目录下，而是位于 three/examples/jsm/controls/OrbitControls 中，因此引入代码为：

```
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
```

> 提醒：在 OrbitControls.js 中，分别导出有 OrbitControls 和 MapControls，所以引入 OribitControls 是需要加 大括号 { }。



**补充说明：**

严格意义上讲，OrbitControls 并不是 Three.js 核心包含的代码内容，OrbitControls 是将最常见的鼠标与 Three.js 场景交互内容的一个额外封装。

由于 OrbitControls 实在是过于频繁使用，最终 Three.js 将 OrbitControls 也包含到了 Three.js 代码包中，只不过不在默认的目录中，而是在 Three.js 示例目录中。

> 在 Three.js 早期的版本中，代码包中并未包含 OrbitControls，若想使用还需要 yarn 安装 three-orbitcontrols 这个包。只不过当后来 Three.js 包含了 OrbitControls 之后，才再也无需额外安装了。



**补充说明2：**

原本 OrbitControl 除了鼠标拖拽可以改变场景视角，还支持键盘上的 4 个方向键来改变场景视角。

只不过 React 对于原生 DOM 事件支持度并不高，React 更倾向于给组件添加 onKeydown 属性处理函数。

> 本质上 组件的 onKeydown 相当于 React 的合成事件。

Three.js 的官方示例使用的是原生 html + js，是完全支持原生键盘事件的。所以 官网的示例 使用键盘方向键控制场景没有问题，但是在 React 项目中却不太容易实现。

在 React 中如果想让 canvas 拥有键盘事件监听，需要做以下 2 处设置：

1. 在 useEffect 中，当第一次挂载完成，添加 canvasRef.current.fouce()，让 canvas 自动获得焦点
2. 在 <canvas /\> 中添加 tabindex 属性，属性值为 -1、0、1 都无所谓，例如：<canvas tabindex={0} /\>

只有满足以上 2 个条件后，canvas 才会监听到键盘事件，但是一旦 canvas 失去焦点，那么就又监听不到了。

> 还是继续使用 鼠标拖拽 来修改查看场景视角吧



**补充说明3：**

事实上如果你真的需要监听 鼠标键盘方向键 ，其实最简单的办法就是把 OrbitControls 监听对象修改为 document.body 上。

```diff
- const controls = new OrbitControls(camera, canvasRef.current)
+ const controls = new OrbitControls(camera, document.body)
```





#### 纹理图片准备：制作纹理图片 checker.png

本示例中需要用到一个类似 3D 场景地面黑白网格的纹理，因此我们需要提前准备好这个纹理图片。

在 PhotoShop 中新建一个 宽高都为 2 像素的画布，然后：

1. 左上角和右下角 的那 1 像素中填充一个较黑的颜色
2. 右上角和左下角 的那 1 像素中填充一个较白的颜色
3. 将图片导出为 checker.png，并保存到 src/assets/imgs/ 目录中

**补充说明：**

虽然制作的纹理图片非常小，只有 2像素 * 2像素，但是我们可以通过设置纹理的重复，来实现渲染出比较大的 黑白网格底盘。

1. 设置纹理 magFilter 的属性值为 Three.NearestFilter
2. 设置纹理 wrapS、wrapT 属性值为 Three.RepeatWrapping
3. 根据 黑白网格的尺寸，计算并设置纹理 repeat 重复次数

以上 3 点刚好都是上一节我们讲解 纹理 时学习到的知识点。



## 灯光示例：HelloLight

先回顾一下 Three.js 中 10 个 光 的类型：

光的原型(Light) + 6种基础光(AmbientLight...) + 光探针原型(LightProbe) + 2种环境光探针(AmbientLightProbe、HemisphereLightProbe) = 1 + 6 + 1 + 2 = 10

### 示例目标：

1. 使用并体验 Three.js 中除 Light 和 LightProbe 之外的其他 8 种光类型
2. 使用并体验 光的辅助对象( XxxLightHelper)

**补充说明：**

1. 环境光 AmbientLight 是没有 辅助对象的、其他光都有辅助对象
2. 矩形光 RectAreaLight 的辅助对象 RectAreaLightHelper 和其他光的辅助对象 引入方式不同



### 代码拆分与梳理：

#### 1、create-scene.ts：创建基础场景

创建 src/components/hello-light/create-scene.ts ，导出一个名为 createScene 的函数，用来专门负责创建基础的场景。

**具体代码梳理：**

1. 该场景中包含 1个黑白网格的地面、1个立方体、1个球体，但是该场景不包含任何光。

2. createScene 创建基础场景时接收一个参数 type，type 只能为以下 2 个值中的其中 1 种：MESH_PHONE_MATERIAL 或 MESH_STANDARD_MATERIAL
3. type 默认值为 MESH_PHONE_MATERIAL

**进一步解释：**

由于 RectAreaLight 只作用在 MeshStandardMaterial 和 MeshPhysicalMaterial 材料物体上，所以我们才设置 type 这个参数。

1. 当使用 RectAreaLight 时，我们告知 createScene，使用 MeshStandardMaterial 材质创建 地面、立方体、球体
2. 当使用 其他 光时，我们告知 createScene，使用 MeshPhongMaterial 材质创建 地面、立方体、球体



#### 2、index.tsx：创建渲染器、镜头、以及不同种类的光

**具体代码梳理：**

1. 当 canvas DOM 初始化后，创建 渲染器、镜头、镜头交互(OrbitControls)
2. 创建 8 个按钮，每个按钮对应一种光
3. 使用 useState 创建一个变量 type，用来记录当前演示 光的类型
4. 点击不同按钮后，修改 当前光的类型 type 的值，从而引发 react 重新渲染
5. 在新一轮的渲染中，通过判断 type 类型，使用 createScene 创建一个新的场景 和 对应的 光



#### 3、index.scss：设置相关样式

**具体样式梳理：**

1. 设置 canvas 对应的样式
2. 设置 8 个按钮对应的样式



### 具体的代码：

#### create-scene.ts：

```
import * as Three from 'three'

export enum MaterialType {
    MESH_PHONE_MATERIAL = 'MESH_PHONE_MATERIAL',
    MESH_STANDARD_MATERIAL = 'MESH_STANDARD_MATERIAL'
}

const createScene: (type?: keyof typeof MaterialType) => Three.Scene = (type = MaterialType.MESH_PHONE_MATERIAL) => {

    const scene = new Three.Scene()

    const planeSize = 40

    const loader = new Three.TextureLoader()
    const texture = loader.load(require('@/assets/imgs/checker.png').default)
    texture.wrapS = Three.RepeatWrapping
    texture.wrapT = Three.RepeatWrapping
    texture.magFilter = Three.NearestFilter
    texture.repeat.set(planeSize / 2, planeSize / 2)

    let planeMat: Three.Material
    let cubeMat: Three.Material
    let sphereMat: Three.Material
    switch (type) {
        case MaterialType.MESH_STANDARD_MATERIAL:
            planeMat = new Three.MeshStandardMaterial({
                map: texture,
                side: Three.DoubleSide
            })
            cubeMat = new Three.MeshStandardMaterial({ color: '#8AC' })
            sphereMat = new Three.MeshStandardMaterial({ color: '#CA8' })
            break
        default:
            planeMat = new Three.MeshPhongMaterial({
                map: texture,
                side: Three.DoubleSide
            })
            cubeMat = new Three.MeshPhongMaterial({ color: '#8AC' })
            sphereMat = new Three.MeshPhongMaterial({ color: '#8AC' })
    }

    const planeGeo = new Three.PlaneBufferGeometry(planeSize, planeSize)
    const mesh = new Three.Mesh(planeGeo, planeMat)
    mesh.rotation.x = Math.PI * -0.5
    scene.add(mesh)

    const cubeGeo = new Three.BoxBufferGeometry(4, 4, 4)
    const cubeMesh = new Three.Mesh(cubeGeo, cubeMat)
    cubeMesh.position.set(5, 2.5, 0)
    scene.add(cubeMesh)

    const sphereGeo = new Three.SphereBufferGeometry(3, 32, 16)
    const sphereMesh = new Three.Mesh(sphereGeo, sphereMat)
    sphereMesh.position.set(-4, 5, 0)
    scene.add(sphereMesh)

    return scene
}

export default createScene
```



#### index.tsx：

```
import { useEffect, useRef, useState } from 'react'
import * as Three from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
//import { RectAreaLightUniformsLib } from 'three/examples/jsm/lights/RectAreaLightUniformsLib'
import { RectAreaLightHelper } from 'three/examples/jsm/helpers/RectAreaLightHelper'

import createScene, { MaterialType } from './create-scene'

import './index.scss'


enum LightType {
    AmbientLight = 'AmbientLight',
    AmbientLightProbe = 'AmbientLightProbe',
    DirectionalLight = 'DirectionalLight',
    HemisphereLight = 'HemisphereLight',
    HemisphereLightProbe = 'HemisphereLightProbe',
    PointLight = 'PointLight',
    RectAreaLight = 'RectAreaLight',
    SpotLight = 'SpotLight'
}

const buttonLables = [LightType.AmbientLight, LightType.AmbientLightProbe, LightType.DirectionalLight,
LightType.HemisphereLight, LightType.HemisphereLightProbe, LightType.PointLight,
LightType.RectAreaLight, LightType.SpotLight]

const HelloLight = () => {

    const canvasRef = useRef<HTMLCanvasElement>(null)
    const sceneRef = useRef<Three.Scene | null>(null)

    const [type, setType] = useState<LightType>(LightType.AmbientLight)

    useEffect(() => {

        if (canvasRef.current === null) {
            return
        }

        const renderer = new Three.WebGLRenderer({ canvas: canvasRef.current as HTMLCanvasElement })

        const camera = new Three.PerspectiveCamera(45, 2, 0.1, 1000)
        camera.position.set(0, 10, 20)

        const controls = new OrbitControls(camera, canvasRef.current)
        controls.target.set(0, 5, 0)
        controls.update()

        const scene = createScene()
        sceneRef.current = scene

        const render = () => {
            if (sceneRef.current) {
                renderer.render(sceneRef.current, camera)
            }
            window.requestAnimationFrame(render)
        }
        window.requestAnimationFrame(render)

        const handleResize = () => {
            const canvas = canvasRef.current
            if (canvas === null) {
                return
            }
            camera.aspect = canvas.clientWidth / canvas.clientHeight
            camera.updateProjectionMatrix()
            renderer.setSize(canvas.clientWidth, canvas.clientHeight, false)
        }
        handleResize()
        window.addEventListener('resize', handleResize)

        return () => {
            window.removeEventListener('resize', handleResize)
        }
    }, [canvasRef])

    useEffect(() => {
        if (sceneRef.current === null) {
            return
        }

        sceneRef.current = null

        let newScene: Three.Scene
        if (type === LightType.RectAreaLight) {
            newScene = createScene(MaterialType.MESH_STANDARD_MATERIAL)
        } else {
            newScene = createScene()
        }
        sceneRef.current = newScene

        switch (type) {
            case LightType.AmbientLight:
                const ambientLight = new Three.AmbientLight(0xFFFFFF, 1)
                newScene.add(ambientLight)
                break
            case LightType.AmbientLightProbe:
                const ambientLightProbe = new Three.AmbientLightProbe(0xFFFFFF, 1)
                newScene.add(ambientLightProbe)
                break
            case LightType.DirectionalLight:
                const directionalLight = new Three.DirectionalLight(0xFFFFFF, 1)
                directionalLight.position.set(0, 10, 0);
                directionalLight.target.position.set(-5, 0, 0)
                newScene.add(directionalLight)
                newScene.add(directionalLight.target)

                const directionalLightHelper = new Three.DirectionalLightHelper(directionalLight)
                newScene.add(directionalLightHelper)
                break
            case LightType.HemisphereLight:
                const hemisphereLight = new Three.HemisphereLight(0xB1E1FF, 0xB97A20, 1)
                newScene.add(hemisphereLight)

                const hemisphereLightHelper = new Three.HemisphereLightHelper(hemisphereLight,5)
                newScene.add(hemisphereLightHelper)

                break
            case LightType.HemisphereLightProbe:
                const hemisphereLightProbe = new Three.HemisphereLightProbe(0xB1E1FF, 0xB97A20, 1)
                newScene.add(hemisphereLightProbe)
                break
            case LightType.PointLight:
                const pointLight = new Three.PointLight(0xFFFFFF, 1)
                pointLight.position.set(0, 10, 0)
                newScene.add(pointLight)

                const pointLightHelper = new Three.PointLightHelper(pointLight)
                newScene.add(pointLightHelper)
                break;
            case LightType.RectAreaLight:
                //RectAreaLightUniformsLib.init() //实际测试时发现即使不添加这行代码，场景似乎也依然正常渲染，没有看出差异

                const rectAreaLight = new Three.RectAreaLight(0xFFFFFF, 5, 12, 4)
                rectAreaLight.position.set(0, 10, 0)
                rectAreaLight.rotation.x = Three.MathUtils.degToRad(-90)
                newScene.add(rectAreaLight)

                const rectAreaLightHelper = new RectAreaLightHelper(rectAreaLight)
                newScene.add(rectAreaLightHelper)
                break
            case LightType.SpotLight:
                const spotLight = new Three.SpotLight(0xFFFFFF, 1)
                spotLight.position.set(0, 10, 0);
                spotLight.target.position.set(-5, 0, 0)
                newScene.add(spotLight)
                newScene.add(spotLight.target)

                const spotLightHelper = new Three.SpotLightHelper(spotLight)
                newScene.add(spotLightHelper)
                break
            default:
                console.log('???')
                break
        }
    }, [type])

    return (
        <div className='full-screen'>
            <div className='buttons'>
                {
                    buttonLables.map((label, index) => {
                        return <button
                            className={label === type ? 'button-selected' : ''}
                            onClick={() => { setType(label) }}
                            key={`button${index}`}
                        >{label}</button>
                    })
                }
            </div>
            <canvas ref={canvasRef} />
        </div>

    )

}

export default HelloLight
```

> create-scene.ts 和 index.tsx 的代码非常多，并且我也没有写注释。  
> 但是如果之前章节中的示例你也都跟着 敲一遍，应该很容易看懂 这两个文件里的代码。



#### index.scss：

```
.full-screen, canvas {
    display: block;
    height: inherit;
    width: inherit;
}

.buttons {
    display: flex;
    justify-content: center;
    width: 100%;
    position: fixed;
    top: 30px;
}

.buttons button {
    width: 200px;
    height: 30px;
    margin-left: 20px;
    font-size: 18px;
    cursor: pointer;
}

.buttons button:first-child {
    margin-left: 0;
}

.button-selected{
    background-color:green;
    color: white;
}
```



若一切正常，实际运行后：

1. 点击网页中不同顶部的按钮，可以切换不同光对应的场景效果
2. 点击并拖动鼠标 或 滚动鼠标滚轴，可以切换场景视角



### 遗留的疑惑

#### 第1个疑惑：RectAreaLightUniformsLib

按照官方文档的说法，在使用 RectAreaLight 时，必须要执行 RectAreaLightUniformsLib.init() 的，但实际试验发现不执行这行代码也没有任何问题。

**为什么会这样？**



#### 第2个疑惑：RectAreaLightHelper

实际运行发现，RectAreaLightHelper 所展示的光源的位置和方向与实际不符。

矩形面灯光的代码为：

```
const rectAreaLight = new Three.RectAreaLight(0xFFFFFF, 5, 12, 4)
rectAreaLight.position.set(0, 10, 0)
rectAreaLight.rotation.x = Three.MathUtils.degToRad(-90)
newScene.add(rectAreaLight)

const rectAreaLightHelper = new RectAreaLightHelper(rectAreaLight)
newScene.add(rectAreaLightHelper)
```

我们对 rectAreaLight 进行了 position(位置)、rotation(旋转) 设置，**难道 对应的辅助对象并不会跟随同步变化？**



## 额外的一些唠叨话

我是一边学习 Three.js 官方教程，一边写本系列文章。

本片文章的源头，对应的原始教程是：https://threejsfundamentals.org/threejs/lessons/threejs-lights.html

因为之前刚好学过 场景，按照当时编写的示例代码：

1. 月球围绕地球
2. 地球围绕太阳
3. 太阳自转

我当时得出了一个结论：每一个 scene 实例都是一个相对独立的 场景空间，**我想当然得认为这里面的独立也包含 场景中的 光**。

所以最初我想实现 光 示例时这样的：

1. 一个主场景
2. 主场景内，2 行 3 列 分布着 6 个子场景
3. 这 6 个子场景里，分别包含这 6 中基础光源

按照这个目标，我编写了代码，结果渲染后的场景画面，**完全不是我预期的，实际结果是：6 个子场景上的灯光，完全混合在一起，并不是相互独立的**。



**当时我完全懵的状态。**

经过 1 天的迷惑，查资料、QQ交流群里询问其他人，直至我最终查阅了 scene.add() 函数源码，才明白过来，我那个结论是错误的。

```
sceneB.add(lightB)
sceneA.add(sceneB)
```

以上代码最终真正的执行，会将 lightB 添加到 sceneA 中。

尤其假设 lightB 光的类型为 环境光，而环境光是无处不在的，那么 lightB 会影响到所有 sceneA 中的物体。

这也就解释了为什么 “6 个子场景上的灯光，完全混合在一起，并不是相互独立的”。



或许你会疑惑？你说的这些不都已经在 [Three.js基础之场景.md](https://github.com/puxiao/threejs-tutorial/blob/main/08%20Three.js%E5%9F%BA%E7%A1%80%E4%B9%8B%E5%9C%BA%E6%99%AF.md) 一节中讲过了，怎么又说了一遍？

事实是我在进一步理解 场景、光 之后，又重新修改编辑了 之前章节的错误观点，所以你看到的时候都才是正确的。

我是一边学习，一边有新的知识领悟，然后再不断回头修改、补充之前文章中的相关知识点。



**我唠叨的这些目的，其实想表达 2 个事情：**

1. 学习 Three.js 的 类、函数、方法、属性时候，最好去看一下 three.js 的源码，绝对会加深你的 Three.js 理解和功力。
2. 学习 Three.js 真的挺难，需要不断打破已有认知，若有些地方暂时无法理解也没关系，只要继续加油，终会搞明白的。



唠叨结束。



本文学习了 光(Light)，按道理接下来应该学习 阴影(LightShadown)，但是下节我们先学习 镜头(Camera)，学完之后再回过头学习 阴影。



> 明天是2020年12月19日，阿里巴巴 前端 D2 技术分享大会开幕，19号、20号 为期 2 天的前端技术直播会议分享。

> 花 98 元买的直播观看门票，不能浪费了，所以，换换脑子，未来 2 天暂停 Three.js 的学习。

