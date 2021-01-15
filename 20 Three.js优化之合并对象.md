# 20 Three.js优化之合并对象

前面学习了 Three.js 入门、基础、技巧，今天开始学习 Three.js 的性能优化。

关于性能优化有很多方式，最基础也是最常见的方式就是——合并几何对象。

> 在 “Three.js 基础之图元” 那篇文章中，我们将几何体称呼为 图元，现在我们修改一下这个称呼，本文以后，绝大多数情况下我们都使用 “几何体” 来代替 “图元”。



既然谈到性能优化，就不能使用简单的示例，要不然根本无法体现出 优化前和优化后 的区别。

激动人心的时刻到了。

#### 本文我们的目标：制作并优化一个显示地球人口人数分布的可视化3D地球

你可以先访问以下网址，先感受一下我们本文要模仿的效果：

https://globe.chromeexperiments.com/

> 补充说明：这个网站，是谷歌浏览器为了向大众演示 WebGL 技术而制作的一个演示网页。

我相信你第一次看到这种基于浏览器的 3D 地球数据展示，一定会被震撼到的。

接下来我们就要逐步分析，找出实现方式。

**我们先考虑怎么把这个场景实现出来，然后再考虑优化的事。**



#### 核心模块分析

我们要先搞明白这个 3D 数字地球的核心模块。

1. 3D 地球

   > 就是一个球体，添加一个地球纹理图片
   >
   > > 图片为一个矩形地球展开图，本示例使用的地球纹理图片资源：  
   > > https://threejsfundamentals.org/threejs/resources/images/world.jpg

2. 表示人口多少的柱状物

   > 某个地区人口多则柱状物就比较高，反之人口少则柱状物比较低

3. 鼠标可交互

   > 这个直接使用 OrbitControls 就可以，不需要再详细讲解

上面的核心模块 1 、3 都很容易实现，重点我们进一步拆解一下 “表示人口多少的柱状物”。



#### 如何实现“表示人口多少的柱状物”？

我们通过以下 3 个灵魂追问，来梳理思路。

**第1问：人口数据从哪里来？**

原网页提供 3 个年份的人口数量统计，分别是 1990、1995、2000年。

NASA提供的统计结果介绍页：
https://sedac.ciesin.columbia.edu/data/set/gpw-v4-admin-unit-center-points-population-estimates-rev11/data-download

请注意，该页面还提供更加新的统计结果，但是下载时候提示需要注册。

我们选择不使用最新的 2020 年数据，而是使用 2010 年的结果。

为了方便你获得到 2010 年人口统计结果，你可以直接点击下面这个地址，直接下载：

https://threejsfundamentals.org/threejs/resources/data/gpw/gpw_v4_basic_demographic_characteristics_rev10_a000_014mt_2010_cntm_1_deg.asc

> 反正我们本文的重点是模仿效果，至于数据时效性不必纠结

> 该人口统计数据文件格式为 .asc，至于如何解析该文件，我们会稍后讲解



**第2问：人口数据和地区的对应关系，如何表现在地球上？**

我们把刚才下载得到的人口统计数据文件，重命名为 gpw_v4_2010.asc，然后将该文件移动到：src/assets/data/ 目录中。

点击该文件，用记事本查看该文件内容，你会发现里面大致为以下内容：

```
ncols         360
nrows         145
xllcorner     -180
yllcorner     -60
cellsize      0.99999999999994
NODATA_value  -9999
-9999 -9999 -9999 -9999 -9999 -999...
...
...
```

不考虑细节，我可以先告诉你的是，这里面的数据是：**矩形地球地图上，不同点(经纬度)对应的数值(人口数量)**。

我们需要将数据与地图纹理图片进行点对点的位置匹配。



**第3问：根据人口多少，如何创建对应的柱状物？**

当得到地球某个经纬度(地球纹理图片上的某个坐标)上对应的人口数据后，就可以根据人口数量按照一定比例，创建柱状物。



原理讲过后，那接下来就是实际操作了。



## 基础示例：HelloEarth

接下来，将通过以下几个步骤，逐步实现我们的目标示例。



#### 第1步：加载人口数据(gpw_v4_2010.asc)



#### 第2步：解析人口数据



#### 第3步：加载地球纹理图片



#### 第4步：将人口数据与地球纹理图片进行位置上的匹配





## 

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



