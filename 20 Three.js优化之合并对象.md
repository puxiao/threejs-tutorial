# 20 Three.js优化之合并对象

前面学习了 Three.js 入门、基础、技巧，今天开始学习 Three.js 的性能优化。

关于性能优化有很多方式，最基础也是最常见的方式就是——合并几何对象。

> 在 “Three.js 基础之图元” 那篇文章中，我们将几何体称呼为 图元，现在我们修改一下这个称呼，本文以后，绝大多数情况下我们都使用 “几何体” 来代替 “图元”。



在演示 Three.js 性能优化之前，我们先要创建一个 3D地球 示例。

> 激动人心的时刻到了，创建一个可视化的地球，不错，有成就感。



## 基础示例：HelloEarth

#### 示例目标：

1. 一个球体

2. 球体表面贴上地球纹理图片

   > 地球表面的纹理图片，你可以从这个地址上获取该图片：https://threejsfundamentals.org/threejs/resources/images/world.jpg



#### 代码思路：

创建球体、加载纹理图片，这些操作我们之前都讲解过，这些并没有什么难度。



#### 具体代码：

地球贴图纹理图片位于：src/assets/imgs/world.jpg

HelloEarth 位于：src/components/hello-earth/index.tsx

```
import { useEffect, useRef } from 'react'
import * as Three from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

import './index.scss'

const HelloEarth = () => {
    const canvasRef = useRef<HTMLCanvasElement>(null)

    useEffect(() => {
        if (canvasRef.current === null) { return }

        const canvas = canvasRef.current
        const renderer = new Three.WebGLRenderer({ canvas })
        const camera = new Three.PerspectiveCamera(45, 2, 0.1, 100)
        camera.position.z = 7
        const scene = new Three.Scene()

        const controls = new OrbitControls(camera, canvas)
        controls.enableDamping = true
        controls.update()

        const light1 = new Three.HemisphereLight(0xFFFFFF, 0xFFFFFF, 0.8)
        scene.add(light1)
        const light2 = new Three.DirectionalLight(0xFFFFFF, 0.4)
        light2.position.set(2, 2, 0)
        scene.add(light2)

        const loader = new Three.TextureLoader()
        const texture = loader.load(require('@/assets/imgs/world.jpg').default)
        const material = new Three.MeshPhongMaterial({
            map: texture
        })
        const geometry = new Three.SphereBufferGeometry(2, 32, 32)
        const earth = new Three.Mesh(geometry, material)
        scene.add(earth)

        const render = () => {
            controls.update()
            renderer.render(scene, camera)
            window.requestAnimationFrame(render)
        }
        window.requestAnimationFrame(render)

        const handleResize = () => {
            const width = canvas.clientWidth
            const height = canvas.clientHeight
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

export default HelloEarth
```



