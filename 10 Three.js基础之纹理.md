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



## 示例1：编写一个最简单的纹理示例

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



## 示例2：实现一个骰子

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