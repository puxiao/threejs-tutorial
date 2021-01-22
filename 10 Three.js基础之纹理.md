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

修改项目中 tsconfig.paths.json，向  paths 中添加 src/assets 目录的资源别名：

```
 "@/assets/*": ["./src/assets/*"]
```

> 备注：以后的章节中，我们还会陆续向 assets 目录中添加其他类型的文件，例如模型文件、asc数据文件等等。



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
  (texture) => {
      console.log('纹理图片加载完成')
      console.log(texture)
      console.log(texture.image.currentSrc) //此处即图片实际加载地址
    },
    (event) => {
      console.log('纹理图片加载中...')
      console.log(event)
    },
    (error) => {
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

在示例3中，我们可以直接给 TextureLoader 的 load 方法中传递加载事件处理回调函数。

假设我们需要加载多个 纹理图片资源，可以新建一个 Three.LoadingManager 实例，并把该实例传递给 TextureLoader 的构造函数。这样，LoadingManager 将来托管和处理 该 TextureLoader 所有的图片加载。 

**具体代码示例：**

```
//创建所有纹理加载的管理器
const loadingManager = new Three.LoadingManager()
//创建一个 纹理加载器
const loader = new Three.TextureLoader(loadingManager)
//创建 6 个面对应的材质
const materialArr: Three.MeshBasicMaterial[] = []
for (let i = 0; i < 6; i++) {
  materialArr.push(new Three.MeshBasicMaterial({
    map: loader.load(require(`@/assets/imgs/dice${i}.jpg`).default)
  }))
}

//添加加载管理器的各种事件处理函数
loadingManager.onLoad = () => {
  console.log('纹理图片资源加载完成')
}
loadingManager.onProgress = (url, loaded, total) => {
  console.log(`图片加载中, 共 ${total} 张，当前已加载 ${loaded} 张 ${url}`)
}
loadingManager.onError = (url) => {
  console.log(`加载失败 ${url}`)
}
```

> 请注意，onProgress 中的 loaded 和 total 分别指：  
>
> 1. loaded：已加载完成的纹理图片数量
> 2. total：总共需要加载的纹理图片数量

**补充说明：**

在上面代码中，虽然是先执行的 loader.load( ... )，后定义了onLoad、onProgress、onError，但是由于 load 加载是异步的，执行完成存在延迟性，所以 onLoad、onProgress、onError 是一定会被正确触发执行的。



## 纹理占用的内存

### 只与图片尺寸(宽高)有关，与图片文件体积无关

#### 原则1：纹理图片文件的体积只影响加载完成所需时间

加载完成所需时间是由 服务器带宽和你本机下载速度 决定的。

#### 原则2：纹理图片的尺寸(宽高)决定所占内存大小

**占用内存计算公式：width * height * 4 * 1.33 字节**

举例，假设某个纹理图片的尺寸为 宽 200，高300，那么这个纹理所占的内存为：

200 * 300 * 4 * 1.33 = 319200 byte(字节) ≈ 319 kb

#### 原则3：纹理图片的清晰度(图片质量)并不影响所占内存大小

纹理图片所占内存只与图片宽高有关，至于图片清晰度(图片质量)，对于所占内存而言是没有区别的。

#### 原则4：同样尺寸的JPG和PNG图片所占内存大小相同

纹理图片所占内存只与图片宽高有关，至于图片是哪种格式(JPG 或 PNG)，对于所占内存而言是没有区别的。

但是请注意，**由于 PNG 图片包含透明度信息，对于非图像贴图(例如 法线贴图)而言，PNG 包含的像素透明度信息非常有用，此时应选择 PNG 格式。**



无论哪种方式，关于纹理图片渲染所占内存和相关计算，都是交由 GPU 来负责计算的。

**小总结：**

1. 纹理图片所占内存大小仅和宽高有关，因此在保证贴图清晰度的前提下应尽量减小图片尺寸。
2. 纹理图片的格式、清晰度仅会影响加载资源所需时间。



## 纹理图片尺寸与物体渲染尺寸的关系处理

**纹理图片尺寸：是指图片本身的宽高尺寸**

**物体渲染面尺寸：是指最终在镜头中物体某一个面所渲染出的尺寸**

> 物体渲染尺寸是由 物体本身大小和物体距离镜头的远近来共同决定的。



纹理图片尺寸和物体渲染面尺寸几乎是不可能刚好完全相同的。

> 这个几率几乎不存在

那么我们就需要考虑，假设 2 者尺寸不相同时，Three.js 是如何处理的。



首先我们先了解一下 mipmap 算法模式。

#### 什么是 mipmap ？

mipmap 是目前 3D 应用最为广泛的纹理映射技术之一。

mipmap 的原则是将图片的每个边(宽和高)对应的分辨率只取上一级的 1/2。

这样便可以通过计算，不断得到 面积为上一级 1/4 的图片数据，直至最终图片为 1像素 * 1 像素。

而 Three.js 会选择最接近于物体渲染面尺寸 的那一级渲染图片，并渲染出效果。

假设 纹理图片尺寸大于渲染面尺寸，此时需要对纹理进行缩小。

> 此处为我个人的理解：假设纹理图片尺寸小于渲染面尺寸，那么此时使用相反计算过程，最终得到一个比较模糊但尺寸符合的贴图数据。

**这样做的好处是？**

答：这种策略其实相当于牺牲掉了纹理贴图的精准性，换来了计算所需性能上的提升。

这也解释了为什么有时候即使纹理图片尺寸非常大，但某些时候渲染出的物体实际上还是略有模糊。



#### 纹理的缩放模式有哪几种？

大体上可以分为 3 种：

1. NearestFilter

2. LinearFilter

3. Mipmap相关的模式

   > mipmap 与 Nearest、Linear的各种结合

| 缩放模式                   | 模式说明                                               |
| -------------------------- | ------------------------------------------------------ |
| NearestFilter              | 最接近模式，选择最接近的像素                           |
| NearestMipmapNearestFilter | 选择最贴近目标解析度的Mip，然后线性过滤器将其渲染      |
| NearestMipMapNearestFilter |                                                        |
| NearestMipmapLinearFilter  | 选择层次最近的2个 Mip，将2个Mip使用线性模式将其混合    |
| NearestMipMapLinearFilter  |                                                        |
| LinearFilter               | 线性模式，选择4个像素并将进行混合                      |
| LinearMipmapNearestFilter  | 选择最贴近目标解析度的1个Mip，然后使用线性模式将其混合 |
| LinearMipMapNearestFilter  |                                                        |
| LinearMipmapLinearFilter   | 选择层次最近的2个Mip，然后使用线性模式将其混合         |
| LinearMipMapLinearFilter   |                                                        |

> 上述表格中 Mipmap 和 MipMap 应该是同一个意思的不同书写方式而已。
>
> 除了 NearestFilter 和 LinearFilter 之外，其他模式都属于 Mipmap 模式的变种。

当第一次见到 Mipmap 模式时，一定是有点摸不着头脑，不清楚究竟他们区别是什么，暂时不用担心，先大致了解即可。

请记住以下原则：

**Nearest：最接近算法，精确度高，像素感比较强烈，锐化程度比较强烈，所用计算量大**

**Linear：线性算法，精确度不高，像素感不强烈，锐化程度不强，相对比较模糊和平滑，所用计算量小**

NearestMipmapNearestFilter、NearestMipmapLinearFilter、LinearMipmapNearestFilter、LinearMipmapLinearFilter 都是在 Mipmap 模式下，分别使用 nearest 和 linear 算法的相互结合产物。



#### 修改之前示例中的纹理相关代码：

在之前示例代码中，我们给某个材质添加纹理都是通过：

```
const loader = new Three.TextureLoader()
const material = new Three.MeshBasicMaterial({
  map: loader.load( ... )
})
```

上面代码中的 loader.load() 返回值就是 Three.Texture 的一个实例。

为了方便我们设置 Texture，我们可以将上面代码修改为：

```
const loader = new Three.TextureLoader()
const texture: Three.Texture = loader.load( ... )
const material = new Three.MeshBasicMaterial({
  map: texture
})
```



### 情况1：纹理图片尺寸 大于 物体渲染面的尺寸

设置纹理缩小模式的属性：**magFilter**

**当纹理图片尺寸 大于 物体渲染面尺寸时，可以通过设置 texture.magFilter 的值来设置纹理的清晰度模式：**

1. texture.magFilter = **Three.NearestFilter：最接近模式**

   > 该模式下最终渲染效果更加清晰，但所需内存更多，渲染时间慢

2. texture.magFilter = **Three.LinearFilter：线性模式**

   > 该模式下最终渲染效果略微模糊，但所需内存更少，渲染时间快

**默认，magFilter 的值是 Three.LinearFilter。**

> 注意：纹理图片缩小模式，不可以选择 Mipmap 中的任何一种模式，只能从 NearestFilter、LinearFilter 选择其一。



### 情况2：纹理图片尺寸 小于 物体渲染面的尺寸

设置纹理放大模式的属性：**minFilter**

**当纹理图片尺寸 小于 物体渲染面尺寸时，可以通过设置 texture.minFilter 的值来设置纹理的清晰度模式：**

1. texture.minFilter = **Three.NearestFilet (最接近模式)**
2. texture.minFilter = **Three.LinearFilter (线性模式)**
3. texture.minFilter = **Three.NearestMipmapNearestFileter**
4. texture.minFilter = **Three.NearestMipmapLinearFilter**
5. texture.minFilter = **Three.LinearMipmapNearestFilter**
6. texture.minFilter = **Three.LinearMipmapLinearFilter**

**默认，minFilter 的值是 LinearMipmapLinearFilter。**



### 究竟该选择哪种模式？

**纹理需要缩小时，如果对渲染清晰度要求比较高，则选择 NearestFilter**

**纹理需要放大时，如无特殊需要，推荐使用默认的 LinearMipmapLinearFilter**



### 在不考虑性能的前提下，选择 Nearest 是否渲染效果最佳？

事实上，除了性能方面的考虑之外，无非特殊必要，并不推荐使用 Nearest 相关模式。

原因是若使用 Nearest 相关模式，会让物体无论哪个位置像素感都比较强，也就是 锐化 程度比较强烈，假设物体远处渲染的面比较小，那么此时物体远处渲染效果就会出现 “像素抖动” 的情况。

而使用 Linear 相关模式，则物体远处像素会比较平滑，不会产生抖动感。



**补充说明：**

**有一种情况除外：采用纹理重复的方式，用极致的小图去渲染比较大的面**

满足以下条件：

1. 纹理图片极其小，例如 2 像素 X 2 像素
2. 希望通过 纹理重复 ，来将比较小的 图片 铺满 整个渲染面

这个时候就推荐使用 Nearest 模式了，因为只有这样才可以保证渲染出的效果比较清晰。

例如：用一个 2 X 2 像素的黑白相间的纹理图片，作为某个大的、类似围棋盘一样的黑白纹理贴图，那么此时就需要将模式设置为 Nearest。

> 提前预告：下一节我们将讲解 灯光，其中就会用到这个知识点。



## 纹理的重复、偏移、旋转

默认情况下，纹理是不会重复、偏移和旋转的。

> 默认将纹理通过 伸缩 以适用于渲染面。

接下来讲一下如何设置纹理的重复、偏移、旋转。



### 设置重复方式

纹理重复分为：水平重复(warpS)、垂直重复(warpT)

**重复有 3 种形式：**

1. Three.ClampToEdgeWrapping：每个边缘最后一个像素将永远重复
2. Three.RepeatWrapping：重复整个纹理
3. Three.MirroredRepeatWrapping：纹理被镜像(对称反转)并重复

**设置重复代码：**

```
texture.wrapS= Three.RepeatWrapping
texture.wrapT= Three.RepeatWrapping
```



### 设置重复次数

**设置重复次数代码：**

```
texture.repeat.set(2,3) //设置水平方向重复 2 次、垂直方向重复 3 次
```



### 设置偏移

**设置偏移的 1 单位 = 1 个纹理图片大小。**

**设置偏移的代码：**

```
texture.offset.set(0.5,0.25) //设置纹理水平方向偏移 0.5 个纹理宽度、垂直方向偏移 0.25 个纹理高度
```

> 0.5 个纹理宽度也就相当于 一半的宽度偏移量
>
> 0.25 个纹理高度也就相当于 1/4 的高度偏移量



### 设置旋转

**通过修改 rotation 属性来设置旋转弧度。**

> 在 Three.js 中，有内置的可以将角度转变为弧度的方法

**通过修改 center 属性来确认旋转中心点。**

**纹理的中心点坐标体系，相当于传统的四象限坐标。**

**中心点的 2 个原则：**

1. 默认的旋转中心点为纹理图片的左下角，坐标为 (0,0)

2. 坐标的单位为 1 单位 = 1 个纹理图片对应大小

   > 例如 (0.5, 0.5) 坐标对应的是 纹理图片的中心位置

**示例代码：**

```
texture.center.set(0.5,0.5) //将旋转中心点改为图片的正中心位置
texture.rotation = Three.MathUtils.degToRad(45) //设置纹理旋转弧度
//MathUtils.degToRad() 方法作用是将度数转化为弧度
```



关于纹理，目前就先讲到这里。在以后的学习中，还会对纹理坐标以及其他几种类型的纹理进行学习。

纹理是目前学习中遇到的内容量最大的一节。

**好好加油，下一节讲解 灯光。**