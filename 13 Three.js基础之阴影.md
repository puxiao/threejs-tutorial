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

```
import { useRef, useEffect } from 'react'
import * as Three from 'three'
import { PlaneBufferGeometry } from 'three'

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
        const planeGeo = new PlaneBufferGeometry(planeSize, planeSize)
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



