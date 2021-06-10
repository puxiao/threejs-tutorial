# 24 Three.js解决方案之加载.gltf模型

上一章节，我们讲解了加载 .obj 模型，本文将讲解加载 .gltf 模型。



> 注：虽然我们标题写的是 “加载.gltf模型”，但更加准确地说法应该是 "加载 gtTF 文件"



<br>

首先我们先回顾一下 .obj 文件格式的模型一些特征：文件格式简单(纯文本)、除模型外无法提供其他场景元素(例如摄像机、灯光等)。

本文要讲解的 .gltf 格式文件可以包含的数据内容和类型要比 .obj 多很多。



<br>

## 常见 3D 文件格式 和 gltf 的区别

**我们先将常见的 3D文件格式进行划分**



**第1类(原始文件)：**

3D 建模软件本身特有的、原始的文件格式，例如：

1. Blender 对应的是 .blend
2. 3D Max 对应的是 .max
3. Maya 对应的是 .ma
4. C4D 对应的是 .c4d



<br>

**第2类(中转文件)：**

多个 3D 建模软件彼此都可以打开，能够读取的文件格式，例如：

1. .obj
2. .dae
3. .fbx

> 所谓“中转”，是指这些格式的作用实际上相当于将某个模型从 A 软件 导出 然后再 B 软件中可以打开并读取。



<br>

**第3类(特定格式)：**

某些 3D 应用独有的文件格式。

例如王者荣耀这款游戏中 3D 模型就可能是自己独有的文件格式。



<br>

**第4类(传输格式)：**

> 这里的 “传输” 是英文单词 “Transmission” 的翻译

gltf 就属于传输格式类型的文件。gltf格式可以做到其他格式都无法做到的事情。



<br>

### GLTF文件格式简介

**GLTF是英文：Graphics Language Transmission Format 的缩写**

> WebGL、OpenGL 中的 “GL” 和 GLTF 中的 “GL” 是相同的单词。

**GLTF中文全称为：图形语言传输格式**

> GLTF 本身就是由 OpenGL 和 Vulkan 背后的 3D 图形标准组织 Khronos 定义的。

所以你可以想象得到，gltf 本身就是为了网络传输、浏览器渲染 3D 而生的。



关于更多 gltf 信息，可以查看其官网：https://www.khronos.org/gltf/



<br>

**GLTF的支持度：几乎所有的 Web 3D 图形框架都支持 GLTF**

> 除了 Three.js 框架外 ，其他 3D JS 引擎框架也都支持 GLTF



<br>

**gltf优点 1：体积小，便于传输**

gltf 文件中模型的数据都以二进制存储，当下载(使用) gltf 文件时可以将这些二进制数据直接在 GPU 中使用。

反观 vrml、.obj 或 .dae 等格式，他们是将数据存储为文本(例如纯文本或JSON格式)，也就是说 GPU 在读取这些模型文件时还需要进行文本解析。

在文件体积方面，通常相同的模型顶点数据如果用文本形式存储，要比二进制存储体积大 3 到 5 倍。



<br>

**gltf优点 2：直接渲染**

gltf 文件中模型的数据是直接要渲染的，而不是要再次编辑的。

> 换句话说 gltf 文件中的模型是不可以再次编辑的

> 而其他类型的文件，例如 .obj 中模型是可以在 Three.js 中加载完成后二次编辑的

> 你可以简单的把 gltf 想象成 jpg 图片，而其他格式的 3D 文件是 PSD 文件，当我们仅仅是为了看到图片，无需编辑该图片时，肯定是 .jpg 图片体积小，打开速度快。

正因为是不可编辑，所以一些对于渲染而言不重要的数据通常都已被删除，例如多边形都已转化为三角形。



<br>

**gltf优点 3：内嵌材质信息**

gltf 文件中模型的材质信息是被内嵌进去。

请注意我们这里说的是 “材质信息”，也就是相当于 .obj 对应的 .mtl 中的数据，但是对于纹理图片资源(xxx.jpg)本身来说，并不会内嵌进去。

> 所以，这里隐含的一个事情就是，我们依然需要将 .gltf 文件对应的纹理图片资源 .jpg 放在 pulic 目录中。



<br>

**结论：gltf 格式非常有针对性，是专门为渲染而设计的，文件体积小，且 GPU 读取快速。**

**因此，推荐使用 gltf 格式。**



<br>

### 在Blender中导出gltf文件

讲了这么多 gltf 文件的优点，那么我们打开之前创建的 hello.blend 文件，导出一下 gltf 文件看看。

**导出步骤：**

1. 打开 hello.blend

2. 文件 > 导出 > glTF 2.0(.glb/.gltf)

3. 在弹窗对话框中，使用默认导出项，我们直接点 `导出`

   > 尽管我们使用的是默认导出项，但是还请你留意一下这几项内容：
   >
   > `包括`：选定的物体(未勾选)、自定义属性(未勾选)、相机(未勾选)、精确灯光(未勾选)
   >
   > `变换`：Y 向上(已勾选)
   >
   > `几何数据`: 应用修改器(未勾选)、UV(已勾选)、法向(已勾选)、切向(未勾选)、顶点色(已勾选)、材质(导出)、图像(自动)、压缩(未勾选)
   >
   > `动画`：动画(已勾选)、形态键(已勾选)、蒙皮(已勾选)
   >
   > > 尽管我们创建的 hello.blend 中并未设置任何动画，你可以选择取消动画相关的勾选



<br>

此时去导出目录里，我们会发现多出来了一个 `hello.glb` 的文件。

> 特别强调：由于纹理图片资源 metal_texture.jpg 本身就在目录中，所以我们只是从直观上感觉多出了 1 个文件而已。



<br>

**.glb ？不是 .gltf ？**

额~，别着急，我们补充一下 GLTF 格式的知识。



<br>

#### GLTF是一种3D文件格式规范，但是却有 3 种表现形式

**3 种表现形式分别为：分离式、二进制、嵌入式**



**第1种表现形式(分离式)：.gltf  + .bin + 纹理贴图资源(.jpg、.png)**

1. gltf：3D 场景的所有概要信息，包括灯光、纹理贴图等信息

   > 该文件的内容具体形式为 JSON

2. .bin：模型的二进制数据

3. 纹理贴图资源：这个就不多说了，就是纹理图片 xxx.jpg 或  .png



<br>

**第2种表现形式(二进制)：.glb**

.glb：包含场景所有的信息的二进制数据。

> .glb === .gltf + .bin + 纹理图片资源



<br>

**第3种表现形式(嵌入式)：.gltf**

.gltf：以 JSON 形式保存所有场景信息数据，包括材质和纹理信息。

> 这种形式由于文件内容是 json，因此是可以通过文本再次编辑的



<br>

**Blender 默认导出 glTF 2.0 格式时，采用的是 .glb 后缀形式。**

想要更改成别的导出形式，我们可以在 Blender 导出项 `格式`下拉框中更改为 “.gltf 分离(.gltf + .bin + 纹理)” 或 "glTF嵌入式(.gltf)"。

那么此时**导出的文件格式就是 .gltf 后缀形式。**



<br>

**3种形式的对比：**

> 以下纯粹是我个人的观点，仅供参考

比较常见的是前 2 种：分离式(.gltf + .bin + 纹理)、二进制形式(.glb)

如果你的项目中，模型数据不会发生变化，但是纹理贴图可能容易发生变化，那么可以选择 分离式的。

> 分离式的贴图资源本身就是单独存在的，因此方便替换修改。

如果你是要发送给其他人使用、且不会发生材质变更的，则可以采用 .glb 形式的。

> 由于所有数据都只在 .glb 文件中，就 1 个文件也利于文件发送。



<br>

补充一点：有一个网站 https://gltf-viewer.donmccurdy.com/ ，他可以提供 .glb 文件在线预览。

同时在 NPM 上面，有很多针对 .glb 和 .gltf 格式互转的工具包，例如：[gltf-import-export](https://www.npmjs.com/package/gltf-import-export)



<br>

对于 Three.js 来说，加载 glTF 格式的文件，无论哪种形式，均支持。



<br>

## 使用GLTFLoader加载glTF文件的示例

在 Three.js 中负责加载 glTF 格式文件的加载器为 GLTFLoader。

用法和之前 OBJLoader 用法完全相同，废话不多说，直接看代码。



<br>

我们先加载 .glb 格式的文件，代码如下：

src/components/hello-gltfloader

> 由于 .glb 文件是单独 1 个存在，所以我们这次可以将 hello.glb 文件放在 src/assces/model/ 目录下了。

```
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader"

...

const loader = new GLTFLoader()
loader.load(require('@/assets/model/hello.glb').default, (gltf) => {
    scene.add(gltf.scene)
})

```

> 请注意当加载完成后，执行的是 scene.add(gltf.scene)
>
> 加载完成得到的 gltf 中包含的数据有：
>
> 1. gltf.animations; // Array<THREE.AnimationClip>
> 2. gltf.scene; // THREE.Group
> 3. gltf.scenes; // Array<THREE.Group>
> 4. gltf.cameras; // Array<THREE.Camera>
> 5. gltf.asset; // Object
>
> 我们此刻只是将 gltf.scene 添加到了场景中而已，其他数据暂时并未使用到。



<br>

和加载 .glb 类似，如果我们的 3D 数据文件为 分离式的 .gltf，则将上述代码修改为：

```
loader.load('./model/hello.gltf', (gltf) => {
    scene.add(gltf.scene)
})
```

> 注意：我们只需将 hello.gltf 传递给 loader 即可，loader 会读取 .gltf 中的数据，自动去加载对应的 hello.bin 和 纹理图片 hello.jpg。

> 由于牵扯到不同的文件 webpack 编译，所以我们选择将 .gltf、.bin、.jpg 文件放在 src/public/ 目录中。



<br>

至此，加载 .gltf 文件讲解完成。

就这？明明就几行代码的事情，为什么还要花这样大的篇幅来讲解 .obj 和 .glb、gltf ？

答：要想学得深入，就一定要知道原理，知道 obj 和 gltf 的差异，知其然也要知其所以然。



<br>

我这里提供几个上找到的 glTF 文件资源，方便自己练习使用。

**一个黄色的小鸭子：**

1. https://vr.josh.earth/assets/models/duck/duck.gltf
2. https://vr.josh.earth/assets/models/duck/duck.bin
3. https://vr.josh.earth/assets/models/duck/duck.png



<br>

**一个简易3D社区**

https://threejsfundamentals.org/threejs/resources/models/cartoon_lowpoly_small_city_free_pack/scene.gltf

> 这个小区模型比较大，你需要适当调整一下镜头参数，才可以看清楚全貌



<br>

**一个酷酷的头盔**

https://cdn.khronos.org/assets/api/gltf/DamagedHelmet.glb



<br>

**一个宇航员**

https://modelviewer.dev/shared-assets/models/Astronaut.glb

> 真的好酷！





<br>

## 谷歌开源的一个JS库：model-viewer

在搜索 glTF 相关文章时，我无意中发现另外谷歌公司开源的一个 JS 项目： model-viewer



项目 Github 地址：https://github.com/google/model-viewer

项目官网：https://modelviewer.dev/

项目介绍：Easily display interactive 3D models on the web & in AR

> 简单来说就是：在 Web 或 AR 中，一个简单的用来显示 3D 模型的 JS 库。



<br>

具体用法：

```
<script type="module" src="https://unpkg.com/@google/model-viewer/dist/model-viewer.min.js"></script>

<model-viewer src="shared-assets/models/Astronaut.glb" alt="A 3D model of an astronaut" auto-rotate camera-controls></model-viewer>
```

> 确实够简单了，就是引入 viewer ，然后可以使用 <model-viewer> 标签插入模型渲染显示标签。
>
> 简直和插入图片标签 <img> 没啥区别。



<br>

交互效果：除了可以渲染出 3D 模型文件，还默认配备有类似 OrbitControls 相同的交互效果。



<br>

兼容性：目前 苹果浏览器 Safari、火狐 Firefox 并不支持。



<br>

至此，关于如何加载 glTF 文件已讲解完毕。



**但是有一点我们没有提到，就是使用 glTF 中自带的灯光、镜头、动画等内容。**

由于目前我还不会在 Blender 创建动画，所以这一块我们暂且保留，等待以后有机会再继续学习。



<br>

在 Three.js 中，还有很多其他文件格式的加载器，我们就不逐个讲解了，具体的可以查阅官方文档。



<br>

你以为本文结束了？没有！

在上面示例中，我们实际上漏掉了一个非常重要的知识点：加载被压缩过的 .glb 文件



<br>

## glTF文件压缩和加载(解压)——Draco

在本文的示例中，所演示加载的 .glb 文件是我自己在 Blender 中创建导出的。

如同图片文件一样，也有专门针对 .glb 文件压缩的工具，最为著名的就是谷歌公司开源的：draco



### Draco简介

Draco 是一种库，用于压缩和解压缩 3D 几何网格(geometric mesh) 和 点云(point cloud)

draco官网：https://google.github.io/draco/

draco源码：https://github.com/google/draco



<br>

draco 底层是使用 c++ 编写的。

draco 可以在不牺牲模型效果的前提下，将 .glb 文件压缩体积减小很多。

> 就好像将普通文件压缩成 .zip 一样

> 至于文件减少多少，这个暂时没有查询到



<br>

#### Draco使用流程是：

1. 使用 Draco 将模型压缩，最终压缩后的文件格式为 .drc 或 .glb

   > Draco 可以压缩众多 3D 格式文件，.glb 仅仅是其中一种

2. 在 .glb 文件内部有一个特殊字段，用来表述本文件是否经过了 draco 压缩

3. 当客户端(JS) 使用 GLTFLoader 去加载某个 .glb 文件时会去读取该标识

4. 若判断该 .glb 文件未被压缩则直接进行加载和解析

5. 若判断该 .glb 文件是被 draco 压缩过的，则会尝试调用 draco 解压类，下载 .glb 文件的同时进行解压，最终将下载、解压后的 .glb 数据传递给 GLTFLoader 使用

   > 这就引申出来一个事情：我们需要提前将负责 draco 解压的类传递给 GLTFLoader，具体如何做请看后面的讲解。



<br>

#### 如何使用 Draco 压缩 .glb 文件？

具体如何操作实现，暂时我也没有学习，先搁置一下。

> 敬请期待以后的更新



<br>

#### 如何在Three.js 中加载压缩过的 .glb 文件？

关于 Draco 的介绍，可以查看 Three.js 对于 Draco 的介绍描述：

https://github.com/mrdoob/three.js/tree/dev/examples/js/libs/draco



Three.js 源码包中 draco 针对 gltf 文件的解压文件库：

1. draco/ 目录下有 4 个文件：draco_decoder.js、draco_decoder.wasm、draco_encoder.js、draco_wasm_wrapper.js

2. draco/gltf/ 目录下面同样有 4 个文件

   > 请注意 draco/ 和 draco/gltf/ 目录下的 4 个文件虽然是名字一样，但是他们内容并不相同。

分别解释一下这 4 个文件：

1. draco_decoder.js

   > draco 解压(解码) 相关 js

2. draco_decoder.wasm

   > .wasm 文件是 WebAssembly 解码器
   >
   > 关于 WebAssembly 更多知识，请执行查阅：https://www.wasm.com.cn/

3. draco_encoder.js

   > draco 压缩(编码) 相关 js

4. draco_wasm_wrapper.js

   > 用于封装 .wasm 解码器的 js



重点来了...

<br>

#### 第1步：拷贝 draco 文件到项目 public 中

我们将 Three.js 中 examples/js/libs/draco 目录拷贝到 React 项目的 public 目录中。

> draco 属于第 3 方库，我们目前暂时采用拷贝到 public 目录中这种形式
>
> 请记得一定拷贝的是 draco/，其中包含 draco/gltf/ 目录



#### 第2步：实例化一个 DRACOLoader，并传递给 GLTFLoader

> 关于 DRACOLoader 的详细解释，请参考官方文档：
>
> https://threejs.org/docs/#examples/zh/loaders/DRACOLoader

<br>

我们将之前 GLTFLoader  的代码修改如下：

```diff
+	import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader'

	const gltfLoader = new GLTFLoader()

+	const dracoLoader = new DRACOLoader()
+	dracoLoader.setDecoderPath('./examples/js/libs/draco/')
+	dracoLoader.setDecoderConfig({ type: 'js' })
+	gltfLoader.setDRACOLoader(dracoLoader)

	gltfLoader.load('./model/vivo.glb', (gltf) => {
    	scene.add(gltf.scene)
	})
```



<br>

下面就针对上面 4 行核心代码进行解释说明：

1. `const dracoLoader = new DRACOLoader()`

   实例化一个 DRACOLoader

2. `dracoLoader.setDecoderPath('./examples/js/libs/draco/')`

   设置 dracoLoader 应该去哪个目录里查找 解压(解码) 文件

3. `dracoLoader.setDecoderConfig({ type: 'js' })`

   设置 dracoLoader 的配置项

4. `gltfLoader.setDRACOLoader(dracoLoader)`

   将 dracoLoader 传递给 gltfLoader，供 gltfLoader 使用

至此，结束！



<br>

虽然 draco 非常复杂，但是对于我们使用者而言却很简单，仅仅上面 4 行代码即可实现加载被 draco 压缩过的 .glb 文件。



<br>

## 加载.drc模型文件

在上面示例中，我们加载的是被 draco 压缩过的 .glb 文件。

那如果是被 draco 压缩过的 .drc 文件呢？

答：更加简单，直接使用 DRACOLoader即可。



<br>

DRACOLoader使用示例代码如下：

```
const loader = new DRACOLoader();
loader.setDecoderPath( '/examples/js/libs/draco/' );
loader.preload();

loader.load('./xxx/model.drc',
	function ( geometry ) {
		const material = new THREE.MeshStandardMaterial( { color: 0x606060 } );
		const mesh = new THREE.Mesh( geometry, material );
		scene.add( mesh );
	}
}
```



<br>

## 补充说明：修改模型位置偏差

无论加载 .obj 文件，还是本章讲解的加载 .gltf 文件，假设模型在建模软件中位置中心并不是原点，而是非常偏远的位置。

那么文件加载完成后，将模型添加到场景中，模型的位置并不在场景视角的中心位置，如果位置过于偏远，甚至有可能根本看不见模型。

我们可以通过以下方式，计算模型的位置偏差，并修正模型的位置，使其出现在视野中心位置。

```
const loader = new GLTFLoader()
loader.load('./model/lddq.gltf', (gltf) => {
    const group = gltf.scene

    const box = new Three.Box3().setFromObject(group)
    const center = box.getCenter(new Three.Vector3())

    group.position.x += (group.position.x - center.x)
    group.position.y += (group.position.y - center.y)
    group.position.z += (group.position.z - center.z)

    scene.add(group)
})
```

> Box3 的介绍请执行查阅官方文档。



<br>

下一章节，我们要学习如何添加 场景背景，呵， VR 看房效果要来了！