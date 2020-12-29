# 13 Three.js基础之阴影

## 阴影(Shadow)概述

#### LightShadow

在 Three.js 中，所有阴影的基类都是 Three.LightShadow。

请注意：不能直接去实例化 LightShadow，平时真正去创建的阴影，都是该类的之类。



#### 阴影类型

目前，一共有 3 种阴影，通过名字基本都可以猜出对应的灯光的类型。

| 阴影类型(都继承于LightShadow) | 阴影名称   | 解释说明                   |
| ----------------------------- | ---------- | -------------------------- |
| PointLightShadow              | 点光源阴影 | 对应 PointLight 光源       |
| DirectionalLightShadow        | 平行光阴影 | 对应 DirectionalLight 光源 |
| SpotLightShadow               | 聚光灯阴影 | 对应 SpootLight 光源       |

**注意：DirectionalLightShadow只能使用 OrthographicCamera 镜头来计算阴影，无法在 PerspectiveCamera 镜头下使用。**

> 这是因为 PerspectiveCamera 的光线是平行的。



#### 特别强调：你无需创建阴影实例，阴影实例是由 灯光 内部创建的。



#### 阴影贴图

默认情况下 Three.js 使用阴影贴图来绘制阴影。

**何为 “阴影贴图” ？**

答：所谓 ”贴图“，你可以想象成 ”一层层窗户纸“。

假设现在有一个窗户，你可以一层一层的粘贴不同透明度的窗户纸，每一层窗户纸都会叠加到之前的那一层，最终窗户纸所呈现的效果是所有窗户纸最终合并后一块呈现的效果。

当然，由于阴影都是黑灰色，不存在彩色，所以上面的举例中，窗户纸都是不同透明度(明暗度)的，不需要考虑窗户纸的颜色。



**阴影的实际渲染过程：**

假设目前场景下，有 5 个可产生阴影的灯光、20 个阴影对象。

那么最终渲染出的阴影，所经历的过程如下：

1. 使用 第1个灯光 渲染出一份 20 个阴影对象
2. 使用 第2个灯光 渲染出一份 20 个阴影对象
3. ...
4. 使用 第5个灯光 渲染出一份 20 个阴影对象
5. 最终将 5 次渲染的阴影结果进行合并，得出最终的阴影

从上述渲染过程可以看出，每多一个可产生阴影的光源、阴影对象，则需要多渲染一次场景。

**由此可见，阴影的渲染需要大量的计算和性能。**



#### 降低阴影所需性能的解决办法

**方案1：可以有多个灯光，但只有一个平行光可产生阴影**

**方案2：使用 光照贴图 或 环境光照遮挡贴图 来预先计算离线照明的效果**

**方案3：使用 假阴影，添加一个平面放到物体下方的地面上，同时赋予一个看着像阴影的纹理图片材质**

方案1、方案2 我们会在以后再详细讲述学习，本文先实现一下方案3。



**补充说明：**

对于绝大多数 游戏场景 来说，人物脚下的阴影都是采用 方案3 的策略，一般人们是可以接受这种假阴影。

这种假阴影对渲染性能的提升非常大。



## 假阴影示例：HelloFakeShadow

#### 示例目标：

1. 一个类似黑白棋盘一样的地面(之前示例已经使用多次)

2. 在这个平台上面有 15 个跳动的小球

3. 每个小球颜色均不同

4. 每个小球在一定范围内循环、且有规律的跳动

5. 每个小球都有自己的在地面上的阴影

   > 注意这个阴影是假的阴影，并不是光照产生的，而是我们模拟出来的

6. 小球阴影随着小球跳动而做出对应的位置、浓度(明暗)变化



#### 代码思路：

**1、如何创建地面？**

答：我们之前示例已经演示过，继续使用 黑白纹理图片 产生地面，不再多言。

**2、如何创建 15 个颜色不同的小球？**

答：可以通过 for 循环来创建 15 个小球，在每一次 for 循环中，设置不同的小球材质颜色。

大体代码如下：

```
const numSphere = 15
for(let i = 0; i<numSphere; i++){
  const sphereMat = new Three.MeshPhongMaterial()
  sphereMat.color.setHSL(i / numSphere, 1, 0.75)
}
```

**3、如何创建假阴影？**

答：首先创建假阴影对应的 PNG 图片，像墨水滴在水上后的样子。图片中间颜色深，越往外扩颜色越淡直至透明。

![](https://threejsfundamentals.org/threejs/resources/images/roundshadow.png)

> 意外不意外，本文终于有一张配图了^_^

我们通过 纹理加载器，加载这张图片作为阴影的纹理，然后创建一个 Three.PlaneBufferGeometry 几何体，通过创建网格让几何体和材质最终变成阴影物体网格。

特别提醒：

1. 在小球几何体的构造参数中，我们需要设定 `transparent: true`，启用透明度，否则渲染出的阴影为一个整体的黑块。
2. 同时设定 `depthWrite: false`，以降低阴影渲染的精确度，让阴影更加模糊平滑。

**4、如何让某个小球和它的阴影为一个整体？**

答：创建一个 Three.Object3D 的对象，将 小球 和 阴影 都添加进去，让他们形成一个整体。在内部设置阴影的 position.y = 0.001，让阴影看上去更加贴近地面，同时设置小球的 position 为 其他值，让小球和阴影在 y 轴上保持一定的距离。

**5、如何让小球循环、且有规律的跳动？**

答：这个牵扯一些具体的数学算法，参见后面示例中贴出来的具体代码吧。

**6、如何让阴影随着小球跳动高度而发生对应的变化？**

答：每次更新小球 y 轴的值时，根据一定比例，修改 阴影对应的透明度，以此来从视觉上感受到阴影的变化。

请注意：并没有更改阴影的尺寸大小，而仅仅更改阴影的透明度来模拟阴影的明暗变化。

**7、阴影和小球讲清楚了，那灯光呢？**

答：为了让整个场景能够有光，我们需要添加 1 个半球环境光源。

为了个小球看上去有光泽感，我们需要添加 1 个平行光，负责照射在小球上。

也就是说，场景中，我们一共要添加 2 个光源。



**关键点补充：**

我们需要将渲染器 WebGLRenderer 的 physicallyCorrectLights 的值设置为 true，以按照物理校正正确光照的模式来渲染场景。

> 该值默认为 false，如果不设置该值为 true，则渲染出的小球特别亮。



#### 示例代码

我们创建 src/components/hello-fake-shadow/index.tsx 文件。

```
import { useRef, useEffect } from 'react'
import * as Three from 'three'

import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

import './index.scss'

interface SphereShadowBase {
    base: Three.Object3D,
    sphereMesh: Three.Mesh,
    shadowMesh: Three.Mesh,
    y: number
}

const HelloFakeShadow = () => {
    const canvasRef = useRef<HTMLCanvasElement>(null)

    useEffect(() => {

        if (canvasRef.current === null) {
            return
        }

        const renderer = new Three.WebGLRenderer({ canvas: canvasRef.current })
        renderer.physicallyCorrectLights = true

        const camera = new Three.PerspectiveCamera(45, 2, 0.1, 1000)
        camera.position.set(0, 10, 20)

        const scene = new Three.Scene()
        scene.background = new Three.Color(0xFFFFFF)

        const hemisphereLight = new Three.HemisphereLight(0xB1E1FF, 0xB97A20, 2)
        scene.add(hemisphereLight)

        const directionalLight = new Three.DirectionalLight(0xFFFFFF, 1)
        directionalLight.position.set(0, 10, 5)
        directionalLight.target.position.set(-5, 0, 0)
        scene.add(directionalLight)
        scene.add(directionalLight.target)

        const planeSize = 40
        const loader = new Three.TextureLoader()
        const texture = loader.load(require('@/assets/imgs/checker.png').default)
        texture.wrapS = Three.RepeatWrapping
        texture.wrapT = Three.RepeatWrapping
        texture.magFilter = Three.NearestFilter
        texture.repeat.set(planeSize / 2, planeSize / 2)
        const planeMaterial = new Three.MeshBasicMaterial({
            map: texture,
            side: Three.DoubleSide
        })
        planeMaterial.color.setRGB(1.5, 1.5, 1.5) //在纹理图片颜色的RGB基础上，分别乘以 1.5，这样可以不修改纹理图片的前提下让纹理图片更加偏白一些
        const planeGeo = new Three.PlaneBufferGeometry(planeSize, planeSize)
        const mesh = new Three.Mesh(planeGeo, planeMaterial)
        mesh.rotation.x = Math.PI * -0.5
        scene.add(mesh)

        const shadowTexture = loader.load(require('@/assets/imgs/roundshadow.png').default)
        const basesArray: SphereShadowBase[] = [] //所有球体假阴影对应的数组

        const sphereRadius = 1
        const sphereGeo = new Three.SphereBufferGeometry(sphereRadius, 32, 16)
        const shadowSize = 1 //假阴影的尺寸
        const shadowGeo = new Three.PlaneBufferGeometry(shadowSize, shadowSize) //假阴影对应的平面几何体

        const numSphere = 15 //将随机创建 15 个球体
        for (let i = 0; i < numSphere; i++) {
            const base = new Three.Object3D() //创建 球和阴影 的整体对象
            scene.add(base)

            const shadowMat = new Three.MeshBasicMaterial({
                map: shadowTexture,
                transparent: true,
                depthWrite: false
            })

            const shadowSize = sphereRadius * 4
            const shadowMesh = new Three.Mesh(shadowGeo, shadowMat)
            shadowMesh.position.y = 0.001
            shadowMesh.rotation.x = Math.PI * -0.5
            shadowMesh.scale.set(shadowSize, shadowSize, shadowSize)
            base.add(shadowMesh)

            const sphereMat = new Three.MeshPhongMaterial()
            sphereMat.color.setHSL(i / numSphere, 1, 0.75) //给 球 设置不同颜色
            const sphereMesh = new Three.Mesh(sphereGeo, sphereMat)
            sphereMesh.position.set(0, sphereRadius + 2, 0)
            base.add(sphereMesh)

            basesArray.push({
                base,
                sphereMesh,
                shadowMesh,
                y: sphereMesh.position.y
            })
        }

        const controls = new OrbitControls(camera, canvasRef.current)
        controls.target.set(0, 5, 0)
        controls.update()

        const render = (time: number) => {
            time *= 0.001

            basesArray.forEach((item, index) => {
                const { base, sphereMesh, shadowMesh, y } = item

                const u = index / basesArray.length
                const speed = time * 0.2
                const angle = speed + u * Math.PI * 2 * (index % 1 ? 1 : -1)
                const radius = Math.sin(speed - index) * 10

                base.position.set(Math.cos(angle)* radius,0,Math.sin(angle)* radius)
                const yOff = Math.abs(Math.sin(time*2+index))
                sphereMesh.position.y = y + Three.MathUtils.lerp(-2,2,yOff);
                (shadowMesh.material as Three.Material).opacity = Three.MathUtils.lerp(1,0.25,yOff)
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

export default HelloFakeShadow
```

运行后，就会看到 15 个跳动且有阴影的小球。



**补充一点：物体碰撞**

如果你把视角变为顶视图，你会发现这个现象：在跳动过程中若两个小球发生位置上的重叠，小球会直接融入到对方之中。

这一点并不像我们现实中，小球会发生碰撞互相作用之类的物体现象。

**Three.js 默认并不包含物体碰撞物理殷勤。**

若你真的需要在 Three.js 中需要物体之间的碰撞，那么你可以引入第三方编写的 物体碰撞模块，具体如何引入我们先略过。

你只需知道 Three.js 这个特性就好。

**Three.js 侧重点时场景渲染，而不是 3D 游戏。**

> 事实上这是 Three.js 的一个特点，也是缺点

> 若开发 3D 游戏类，通常并不会选择 Three.js，会选择其他 3D 游戏引擎，例如：Babylon.js、PlayCanvas.js



## 平行光阴影(DirectionalLightShadow)示例：HelloShadow

#### 示例目标：

1. 一个黑白相间的地面上，放置有 一个立方体和一个球体

2. 场景中有一个 平行光源

3. 地面上显示出 立方体和球体 真正的阴影

   > 这里说的阴影是真的阴影对象，而不是“物体显示出的明暗”或“像 HelloFakeShadow 那样假的阴影”

4. 为了方便我们观察，场景中需要添加：灯光的辅助对象、镜头的辅助对象、“阴影的辅助对象”

   > 请注意 “阴影的辅助对象”其实是加上了双引号，事实上并不存在阴影的辅助对象，这里的辅助对象实际上指的是 “灯光中对应的阴影镜头的辅助对象”。



#### 代码思路：

**1、基础场景的搭建**

这个场景在之前示例中已经反复出现了，不再过多讲述。

**2、阴影的相关设置有哪些？**

1. 渲染器(WebGLRenderer)开启渲染阴影

   ```
   renderer.shadowMap.enable = true
   ```

2. 平行灯光(DirectionalLight)开启投射阴影

   ```
   light.castShadow = true
   ```

3. 地面开启 接收投影

   ```
   planeMesh.receiveShadow = true
   ```

4. 立方体和球体都开启 接收和投射阴影

    ```
   boxMesh.castShadow = true
   boxMesh.receiveShadow = true
   
   sphereMesh.castShadow = true
   sphereMesh.receiveShadow = true
   ```

**3、阴影的辅助对象？**

灯光、镜头 的辅助对象在之前章节中，已经使用过了，那么说一下 “阴影的辅助对象”。

首先在 Three.js 中根本不存在阴影的辅助对象，“阴影的辅助对象”其实是我个人想出来的一个词。

这个所谓的“阴影的辅助对象”其实是 灯光中阴影的镜头对应的辅助对象。

> 目的是为了方便我们通过灯光的辅助对象来观察灯光所能投射的阴影可见区域。

```
const shadowCamera = light.shadow.camera
const shadowHelper = new Three.CameraHelper(shadowCamera)
scene.add(shadowHelper)
```

**4、修改灯光中阴影的镜头属性，覆盖住所有物体，好让阴影显示完整**

如果你在编写示例时发现物体在地面上的影子显示不完整(看着影子似乎被裁切掉了一部分)，那么这种情况多数原因就是 灯光中阴影的镜头照着范围不足以覆盖物体，所以才出现了影子被裁切掉的情况。

在本示例中，我们就需要手工修改阴影镜头视椎的可见范围，让视椎刚好可以覆盖住物体。同时我们要尽量保证视椎又不是特别大，而是刚好比较合适的大小。

特别强调：

1. 平行光中影子(DirectionalLightShadow)的镜头使用的是 OrthographicCamera，不是 PerspectiveCamera
2. 其他 2 种阴影 PointLightShadow、SpotLightShadow 镜头默认使用的是 PerspectiveCamera。
3. 因此在本示例中，当我们要修改 阴影的镜头覆盖区域时修改的是 OrthographicCamera 的 left、right、top、bottom 属性。

```diff
const shadowCamera = light.shadow.camera
+ shadowCamera.left = -10
+ shadowCamera.right = 10
+ shadowCamera.top = 10
+ shadowCamera.bottom = -10
+ shadowCamera.updateProjectionMatrix()

const shadowHelper = new Three.CameraHelper(shadowCamera)
scene.add(shadowHelper)
```

> 经过修改后的视椎，实际宽 20、高 20

在本示例中我们添加的 shadowHelper 事实上就是为了方便我们观察视椎是否完全覆盖住了物体。

> 这里我们先提出一个问题：为了省事我们是否可以直接将阴影镜头视椎的范围设置成特别大，这样就不用担心万一物体不在视椎范围里了。可以这样做吗？
>
> 答案是：不可以！
>
> 至于为什么，会在后面详细讲述。



#### 注意事项：

1. 在 假阴影示例(HelloFakeShadow)中，我们将 地面平台使用的是不反光材质 MeshBasicMaterial，但是在本示例中，需要将地面平台设置为可反光材质 MeshPhoneMaterial。

   > 如果地面平台材质继续使用 MeshBasicMaterial，则永远显示不出阴影。

2. 再次强调一遍，你无需要创建 DirectionalLightShadow 实例，阴影实例是由 灯光(DirectionalLight) 内部创建的。

3. 目前我们一直采用的是 简单光照模型，所以物体的阴影是正确的，但是物体的明暗与实际生活中的效果在某些地方略微不同，说直白点就是有些地方的明暗不符合自然现象。

4. 由于场景中添加了 镜头交互对象 OrbitControls，所以在每次渲染时，也要将 各个辅助对象进行更新。

   ```
   const render = () => {
     cameraHelper.update()
     lightHelper.update()
     shadowHelper.update()
     ...
   }
   ```



#### 示例代码：

示例代码位于 src/components/hello-shadow/index.stx

```
import { useEffect, useRef } from 'react'
import * as Three from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

import './index.scss'

const HelloShadow = () => {

    const canvasRef = useRef<HTMLCanvasElement>(null)

    useEffect(() => {
        if (canvasRef.current === null) {
            return
        }

        const renderer = new Three.WebGLRenderer({ canvas: canvasRef.current })
        renderer.shadowMap.enabled = true

        const scene = new Three.Scene()
        scene.background = new Three.Color(0x333333)

        const camera = new Three.PerspectiveCamera(45, 2, 5, 100)
        camera.position.set(0, 10, 20)
        scene.add(camera)

        const helperCamera = new Three.PerspectiveCamera(45, 2, 5, 100)
        helperCamera.position.set(20, 10, 20)
        helperCamera.lookAt(0, 5, 0)
        scene.add(helperCamera)

        const cameraHelper = new Three.CameraHelper(helperCamera)
        scene.add(cameraHelper)

        const controls = new OrbitControls(camera, canvasRef.current)
        controls.target.set(0, 5, 0)
        controls.update()

        const light = new Three.DirectionalLight(0xFFFFFF, 1)
        light.castShadow = true
        light.position.set(0, 10, 0)
        light.target.position.set(-4, 0, -4)
        scene.add(light)
        scene.add(light.target)

        const shadowCamera = light.shadow.camera
        shadowCamera.left = -10
        shadowCamera.right = 10
        shadowCamera.top = 10
        shadowCamera.bottom = -10
        shadowCamera.updateProjectionMatrix()

        const lightHelper = new Three.DirectionalLightHelper(light)
        scene.add(lightHelper)

        const shadowHelper = new Three.CameraHelper(shadowCamera)
        scene.add(shadowHelper)

        const planeSize = 40

        const loader = new Three.TextureLoader()
        const texture = loader.load(require('@/assets/imgs/checker.png').default)
        texture.wrapS = Three.RepeatWrapping
        texture.wrapT = Three.RepeatWrapping
        texture.magFilter = Three.NearestFilter
        texture.repeat.set(planeSize / 2, planeSize / 2)

        const planGeo = new Three.PlaneBufferGeometry(planeSize, planeSize)
        const planeMat = new Three.MeshPhongMaterial({
            map: texture,
            side: Three.DoubleSide
        })
        const planeMesh = new Three.Mesh(planGeo, planeMat)
        planeMesh.receiveShadow = true
        planeMesh.rotation.x = Math.PI * -0.5
        scene.add(planeMesh)

        const material = new Three.MeshPhongMaterial({
            color: 0x88AACC
        })
        const boxMat = new Three.BoxBufferGeometry(4, 4, 4)
        const boxMesh = new Three.Mesh(boxMat, material)
        boxMesh.castShadow = true
        boxMesh.receiveShadow = true
        boxMesh.position.set(5, 3, 0)
        scene.add(boxMesh)

        const sphereMat = new Three.SphereBufferGeometry(3, 32, 16)
        const sphereMesh = new Three.Mesh(sphereMat, material)
        sphereMesh.castShadow = true
        sphereMesh.receiveShadow = true
        sphereMesh.position.set(-4, 5, 0)
        scene.add(sphereMesh)

        const render = () => {
            cameraHelper.update()
            lightHelper.update()
            shadowHelper.update()

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

export default HelloShadow
```

以上代码正常运行后，这会在地面上显示出比较清晰、比较黑的物体影子。



#### 回顾之前提出的问题：为什么阴影镜头的视椎既要覆盖物体，又要范围合适，不要过大？

先不解释为什么，我们可以先尝试将阴影镜头的视椎修改得比较大，例如扩大 10 倍，然后看会发生什么？

```diff
- shadowCamera.left = -10
- shadowCamera.right = 10
- shadowCamera.top = 10
- shadowCamera.bottom = -10

+ shadowCamera.left = -100
+ shadowCamera.right = 100
+ shadowCamera.top = 100
+ shadowCamera.bottom = -100
```

当阴影视椎变得比较大，此时运行，实际渲染中会发现——物体在地面上的阴影变得不够平滑、有类似马赛克的块状效果，这是为什么？

答：默认阴影贴图尺寸(shadow mapSize) 为 512 x 512，当灯光中阴影镜头视椎范围越大，所需要对应的阴影贴图尺寸也要越大。当视椎特别大而阴影贴图尺寸并不足够大时就会产生这种块状阴影。

当然你可以通过修改 `light.shadow.mapSize.widht` 和 `light.shadow.mapSize.height` 的值，将属性值调大(默认是512)。这样看似可以解决阴影块状问题，但是当阴影贴图尺寸越大，对应渲染所需的计算和性能也就越大，这并不是我们希望的效果。

所以，**最简单的方式就是不要将视椎范围调整得过大**，让视椎可覆盖物体但有不是特别大，这才是我们应该做的。

> 事实上是视椎和阴影贴图尺寸 越合适、越贴近，阴影渲染的效果越好。



**补充说明：阴影贴图尺寸最大能设置为多少？**

答：渲染器中 `renderer.capabilities.maxTextureSize`  的值就是渲染器可支持的贴图最大尺寸。



#### 如何让影子不那么黑，显示得比较淡一些？

我们是无法直接修改影子的明暗度，但是可以通过给场景中添加一个半球环境光，让场景整体更加亮一些，从而影子也就变得淡了一些。

```diff
const renderer = new Three.WebGLRenderer({ canvas: canvasRef.current })
renderer.shadowMap.enabled = true
+ renderer.physicallyCorrectLights = true

...

//添加半球环境光
+ const hemisphereLight = new Three.HemisphereLight(0xFFFFFF, 0x000000, 2)
+ scene.add(hemisphereLight)
```



## 聚光灯阴影(SpotLightShadow)示例

我们将在平行光阴影示例的基础上，修改成 聚光灯阴影示例。

注意：

1. 为了方便我们观察聚光灯阴影，我们将不添加 半球环境光。
2. 聚光灯阴影对应的镜头为 PerspectiveCamera，这种镜头上我们没有必要再去修改视椎范围

修改后的代码如下：

```diff
const renderer = new Three.WebGLRenderer({ canvas: canvasRef.current })
renderer.shadowMap.enabled = true
- renderer.physicallyCorrectLights = true

- const hemisphereLight = new Three.HemisphereLight(0xFFFFFF, 0x000000, 2)
- scene.add(hemisphereLight)

- const light = new Three.DirectionalLight(0xFFFFFF, 1)
+ const light = new Three.SpotLight(0xFFFFFF, 1)
light.castShadow = true
...

const shadowCamera = light.shadow.camera
-  shadowCamera.left = -10
-  shadowCamera.right = 10
-  shadowCamera.top = 10
-  shadowCamera.bottom = -10
shadowCamera.updateProjectionMatrix()

-  const lightHelper = new Three.DirectionalLightHelper(light)
+  const lightHelper = new Three.SpotLightHelper(light)
scene.add(lightHelper)
```



## 点光源阴影(PointLightShadow)示例

#### 点光源(PointLight)的特殊之处：

点光源是朝着四面八方发射光的，也就是说相当于朝着 上、下、左、右、前、后 6 个面都发光，这就意味着它会在这 6 个面上都产生阴影。

> 平行光、聚光灯 他们 都只会朝着一个方向发射光，且只产生一个面上的阴影。

也就是说 点光源阴影 渲染一次(实际是渲染 6 个面的阴影)要比别的阴影类型计算量更大、渲染所需时间多。



#### 示例的特殊改动之处：

由于之前示例中，我们只有一个 地面，而 点光源阴影会在 6 个面中都投射阴影，所以我们需要改造我们的场景。

向场景中添加其他几个面，以便我们看到不同面上的阴影。

**如何添加 6 个面？**

答：我们只需在场景中添加一个尺寸更大一点的一个立方体，为了方便和之前的立方体做区分，我们暂且将这个立方体称呼为 room。

我们需要做的事情是：

1. 将 room 嵌套住原本的立方体和球体

2. 将 room 的材质的 side 设置为 Three.BackSide，这样渲染器就会渲染该物体的内部，而不是外部。

   > 由于原本的立方体、球体、点光源 均被包裹 在 room 中，所以我们就会看到 他们 在 room 内部各个面上的投影。

   > 这里所说的 “被包裹” 其实只不是 他们(立方体、球体)尺寸比较小，且位置(立方体、球体、点光源)坐标刚好位于 room 内部而已。



#### 补充说明：

尽管 room 是 6 个面，但是实际渲染的结果，正对着你视觉的那些面是不会被渲染出来的。

> 视觉正对着的面，有可能是 1 个，也有可能是 2 个、3 个，这完全取决于你观察的视觉角度。

因为如果这个面也被渲染，那你实际上是看不到 room 内部的，这不是我们希望的。

> 因为我们 room 材质里设定的是 side:Three.BackSide



#### 具体的代码：

我们将代码放置于 scr/components/hello-shadow/hello-point-light-shadow.tsx

这次我们以 聚光灯阴影(SpotLightShadow)示例 来做修改说明：

```diff
- light = new Three.DirectionalLight(0xFFFFFF, 1)
+ const light = new Three.PointLight(0xFFFFFF, 1)
light.castShadow = true
light.position.set(0, 10, 0)
- light.target.position.set(-4, 0, -4)
scene.add(light)
- scene.add(light.target)

- const lightHelper = new Three.SpotLightHelper(light)
+ const lightHelper = new Three.PointLightHelper(light)

//新增 room 立方体
+ const roomMat = new Three.MeshPhongMaterial({
+   color:0xCCCCCC,
+   side:Three.BackSide //注意此处的设置
+ })
+ const roomGeo = new Three.BoxBufferGeometry(30,30,30)
+ const roomMesh = new Three.Mesh(roomGeo,roomMat)
+ roomMesh.receiveShadow = true //作为背景墙面，只需接收阴影，无需设置 投射阴影(castShadow)
+ roomMesh.position.set(0,14.9,0) //这个 y 值 14.9 是有玄机的，稍后解释
+ scene.add(roomMesh)
```

编译运行，就可以看到 球体 在地面和某个侧面上的阴影。



#### 注意：为什么物体下方显示的阴影 是在地面(planeMesh)而不是 roomMesh 的底面？

究竟显示 地面还是 room 的底面，这个完全取决于 地面(planeMesh)、roomMesh 的相对位置，谁位置更加靠上就显示谁。

由于 room 的高度为 30，高度的一半为 15，为了避免 地面和room 的底面 不分高低，所以我们故意将 roomMesh 的 position.y 的值设置为 14.9。

这样可以十分明确 地面 高于 roomMesh 的底面，所以我们看到的就是地面。



若将上面的代码，修改为：

```
roomMesh.position.set(0,15.1,0)
```

此时 roomMesh 底面在 地面 之上，所以渲染后只会看到 roomMesh 的底面，看不见地面了。



**在实际的场景中，确实避免 2 个面极度紧贴，是会选择将其中一个面的位置故意设置偏低一些，保证 Three.js 能够明确区分出 哪个面谁在上谁在下。**



至此，关于阴影我们就学习到此。

下一节学习一个特殊场景——雾，充满雾气的场景。



