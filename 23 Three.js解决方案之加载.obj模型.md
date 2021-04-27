# 23 Three.js解决方案之加载.obj模型

本文将开始 Three.js 新的篇章—— Three.js 各种应用场景的解决方案。

有 2 点会与之前的篇章不同：

1. 讲解内容时尽量简要，不再像前面 01-22 篇章那样啰嗦
2. 和原版官方教程内容上略有不同，删除我认为没有必要的内容、增加上我自己补充的内容。



<br>

**特别强调 1：从本文示例开始，我们将升级 React 和 Three.js 的版本**

1. react  由原来的 17.0.2 升级为 17.0.3

2. 重点是 Three.js 由原来的 0.125.2 升级为 0.127.0

   > 由于 Three.js 版本迭代，对于某些类的引用路径、方法和属性的使用 难免会有一些变化。

3. 由于 Three.js 在 0.126.0 版本时已经默认不包含 .d.ts 文件，所以我们还需要额外单独安装 对应的 typescript 类型定义文件包

   ```
   yarn add @types/three
   
   //npm i @types/three
   ```



<br>

**特别强调 2：尽管我们在前一章节花了非常多的时间研究出如果在 Web Worker 中渲染和控制 Threejs 3D 场景，但是本示例并不会选择使用 web worker，依然会选择直接在 JS 主场景中创建 3D 场景。**



<br>

正文开始...



## 加载.obj(模型)文件

<br>

**前言小絮**

在之前所有的示例中，场景中的 3D 物体元素都是直接在 Three.js 中创建的，但是 Three.js 毕竟不是专业的 3D 建模软件，所以实际工作中，我们更多的都是在传统 3D 建模软件中创建好模型，然后将模型导出成特定格式的文件，然后在 Three.js 中使用特定的加载器，将这些模型加载进来。



<br>

#### 让我们开始使用 Blender

尽管我更喜欢 C4D 这个软件，但是由于 C4D 为收费软件，所以在本教程中，我们将更多的使用 Blender 这个免费的 3D 建模软件。

国外人很注意版权，所以很多教程中也都使用的是 Bleander。

假设你希望自己创建的模型或者项目要跟别人交流，而你使用破解版的 C4D ，多少有一些隐患。



<br>

#### Blender软件下载安装

Blender 不光免费，而且还支持简体中文。

> 当前版本为 2.92.0

下载地址：https://www.blender.org/download/

> 软件安装好之后，第一次启动 Blender 时会有一个弹窗，在弹窗中将界面语言由 English 修改为 简体中文，这样软件界面就变成 简体中文了。



<br>

**关于Blender的基础用法，请参考我另外一篇学习笔记：[Blender基础教程](https://github.com/puxiao/notes/blob/master/Blender%E5%9F%BA%E7%A1%80%E6%95%99%E7%A8%8B.md)**

我也是刚开始学习 Blender，感兴趣一起学习吧。

> 我本身就会一些基础的 C4D 软件操作，所以在学习 Blender 时并没有感觉特别吃力，只是觉得 Blender 各种操作的快捷键实在是太多了，有点难记。如果你是第一次接触 3D 建模软件，那...只能说，加油！ 



<br>

#### 使用Blender创建并导出3D模型——.obj文件

启动 Blender 之后，默认场景上有一个立方体，我们再添加几个其他元素，如 圆柱体、圆锥体、球体。



**关于Blender中“球体”的特别说明**

我们添加球体的操作是：物体模式下，点击 “添加 > 表(曲)面 > 球体”

千万不要误操作成：“添加 > 融球 > 球”

因为 Blender 中 融球 和 球体 虽然看上去都像球体，但是 2 者有很大的差别。

> 这句话是废话，要是没差别也不会是不同操作了。
>
> 所谓 融球 就是当 2 个融球彼此靠近一定程度后，2 个融球边缘会自动融合在一起，就是因为这个特性所以才被称为融球。融球本质上是 Blender 一种即时计算的数据公式。
>
> 而 球体 就是属于普通的网格，2 个球体就算彼此靠得再近也不会发生相融的场景。



<br>

我们只是简单将这几个元素修改一下位置，不做任何属性的调整，然后就直接导出模型。

> 为什么不做复杂的调整？  
> 答：因为我也刚学 Blender，还不太会，就先这样吧。



<br>

现在我们要导出刚才创建的 3D 场景元素，执行：

**文件 > 导出 > Wavefront(obj)**

在弹出的导出选项弹出中，我们什么也不修改，直接点击 导出OBJ，我们将文件名设置为 hello.obj。

> 以下为默认的导出参数
>
> 在 Objects as 选项中：
>
> 1. OBJ 物体 (默认已勾选)
> 2. OBJ组 (默认未被勾选)
> 3. 材质组 (默认未被勾选)
> 4. 动画 (默认未被勾选)
>
> 在 变换 选项中：
>
> 1. 缩放：1.00 (默认值)
> 2. 路径模式：自动  (默认值)
> 3. 前进：-Z 前进  (默认值)
> 4. 向上：Y 向上  (默认值)
>
> 在 集合数据 选项中：
>
> 1. 应用修改器 (默认已勾选)
> 2. 平滑组 (默认未被勾选)
> 3. Bitflag平滑组 (默认未被勾选)
> 4. 写入法线 (默认已勾选)
> 5. 包括UV (默认已勾选)
> 6. 写入材质 (默认已勾选)
> 7. 三角面 (默认未被勾选)
> 8. Curves as NURBS (默认未被勾选)
> 9. 多边形组 (默认未被勾选)
> 10. 保持顶点顺序 (默认未被勾选)



<br>

当导出完成后，我们会看到实际上产生了 2 个新文件：

1. hello.obj
2. hello.mtl

特别说明一下：

1. .obj 这个文件是用来储存 3D模型 数据的

   > 请注意这里面只包含 3D模型 的数据和模型位置信息，并不包含 Blender 场景中的镜头和灯光

2. .mtl 这个文件是用来储存 材质 数据的

   > 我们在 Blender 中创建的几个物体元素，**实际上 Blender 已经给他们添加了默认的一个可反光材质**。
   >
   > 由于是可反光材质，所以请记得一定要在 Three.js 场景中添加灯光，否则这些物体都会看不见的。
   
   > 在 Blender 中材质包含有 纹理贴图，尽管此刻我们并未给材质设置任何纹理贴图。



<br>

我们将得到的 hello.obj 文件拷贝到 React 项目中，路径为 src/assets/model/hello.obj

接下来我们要开始编写 Three.js 相关代码了。



<br>

#### 加载 .obj 模型文件对应的类(加载器)为 OBJLoader

OBJLoader 的用法也非常简单：

> 最常用的方法为 .load() 

```
import { OBJLoader } from "three/examples/jsm/loaders/OBJLoader"

...

const loader = new OBJLoader()
    loader.load(require('@/assets/model/hello.obj').default, (group) => {
        console.log(group)
        scene.add(group)
    }, (event) => {
        console.log(Math.floor((event.loaded * 100) / event.total) + '% loaded')
    }, (error) => {
        console.log(error.type)
})
```



<br>

也可以使用异步的 loadAsync()，用法为：

```
const promise = loader.loadAsync(require('@/assets/model/hello.obj').default, (event) => {
            console.log(Math.floor((event.loaded * 100) / event.total) + '% loaded')
        })
promise.then((group) => {
            scene.add(group)
})
```



<br>

**关于Three.js官方文档的一个小说明：**

如果你查看 OBJLoader 的官方文档，你会发现只介绍了 load() 方法，没有提及 loadAsync() ，这是为什么？

> 即使你查看 OBJLoader 的源码，也会发现根本没有 loadAsync() 方法，那......

这其实是因为 OBJLoader 继承于 Loader，而 loadAsync() 是由 Loader 定义的，所以才未出现在 OBJLoader 的文档中。

Three.js 仓库管理员非常严谨，每次提交 PR 涉及修改一定要求你去修改对应的文档，所以如果以后发现某个方法并未出现在 要使用的类的介绍中，那么就去他的父类里查找即可。



<br>

> 像别的编程语言文档，可能都会在某个类的介绍文章中，列出所继承父类的属性和方法，但是在 Three.js 中并未列出。

> 我向官方提交了一个建议：https://github.com/mrdoob/three.js/issues/21640
>
> 得到官方的回复内容为：
>
> ```
> Indeed. And that's the reasons why I not vote to do this. This approach would require a lot of manual effort and thus is error prone when things change.
> 
> The documentation pages contain references like:
> 
> See the base Material class for common properties.
> 
> Same for methods. Considering that the documentation is not generated, I think it's better to stick with this approach.
> ```
>
> 官方回复的意思是：由于现在 Three.js 文档并不是自动生成的，如果那样做当发生变更时，每次都会产生大量的工作内容，所以...就先保持现状吧。



<br>

完整的示例代码如下：

src/components/hello-objloader/index.tsx

```
import { useRef, useEffect } from "react"
import * as Three from 'three'
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
import { OBJLoader } from "three/examples/jsm/loaders/OBJLoader"

import './index.scss'

const HelloOBJLoader = () => {
    const canvasRef = useRef<HTMLCanvasElement | null>(null)

    useEffect(() => {

        if (canvasRef.current === null) { return }

        const renderer = new Three.WebGLRenderer({ canvas: canvasRef.current })
        const scene = new Three.Scene()
        const camera = new Three.PerspectiveCamera(45, 2, 0.1, 100)
        camera.position.set(10, 0, 10)

        const light = new Three.HemisphereLight(0xFFFFFF, 0x333333, 1)
        scene.add(light)

        const control = new OrbitControls(camera, canvasRef.current)
        control.update()

        const loader = new OBJLoader()
        loader.load(require('@/assets/model/hello.obj').default, (group) => {
            console.log(group)
            scene.add(group)
        }, (event) => {
            console.log(Math.floor((event.loaded * 100) / event.total) + '% loaded')
        }, (error) => {
            console.log(error.type)
        })

        const render = () => {
            renderer.render(scene, camera)
            window.requestAnimationFrame(render)
        }
        window.requestAnimationFrame(render)

        const handleResize = () => {
            if (canvasRef.current === null) { return }

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
    }, [])

    return (
        <canvas ref={canvasRef} className='full-screen' />
    )
}

export default HelloOBJLoader
```

> 切记一定要给场景添加灯光，否则即使加载模型成功了，场景漆黑一片，也啥也看不见。



<br>

此刻，我们在 Blender 中创建的 3D 模型已经加载并显示出来了，但是模型并没有任何材质，接下来我们要完善一下示例，给物体添加上材质。



<br>

## 加载.mtl(材质和纹理)文件

首先，我们需要在 Blender 中给物体的材质添加纹理贴图。



<br>

#### 在Blender中给物体设置材质，添加纹理

具体步骤为：

1. 先准备一张 金属纹理贴图，将该图片文件命名为 metal_texture.jpg，暂且存放在 hello.blend 同级目录下。

   > 随着项目复杂，纹理图片资源变多，更加合理的是创建一个 texture 的目录，用来专门存放纹理图片资源

2. 在不选择任何物体的前提下，在 Blender 右侧选项面板中，找到 “材质属性” 面板，我们会看到 默认材质 Material，在 “表(曲)面 - 基础色” 设置中，找到 “基础色”，点击左侧的 小圆点，在弹出菜单中选择 “图像纹理”。

   > 如果你不小心点击的是 “基础色”  右侧的区域，则会出现 调色板

3. 此时 基础色 已修改为 图像纹理 状态，我们点击 “打开”，在弹窗中找到 metal_texture.jpg 并点击 “打开图片"，这样该纹理图片已添加到 材质 Material 中了。

4. 接下来我们需要做的就是依次选中(或全选)立方体、圆柱体、椎体、球体，然后在材质属性面板中，找到 “材质图标”，鼠标放上去会显示提示文字：浏览要关联的材质，点击该图标，选择 Material

   > 请注意是一个图标，而不是什么按钮，更不要点击 “新建”

5. 至此，我们已经将所有物体均设置一个相同材质纹理图片。

   > 如果感兴趣，你完全可以自己给不同物体设置不同的材质纹理图片



<br>

让我们先简单渲染一下，看看效果吧。

首先我们调整好场景视角，然后执行 “视图 > 对齐视图 > 活动相机对齐当前视角”，这一步的快捷键为 `Ctrl + Alt + Numpad0`。

然后我们点击 Blender 顶部菜单 “渲染 > 渲染图像”，这一步的快捷键为 F12，接下来就渲染弹窗中就可以看到物体渲染后的样子。



<br>

> 假设你不会 Blender，也没有关系，可以忽略我上面这段关于如何在 Blender 中创建模型和添加材质、纹理的操作。



<br>

**导出模型**

"文件 > 导出 > Wavefront(.obj)"



<br>

**我们看一下 Three.js 官方文档中对 .mtl 文件的描述：**

材质模版库（MTL）或 .MTL 文件格式是 .OBJ 的配套文件格式， 用于描述一个或多个 .OBJ 文件中物体表面着色（材质）属性。



<br>

**实际看看 hello.mtl 文件具体是什么内容**

我们可以使用 记事本 或 vscode 打开 hello.mtl 文件，内容如下：

```
# Blender MTL File: 'hello.blend'
# Material Count: 1

newmtl Material
Ns 323.999994
Ka 1.000000 1.000000 1.000000
Kd 0.800000 0.800000 0.800000
Ks 0.500000 0.500000 0.500000
Ke 0.000000 0.000000 0.000000
Ni 1.450000
d 1.000000
illum 2
map_Kd metal_texture.jpg

```

我们可以看到最后一行 `map_Kd metal_texture.jpg`，大概也可以猜到这一行表明材质纹理图片的路径和文件名，并且路径是相对路径。

> 由于我们将 metal_texture.jpg 存放在 hello.blend 同一目录下。



<br>

**关于.mtl文件内容的补充说明：**

> 下面我将针对上面的 .mtl 文件内容进行逐一解释

1. #：注释内容
2. newtml：材质的名称
3. Ns：反射指数，即反射高光度，值越高则高光越密集，一般取值范围 0 - 1000
4. Ka：材质的环境光，一般取值范围 0 - 1
5. Kd：散射光
6. Ks：镜面光
7. Ni：折射光，一般取值范围 0.01 - 10
8. d：渐隐指数，取值范围为 0 - 1，当值为 1 时完全不透明，当值为 0 时完全透明
9. illum：材质的光照模型，值为整数，取值范围为 0 - 10
10. map_Kd：漫反射指定颜色纹理贴图文件路径



<br>

上面属性中 illum 实际上是 “illumination model”，即光照模型。

illum 的取值分别对应的是：

| 取值 | 光照模型                                                     | 中文名                                      |
| ---- | ------------------------------------------------------------ | ------------------------------------------- |
| 0    | Color on and Ambient off                                     | 色彩开，阴影色关                            |
| 1    | Color on and Ambient on                                      | 色彩开，阴影色开                            |
| 2    | Highlight on                                                 | 高光开                                      |
| 3    | Reflection on and Ray trace on                               | 反射开，光线追踪开                          |
| 4    | Transparency: Glass on / Reflection: Ray trace on            | 透明： 玻璃开 反射：光线追踪开              |
| 5    | Reflection: Fresnel on and Ray trace on                      | 反射：菲涅尔衍射开，光线追踪开              |
| 6    | Transparency: Refraction on / Reflection: Fresnel off and Ray trace on | 透明：折射开 反射：菲涅尔衍射关，光线追踪开 |
| 7    | Transparency: Refraction on / Reflection: Fresnel on and Ray trace on | 透明：折射开 反射：菲涅尔衍射开，光线追踪开 |
| 8    | Reflection on and Ray trace off                              | 反射开，光线追踪关                          |
| 9    | Transparency: Glass on / Reflection: Ray trace off           | 透明： 玻璃开 反射：光线追踪关              |
| 10   | Casts shadows onto invisible surfaces                        | 投射阴影于不可见表面                        |



<br>

**那么问题来了，由于 hello.mtl 中包含纹理图片的路径，如果我们将 图片资源(metal_texture.jpg) 拷贝到 src/assets/model 中，.jpg 图片会被 webpack 重新编译成别的文件名，这会造成我们加载不到 纹理图片 资源。**



<br>

**最简单的解决办法，就是将 .obj/.mtl/.jpg 这 3 个文件资源存放在 public 目录中，而不是在 src 目录中。**

> public 目录里的文件是不会被 webpack 编译重命名的。



<br>

**拷贝文件到 Three.js 项目中**

我们在React 项目根目录的 public 目录下创建 model 目录，然后将新导出的 3 个文件：hello.obj、hello.mtl、metal_texture.jpg 拷贝到 public/model/ 中。



<br>

至此，前期准备工作完毕，接下来转到 Three.js 项目代码中。



<br>

#### 加载 .mtl 表面着色器(材质)对应的类(加载器)为 MTLLoader

> 在 Three.js 中，各种加载器的用法几乎完全相同。



我们需要做的事情就是：

1. 先使用 MTLLoader 实例 加载材质(纹理图片)资源

   > 特别强调一下，MTLLoader 实例 加载得到的对象类型并不是 Three.Metrial ，而是 MTLLoader.MaterialCreator

2. 加载完成后，将得到的 MaterialCreator 实例 赋予给 OBJLoader 实例

   > 这样 OBJLoader 以后加载的任意模型都会自动应用该 MaterialCreator 材质

3. 然后再让 OBJLoader 实例 去加载模型

   > 加载完成后，将模型添加到场景中



<br>

**实际代码：**

```
import { MTLLoader } from "three/examples/jsm/loaders/MTLLoader"
import { OBJLoader } from "three/examples/jsm/loaders/OBJLoader"

...

const mtlLoader = new MTLLoader()
const objLoader = new OBJLoader()
mtlLoader.load('./model/hello.mtl', (materialCreator) => {
    objLoader.setMaterials(materialCreator)
    objLoader.load('./model/hello.obj', (group) => {
        scene.add(group)
    })
})
```

> 请注意上述代码加载的资源地址为 “./model/hello.mtl” 和 “./model/hello.obj”，这里是相对路径，相对编译之后的 index.html 而言。



<br>

**完整的代码：**

```
import { useRef, useEffect } from "react"
import * as Three from 'three'
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
import { MTLLoader } from "three/examples/jsm/loaders/MTLLoader"
import { OBJLoader } from "three/examples/jsm/loaders/OBJLoader"

import './index.scss'

const HelloOBJLoader = () => {
    const canvasRef = useRef<HTMLCanvasElement | null>(null)

    useEffect(() => {

        if (canvasRef.current === null) { return }

        const renderer = new Three.WebGLRenderer({ canvas: canvasRef.current })
        const scene = new Three.Scene()
        const camera = new Three.PerspectiveCamera(45, 2, 0.1, 100)
        camera.position.set(10, 0, 10)

        const light = new Three.HemisphereLight(0xFFFFFF, 0x333333, 1)
        scene.add(light)


        const mtlLoader = new MTLLoader()
        const objLoader = new OBJLoader()
        mtlLoader.load('./model/hello.mtl', (materialCreator) => {
            objLoader.setMaterials(materialCreator)
            objLoader.load('./model/hello.obj', (group) => {
                scene.add(group)
            })
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

            camera.aspect = width / height
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

export default HelloOBJLoader
```



<br>

至此，我们已经讲解完如何加载 .mtl 和 .obj 文件。



<br>

## .obj文件类型的一些特别之处

本文讲解的是在 Blender 中导出 wavefront(.obj) 文件格式。

这里要针对性的进行补充：

1. Blender 可以导出 N 多种文件格式

   > .obj 仅仅是其中一种

2. Three.js 支持多种 3D 文件模型格式

   > 本文讲解的是加载 .obj



<br>

**.obj格式的一些特点：**

1. 行业内比较广泛使用

2. .obj是一种纯文本格式

   > 本文中我们使用记事本打开了 .mtl 文件，并未打开 .obj 文件，感兴趣的可以自己尝试看看具体都是什么内容。

3. .obj包含的内容有：Mesh(网格)、按组/物体分离、材质/纹理、NURBS曲线和曲面

4. .obj不包含(不支持导出)的内容有：网格顶点颜色、骨架、动画、灯光、相机、空物体、父子关系或变换



<br>

以上特性中不难看出 .obj 有优点，也有缺点。



**优点：只包含物体模型数据本身**

**缺点：不包含其他复杂元素(动画、相机、灯光等)**



<br>

接下来，我们将在下一章节中，讲解另外一种 3D 模型格式文件：.gLTF

gLTF 格式可以包含更多复杂的元素数据。



<br>

## 为什么要自己学习Blender？

我个人认为，如果你想深入学习 3D，学习 Three.js，那么你就需要掌握一门 3D 建模软件。

你可以不精通，但是基础的操作你要掌握，这样非常利于你对于 Three.js 3D 概念的理解。



<br>

如果你不会 3D 建模软件，那么你只能直接在 Three.js 中去创建模型，这将是一件非常费力的事情，并且也无法做到精准建模，对模型的细腻雕刻。



<br>

你总不能完全依靠别人给你提供建好的模型，做到饭来张口的状态吧。

自己能够掌握 3D 建模，越建越快乐。



<br>

这也是为什么本文花了大量篇幅在一步一步讲解 Blender 中的各种操作的原因。

> 我也是 Blender 新手小白，希望我们一起学习，一起加油。



<br>

如果本文中讲解的 Blender 你根本不会操作，也不感兴趣安装学习，那么你可以直接在网上搜索一些 .obj 和 .mtl 文件进行代替学习。

这里提供 2 个文件资源，你可以直接下载使用：

https://threejsfundamentals.org/threejs/resources/models/windmill_2/windmill-fixed.mtl

https://threejsfundamentals.org/threejs/resources/models/windmill_2/windmill.obj



<br>

我们下一章节见。

