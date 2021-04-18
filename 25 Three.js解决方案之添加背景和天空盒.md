# 25 Three.js解决方案之添加背景和天空盒

在我们之前所有演示的案例中，场景中的背景往往使用默认的黑色，或者是其他纯颜色。

下面我们讲解一下 Three.Screen 的背景设置方式。



<br>

## 设置场景背景

场景(Three.Screen)有一个属性 .background。我们可以通过设置这个属性来给场景添加背景。



<br>

**.background属性值类型**

场景背景属性值一共有 3 种类型：

1. 默认为 null

   > .background 属性值为 null，场景显示为黑色

2. 某颜色 Three.Color

   > Three.Color 可接受 字符串或数字类型的颜色值，例如:
   >
   > 1. new Three.Color('#333')
   > 2. new Three.Color('green')
   > 3. new Three.Color(0x333333)

3. 某纹理 Three.Texture



<br>

**设置背景色示例代码：**

```
const scene = new Three.Scene()
scene.background = new Three.Color(0x333333)
```



<br>

**设置背景纹理图片示例代码：**

```
const scene = new Three.Scene()

const textureLoader = new Three.TextureLoader()
scene.background = textureLoader.load(require('@/assets/imgs/blue_sky.jpg').default)
```

或者是

```
const textureLoader = new Three.TextureLoader()
textureLoader.load(require('@/assets/imgs/blue_sky.jpg').default, (texture) => {
    scene.background = texture
})
```



<br>

上面设置场景纹理背景图，实际运行后你会发现虽然背景图显示了，但是背景图却有可能是变形着的。

这是由于背景图片本身就一个宽高比，而画布(Canvas)本身也有一个宽高比。

> 实际上是渲染器渲染尺寸的宽高，例如每次浏览器窗口尺寸发生变化时，我们都会重新设置 渲染尺寸
>
> ```
> renderer.setSize(width, height, false)
> ```



<br>

**判断高宽比，不让背景图变形且可以铺满整个背景**

假设我们不能接受背景图变形，那么我们就需要计算一下 2 者的宽高比，然后找出合适的比例进行修改。

这个不让背景图变形的计算过程是：

1. 计算出画布宽高比，例如 canvasAspect
2. 计算出背景图宽高比，例如 imgAspect
3. 然后计算 imgAspect/canvasAspect，得到 最终背景图在不变形的前提下的缩放比，例如 const resultAspect = imgAspect / canvasAspect
4. 然后依次设置背景图纹理的偏移(offset.x、offset.y)，以及判断是否需要重复平铺背景图(repeat.x、repeat.y)



<br>

**示例代码如下：**

```
const textureRef = useRef<Three.Texture | null>(null)

...

const textureLoader = new Three.TextureLoader()
textureLoader.load(require('@/assets/imgs/blue_sky.jpg').default, (texture) => {
    textureRef.current = texture
    scene.background = textureRef.current
    handleResize() //此处是当纹理图片加载完成后，需要调用执行一下 handleResize()
})

...

const handleResize = () => {
    const canvasAspect = width / height  //第1步：计算出画布宽高比
    if (textureRef.current !== null) {
        const bgTexture = textureRef.current
        const imgAspect = bgTexture.image.width / bgTexture.image.height  //第2步：计算出背景图宽高比

        const resultAspect = imgAspect / canvasAspect  //第3步：计算出最终背景图宽缩放宽高比

        //第4步：设置背景图纹理的偏移和重复
        bgTexture.offset.x = resultAspect > 1 ? (1 - 1 / resultAspect) / 2 : 0
        bgTexture.repeat.x = resultAspect > 1 ? 1 / resultAspect : 1

        bgTexture.offset.y = resultAspect > 1 ? 0 : (1 - resultAspect) / 2
        bgTexture.repeat.y = resultAspect > 1 ? 1 : resultAspect
    }
}
```



<br>

**完整的示例代码如下：**

```
import { useRef, useEffect } from "react"
import * as Three from 'three'
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader"

import './index.scss'

const HelloSkybox = () => {
    const canvasRef = useRef<HTMLCanvasElement | null>(null)
    const textureRef = useRef<Three.Texture | null>(null)

    useEffect(() => {

        if (canvasRef.current === null) { return }

        const renderer = new Three.WebGLRenderer({ canvas: canvasRef.current })

        const scene = new Three.Scene()
        const textureLoader = new Three.TextureLoader()
        textureLoader.load(require('@/assets/imgs/blue_sky.jpg').default, (texture) => {
            textureRef.current = texture
            scene.background = textureRef.current
            handleResize()
        })
        scene.background = textureRef.current

        const camera = new Three.PerspectiveCamera(45, 2, 0.1, 100)
        camera.position.set(10, 0, 10)

        const light = new Three.HemisphereLight(0xFFFFFF, 0x333333, 1)
        scene.add(light)

        const loader = new GLTFLoader()
        loader.load(require('@/assets/model/hello.glb').default, (gltf) => {
            scene.add(gltf.scene)
        })

        const control = new OrbitControls(camera, canvasRef.current)
        control.update()

        const render = () => {
            renderer.render(scene, camera)
            window.requestAnimationFrame(render)
        }
        window.requestAnimationFrame(render)

        const handleResize = () => {
            if (canvasRef.current === null) { return }

            const width = canvasRef.current.clientWidth
            const height = canvasRef.current.clientHeight
            const canvasAspect = width / height

            if (textureRef.current !== null) {
                const bgTexture = textureRef.current
                const imgAspect = bgTexture.image.width / bgTexture.image.height

                const resultAspect = imgAspect / canvasAspect

                bgTexture.offset.x = resultAspect > 1 ? (1 - 1 / resultAspect) / 2 : 0
                bgTexture.repeat.x = resultAspect > 1 ? 1 / resultAspect : 1

                bgTexture.offset.y = resultAspect > 1 ? 0 : (1 - resultAspect) / 2
                bgTexture.repeat.y = resultAspect > 1 ? 1 : resultAspect
            }

            camera.aspect = canvasAspect
            camera.updateProjectionMatrix()
            renderer.setSize(width, height, false)
        }
        handleResize()
        window.addEventListener('resize', handleResize)

        return () => {
            window.removeEventListener('resize', handleResize)
        }
    }, [])

    return (
        <canvas ref={canvasRef} className='full-screen' />
    )
}

export default HelloSkybox
```



<br>

请注意，上面讲述的是将背景图片加载进 Three.js 中，并当做纹理来使用。我们可以通过修改纹理各种属性来修改和控制背景图。

> 上面示例代码中仅仅是对纹理的偏移和重复进行了设置

但是，假设就仅仅为了达到上述效果，实际上我们根本不用搞这么复杂，直接给网页中 <canvas\> 标签设置一个背景图片即可。



<br>

**第1步：添加渲染器参数 alpha:true**

```
const renderer = new Three.WebGLRenderer({ canvas: canvasRef.current, alpha: true })
```



<br>
**第2步：给画布标签(<canvas\>)添加背景图**

第1种添加方式：通过 css 添加

```
.full-screen {
    display: block;
    width: inherit;
    height: inherit;
    background: url(./imgs/blue_sky.jpg) no-repeat center center;
    background-size: cover;
}
```

> 请注意：上面 .scss 中我们给背景设置的图片路径，其实指向项目的 public 目录



<br>

第2种添加方式：通过 JS 添加

```
const canvasStyle = {
    background: `url(${require('@/assets/imgs/blue_sky.jpg').default}) center center no-repeat`,
    backgroundSize: 'cover'
}
    
<canvas ref={canvasRef} className='full-screen' style={canvasStyle} />
```

> React 在编译时，会自动将 `style={canvasStyle}` 中的样式转化为 CSS  样式



<br>

至此，关于如何设置场景背景图片，讲解完毕。

接下来要讲解一个常见的 Three.js 应用场景：SkyBox(天空盒)。



<br>

## 天空盒(Skybox)

假设我们身处一个立方体内部，我们可以观察到立方体内部 6 个面的背景贴图。

> 这不就是我们身处某个房间内吗？

这类应用场景，通常被称呼为 Skybox，也就是 天空盒。

这也是我们日常听到对最多的  Web 3D 应用：VR 看房



<br>

上面对于天空盒的解释正确吗？

答：正确但不严谨！



<br>

通常我们所说天空盒(Skybox) 一个非常重要的特性就是：像天空一样大的盒子

进一步解释就是：这个盒子空间像天空一样无边无际，永远不到头。

> 说直白点，天空盒就好像我们平时的场景(Three.Scene)，无论缩小到什么限度，还是放大到什么限度，永远走不出场景之外。

而本文下面所有的示例，其实都是针对场景背景添加纹理贴图，所以下面示例中的天空盒(skybox)空间等同于场景本身。



<br>

**天空盒一共有 2 种形式的贴图资源：**

1. 全景图(hdri)，又名 天空图
2. 立方体贴图(cubemap)



<br>

#### 第1种实现天空盒的方法：全景图(hdri)

很明显，我们最容易想到的实现方式为：

1. 我们把整个 Three.Screen 当做立方体，也就是将整个场景当做立方体

   > 再次重复一遍：我们并不是在场景中创建一个立方体，而是直接将整个场景当做立方体
   >
   > 假设你要给场景中某个立方体设置类似的效果，那么你要做的事情是：
   >
   > 1. 创建纹理，使用立方体纹理加载器(Three.CubeTextureLoader)加载图片资源
   >
   > 2. 创建材质，除了设置材质的纹理之外，还要设置 .side 属性，将值为 Three.BackSide，例如
   >
   >    ```
   >    new MeshPhongMaterial({ map:xxxx, side: Three.BackSide })
   >    ```
   >
   > 3. 最终创建立方体网格(Three.Mesh)

   > 请注意：绝大多数 VR 看房，都是将场景当做立方体即可。

2. 按照指定顺序，获取(加载) 6 个面的纹理贴图，得到纹理

   > 请注意，这次加载我们并不使用 Three.TextureLoader，而是采用立方体专有的纹理加载器 Three.CubeTextureLoader

3. 将得到的纹理作为场景背景

4. 设置相机坐标 z 的值，确保我们可以看到物体，例如

   ```
   camera.position.set(0, 0, 10)
   ```

5. 然后正常渲染，我们就会感觉此刻身在房间中



<br>

**示例代码：**

房间图片素材：

我们使用网上找到的某房间 6 个面的纹理贴图

1. https://threejsfundamentals.org/threejs/resources/images/cubemaps/computer-history-museum/pos-x.jpg
2. https://threejsfundamentals.org/threejs/resources/images/cubemaps/computer-history-museum/neg-x.jpg
3. https://threejsfundamentals.org/threejs/resources/images/cubemaps/computer-history-museum/pos-y.jpg
4. https://threejsfundamentals.org/threejs/resources/images/cubemaps/computer-history-museum/neg-y.jpg
5. https://threejsfundamentals.org/threejs/resources/images/cubemaps/computer-history-museum/pos-z.jpg
6. https://threejsfundamentals.org/threejs/resources/images/cubemaps/computer-history-museum/neg-z.jpg



<br>

实际代码：

```
const cubeTextureLoader = new Three.CubeTextureLoader()
cubeTextureLoader.load([
    'https://threejsfundamentals.org/threejs/resources/images/cubemaps/computer-history-museum/pos-x.jpg',
    'https://threejsfundamentals.org/threejs/resources/images/cubemaps/computer-history-museum/neg-x.jpg',
    'https://threejsfundamentals.org/threejs/resources/images/cubemaps/computer-history-museum/pos-y.jpg',
    'https://threejsfundamentals.org/threejs/resources/images/cubemaps/computer-history-museum/neg-y.jpg',
    'https://threejsfundamentals.org/threejs/resources/images/cubemaps/computer-history-museum/pos-z.jpg',
    'https://threejsfundamentals.org/threejs/resources/images/cubemaps/computer-history-museum/neg-z.jpg'
], (texture) => {
    scene.background = texture
})
```



<br>

**针对贴图资源顺序的补充说明：**

在上面示例代码中，我们可以看到加载立方体 6 面图片贴图资源的顺序是固定的，依次为：

pos-x.jpg、neg-x.jpg、pos-y.jpg、neg-y.jpg、pos-z.jpg、neg-z.jpg

<br>

**"pos" 是单词 "positive" 的缩写，在 3D 坐标系中意思是 正**

> positive：积极的、正向的、乐观的

**"neg" 是单词 "negative" 的缩写，在 3D 坐标系中意思是 负**

> negative：消极的、负面的



<br>

### 左手坐标系统 VS 右手坐标系统

我们先介绍一下 右手坐标系统，你就自然明白什么是左手坐标系统了。

我们看一下百度百科的介绍：

右手系(right-hand system)是在空间中规定直角坐标系的方法之一。此坐标系中x轴，y轴和z轴的正方向是如下规定的：把右手放在原点的位置，使大拇指，食指和中指互成直角，把大拇指指向x轴的正方向，食指指向y轴的正方向时，中指所指的方向就是z轴的正方向。



<br>

同样的操作你换做左手，那么就是左手坐标系统。

**左手坐标系统和右手坐标系统在 Y 轴、Z 轴 方面没有区别，但是在 X 轴上是彼此相反的。**



<br>

**对于立方体贴图，使用的是左手系统！**

因此上面图片名称的含义为：

| 名称  | 含义       | 对应立方体内部的面来说 | 对于站在立方体内部中间的人来说 |
| ----- | ---------- | ---------------------- | ------------------------------ |
| pos-x | X 轴正方向 | 左面                   | 视觉左方                       |
| neg-x | X 轴负方向 | 右面                   | 视觉右方                       |
| pos-y | Y 轴正方向 | 上面                   | 视觉上方                       |
| neg-y | Y 轴负方向 | 下面                   | 视觉下方                       |
| pos-z | Z 轴正方向 | 后面                   | 视觉后方                       |
| neg-z | Z 轴负方向 | 前面                   | 视觉前方                       |

> 请注意，立方体的前面或后面完全是由观察者所处的位置来决定的。
>
> 如果你在立方体外面去看立方体，那么正面看到的是立方体的正面，看不到的是立方体的背面。
>
> 但是我们现在做的是站在立方体内部去观察立方体，所以此刻 “立方体的正面” 实际上对于我们观察者而言是在我们身后，也就是我们的视觉后方。



<br>

**对于Three.js渲染使用的是 右手坐标系统！**

**不过你不用担心左右方向相反这个事情，因为在 Three.js 内部在渲染的时候会自动帮我们将左右对调。**

> 重复一遍：
>
> 1. 一般立方体模型贴图使用左手坐标系统
> 2. Three.js 整体使用右手坐标系统
> 3. 但在渲染立方体内部贴图时，Three.js 会自动帮我们做好左右兑换
> 4. 因此我们在传递纹理贴图时，贴图顺序使用的是左手坐标系统



<br>

> 糟糕，我也是今天学习到这里才彻底明白坐标系统，我可能在之前的文章中对于 上下左右前后 讲解错了，但是我暂时记不清是哪一章节。



<br>

#### 第2种实现天空盒的方法：立方体贴图(cubemap)

我们第1种实现天空盒，实际上使用的是 6 个面的图片资源组合成了一个 3D 空间。

接下来我们学习使用一张 360° 球形相机拍摄的照片，来实现 3D 空间立方体。



<br>

首先你从网上找到一张 360° 的场景图片资源：

https://threejsfundamentals.org/threejs/resources/images/equirectangularmaps/tears_of_steel_bridge_2k.jpg

> 请注意，这类图片尺寸宽高比例为 2:1，经常称呼这类图片为 “全景图”



<br>

**实现思路：**

1. 使用纹理加载器加载该图片资源

   > 这种 2:1 的图片，被称为 “等矩形图像”

2. 实例化一个 Three.WebGLCubeRenderTarget，构造函数中的 size 属性为图片资源的高

   > WebGLCubeRenderTarget 继承于 WebGLRenderTarget，属于离屏渲染的一种特例(专门针对立方体模型)

   > 在 WebGLRenderTarget 的源码中可以看到这句代码：super( size, size, options );
   >
   > WebGLRenderTarget 构造函数需要传递 width 和 height，但是 WebGLCubeRenderTarget 构造函数只需传入 1 个 size，因为正方体，所以宽高一样。

3. 调用该实例化对象的 fromEquirectangularTexture() 函数

   > 将等距图像 转化为 立方体模型贴图
   >
   > > 可以简单理解成：就是将 1 整张图片转化为 6个面的立方体模型贴图，并进行渲染

4. 将场景背景设置为该实例对象

   > 这样相当于将场景背景图的值设置为 WebGLCubeRenderTarget  的渲染结果



<br>

**具体代码：**

```
const textureLoader = new Three.TextureLoader()
textureLoader.load(require('@/assets/imgs/tears_of_steel_bridge.jpg').default,
    (texture) => {
        const crt = new Three.WebGLCubeRenderTarget(texture.image.height)
        crt.fromEquirectangularTexture(renderer,texture)
        scene.background = crt.texture
    }
)
```

> 补充说明：scene.background 的类型为 WebGLBackground
>
> 请留意上述代码中 `scene.background = crt.texture`，事实上在以前的一些教程中可以写成 `scene.background = crt`，WebGLBackground 会在内部进行判断，如果 background 类型为 WebGLRenderTarget，则使用该实例的 .texture 属性值。
>
> 但是在最新版 r127 中已经删除了该判断代码，所以现在必须写成 `scene.background = crt.texture`。
>
> 就这个问题，我已向官网教程进行了修改提交：https://github.com/gfxfundamentals/threejsfundamentals/pull/205



<br>

### 补充说明：全景图(hdri) 与 立方体贴图(cubemap) 互转

网上有人提供了 全景图与立方体模型图 之间的转化工具包：

在线地址：https://matheowis.github.io/HDRI-to-CubeMap/

项目源码：https://github.com/aunyks/hdri-to-cubemap



<br>

你以为本文结束了？没有！

我们上面示例都是 天空盒(skybox)，那如果是真的一个立方体呢？

> 天空盒 是没有尺寸，空间无限大的，而普通立方体则是有尺寸的。

下面示例我们将创建一个立方体，然后对立方体内部进行贴图，并渲染和观察立方体盒子内部。



<br>

## 普通立方体内部贴图和渲染



