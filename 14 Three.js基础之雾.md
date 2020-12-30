# 14 Three.js基础之雾

## 雾(Fog)概述

这里的 雾(Fog) 就是指我们日常生活中的雾气。

我们之前所有的示例的 scene 中，都是完全清晰、透明的空间，如果想创建出有雾气的场景，就需要 雾 了。



#### 雾的特点：

1. 越靠近镜头 雾气越小
2. 越远离镜头 雾气越大
3. 雾气本身只会影响物体的渲染效果，但雾气本身并不会流动
4. 默认所有材质都可以被雾影响，若某物体不想被雾影响，可以将该物体材质的 fog 属性设置为 false



#### 雾的 2 种类型

在 Three.js 中，一共有 2 种 雾的类型；

| 雾的类型 | 名称   | 解释                      |
| -------- | ------ | ------------------------- |
| Fog      | 雾     | 雾的密度随着距离 线性增大 |
| FogExp2  | 指数雾 | 雾的密度随着距离 指数增大 |



#### Fog构造参数

```
Fog( color : Integer, near : Float, far : Float )
```

1. color：雾的颜色

2. near：开始应用雾的最小距离，默认值为 1

   > 假设 雾的 near 数值 小于 镜头 near 的值，则该区域的物体不会被雾所影响。
   >
   > 因为小于镜头 near 区域的物体根本就不可见，Three.js 也不会渲染该区域。

3. far：应用雾的最大距离，默认值为 1000

   > 假设 雾的 far 数值 大于 镜头 far 的值，则该区域的物体不会被雾所影响。



#### FogExp2构造参数

```
FogExp2( color : Integer, density : Float )
```

1. color：雾的颜色
2. density：定义雾的密度将会增加的有多快，默认值为 0.00025



#### 如何把雾添加到场景中？

添加的方式非常简单：

```
scene.fog = new Three.Fog(0xFFFFFF,10,100)
或
scene.fog = new Three.FogExp2(0xFFFFFF,0.001)
```



#### 究竟该选择哪种雾？

从实际渲染效果 真实度 而言，FogExp2 更加逼真。

但实际项目中，往往更多选择 Fog，因为 Fog 更加简单。

> Fog 还允许你调整 near 和 far 的值，而 FogExp2 只允许调整指数值，若想对雾气距离更加精准控制，Fog 是第一选择。



#### 关于雾的颜色的补充说明

为了让 雾和物体、场景融合比较好，通常情况下我们会将 雾的颜色和场景的背景色 设置成相同值。

当然如果你希望 场景背景色 和 雾气颜色不相同，完全没有问题，根据实际需求来设定就好了。



## 雾的示例：HelloFog

#### 示例目标

1. 场景上有 3 个不断旋转、不同颜色的立方体
2. 场景中添加 雾，让 3 个立方体被雾气包围



#### 实现思路

额~，这个场景除了 雾 之外其他的实现，和我们最初刚开始学 “03 编写HelloThreejs.md” 那篇文章一样，具体就不多说了，直接上代码。



#### 示例代码

代码位于 scr/components/hello-fog/index.tsx

```
import { useEffect, useRef } from 'react'
import * as Three from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

import './index.scss'

const HelloFog = () => {

    const canvasRef = useRef<HTMLCanvasElement>(null)

    useEffect(() => {

        if (canvasRef.current === null) {
            return
        }

        const renderer = new Three.WebGLRenderer({ canvas: canvasRef.current })

        const scene = new Three.Scene()
        scene.background = new Three.Color(0xadd8e6)
        scene.fog = new Three.Fog(0xadd8e6, 1, 2) //向场景中添加 雾
        //scene.fog = new Three.FogExp2(0xadd8e6,0.8) //向场景中添加 指数雾

        const camera = new Three.PerspectiveCamera(75, 2, 0.1, 5)
        camera.position.z = 2

        const controls = new OrbitControls(camera, canvasRef.current)
        controls.update()

        const light = new Three.DirectionalLight(0XFFFFFF, 1)
        light.position.set(-1, 2, 4)
        scene.add(light)

        const colors = ['blue', 'red', 'green']
        const boxs: Three.Mesh[] = []

        colors.forEach((color, index) => {
            const mat = new Three.MeshPhongMaterial({ color })
            const geo = new Three.BoxBufferGeometry(1, 1, 1)
            const mesh = new Three.Mesh(geo, mat)
            mesh.position.set((index - 1) * 2, 0, 0)
            scene.add(mesh)
            boxs.push(mesh)
        })

        const render = (time: number) => {
            time *= 0.001

            boxs.forEach((box) => {
                box.rotation.x = time
                box.rotation.y = time
            })

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

export default HelloFog
```



## 如何让材质不受雾的影响？

**所有材质默认都会受到雾的影响和作用。**

若希望物体不受雾的影响(即使物体处于雾气当中)，那么可以将物体材质的 fog 属性设置为 false 即可。



#### 试想一下以下场景：

1. 我们有一个房子，房子四周被雾气包围
2. 此时我们打开窗户，我们希望的效果是：窗外的物体继续被雾气环绕，但屋内的物体并不受雾气影响。
3. 为了实现这个效果，我们只需将屋内的物体材质 fog 设置为 false 即可



#### 举例演示

我们修改 HelloFog 中的代码，我们让中间的红色立方体不受雾气影响，代码如下：

```diff
colors.forEach((color, index) => {
            const mat = new Three.MeshPhongMaterial({ color })
            const geo = new Three.BoxBufferGeometry(1, 1, 1)
            const mesh = new Three.Mesh(geo, mat)
            mesh.position.set((index - 1) * 2, 0, 0)
            scene.add(mesh)
            boxs.push(mesh)
        })

+ const redBox = boxs[1].material as Three.Material //找到中间 红色立方体
+ redBox.fog = false //让红色立方体的材质不受雾的影响
```

运行后，左右两侧的立方体继续受到雾气影响，若隐若现，但中间红色立方体则不受雾气任何影响。

> 滚动鼠标中轴，调整场景上的观察视角，拉远观察距离，左右两侧立方体可能会完全消失在雾气中，但中间红色立方体不会消失，会一直处于可见状态。



至此，关于 雾 讲解完毕。

这一节可能是最近一系列文章中，最简单的一篇了。

下一节，讲解 离屏渲染(render target)