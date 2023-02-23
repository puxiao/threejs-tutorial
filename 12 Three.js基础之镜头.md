# 12 Three.js基础之镜头

在之前所有的示例中，关于镜头，我们使用的都是 PerspectiveCamera(透视镜头)。

> 再次强调一下，我个人偏好是喜欢将 Camera 称为 “镜头”，但是 Three.js 官方或其他教程中称呼其为 “相机”

**在 Three.js 中，一共有 5 种镜头：**

| 镜头类型(都继承于Three.Camera) | 镜头名称 | 解释说明                                      |
| ------------------------------ | -------- | --------------------------------------------- |
| ArrayCamera                    | 镜头阵列 | 一组已预定义的镜头                            |
| CubeCamera                     | 立方镜头 | 6个面的镜头(前、后、左、右、顶、底)           |
| OrthographicCamera             | 正交镜头 | 无论物体距离镜头远近，最终渲染出的大小不变    |
| PerspectiveCamera              | 透视镜头 | 像人眼睛一样的镜头，远大近小，最常用的镜头    |
| StereoCamera                   | 立体镜头 | 双透视镜头，常用于创建 3D 立体影像或 视差屏障 |



**所有镜头的辅助对象都是：Three.CameraHelper**



由于 透视镜头(PerspectiveCamera) 是日常中使用最频繁的镜头类型，因此我们先从 透视镜头 开始讲解。



## 镜头的一些知识

> 我们通过透视镜头，来讲解一些镜头的知识  
> 透视镜头(PerspectiveCamera) 所呈现出的效果，和我们用有眼睛观察世界是一模一样的。

#### 平截面

无论所观察的物体是什么类型，例如球体、立方体、椎体等，在我们的视野中都会形成 2 个截面：

1. **远截面(far)：物体最远处的截面**
2. **近截面(near)：物体最近处的截面**

若以某个特定角度，当镜头(眼睛)观察物体时，物体远截面和近截面完全相同，那么此时近截面就会遮挡远截面，我们只能看到近截面。

例如我们在一个立方体的正前方，此时近截面完全遮挡住远截面，此时我们观察到的立方体更像是一个平面。

> 但是由于可能存在阴影，我们依然能够感知到这是一个 “3D立体物体”。



#### 视椎

想象一下，假设把我们的镜头(眼睛) 当做一个点，由这个点依次与物体的近截面、远截面的顶点进行连接，就会形成一个 椎体，而这个虚构出来的椎体，就是我们镜头(眼睛)与物体在空间上存在的视椎。

如果我们眼睛不是与物体，而是与 **“场景(Three.Scene)的近截面、远截面”** 形成的视椎，就是正常 Three.js 场景中可见空间。

请注意，上面提到的 场景的远截面和近截面 是加了引号，事实上没有办法直接设置场景的远近截面，场景的近远视椎是由镜头的以下几个参数最终计算出的：

1. 镜头的观察角度(fov)
2. 镜头画面的宽高比(aspect)
3. 镜头的最近可见距离(far)
4. 镜头的最远可见距离(near)
5. 一个隐含因素：镜头本身的位置(camera.position)

> 近截面和远截面决定了物体是否在镜头内可见  
> 物体与镜头的距离决定物体在视觉上的大小

当我们初始化一个透视镜头时，构造函数需要传递的 4 个参数，就是上面前 4 个元素。

**透视镜头默认参数值：fov=50、aspect=1、near=0.1、far=2000**



**补充说明：**

在 Three.js 官方文档中对以上 4 个参数的解释是：

1. fov：摄像机视椎体垂直视野角度
2. aspect：摄像机视椎体长宽比(宽高比)
3. near：摄像机视椎体近端面
4. far：摄像机视椎体远端面



> 虽然我的描述和官方描述文字上存在差异，但意思相同。我认为我的用词更加口语化，容易理解，所以在本系列文章中，我会继续使用我的描述语言。
>
> 因为受到自己对 Three.js 的理解程度，或许偏个人化语言描述或许是不正确的。



## 关于镜头近截面与远截面的补充说明：计算量与性能

我们知道透视镜头默认参数值：fov=50、aspect=1、near=0.1、far=2000，而我们之前文章中的示例，通常镜头设置参数为：new Three.PerspectiveCamera(45,2,0.1,1000)

> 也就是说，近截面(镜头最近可见距离)通常设置为 0.1、远截面(镜头最远可见距离)通常为 1000

若超出这个范围内的物体或物体局部则都将不可见。

#### 思考一下

假设我们直接将 near 由 0.1 修改为 0.0001、far 由 1000 修改为 1000000，那是不是场景最近可见度更加精细、可见范围变得更大，能够承载更多的物体呢？

答案是肯定的，但场景越大，可见度越微观，渲染所需计算量也越大。

**请记得：当计算量大到一定程度后，就会出现渲染异常，画面会出现一些意外的、不符合预期结果。**

**通常表现为物体表面像素紊乱、破碎、闪烁、像素前后失调，这是因为 GPU 没有足够的精度来确认哪些像素应该在前，哪些在后。**

> 我十分确信此刻我是在讲解 Three.js，而不是 大姨妈。



#### 解决办法(并不推荐)：将渲染器的logarithmicDepthBuffer设置为true

**logarighmicDepthBuffer：对数深度缓存器**

```
const renderer = new Three.WebGLRenderer({
  canvas:xxxx,
  logarithmicDepthBuffer:true
})
```



**注意事项：**

1. 通常电脑浏览器都已支持 logarithmicDepthBuffer 属性，但很多手机目前还不支持。
2. logarithmicDepthBuffer 为 true 只是在一定程度上能够缓解问题，但若 near 足够小、far 足够大时，依然会出现 GPU 计算精度不够，造成画面渲染紊乱。



#### 解决办法(推荐做法)：不解决

**请记得：你本就不应该把 near 设置过小、far 设置过大！**



#### near 和 far 正确的设定原则

尽可能让 near 和 far 更接近镜头不远的位置，当然前提是不让任何物体超出消失的范围内。

注意，这里的 “更接近镜头不远的位置” 是指 “合适、适当的位置”，并不是指小数点后精确多少位。

在保证精度的前提下，尽可能设置合适的 近截面和远截面，这样让 镜头与物体产生的视椎 “更小、更接近”，以节省渲染所需计算量和性能。



**举一个例子：**

假设你需要渲染出一个 足球 的特写，那么把足球放置在一个 比较小的平台或地面即可，而不是创建一个城市一样大小的场景，却只渲染出一个 足球的近距离特写。

> 但是假设你的场景确确实实需要非常大，此时就需要多参考网上其他人是如何处理类似场景的。  
> 或许后续文章中也会有讲解，此时此刻你只需知道 near 和 far 设置合适即可，没必要过于精细或巨大。



## 镜头示例1：使用CameraHelper来观察镜头

关于 透视镜头 PerspectiveCamera 我们之前示例中已经使用多次。

本示例主要演示 通过 镜头辅助对象(CameraHelper) 来观察镜头。



#### 思考一下：

在正常情况下，一个镜头只能看到别的物体但无法看到自己。

就好像我们的眼睛可以看到这个世界，但是眼睛本身自己没法看到自己(眼睛)。

> 当然除非对着镜子，不过对着镜子的本质依然是眼睛去看别的物体。

更加直白一点：就算你眼睛长得再大，你也做不到左眼直接看到自己右眼。

> 我们人虽然是 2 个眼球，但是在 Three.js 的相关举例中，是将 左右两个眼睛当成 是 1 个镜头来阐述的。



### HelloCamera示例目标

1. 创建一个包含物体的场景
2. 创建 2 个镜头，镜头A和镜头B
3. 将网页画面一分为二
4. 左侧显示 镜头A 所看到的场景
5. 右侧使用使用 镜头B 来观察 镜头A

**补充说明：**

1. 事实上是 镜头B 观察并显示 镜头A 对应的辅助对象 (CameraHelper)
2. 为了省事，我们直接使用前文讲解灯光时编写的 create-scene.ts 来创建场景



### 代码实现思路

#### 关键点 1：同一个场景渲染出 2 个不同的画面

**实现 2 个画面：**添加左右 2 个镜头，每个镜头设置不同，最终呈现出的场景不同

```
const leftCamera = new Three.PerspectiveCamera(45, 2, 5, 100)
leftCamera.position.set(0, 10, 20)

const rightCamera = new Three.PerspectiveCamera(60, 2, 0.1, 200)
rightCamera.position.set(40, 10, 30)
rightCamera.lookAt(0, 5, 0)
```



**实现 2 个交互：**添加 左右 2 个 div、2 个 OrbitControls，覆盖于canvas 之上

```
const leftControls = new OrbitControls(leftCamera, leftViewRef.current)
leftControls.target.set(0, 5, 0)
leftControls.update()

const rightControls = new OrbitControls(rightCamera, rightViewRef.current)
rightControls.target.set(0, 5, 0)
rightControls.update()

...

<div className='full-screen'>
    <div className='split'>
        <div ref={leftViewRef}></div>
        <div ref={rightViewRef}></div>
    </div>
    <canvas ref={canvasRef} />
</div>
```



**渲染 2 个画面：**使用渲染器的 裁减 功能

渲染器裁减功能，涉及到的方法有 setScissor()、setScissorTarget()、setViewport() 。

> 我们之前示例中，为了适应浏览器窗口大小的改变，我们使用过 渲染器的 setSize()

**setScissor ( x : Integer, y : Integer, width : Integer, height : Integer ) : null**
将剪裁区域设为(x, y)到(x + width, y + height) Sets the scissor area from

**setScissorTest ( boolean : Boolean ) : null**
启用或禁用剪裁检测. 若启用，则只有在所定义的裁剪区域内的像素才会受之后的渲染器影响。

**setViewport ( x : Integer, y : Integer, width : Integer, height : Integer ) : null**
将视口大小设置为(x, y)到 (x + width, y + height)



**请额外留意在后面实际代码中，我们定义的 setScissorForElement() 函数。**

> 多敲几遍，记住 setScissorForElement() 函数中获得裁减区域的代码套路



#### 关键点 2：镜头辅助对象

镜头辅助对象为 Three.CameraHelper，他的用法很简单：

```
const helper = new THREE.CameraHelper( leftCamera )
scene.add( helper )
```



#### 关键点 3：如何渲染

在之前所有的示例代码中，渲染代码都为：

```
renderer.render(scene, camera)
```

但本示例中，我们是同一个 scene，但是 2 个不同的 camera，因此与之对应的渲染代码也要发生变化。

需要依次分别渲染出 左侧镜头视角 和 右侧镜头视角。

```
//leftCamera一些更新操作
...
renderer.render(sceneRef.current, leftCamera)


//rightCamera一些更新操作
...
renderer.render(sceneRef.current, rightCamera)
```



### 具体的代码

#### create-scene.ts

我们直接使用之前 **“11 Three.js基础之灯光.md”** 中已写好的代码。



#### index.scss

```
.full-screen,canvas {
    display: block;
    height: inherit;
    width: inherit;
}

.split {
    position: fixed;
    display: flex;
    width: inherit;
    height: inherit;
}

.split div {
    width: inherit;
    height: inherit;
}
```



#### index.tsx

```
import { useRef, useEffect } from 'react'
import * as Three from 'three'
import createScene from '@/components/hello-light/create-scene'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

import './index.scss'

const HelloCamera = () => {

    const canvasRef = useRef<HTMLCanvasElement>(null)
    const sceneRef = useRef<Three.Scene | null>(null)
    const leftViewRef = useRef<HTMLDivElement>(null)
    const rightViewRef = useRef<HTMLDivElement>(null)

    useEffect(() => {
        if (canvasRef.current === null || leftViewRef.current === null || rightViewRef.current === null) {
            return
        }

        const renderer = new Three.WebGLRenderer({ canvas: canvasRef.current as HTMLCanvasElement })
        renderer.setScissorTest(true)

        const scene = createScene()
        scene.background = new Three.Color(0x000000)
        sceneRef.current = scene

        const light = new Three.DirectionalLight(0xFFFFFF, 1)
        light.position.set(0, 10, 0)
        light.target.position.set(5, 0, 0)
        scene.add(light)
        scene.add(light.target)

        const leftCamera = new Three.PerspectiveCamera(45, 2, 5, 100)
        leftCamera.position.set(0, 10, 20)

        const helper = new Three.CameraHelper(leftCamera)
        scene.add(helper)

        const leftControls = new OrbitControls(leftCamera, leftViewRef.current)
        leftControls.target.set(0, 5, 0)
        leftControls.update()

        const rightCamera = new Three.PerspectiveCamera(60, 2, 0.1, 200)
        rightCamera.position.set(40, 10, 30) //为了能够看清、看全镜头，所以将右侧镜头的位置设置稍远一些
        rightCamera.lookAt(0, 5, 0)

        const rightControls = new OrbitControls(rightCamera, rightViewRef.current)
        rightControls.target.set(0, 5, 0)
        rightControls.update()

        const setScissorForElement = (div: HTMLDivElement) => {
            if (canvasRef.current === null) {
                return
            }

            //获得 canvas 和 div 的矩形框尺寸和位置
            const canvasRect = canvasRef.current.getBoundingClientRect()
            const divRect = div.getBoundingClientRect()

            //计算出裁切框的尺寸和位置
            const right = Math.min(divRect.right, canvasRect.right) - canvasRect.left
            const left = Math.max(0, divRect.left - canvasRect.left)
            const bottom = Math.min(divRect.bottom, canvasRect.bottom) - canvasRect.top
            const top = Math.max(0, divRect.top - canvasRect.top)
            const width = Math.min(canvasRect.width, right - left)
            const height = Math.min(canvasRect.height, bottom - top)

            //将剪刀设置为仅渲染到画布的该部分
            const positiveYUpBottom = canvasRect.height - bottom
            renderer.setScissor(left, positiveYUpBottom, width, height)
            renderer.setViewport(left, positiveYUpBottom, width, height)

            //返回外观
            return width / height
        }

        const render = () => {

            if (leftCamera === null || rightCamera === null || sceneRef.current === null) {
                return
            }
            
            const sceneBackground = sceneRef.current.background as Three.Color

            //渲染 左侧 镜头
            const leftAspect = setScissorForElement(leftViewRef.current as HTMLDivElement)

            leftCamera.aspect = leftAspect as number
            leftCamera.updateProjectionMatrix()

            helper.update()
            helper.visible = false

            sceneBackground.set(0x000000)
            renderer.render(sceneRef.current, leftCamera)

            //渲染 右侧 个镜头
            const rightAspect = setScissorForElement(rightViewRef.current as HTMLDivElement)

            rightCamera.aspect = rightAspect as number
            rightCamera.updateProjectionMatrix()

            helper.visible = true

            sceneBackground.set(0x000040)
            renderer.render(sceneRef.current, rightCamera)

            window.requestAnimationFrame(render)
        }
        window.requestAnimationFrame(render)

        const handleResize = () => {
            if (canvasRef.current === null) {
                return
            }

            const width = canvasRef.current.clientWidth
            const height = canvasRef.current.clientHeight

            renderer.setSize(width, height, false)
        }
        handleResize()
        window.addEventListener('resize', handleResize)

        return () => {
            window.removeEventListener('resize', handleResize)
        }
    }, [canvasRef])

    return (
        <div className='full-screen'>
            <div className='split'>
                <div ref={leftViewRef}></div>
                <div ref={rightViewRef}></div>
            </div>
            <canvas ref={canvasRef} />
        </div>
    )
}

export default HelloCamera
```



执行以后，就会看到浏览器中左右 2 个可交互的画面。其中右侧画面中包含左侧灯光辅助对象。



## 镜头示例2：OrthographicCamera

第二个比较经常用的镜头是 OrthographicCamera(正交镜头)。

正交镜头与透视镜头最大的区别点在于：

1. 正交镜头的视椎体不是 椎体，而是立方体

2. 正交镜头看到的都是一个“面”

3. 因此，正交镜头没有 “透视(近大远小)”这个概念

   > 我对于以上 2 点理解并不深，先记住以后再慢慢研究



#### OrthographicCamera的用途

1. 用途1：作为 2D 画布
2. 作为 3D 建模程序 的 上、下、左、右、前、后 视图。



#### OrthographicCamera基本用法

```
const camera = new Three.OrthographicCamera(-1, 1, 1, -1, 5, 50)
camera.zoom = 0.2
camera.position.set(0,10,20)
```

初始化时，构造函数内的参数依次是：

OrthographicCamera( left : Number, right : Number, top : Number, bottom : Number, near : Number, far : Number )

1. left：视椎体左侧面
2. right：视椎体右侧面
3. top：视椎体顶面
4. bottom：视椎体底面
5. near：视椎体近端面
6. far：视椎体远端面



**从实际的角度来看，一定要注意以下几点：**

1. left 的值不能大于 right，同理 bottom 的值不能大于 top。 如果没有按照这个约定，例如 bottom 大于 top 相当于颠倒了相机。
2. near 设置越小，投影的映像越大
3. left 与 right 之间的距离、top 与 bottom 之间的距离的比例一定要和 canvas 比例相同，否则会导致投影的物体形状变形



#### 正交镜头与透视镜头的几点不同地方

**渲染时，对应的设置不同。**

透视镜头渲染时，需要修改的是 camera.aspect = newAspect

正交镜头渲染时，需要修改的是 camera.left = - new Aspect、camera.right = new Aspect



### 假设我们在 HelloCamera 示例中使用 OrthographicCamera

**需要修改的地方为：**

```diff
- const leftCamera = new Three.PerspectiveCamera(45, 2, 5, 100)
- leftCamera.position.set(0, 10, 20)
+ const leftCamera = new Three.OrthographicCamera(-1, 1, 1, -1, 5, 50)
+ leftCamera.zoom = 0.2
  leftCamera.position.set(0,10,20)

...

  const leftAspect = setScissorForElement(leftViewRef.current as HTMLDivElement)
- leftCamera.aspect = leftAspect as number
+ leftCamera.left = -(leftAspect as number)
+ leftCamera.right = leftAspect as number
  leftCamera.updateProjectionMatrix()
```

其他代码无需修改，发布调试，即可看到正交镜头辅助对象，此时的视椎不再是椎体，而是一个立方体。



关于其他镜头：ArrayCamera、CubeCamera、StereoCamera 本文不再讲解，以后用到的时候再深入研究。

具体的用法，可查阅：https://threejs.org/docs/index.html#api/zh/cameras/Camera



至此，关于镜头的基础知识讲解完毕。

> 虽然本文是在讲镜头，但本文的核心知识点却是在讲 渲染器的裁切 功能。
>
> 一定要多多复习，熟练掌握 渲染器的 setScissor()、setScissorTest()、setViewport() 方法。



下一节，我们将讲解 Shardown(阴影)

