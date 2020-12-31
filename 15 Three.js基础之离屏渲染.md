# 15 Three.js基础之离屏渲染

## Render Targets(离屏渲染)简介

#### 名词解释：Render Targets

**从字面上直接翻译，“Render Targets” 应该翻译为 “渲染目标”。**

**从实际作用上翻译，”Render Targets“ 应该翻译为 ”离屏渲染目标 或 离屏渲染对象“。**

> 国内绝大多数 Three.js 教程都把 Render Targets 翻译为 离屏渲染。
>
> 我个人认为翻译成：离屏渲染目标 更为合适，但有时我会不自觉使用 离线渲染对象 这个词，所以你只需要明白 虽然称呼不同，但指向的都是同一个东西。



#### 离屏渲染概念解释

首先，我们先说一下 普通的“渲染”。

之前示例中我们使用的渲染器都是 WebGLRenderer。我们创建的渲染器实例 renderer 会根据 场景(含场景中的物体)、灯光 来将视觉结果渲染到网页中。

此时的 渲染 就是普通的渲染，渲染结果直接出现在网页中。



**那什么又是离屏渲染？**

答：渲染器会渲染场景，但是不会吧渲染结果直接呈现在网页中，而是把渲染结果保存到 GPU 内部中。

此时 暂存到 GPU 中的渲染结果(图片)，可以被当做一种纹理(texture)，使用到其他物体中。

离屏渲染 和 正常渲染 整个计算过程完全相同，不同的地方在于 离屏渲染 的结果是保存在 GPU 内存中，而非直接显示在网页中。



#### 离屏渲染的种类

在 Three.js 中，一共有 3 种离屏渲染类型。

| 离屏渲染类型                 | 名称及解释                 |
| ---------------------------- | -------------------------- |
| WebGLMultisampleRenderTarget | WebGL 2 对应的离屏渲染     |
| WebGLRenderTarget            | WebGLRender 对应的离屏渲染 |
| WebGLCubeRenderTarget        | CubeCamera 对应的离屏渲染  |



#### 离屏渲染的用途

假设有这样一个场景：场景中有 3 个不同颜色、不停转动的立方体。

我们之前示例中会使用 WebGLRenderer ，把这个场景画面内容渲染到网页中，这属于正常的渲染。

若我们现在改变需求，我们希望修改成：

1. 场景中有一面镜子
2. 在镜子中显示出 3 个不同颜色、不同旋转的立方体
3. 场景本身当中，是看不见这 3 个立方体的

为了实现这个需求，我们此时就需要用到 离屏渲染。

具体做法是：

1. 创建一个子场景，该子场景中 有 3 个不同颜色、不停旋转的立方体

2. 创建一个总场景、一个渲染器，一面镜子

3. 使用总场景的渲染器，对子场景进行渲染，得到一个离屏渲染结果(图像纹理)

   > 注意：由于是离屏渲染，只是将 3 个立方体渲染出的视觉效果保存到 GPU 内存中，网页中并不会显示出离屏渲染结果

4. 将离屏渲染结果作为一个纹理，作用在镜子面上

5. 使用总场景的渲染器，将镜子渲染到网页中

   > 至此，完成我们的目标。



**再试想另外一个应用场景：**

一辆汽车，汽车的倒车镜中可以显示出汽车后面的场景，这也需要用到 离线渲染。



## 离屏渲染示例：HelloRenderTarget

#### 示例目标：

1. 创建一个 “子场景”，子场景中有 光、镜头、3 个不同颜色的立方体
2. 创建一个 “总场景”，总场景中有 光、镜头、1 个平面圆(镜子)、1个立方体
3. 在 总场景中，控制子场景中的 3 个立方体，让他们不停旋转
4. 通过离屏渲染，将子场景中的 “景象” 作为图片纹理，作用在镜子和立方体的6个面上



#### 代码思路：

**场景搭建：**

子场景和总场景的创建过程，比较简单，不再过多讲述。

由于子场景和总场景中都有 镜头、光、立方体这些，为了方便我们区分，也为了让我们代码更加简单清晰，所以我们会单独创建一个 scr/components/hello-render-target/render-target-scene.ts 的文件，用来创建子场景。

> 注意：子场景中不需要创建渲染器

> 我们说的 总场景，其实就是 HelloRenderTarget 组件本身



子场景需要对外暴露出 场景、立方体、镜头：

```
export default {
    scene,
    boxs,
    camera
}
```



总场景获取子场景中的关键元素：

```
import * as RTScene from './render-target-scene'
...
const rtScene = RTScene.default.scene
const rtBoxs = RTScene.default.boxs
const rtCamera = RTScene.default.camera
```



**创建离屏渲染对象**

```
const rendererTarget = new Three.WebGLRenderTarget(512, 512)
```

> 请注意，上面代码中设置 离屏渲染对象的尺寸为 宽 512 像素、高 512 像素



**创建材质，并将材质纹理与离屏渲染对象的渲染结果纹理进行绑定**

```
const material = new Three.MeshPhongMaterial({
    map: rendererTarget.texture
})
```

> 由于离屏渲染对象的渲染出的纹理尺寸为 512 X 512，这样意味着我们应该将子场景中的镜头宽高比(aspect) 设置为 1

> 同样，也意味着我们将来 总场景中的 物体(镜子和立方体) 渲染的面 宽高比 也应该是 1:1。
>
> 若物体渲染的面 宽高比不是 1:1，那么最终渲染出的面上的图片会变形。



**修改渲染器的渲染目标，让渲染器去渲染离屏渲染对象，当渲染完成后再清除(恢复)渲染器的渲染目标**

```
renderer.setRenderTarget(rendererTarget)
renderer.render(rtScene, rtCamera)
renderer.setRenderTarget(null)
```

> 虽然渲染器的渲染目标最终又被设置为 null，但是 离屏渲染的画面我们已经获得并保存在 rendererTarget 中。



**最终，在使用渲染器把镜子和立方体进行渲染输出**

```
renderer.render(scene, camera)
```

> 至此，整个代码完成



#### 示例代码：

**子场景：**

文件位于 scr/components/hello-render-target/render-target-scene.ts

```
import * as Three from 'three'

const scene = new Three.Scene()
scene.background = new Three.Color(0x00FFFF)

const camera = new Three.PerspectiveCamera(45, 1, 0.1, 10)
camera.position.z = 10

const light = new Three.DirectionalLight(0xFFFFFF, 1)
light.position.set(0, 10, 10)
scene.add(light)

const colors = ['blue', 'red', 'green']
const boxs: Three.Mesh[] = []

colors.forEach((color, index) => {
    const mat = new Three.MeshPhongMaterial({ color })
    const geo = new Three.BoxBufferGeometry(2, 2, 2)
    const mesh = new Three.Mesh(geo, mat)
    mesh.position.x = (index - 1) * 3
    scene.add(mesh)
    boxs.push(mesh)
})

export default {
    scene,
    boxs,
    camera
}
```



**总场景(HelloRenderTarget)：**

文件位于 scr/components/hello-render-target/index.tsx

```
import { useEffect, useRef } from 'react'
import * as Three from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import * as RTScene from './render-target-scene'

import './index.scss'

const HelloRenderTarget = () => {

    const canvasRef = useRef<HTMLCanvasElement>(null)

    useEffect(() => {
        if (canvasRef.current === null) {
            return
        }

        const rtScene = RTScene.default.scene
        const rtBoxs = RTScene.default.boxs
        const rtCamera = RTScene.default.camera

        const renderer = new Three.WebGLRenderer({ canvas: canvasRef.current })
        const rendererTarget = new Three.WebGLRenderTarget(512, 512)

        const scene = new Three.Scene()
        scene.background = new Three.Color(0x333333)

        const light = new Three.DirectionalLight(0xFFFFFF, 1)
        light.position.set(0, 10, 10)
        light.target.position.set(-2, 2, 2)
        scene.add(light)
        scene.add(light.target)

        const camera = new Three.PerspectiveCamera(45, 2, 0.1, 100)
        camera.position.z = 15

        const controls = new OrbitControls(camera, canvasRef.current)
        controls.update()

        const material = new Three.MeshPhongMaterial({
            map: rendererTarget.texture
        })

        const cubeGeo = new Three.BoxBufferGeometry(4, 4, 4)
        const cubeMesh = new Three.Mesh(cubeGeo, material)
        cubeMesh.position.x = 4
        scene.add(cubeMesh)

        const circleGeo = new Three.CircleBufferGeometry(2.8, 36)
        const circleMesh = new Three.Mesh(circleGeo, material)
        circleMesh.position.x = -4
        scene.add(circleMesh)

        const render = (time: number) => {
            time *= 0.001

            rtBoxs.forEach((item) => {
                item.rotation.set(time, time, 0)
            })
            renderer.setRenderTarget(rendererTarget)
            renderer.render(rtScene, rtCamera)
            renderer.setRenderTarget(null)

            cubeMesh.rotation.set(time, time, 0)
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

export default HelloRenderTarget
```

> 补充一下：上面代码中关于灯光的位置、灯光目标的位置、镜头的位置、立方体的位置 都是我随手 填上的，并没有特别的含义，你完全可以适当修改一下。

发布运行，就会在网页中看到 镜子的 1 个面、立方体的 6 个面 上 显示着 3 个不停旋转的立方体。



#### 补充说明1：

在上述示例代码中，离线渲染目标的尺寸我设置的宽高均为 512，你完全可以设置成其他比例的值。

但是为了画面不出现变形内容，所以要遵循以下原则：

**离线渲染目标的宽高比、子场景中镜头的宽高比、总场景中物体被渲染的面的宽高比，这 3 者要保持一致，这样就不会变形。**



假设你想在运行的过程中，修改 离线渲染目标的宽高、以及子场景中镜头的宽高比，其操作方式和修改普通的渲染器或镜头没有什么区别。例如：

```
renderTarget.setSize(newWidth,newHeight)

rtCamera.aspect = newWidth/newHeight
rtCamera.updateProjectionMatrix()
```



#### 补充说明2：3D绘制中的 4 大数据缓冲

1. 颜色缓冲：
2. 像素缓冲：
3. 深度缓冲：depth buffer
4. 模板缓冲：stencil buffer

> stencilBuffer 又被称为 印模缓冲

**模板(stencil)与模板(template)的差异之处：**

单词 stencil 和 template 都可以被翻译为 模板，但是他们 2 者含义是有区别的。

首先这 2 个单词都是来源以 印刷。

1. 模板(template)：形模，例如通过修剪 木板或钢板 的外形，以此外形来进行印刷

2. 模板(stencil)：印模，另外一种印刷技术，例如通过蜡纸来印刷

   > 你把 印模 与 形模 理解成 2 种 不同的印刷方式即可



#### 保存 渲染目标对应的 图片纹理之外，还会额外创建 颜色纹理 和 深度模板纹理。

离屏渲染目标 除了得到并保存 渲染目标对应的 图片纹理之外，还会额外创建 颜色纹理 和 深度模板纹理。

> 图片纹理中，就使用到了 像素缓冲

像我们上面示例中根本就用不到 深度缓冲，那么我们可以在 离屏渲染目标初始化的时候，直接设置 不需要创建 深度缓冲 和 模板缓冲，以节省性能。

```
const rendererTarget = new Three.WebGLRenderTarget(512, 512,{
    depthBuffer:false,
    stencilBuffer:false
})
```

**补充说明：**

1. depthBuffer：深度缓存、默认值为 true
2. stencilBuffer：模板缓冲，默认值为 false



> 关于上述 补充说明2 、补充说明3 中的几个缓冲，我个人理解也不够深，不够透彻，观点仅供参考。



至此，离线渲染 讲解完毕。

今天是 2020年最后一天，大家元旦快乐。

元旦过后，我们将开始学习 Three.js 基础中最后 2 个知识：自定义几何体(current Geometry)、自定义缓冲几何体(current buffer geometry)

