# 03 编写HelloThreejs

终于要真正第一次亲密接触 Three.js 了。

我们先梳理一下创建一个 Three.js 示例所需要的过程，认真阅读并理解整个过程，会更加容易让你读懂我后面的示例代码。



## 从引入Three到创建示例的过程

### 第1环节：引入Three.js

**引入方式1：将 THREE 一次全部引入**

```
import THREE from 'three'

//当需要使用某个具体的模块时，例如创建场景，则代码如下
const scene = new THREE.Scene()
```

> 这种方式会将所有 Three.js 相关模块都引入进来，虽然引入代码简介，但是会造成项目打包输出时文件过大，因此并不建议这样引入。



**请注意：默认 three 模块导出的名字就是全部大写的 “THREE”。我个人非常不习惯 模块名称 全部大写，我的代码习惯是使用 “Three”。**

```
import * as Three from 'three'

const scene = new Three.Scene()
```

> **本系列文章中使用的示例，代码绝大多数都采用这种引入形式，使用 Three 而 不使用 THREE。  
> 所以网上一些教程中可能描述某个类时使用的是 THREE.Xxxx，而我在本系列文章中都会使用 Three.Xxxx 这种方式。**



**引入方式2：按需引入模块**

例如我们需要使用 Scene 模块，则仅引入该模块即可

```
import { Scene } from 'three'

//当需要使用某个具体的模块时，例如创建场景，则代码如下
const scene = new Scene()
```

> 本文示例代码，都将采用按需引入的方式。



### 第2环节：将DOM中的canvas与Threejs中的渲染器进行挂钩

**采用 React 的 useEffect + useRef 来实现所谓 “挂钩” 。**

> 具体参见示例代码



### 第3环节：创建Three.js基础3大元素、场景可见元素

**基础3大元素：**

1. 渲染器 > 本文示例采用的渲染器是 WebGLRenderer
2. 透视镜头 > 本文示例采用的是 PerspectiveCamera
3. 场景 > Scen

**场景可见元素：**

1. 几何体 > 本文示例采用的是 BoxGeometry(立方体)
2. 几何体的材质(颜色、光亮程度) > 本文示例采用的是 MeshBasicMaterial 或 MeshPhongMaterial
3. 网格 > Mesh
4. 光源 > 本文示例采用的是 DirectionalLight(平行光源)

**补充说明：**

你应该发现，除了 场景(Scen)、网格(Mesh) 之外，其他的元素我都注明 “本文示例采用的是...”。

因为无论渲染器，还是几何体，以及其他元素，Three.js 都内置了非常多不同种类的元素构造函数，这个会在以后学习中逐渐详细说明举例。



### 第4环节：使用渲染器渲染出画面

**渲染画面**

就是根据第3环节中所创建出的 3D 场景，渲染出画面，并将画面内容填充到 canvas 中。

本文示例中，为了呈现 3D 动画，使用到了浏览器中 window.requestAnimationFrame() 这个函数。

> 关于 window.requestAnimationFrame() 的用法请参考：https://developer.mozilla.org/zh-CN/docs/Web/API/window/requestAnimationFrame



### 补充说明：3D动画是怎么动起来的？

默认情况下，渲染出的 3D 场景都是静止的，所谓 3D 动画，本质上是因为 “场景” 上发生了 “变化” 被渲染器不断重新渲染。

引起这些所谓 “变化”，简单可归纳为以下几种原因：

1. 镜头不变，但可见场景元素发生了变化，例如几何体发生了变化、网格角度发生了变化等
2. 可见场景元素不变，但是镜头发生了变化，例如镜头的推近、拉远等
3. 镜头变化了，同时场景元素也变化了......



## 示例代码

#### 第1步：创建并编写index.tsx代码内容

在 src/components/hello-threejs 目录下，创建 index.tsx 作为我们自定义的组件。

编写该组件对应的代码内容：

```
import React, { useRef, useEffect } from 'react'
import { WebGLRenderer, PerspectiveCamera, Scene, BoxGeometry, Mesh, DirectionalLight, MeshPhongMaterial } from 'three'

const HelloThreejs: React.FC = () => {
    const canvasRef = useRef<HTMLCanvasElement>(null)

    useEffect(() => {
        if (canvasRef.current) {
            //创建渲染器
            const renderer = new WebGLRenderer({ canvas: canvasRef.current })

            //创建镜头
            //PerspectiveCamera() 中的 4 个参数分别为：
            //1、fov(field of view 的缩写)，可选参数，默认值为 50，指垂直方向上的角度，注意该值是度数而不是弧度
            //2、aspect，可选参数，默认值为 1，画布的宽高比(宽/高)，例如画布宽300像素，高150像素，那么意味着宽高比为 2
            //3、near，可选参数，默认值为 0.1，近平面，限制摄像机可绘制最近的距离，若小于该距离则不会绘制(相当于被裁切掉)
            //4、far，可选参数，默认值为 2000，远平面，限制摄像机可绘制最远的距离，若超出该距离则不会绘制(相当于被裁切掉)
            //以上 4 个参数在一起，构成了一个 “视椎”，关于视椎的概念理解，暂时先不作详细描述。
            const camera = new PerspectiveCamera(75, 2, 0.1, 5)

            //创建场景
            const scene = new Scene()

            //创建几何体
            const geometry = new BoxGeometry(1, 1, 1)

            //创建材质
            //我们需要让立方体能够反射光，所以不使用MeshBasicMaterial，而是改用MeshPhongMaterial
            //const material = new MeshBasicMaterial({ color: 0x44aa88 })
            const material = new MeshPhongMaterial({ color: 0x44aa88 })

            //创建网格
            const cube = new Mesh(geometry, material)
            scene.add(cube)//将网格添加到场景中

            //创建光源
            const light = new DirectionalLight(0xFFFFFF, 1)
            light.position.set(-1, 2, 4)
            scene.add(light)//将光源添加到场景中，若场景中没有任何光源，则可反光材质的物体渲染出的结果是一片漆黑，什么也看不见

            //设置透视镜头的Z轴距离，以便我们以某个距离来观察几何体
            //之前初始化透视镜头时，设置的近平面为 0.1，远平面为 5
            //因此 camera.position.z 的值一定要在 0.1 - 5 的范围内，超出这个范围则画面不会被渲染
            camera.position.z = 2

            //渲染器根据场景、透视镜头来渲染画面，并将该画面内容填充到 DOM 的 canvas 元素中
            //renderer.render(scene, camera)//由于后面我们添加了自动渲染渲染动画，所以此处的渲染可以注释掉

            //添加自动旋转渲染动画
            const render = (time: number) => {
                time = time * 0.001 //原本 time 为毫秒，我们这里对 time 进行转化，修改成 秒，以便于我们动画旋转角度的递增
                cube.rotation.x = time
                cube.rotation.y = time
                renderer.render(scene, camera)
                window.requestAnimationFrame(render)
            }
            window.requestAnimationFrame(render)

        }
    }, [canvasRef])


    return (
        <canvas ref={canvasRef} />
    )
}

export default HelloThreejs
```

#### 第2步：添加对HelloThreejs组件的使用

修改 src/app.tsx 对应的代码：

```
import './App.scss'
import HelloThreejs from '@/components/hello-threejs';

const App = () => {
  return (
    <HelloThreejs />
  )
}

export default App;
```

#### 第3步：查看运行效果

```
yarn start
```

若无意外，你会在浏览器中看到一个 高150像素，宽300像素的 黑色场景，该场景上一直有一个 3D 立方体在旋转。

至此，我们的第一个 Three.js 示例完成。



## 如何让场景有多个立方体？

首先回忆一下 "01 Three.js简介.md" 中 “Three.js中的技术名词” 中关于 网格的介绍。

**网格：一种特定的 几何体和材质 绘制出的一个特定的几何体系。**

**网格包含的内容为：几何体、几何体的材质、几何体的自身网格坐标体系**

**在 Three.js 中，要牢记以下几个概念：**

* 一个几何体或材质，可以同时被多个网格使用(引用)
* 一个场景内，可以添加多个网格



**那和让场景中有多个立方体？**

答：使用相同或不同的几何体(立方体)，以及相同或不同的材质，去创建多个网格(特定的几何体)，然后将多个网格添加到同一个场景中。

> 注意：为了不同的立方体在场景中不叠加在一起，所以我们还要将网格(特定的几何体)的位置设置成不同的值。



#### 具体代码的修改：

1、我们假定继续使用原有示例中的立方体，因此创建几何体的代码不变。

2、为了凸显立方体的区别，我们将创建 3 个不同颜色的材质。

```diff
- //创建纹理
- const material = new MeshBasicMaterial({ color: 0x44aa88 })

+ //创建 3 个纹理
+ const material1 = new MeshPhongMaterial({ color: 0x44aa88 })
+ const material2 = new MeshPhongMaterial({ color: 0xc50d0d })
+ const material3 = new MeshPhongMaterial({ color: 0x39b20a })
```

3、创建 3 个网格，每个网格的水平位置不同

```diff
- //创建网格
- const cube = new Mesh(geometry, material)~~
- scene.add(cube)

+ //创建 3 个网格
+ const cube1 = new Mesh(geometry, material1)
+ cube1.position.x = -2
+ scene.add(cube1)//将网格添加到场景中

+ const cube2 = new Mesh(geometry, material2)
+ cube2.position.x = 0
+ scene.add(cube2)//将网格添加到场景中

+ const cube3 = new Mesh(geometry, material3)
+ cube3.position.x = 2
+ scene.add(cube3)//将网格添加到场景中
```



4、为了便于后面对于不同网格的循环修改，我们将创建包含 3 个网格的一个数组

```
const cubes = [cube1, cube2, cube3]
```

5、修改自动旋转渲染动画的相关代码

```diff
- cube.rotation.x = time
- cube.rotation.y = time
    
+ //通过 cube.map 循环遍历修改网格相关属性
+ cubes.map(cube => {
+     cube.rotation.x = time
+     cube.rotation.y = time
+ })
```



6、保存并重新执行 yarn start，若一切正常此时就会看到 画面中有 3 个不同颜色的立方体同时在做旋转动画。

> 目前 3 个立方体仅仅是颜色和位置不同，你可以尝试将立方体设置为不同的尺寸，不同的旋转频率等等，自己发挥吧。



是不是感觉自己对 Three.js 场景有进一步有所掌握 ^_^。

**下一节，我们将进一步改进这个示例代码。**

