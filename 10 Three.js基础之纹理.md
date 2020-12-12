# 10 Three.js基础之纹理

### 纹理(Texture)概要

前面学习的 材质(material) 更多是表达一个物体表面的物理特性和一些简单的外观颜色。

而 纹理(Texture) 则专门来设置物体表面贴合的彩色图片的，具体做法就是：

1. 使用纹理加载器 TextureLoader 加载外部图片(.jpg或.png)
2. 通过设置 物体的 .map 属性，将加载得到的外部图片贴合在物体表面

> 有些 3D 软件教程会直接将 纹理 称呼为 皮肤、贴图



**纹理加载器的简单示例代码：**

```
const loader = new Three.TextureLoader()
const material = new Three.MeshBasicMaterial({
  map:loader.load('xxx/xx.jpg')
})
```



**补充一点：**

1. 在汉语词语中，纹最初指 乌龟壳上的纹路、理最初指 石头上的纹路和细腻程度
2. 在物理中，纹理指物体表面凸凹不平的沟纹
3. 在 Three.sj 中，纹理指 物体光滑表面上的彩色图案



由于纹理牵扯到 图片资源加载，所以我们先补充一下 react 引入图片资源的相关知识。

## 在React+TypeScript中引入图片资源

在 React + TypeScript 中引入本地图片资源的方法如下。

#### **第一件要做的事情：src/global.d.ts**

```
declare module '*.png';
declare module '*.gif';
declare module '*.jpg';
declare module '*.jpeg';
declare module '*.svg';
declare module '*.css';
declare module '*.less';
declare module '*.scss';
declare module '*.sass';
declare module '*.styl';
declare module 'react-app-rewire-alias';
```

我们先要确保项目源码中 src/global.d.ts 中有针对 图片资源 的声明模块。

例如：`declare module '*.jpg';` 是告诉 TypeScript ，允许在 React 代码中 引入 jpg 格式的文件模块



**补充说明：**

 ```
declare module '*.jpg';
 ```

上面那行声明代码是一种简写，实际他表示的代码为：

```
declare module '*.jpg' {
  const content: string;
  export default content;
}
```



#### 第1种：使用 import 引入和使用图片

假设图片文件位置为 src/assets/imgs/mapping.jpg，那么引入该图片文件的代码为：

```
import imgSrc from '../assets/imgs/mapping.jpg'
```

使用该图片，代码为：

```
<img src={imgSrc}>
```

> 请注意 imgSrc 的实际类型为 string，imgSrc 是 react 编译后该图片资源对应的位置

> 经过 React 编译之后，图片位置最终变为：static/media/mapping.xxxxxxxx.jpg



#### 第2种：使用 require 引入和使用图片

不需要在顶部 import 图片资源，而是直接使用：require('xxx').default 的形式获得图片资源最终路径：

```
<img src={require('../assets/imgs/mapping.jpg').default}>
```

> 请注意，一定要 要有.default 才可以 

> 当需要批量引入很多张图片时，或者动态获得 引入的图片地址时，第 2 种写法就非常方便



**问题延展：**

你是否还记得在讲解 图元 中 TextBufferGeometry 时候，当时字体数据也是靠加载，当时演示的时候加载的是一个网络资源：

```
const url = 'https://threejsfundamentals.org/threejs/resources/threejs/fonts/helvetiker_regular.typeface.json'
```

那如果 字体数据文件 .json 在项目本地资源中，又该如何加载呢？

1. 首先可以肯定的是，引入图片资源的方式并不适用于 .json 文件资源

   > .json 文件 对应的 webpack 加载器为 json-loader，他是可以直接将 json 中数据内容解析出来直接给我们使用的
   >
   > .jpg 文件对应的是 webpack 加载器为 file-loader，他仅仅是将资源更换目录和重命名，并不会对图片(图片也不需要)内容解析的

2. 那该怎么引入？

这个问题我们知识先思考一下，暂时先不去讲如何解决。

让我们回到 HelloTexture 中。



## 示例1：加载一张图，实现一个有贴图的立方体

#### 示例目标

1. 创建一个正方体
2. 通过 TextureLoader 加载一张外部图片
3. 将图片贴在立方体的 6 个面上



#### 具体示例编写

**第1步、添加纹理加载所需的图片资源**

网上随便找了一张风景图，存放在项目 src/assets/imgs 目录中，文件名 mapping.jpg



**第2步：添加图片目录对应的 alias 配置**

修改项目中 tsconfig.paths.json，向  paths 中添加：

```
 "@/assets/imgs/*": ["./src/assets/imgs/*"]
```



**第3步：编写 HelloTexture**

在项目 src/components/hello-texture/ 中创建 index.tsx，代码内容：

```
import { useEffect, useRef } from 'react'
import * as Three from 'three'

import './index.scss'
import imgSrc from '@/assets/imgs/mapping.jpg' //引入图片资源

const HelloTexture = () => {

    const canvasRef = useRef<HTMLCanvasElement>(null)

    useEffect(() => {
        if (canvasRef.current === null) {
            return
        }

        const renderer = new Three.WebGLRenderer({ canvas: canvasRef.current as HTMLCanvasElement })

        const camera = new Three.PerspectiveCamera(40, 2, 0.1, 1000)
        camera.position.set(0, 0, 40)

        const scene = new Three.Scene()
        scene.background = new Three.Color(0xcccccc)

        //创建一个 纹理加载器
        const loader = new Three.TextureLoader()
        //创建一个材质，材质的 map 属性值为 纹理加载器加载的图片资源
        const material = new Three.MeshBasicMaterial({
            map: loader.load(imgSrc) //loader.load('xxx.jpg')返回值为Three.Text类型实例
        })

        const box = new Three.BoxBufferGeometry(8, 8, 8)
        const mesh = new Three.Mesh(box, material)
        scene.add(mesh)

        const render = (time: number) => {
            time = time * 0.001

            mesh.rotation.x = time
            mesh.rotation.y = time
            renderer.render(scene, camera)

            window.requestAnimationFrame(render)
        }
        window.requestAnimationFrame(render)

        const resizeHandle = () => {
            const canvas = renderer.domElement
            camera.aspect = (canvas.clientWidth / canvas.clientHeight)
            camera.updateProjectionMatrix()
            renderer.setSize(canvas.clientWidth, canvas.clientHeight, false)
        }
        resizeHandle()
        window.addEventListener('resize', resizeHandle)

        return () => {
            window.removeEventListener('resize', resizeHandle)
        }
    }, [canvasRef])

    return (
        <canvas ref={canvasRef} className='full-screen' />
    )
}

export default HelloTexture
```

查看最终效果，终端中执行：yarn start

若一切正常，就会看到一个包含风景贴图的立方体。



## 示例2：加载多张图片，实现一个骰子

在上面的示例中，我们是加载了一张图片资源，然后立方体默认 6 个面都采用这张图来进行贴图渲染。

那么假设我们希望立方体每个面都使用不同的图片来进行贴图，还如何实现呢？



#### 骰子的特征就是：立方体 6 个面贴图均不相同

> 骰(tóu)子，也就是 色(shǎi)子。



### 实现方式

在之前的代码中，我们创建物体，也就是网格的代码是：

```
const mesh = new Three.Mesh(box, material)
```

请注意，上述代码中 material 是单个材质，如果我们想实现将立方体各个面都使用不同材质，**我们可以将 material 替换为一个 数组(数组元素为 6 个 material ) 即可**。



### 第1步：添加 骰子 的 6 个面图片资源

假设我们已经有 骰子 6 个面的图片资源，分别是：

1. src/assets/imgs/dice0.jpg
2. ....
3. src/assets/imgs/dice5.jpg

**其中 dice0.jpg 对应骰子 1点、...、dice5.jpg 对应骰子 6点**



### 第2步：引入 6 个图片资源，并创建材质数组

#### 第1种方式：使用 import 引入图片：

**依次引入 6 个图片资源**

```
import imgSrc0 from '@/assets/imgs/dice0.jpg'
import imgSrc1 from '@/assets/imgs/dice1.jpg'
import imgSrc2 from '@/assets/imgs/dice2.jpg'
import imgSrc3 from '@/assets/imgs/dice3.jpg'
import imgSrc4 from '@/assets/imgs/dice4.jpg'
import imgSrc5 from '@/assets/imgs/dice5.jpg'
```

**创建材质数组，元素为骰子的6个面对应的材质**

```
//创建一个纹理加载器
const loader = new Three.TextureLoader()

const imgSraArr = [imgSrc0, imgSrc1, imgSrc2, imgSrc3, imgSrc4, imgSrc5]
//创建一组材质，每个材质对应立方体每个面所用到的材质
const materialArr: Three.MeshBasicMaterial[] = []
imgSraArr.forEach((src) => {
  materialArr.push(new Three.MeshBasicMaterial({
    map: loader.load(src)
  }))
})
```



#### 第2种方式：使用 require 引入图片：

实际上使用的是 for循环 + requrie + 字符串模板 生成出图片资源地址

```
//不需要在顶部 import xxx from 'xxx.jpg'

//创建一个 纹理加载器
const loader = new Three.TextureLoader()

//创建 6 个面对应的材质
const materialArr: Three.MeshBasicMaterial[] = []
for (let i = 0; i < 6; i++) {
    materialArr.push(new Three.MeshBasicMaterial({
        map: loader.load(require(`@/assets/imgs/dice${i}.jpg`).default)
    }))
}
```



### 第3步：创建网格时，使用 材质数组 materialArr

```
const box = new Three.BoxBufferGeometry(8, 8, 8)
const mesh = new Three.Mesh(box, materialArr) //注意，此处使用的不再是单个材质，而是一个材质数组
scene.add(mesh)
```



最终实际运行后，立方体 6 个面分别贴上不同数字的图片，一个 骰子 效果实现了。

你以为完事了？



### 冷静观察并思考：骰子数字的分布

**经过观察，你会发现，骰子数字目前的分布为：**

1. 数字 1 对面是 数字 2
2. 数字 3 对面是 数字 4
3. 数字 5 对面是 数字 6



**现实中的骰子数字分布规则：**

1. 数字 1 对面是 数字 6
2. 数字 2 对面是 数字 5
3. 数字 3 对面是 数字4

简而言之就是 对立面 2 边的数字相加必须等于 7。

如果想模拟出真实的骰子数字分布，那么就要按照这个规则去修改图片编号对应的图片数字。



### 加载多张图作为材质的补充说明：

上面示例 2 中演示了 加载 6 张图，并贴合到 立方体的 6 个面上。现在，我们思考以下几个问题。



**问题1：假设立方体 6 个面，但是材质只有 5 个 或者 4 个，会出现什么情况呢？**

答：缺少 N 个面的材质，立方体就会有 N 个面直接显示为空白，当然你也可以说显示为 “透明”、"缺失"。



**创建网格时：**

* **若 Three.Mesh(xx,xx) 构造函数中第 2 个参数值是 单个 材质，则图元实例上所有的面均采用该材质(纹理)**
* **若第 2 个参数值是 材质组成的数组，则该数组中材质数量不能少于图元实例各个可渲染面的数量。**



**问题2：立方体是 6 个可渲染的面，那其他图元都有几个可渲染的面？**

答：**支持多个材质纹理的图元，可需要渲染的面数量也不同**，例如圆锥体只需要 2 个面(圆锥侧面、圆锥底面)、圆柱体 一共 3 个面，也就是需要 3 个材质贴图(圆柱上下 2 个底 + 圆柱侧面)。

不是所有材质都支持多个材质纹理的，例如平面圆(CircleBufferGeometry)、平面矩形(PlaneBufferGeometry) 都仅只支持 1 个材质纹理。



**问题3：Three.js 纹理是否支持精灵图(雪碧图)？**

> 精灵图也被称为雪碧图，比如在网页 CSS 中，可以将多个小图标图片合并放在一张图片上，当不同地方需要使用不同的图标图片时，通过设置图片中的图标对应的位置来只显示该图标。
>
> 这样做的好处是可以让 N 个小图片资源请求 合并为 1 个图片资源请求，降低服务器请求压力。

答：**Three.js 纹理是完全支持 精灵图(雪碧图) 的。**

实现方式就是将多张图片合并成一张图片，然后创建成一个 纹理图集(texture atlas)，具体的做法会在之后学习的 构建自定义几何图形时讲解。



## 示例3：纹理加载器的不同事件回调函数

让我们在回到 示例1 的代码中，关于纹理加载器的代码片段：

```
//创建一个 纹理加载器
const loader = new Three.TextureLoader()
/创建一个材质，材质的 map 属性值为 纹理加载器加载的图片资源
const material = new Three.MeshBasicMaterial({
  map: loader.load(require('@/assets/imgs/mapping.jpg').default)
})
```

上述代码中，我们只是设置了加载图片资源的文件路径，并没有添加图片加载相关的事件处理回调函数。

关于纹理加载器 load()，相应的 .d.ts 定义为：

```
TextureLoader.load(
  url: string, 
  onLoad?: ((texture: Three.Texture) => void) | undefined, 
  onProgress?: ((event: ProgressEvent<EventTarget>) => void) | undefined, 
  onError?: ((event: ErrorEvent) => void) | undefined
): Three.Texture
```

为了显示更多纹理图片加载过程中的细节，我们可以将代码修改为：

```
//创建一个 纹理加载器
const loader = new Three.TextureLoader()
const material = new Three.MeshBasicMaterial({
  map: loader.load(require('@/assets/imgs/mapping.jpg').default,
  (texture: Three.Texture) => {
      console.log('纹理图片加载完成')
      console.log(texture)
      console.log(texture.image.currentSrc) //此处即图片实际加载地址
    },
    (event: ProgressEvent<EventTarget>) => {
      console.log('纹理图片加载中...')
      console.log(event)
    },
    (error: ErrorEvent) => {
      console.log('纹理图片加载失败！')
      console.log(error)
    }
  )
})
```

**请注意：**

1. 在项目调试过程中，由于图片资源位于本地，所以加载所需时间极其短暂。

2. 若项目加载的是网络图片资源，但是由于目前一般网速都比较快，一张100K 左右的图片下载所需时间非常短暂，所以可能在即运行中，根本触发不了 onProgress 处理函数。

   > 此处是存疑的，因为按照我的理解，就算是图片资源加载再快，onProgress 回调函数至少也应该执行一次才对的，可事实是onProgress 回调函数一次也没有被调用。



**补充说明：谷歌浏览器网络调试**

为了能够模拟出网络加载速度较慢的情况，可以通过设置 谷歌浏览器 调试工具中的 网络面板。

1. 勾选上 Disable cache (禁用缓存)
2. 将网络由 Online 修改为自定义网络网速模式，在创建的自定义网速模式中可是设置下载或上传的网速。



## 示例4：使用纹理加载管理器监控多个图片资源的加载