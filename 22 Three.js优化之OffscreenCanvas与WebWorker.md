# 22 Three.js优化之OffscreenCanvas与WebWorker

我们知道 JS 是单线程，可以通过 WebWorker 将一些复杂计算执行命令从主线程中分离出来。



关于 WebWorker 的使用方法，请参考：

https://developer.mozilla.org/zh-cn/docs/web/api/web_workers_api/using_web_workers

或者查看我写的另外一篇文章：[WebWorker学习笔记.md](https://github.com/puxiao/notes/blob/master/WebWorker%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0.md)



<br>

一些比较新的浏览器(例如谷歌浏览器) 还有另外一个和 WebWorker 搭配使用、针对画布的类：OffscreenCanvas。

**OffscreenCanvas 的作用就是将 canvas 的控制权转让给 Web Worker。**

> 因此 OffscreenCanvas 必须搭配 Web Worker 一起使用。



> 补充一个知识点：
>
> 和 OffscreenCanvas 类似的还有 ArrayBuffer、 MessagePort、ImageBitmap，他们都可以与 WebWorker 搭配使用



<br>

## OffscreenCanvas的概念和用法

#### OffscreenCanvas的基本概念

你可以把 WebWorker 做为各种复杂类型的后台运算线程，作用范围比较广泛。

而 OffscreenCanvas 则专门针对 Canvas 做离屏渲染。

> 注意，这里提到的 离屏渲染  和 我们使用 WebGLRendererTarget 来做的离屏渲染，从概念上是类似的

> OffscreenCanvas 的 “离屏” 是指浏览器 DOM 而言
>
> WebGLRenderTarget 的 “离屏” 只指 Three.js 的主场景(Scene) 而言

你可以把 OffscreenCanvas 看作是针对 Canvas 的特殊 WebWorker 应用场景。

但是请记得：目前绝大多数浏览器均已支持 WebWorker，但是对 OffscreenCanvas 的支持度并不高。



<br>

**如何创建 OffscreenCanvas ？**

不可以使用 new OffscreenWorker() 的方式来创建 OffscreenCanvas，而是使用 canvas.transferControlToOffscreen() 来获得 canvas 对应的 OffscreenCanvas。



<br>

**如何检测当前浏览器是否支持 OffscreenCanvas？**

我们只需检查 canvas 是否存在 transferControlToOffscreen 方法：

```
if(canvas.transferControlToOffscreen !== null){
    console.log('当前浏览器支持 OffscreenCanvas')
}else{
    console.log('当前浏览器不支持 OffscreenCanvas')
}
```

> 我们是通过检查 canvas 对象上是否包含 .transferControlToOffscreen 方法来判断是否当前浏览器支持 OffscreenCanvas 的。
>
> 这里补充一个 JS 知识，如何判断某个对象上是否有某个属性。
>
> 假设有一个对象 
>
> ```
> const obj = {
>   a:undefined,
>   b:null
> }
> ```
>
> 此时我们使用 if(obj.a) 或 if(obj.b) 都是无法准确判断出到底 属性 a、b 是否存在。
>
> 那么这个时候就可以使用以下 2 种方式来进行判断：
>
> ```
> if('a' in obj) { ... }
> 
> if(Reflect.has(obj,'b')){ ... }
> ```
>
> 使用 in 或者 Reflect.has() 就可以准确判断出对象上是否具有某属性或方法，即使该属性的值为 undefined
>
> > 注意：上面 2 种查询方式都需要属性名的字符串值，为了更好的语法提示，我们示例中并不这样用。



<br>

#### OffscreenCanvas的用法

> 再次强调：通常情况下 OffscreenCanvas 必须搭配 Web Worker 一起使用。

我们单独创建一个 JS 文件，将 canvas 绘制的一些 JS 代码写在这个文件中。

**大体步骤：**

1. 创建一个单独的 JS 文件，用来编写 Three.js 场景内容和渲染代码

   > 会以这个 JS 文件来作为 web worker 的调用文件对象

2. 在主场景中，获得 DOM 中的 canvas

3. 获取 canvas 对应的 OffscreenCanvas

   ```
   const offscreen = canvas.transferCountrolToOffscreen()
   ```

4. 创建一个 worker 对象，并且设置一些消息参数

   ```
   const worker = new Window.Worker('xx/xxxx.js',{type:'module'})
   worker.postMessage({type:'main',canvas:offscreen},[offscreen])
   ```

5. 由于 web worker 不允许访问 DOM 事件，例如浏览器窗口尺寸改变事件、鼠标事件等，所以当这些事件发生后，我们需要通知 worker，将事件对应的一些参数和变动发送给 worker，以便 worker 中的 canvas 渲染逻辑作出对应的响应。

   > 窗口尺寸改变事件我们还比较容易解决，无非就是把新的尺寸发送给 worker，比较难的是像 鼠标事件、键盘事件等，需要稍微复杂的一些传递方式才可以解决。
   >
   > 在本文后半部分会有详细讲解。



<br>

**实际差异：**

刚才将的是理论上大体步骤，但是由于我们本教程的示例代码，实际上是运行在 React + TypeScript 环境上的，也就是说我们需要编写的是 worker.ts 而不是 worker.js。

> 当然你也可以采取在编写 worker 时使用 .js 而不是 .ts，只不过这样就失去了 TypeScript 的便利性。



我们推荐的解决方案是使用 webpack 的插件：worker-loader 来解决 react + typescript 环境中编写 worker。

 具体的配置步骤，请参考我写的另外一篇文章：[React内嵌WebWorker代码](https://github.com/puxiao/notes/blob/master/WebWorker%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0.md#React%E5%86%85%E5%B5%8CWebWorker%E4%BB%A3%E7%A0%81)



接下来，我们将通过一个实际的例子，来演示一遍 OffscreenCanvas + Worker。



<br>

## 离屏画布渲染示例：HelloOffscreenCanvas

假设你已经配置好了 worker-loader，那么我们开始本示例。



<br>

#### 示例目标：

1. 场景上有 3 个不同颜色，不断旋转的立方体
2. 我们将场景中的渲染工作，从主程序中抽离出去，让 Web Worker 来负责场景的渲染工作，依次来减轻主程序的运算负担。



<br>

**补充说明：**

1. 我们场景中动画渲染本身计算量并不是很大，即使我们不使用 web worker 浏览器也不会卡顿，但本示例只是为了演示如何使用 OffscreenCanvas + Worker。
2. 我们先假设你的浏览器是支持 OffscreenCanvas 的。



<br>

#### 关键点说明：

默认 主程序(index.tsx 或 index.ts) 与 分线程(Worker.ts) 彼此通过 .postMessage() 互相发送数据。

而 .postMessage() 默认发送的数据是深拷贝，而不是引用。

> 因为本质上 index.tsx 和 worker.ts 就不在同一个线程中，无法共享数据
>
> 支持共享数据的 SharedWorker 目前浏览器支持度还不够高。

假设我们是把场景渲染的计算工作转移到了 worker.ts 中，但是 worker.ts 每次计算好 canvas 画面内容后再发送给 index.tsx，每次都执行一次深拷贝，性能反而低下。

> Web Worker 本身是无法访问 DOM 元素的

幸好 .postMessage() 方法的第 2 个参数，允许我们将一些数据类型比较大的对象，直接将控制权转移给 worker.ts。

**也就是说，OffscreenCanvas + Worker 不是走以下流程：**

1. worker.ts 负责创建 Three.js 场景和物体
2. worket.ts 负责渲染得到 场景画面数据
3. worket.ts 通过 .postMessage() 将离屏渲染得到的 画布画面内容数据 发送给 index.tsx
4. index.tsx 接收 画布画面内容数据 并渲染到 canvas DOM 中

**而是走以下流程：**

1. index.tsx 通过 canvas.transferToOffscreenCanvas() 得到 OffscreenCanvas
2. index.tsx 通过 .postMessage() 第 2 个参数，将 [OffscreenCanavs] 传递给 worker.ts，也就是说将 canvas 的控制权完全交给 worker.ts
3. 接下来就是 worker.ts 负责创建 Three.js 场景和物体，并且渲染场景画面内容直接赋予给 OffscreenCanavas。



<br>

#### 其他补充：

由于 worker 本身无法获取 DOM ，以及无法获取浏览器某些事件，例如浏览器的窗口大小变动事件。

所以当浏览器窗口尺寸发生变化后，我们要让 index.tsx 及时通知 worker.ts 新的浏览器窗口宽高，以便让 worker.ts 作出相应的调整。

> 与窗口尺寸改变相似的还有鼠标移动事件，也可以通过传递当前鼠标坐标位置传递给 Worker 以便做出相应的处理。后期我们会学习如何做场景物体拾取效果，就是鼠标放到某个物体上时物体做出相应变化，这种场景就会需要用到鼠标坐标。



接下来，就开始具体编写代码吧。



<br>

#### message-data.ts

> src/components/hello-offscreen-canvas/message-data.ts

message-data.ts 的作用是定义一些参数名、参数值的类型，以便我们获得好的 TS 语法提示。

> 注意：message-data.ts 会同时被 index.tsx 和 worker.ts 引入，这样做的效果是：
>
> 1. index.tsx 可以比较容易知道 worker.ts 内部定义的函数名叫什么
> 2. worker.ts 可以比较容易知道 index.tsx 传递过来的参数类型是什么

```
//定义画布的尺寸类型数据结构
export type CanvasSize = {
    width: number,
    height: number
}

export enum WorkerFunName {
    main = 'main',
    updateSize = 'updateSize'
}

//定义 MessageEvent data 的数据结构
export type MessageData =
    { type: WorkerFunName.main, params: OffscreenCanvas }
    |
    { type: WorkerFunName.updateSize, params: CanvasSize }
```



<br>

#### worker.ts

> src/components/hello-offscreen-canvas/worker.ts

```
import * as Three from 'three'
import { CanvasSize, MessageData, WorkerFunName } from './message-data'

let renderer: Three.WebGLRenderer
let camera: Three.PerspectiveCamera
let scene: Three.Scene

//定义初始化的函数
const main = (canvas: OffscreenCanvas) => {
    //开始创建 3D 相关场景
    renderer = new Three.WebGLRenderer({ canvas })
    camera = new Three.PerspectiveCamera(45, 2, 0.1, 100)
    camera.position.z = 4
    scene = new Three.Scene()

    const colors = ['blue', 'red', 'green']
    const cubes: Three.Mesh[] = []
    colors.forEach((color, index) => {
        const material = new Three.MeshPhongMaterial({ color })
        const geometry = new Three.BoxBufferGeometry(1, 1, 1)
        const mesh = new Three.Mesh(geometry, material)
        mesh.position.x = (index - 1) * 2
        scene.add(mesh)
        cubes.push(mesh)
    })

    const light = new Three.DirectionalLight(0xFFFFFF, 1)
    light.position.set(-2, 2, 2)
    scene.add(light)

    const render = (time: number) => {
        time *= 0.001
        cubes.forEach((item) => {
            item.rotation.set(time, time, 0)
        })
        renderer.render(scene, camera)
        self.requestAnimationFrame(render)
    }
    self.requestAnimationFrame(render)
}

//定义用来接收画布尺寸更新的函数
const updateSize = (newSize: CanvasSize) => {
    camera.aspect = newSize.width / newSize.height
    camera.updateProjectionMatrix()
    renderer.setSize(newSize.width, newSize.height, false)
}

const handleMessage = ((eve: MessageEvent<MessageData>) => {
    switch (eve.data.type) {
        case WorkerFunName.main:
            main(eve.data.params)
            break
        case WorkerFunName.updateSize:
            updateSize(eve.data.params)
            break
        default:
            throw new Error(`no handle for the type`)
    }
})
self.addEventListener('message', handleMessage)

const handleMessageError = () => {
    throw new Error('Worker.ts: message error ...')
}
self.addEventListener('messageerror', handleMessageError)

//导出 {} 是因为 .ts 类型的文件必须有导出对象才可以被 TS 编译成模块，而不是全局对象
export { }
```



<br>

#### index.tsx

> src/components/hello-offscreen-canvas/index.tsx

```
import { useEffect, useRef } from 'react'
import { WorkerFunName } from './message-data'
import Worker from 'worker-loader!./worker'

import './index.scss'

const HelloOffscreenCanvas = () => {

    const canvasRef = useRef<HTMLCanvasElement | null>(null)

    useEffect(() => {
        if (canvasRef.current === null) { return }

        const canvas = canvasRef.current as HTMLCanvasElement
        const offscreen = canvas.transferControlToOffscreen()

        const worker = new Worker()
        worker.postMessage({ type: WorkerFunName.main, params: offscreen}, [offscreen])

        const handleMessageError = (error: MessageEvent<any>) => {
            console.log(error)
        }
        const handleError = (error: ErrorEvent) => {
            console.log(error)
        }
        worker.addEventListener('messageerror', handleMessageError)
        worker.addEventListener('error', handleError)

        const handleResize = () => {
            worker.postMessage({
                type: WorkerFunName.updateSize,
                params: { width: canvas.clientWidth, height: canvas.clientHeight }
            })
        }
        handleResize()
        window.addEventListener('resize', handleResize)

        return () => {
            worker.removeEventListener('messageerror', handleMessageError)
            worker.removeEventListener('error', handleError)
        }
    }, [canvasRef])


    return (
        <canvas ref={canvasRef} className='full-screen' />
    )
}

export default HelloOffscreenCanvas
```

我们可以看到 index.tsx 中已经没有任何 Three.js 相关的代码了。

运行调试一切正常。



<br>

**接下来我们要解决 2 个问题：**

1. 控制 3D 场景用到的 OrbitControls 类，在新建时需要传递 HTML DOM 元素，交互的过程中需要 DOM 元素的鼠标事件和键盘事件，但是 worker 内部又不能访问 DOM 元素，那该如何解决？

2. 假设浏览器不支持 OffscreenCanvas ，那又该如何拆分我们的代码可以做到兼容？

   > 在软件开发术语中，会使用 “鲁棒性或健壮性” 来指代码的兼容性和容错性。



<br>

## 模拟并添加OrbitControls

> 你需要先忘记我们上面刚才讲过的示例代码，本小节中所有的代码和上面示例代码没有任何关联。



#### 目前无法使用OrbitControls的困境

在以前所有的例子中，我们添加镜头轨道控制器，都是使用以下代码：

```
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

const controls = new OrbitControls(camera,canvas)
或者
const controls = new OrbitControls(camera,window.body)
```

OrbitControls 构造函数第 2 个参数无论是 canvas 还是 window.body，一定是一个 DOM 元素。

我们已经将 Three.js 相关代码都转移到了 Worker 内部，但是Web Worker 内部是无法获取 DOM 元素的，那么意味着 OrbitControls 根本就无法初始化。



> 不要想着尝试将 DOM 元素作为 参数，使用 .postMessage() 函数传递给 Worker 内部，因为 .postMessage() 函数中的参数并不是传递引用，而是直接深度复制一份。



<br>

#### 那么究竟该怎么办呢？

我们先研究一下 OrbitControls 的源码：

https://github.com/mrdoob/three.js/blob/dev/examples/jsm/controls/OrbitControls.js



<br>

我把源码中和 Dom 元素相关的一些关键代码摘录出来：

```
//设置 scope = this
var scope = this; 

var OrbitControls = function ( object, domElement ) {
  //下面这行代码相当于 scope.domElement = domElement
  this.domElement = domElement;
}

//添加鼠标右键(上下文)菜单事件侦听
scope.domElement.addEventListener( 'contextmenu', onContextMenu);

//添加触控笔和鼠标摁下事件侦听
scope.domElement.addEventListener( 'pointerdown', onPointerDown);

//添加鼠标滚轴事件侦听
scope.domElement.addEventListener( 'wheel', onMouseWheel );

//添加触摸开始、结束、移动事件侦听
scope.domElement.addEventListener( 'touchstart', onTouchStart);
scope.domElement.addEventListener( 'touchend', onTouchEnd);
scope.domElement.addEventListener( 'touchmove', onTouchMove);

//添加键盘事件侦听
scope.domElement.addEventListener( 'keydown', onKeyDown);
	
if ( state !== STATE.NONE ) {
    //添加触控笔和鼠标移动事件侦听
    scope.domElement.ownerDocument.addEventListener( 'pointermove', onPointerMove, false );
    //添加触控笔和鼠标松开事件侦听
    scope.domElement.ownerDocument.addEventListener( 'pointerup', onPointerUp, false );
    scope.dispatchEvent( startEvent );
}
```

> 补充说明：在最新的浏览器事件中 pointer 相关事件即包含触控笔，也包含鼠标。所以 pointerdown、pointermove、pointerup 这 3 个事件对应 触控笔或鼠标 对应的事件，相当于是 2 者的合体。

<br>

从上面可以看出，我们初始化传递给 OrbitControls 的 DOM 元素主要是用来添加各种事件侦听。

> 当然 OrbitControls 代码中也有对应的 removeEventListener 移除事件侦听。

> 补充一点：contextmenu 事件虽然目前部分浏览器支持(主要是火狐浏览器)，但是在 MDN 的文档中已经明确该事件即将被废除。



<br>

**重点来了，请听好：**

1. 既然 OrbitControls 需要 DOM 元素的目的是为了获取并添加各种事件侦听

2. 而 Worker 中无法获取 DOM 元素

3. 那么有没有可能我们虚拟出来一个对象，让该对象拥有和 DOM 元素相同的事件 API 

   > 换句话说，就是让这个虚拟出来的对象具备抛出事件的能力

   > 补充一点，这里说的 DOM 事件其实有 2 种：
   >
   > 1. DOM 元素上的各种用户交互 5 种事件：鼠标事件、滚轴事件、键盘事件、触摸事件、右键菜单事件
   > 2. 浏览器窗口尺寸发生变化引发 DOM 元素尺寸发生变化的 window resize 事件
   > 3. 我们需要做的就是分别模拟出以上 6 种事件

4. 然后我们让 OrbitControls 去侦听这个虚拟对象所发出的各种事件

5. 也就是说让这个虚拟对象代替真实的 DOM 元素，以此来解决我们目前的困境



<br>

**思路有了，那该具体怎么做呢？**

首先我们要明白一件事，原生 DOM 对应的属性、方法、事件、以及事件携带的属性 种类繁多且复杂。

而 OrbitControls 并不是每一个属性、方法、事件、事件每一个属性 都使用到了，也就是说我们所谓的 “模拟”不需要 100% 一模一样，我们只需要提供 OrbitControls 需要的即可。

究竟需要模拟出 DOM 元素哪些属性和方法，我们会在具体代码时讲解。

此刻，我们只以事件携带的属性来举例说明：

1. 对于鼠标滚轴滚动来说，OrbitControls 只需要使用到该事件的 deltaY 值

2. 对于键盘事件来说，OrbitControls 只需要使用到该事件的 ctrlKey、metaKey、shiftKey、keyCode 值

   > meta 键？
   >
   > 这个 meta 键在 Windows 键盘上相当于 windows 键、在苹果键盘上是一个四瓣的小花。

   > 在 OrbitControls 内部并未使用到 altKey 值

   > 注意：由于 OrbitControls 仅使用到键盘 上/下/左/右 4 个键，我们还可以主动过滤掉一些无用的摁键，换句话说也就是提前判断一下是否是以上 4 个方向键，如果不是，则直接跳过，不传递该事件

   > 注意：对于目前版本 Three.js r126 版本而言，OrbitControls 键盘事件读取使用的是 keyCode，但是 event.keyCode 事实上已经不被推荐使用，建议使用 event.code 属性。
   >
   > 所以我顺带向 Three.js 官方提交了 PR，将 keyCode 修改为 code，这个 PR 已经被官方审查通过了，会在 r127 版本中使用。因此，我也成为 Three.js 代码贡献者了。
   >
   > 我提交的这个 PR：https://github.com/mrdoob/three.js/pull/21409

   > 关于为什么不再推荐使用 event.keyCode 主要是因为 keyCode 不能够比较清晰正确返回键盘所摁键，例如 冒号和分号 都是同一个键，此时 keyCode 就无法精确区分。

3. 对于鼠标事件，OrbitControls 需要使用到该事件的 pointerType、button、clientX、clientY、ctrlKey、metaKey、shiftKey 值

4. 对于触摸(touch)事件，OrbitControls 需要使用每一个 触摸点的 pageX、pageY 属性值

5. ...



<br>

我们需要让 “虚拟对象” 抛出 “虚拟事件”，并且虚拟事件拥有上面那些属性值。

> 补充说明：这里所说的 “抛出事件” 暗含 2 种事件：
>
> 1. 用户交互事件：鼠标事件、滚轴事件、键盘事件...
>
> 2. 浏览器窗口变化引发 “DOM 元素” 内部尺寸属性相关的修改事件
>
>    > 除了 window 拥有 resize 事件之外，普通 DOM 元素是不具备 resize 事件的，我们可以将这个原本不存在的 DOM 元素 resize 追加到 我们的消息中，对于 用户交互事件 我们选择直接抛出、对于 window resize 引发的尺寸修改事件，我们选择内部直接处理。



<br>

**属性补充说明：**

在 OrbitControls 内部，还会对监听的 DOM 元素的根元素添加 2 个事件侦听。

```
// scope = this
scope.domElement.ownerDocument.addEventListener( 'pointermove', onPointerMove );
scope.domElement.ownerDocument.addEventListener( 'pointerup', onPointerUp );
```

因此我们模拟的元素还要拥有 .ownerDocument 属性值。

> 在后面的实际代码中你会看到，我们会让 模拟元素 .ownerDocument 指向自身。
>
> this.ownerDocument = this



<br>

**document补充说明：**

在 OrbitControls 类的构造函数中，有以下代码：

```
if ( domElement === undefined ) console.warn( 'THREE.OrbitControls: The second parameter "domElement" is now mandatory.' );

if ( domElement === document ) console.error( 'THREE.OrbitControls: "document" should not be used as the target "domElement". Please use "renderer.domElement" instead.' );
```

也就是说在初始化 OrbitControls 时会对要侦听的 DOM 元素进行检查。在第 2 行代码中需要使用到 document 这个对象，但是在 web worker 中根本无法访问 document，所以我们需要给 worker 添加一个 document 对象。代码如下：

```
//@ts-ignore
self.document = {}
```

1. 添加 //@ts-ignore 这样注释，可以让 TypeScript 忽略下面一行代码的检查。

   > 因为 window.document 在 TS 版本里被定义为 只读对象，是无法修改的。
   >
   > 如果我们不选择忽略 TS 检查，当去执行 self.document = {} 时 TS 就会报错。

2. 我们添加的 self.document = {} 纯粹是为了让 OrbitControls 构造函数可以去访问到 document 对象，避免报错。



<br>

**所谓的 “抛出事件” 实现的途径是由 我们虚构出的一个 事件派发器 来实现的。**

**该事件派发器的 3 个功能责任：**

1. 监听功能：监听真实DOM元素事件：用户交互事件、浏览器尺寸变化事件
2. 事件转化：将监听到的事件内容，转化为一条数据，该数据包含该事件中 OrbitControls 所需要的各种属性值
3. 消息传递：将事件转化后的数据，通过 web worker 的 postMessage() 传递给 web worker



<br>

**所谓的“模拟的DOM元素”，也就是在 web worker 中工作的 DOM 元素。**

**该“模拟的DOM元素”的 2 个功能责任：**

1. 模拟 DOM 元素的属性和方法
2. 处理 DOM 元素需要处理的各种事件

**该“模拟的DOM元素”的生命工作流程为：**

1. index.tsx 中添加对 真实 DOM 元素的监听，并且通知  worker 创建“模拟元素”的消息

2. worker.ts 中接收消息，开始创建一个 “模拟元素”，并且把该元素传递给 OrbitControls

4. index.tsx 中监听到真实 DOM 元素触发的各种事件，将该事件分析处理，转化为一条约定好的消息 并发送给 worker

6. worker.ts 中接收到事件消息后，将该消息转发给 “模拟的DOM元素”

7. “模拟的DOM元素” 接收该消息，然后将该消息(包含事件类型、事件属性)抛出，供 OrbitControls 使用

   > 我们抛出的事件，也是模拟出来的事件，并不是原始 DOM 事件，只不过我们模拟出来的事件中恰好包含 OrbitControls 需要的所有属性和方法。
   
   > 在 web worker 中工作的 OrbitControls，从始至终都不知道自己操作的其实是一个假的 DOM 元素，以及监听的事件也是假的 DOM 事件。



<br>

**整个事件的流程为：**

**真实DOM事件 > 转化为消息 > 发送给 worker > 传递给“模拟的DOM元素” > 抛出该消息(相当于抛出该事件)**

通过 事件 > 消息 > 消息 > 事件 这样一波操作过后，可以让 OrbitControls 就像监控真实 DOM 元素一样监控运行在 web worker 中的那个 “模拟的DOM元素”。

> 我故意一直使用 “模拟的DOM元素”这个词，而没有使用 “虚拟DOM”，就是为了让我们避免和 react 中 虚拟DOM 的一词弄混淆。



<br>

**关于 事件抛出 的补充说明：**

我们让 “虚构元素” 继承于 Three 已内置的 EventDispatcher，这样 “虚构元素” 就 具备 .dispatcheEvent() 方法。

```
import { EventDispatcher } from 'three'
```

<br>



为什么不使用 原生 JS 提供的 EventTarget ？

这是因为原生 JS 提供的 EventTarget 虽然也有 .dispatcheEvent()，但问题是它只可以抛出 JS 中的 Event 实例，而我们在 worker 中并不能使用 Event。

<br>



**补充说明：**

无论 JS 原生的 EventTarget，还是 Three.js 内置的 EventDispatcher，他们内部本质上都是执行的是函数调用，所以他们的执行过程都是 同步的。

> 浏览器中原生的各种事件处理函数 其实是异步的。



<br>

**整体解决思路回顾：为什么我们可以实现？**

回顾一下本小节开头的困境问题：web worker 本身不可以访问真实 DOM 元素，当然也包括 DOM 事件。



<br>

**那么我们究竟是怎么做到的？以及为什么我们可以做到？**

**答：机缘巧合**

**第1种机缘巧合：**虽然 web worker 无法访问真实的 DOM 元素，但是 canvas 元素对应的 OffscreenCanvas 却是一个例外，通过主线程让出 canvas 绘制控制权，让 web worker 拥有了可以操作并绘制 canvas 元素的能力。

> 目前火狐浏览器并不支持 OffscreenCanvas，所以本示例还要考虑在非 worker 情况下的场景创建。



<br>

**第2种机缘巧合：**虽然 web worker 无法直接监听真实 DOM 元素事件，但是 web worker 内部却可以运行 抛出事件 这个操作。于是我们通过 事件 > 消息 > 消息 > 事件 的操作让 web worker 内部模拟出的 DOM 元素拥有了像真实 DOM 元素一样的各种事件抛出机制。

> 再次提醒一下：在 web worker 中，不光 DOM 元素是我们模拟出来的，就连抛出的 DOM 元素事件也是我们模拟出来的。

最终我们让 web worker 中 OrbitControls 正常运行起来了。



<br>

> 如果你对我上面的讲述还不太理解，那么多读几遍，不要着急接着往下看。因为如果整体的思路你没有理解透，那么下面这些具体实现的代码，阅读起来也会一头雾水。

> 实话实说，在学习本章内容，看英文原版教程，我花了将近一周的时间，才弄明白整个原理。
>
> 本文讲的内容可能是整个 three.js 教程中最绕、最复杂的，但是为了性能优化，学习一下是非常值得的。



<br/>

**补充一个和本文无关的知识点：**

除了 window 之外，其他 DOM 元素尺寸发生变化时，是不会触发任何事件的。

1. 通过 css 修改 DOM元素尺寸
2. 因为 window resize 事件而修改 DOM 元素尺寸

如果你想监听某 DOM 元素尺寸的变化，在最新的 DOM3 标准中，可以通过 MutationObserver 来监控，具体用法请查阅 MDN 官方文档。

https://developer.mozilla.org/zh-CN/docs/Web/API/MutationObserver

<br/>



### 整体思路示意图

![using-orbitcontrols-in-worker.jpg](https://puxiao.com/demo/using-orbitcontrols-in-worker/using-orbitcontrols-in-worker.jpg)



<br>

接下来开始讲解具体代码如何实现。

> 由于我使用的是 React + TypeScript，加上我有一些自己代码理解和划分，所以我下面讲述的代码和原教程很多地方都不一样。

> 过程有点复杂，希望我能讲清楚。



<br>

#### 代码模块规划

1. 定义一个类 FictionalElement 继承于 EventDispatcher，用这个类的实例来模拟 “DOM元素本身”

   > 请注意：FictionalElement 只具备(模仿、模拟) “DOM元素” 本身的一些属性和方法(例如 width、height、left、top、getBoundingClientRect() 等)，但并不具备可以直接和 web worker 通信的能力

   > 在模拟一些 “无用” 的方法时，例如 focus()、preventDefault()、stopPropagation()，将该方法代码体内不添加任何代码，只是保证有这个方法但无需有实际执行内容。

2. 定义一个类 FictionalElementManager 用来创建和管理所有 FictionalElement 实例

   > 事实上我认为这一步骤不是必须的，因为实际项目中，绝大多数情况下网页中都只会有一个 用户交互元素(通常为 canvas 或 document.body)，并不会有太多元素需要我们管理。但是本文对应的 [原版教程](https://threejsfundamentals.org/threejs/lessons/threejs-offscreencanvas.html) 中有这一环节(管理层)，那么我们也继续遵循。

   > 为了区别不同的管理对象，我们将在内部给每一个 ElementProxyReceiver 添加一个对应的 id
   >
   > 原版教程中，id 是由 FictionalElementManager 提供的，但是我认为这样并不合理，我采用的是通过外部传递 id 为 string 类型的值。

3. 定义一个类 FictionalWindow 用来模拟 window

   > 请注意，目前来说 FictionalWindow 仅仅是继承了 EventDispatcher，并没有做其他设置，先这样做，也许未来有其他需求了，再根据需要扩展它。

4. 定义一个类 ControlsProxy 用来负责基础的 真实DOM元素与 Web Worker 之间通信。

   > 当浏览器窗口尺寸发生变化时，我们模拟的 "DOM元素本身"也应该会发生尺寸变化，而这个变化的触发并不是由 FictionalElement 完成的，而是由 ControlsProxy 来完成的。 

5. 定义一个类 OrbitControlsProxy 继承于 ControlsProxy，然后重写 configEventListener() 和 dispose()

   > 你可能疑惑为什么我没有直接把代码写在同一个类里，而是要拆分成 2 个。我这样做的原因是想将一些基础的、共性的属性和方法抽离出来，这样未来有一天我们要去实现其他轨道控制器，都可以继承于 ControlsProxy

6. 定义一个类 WorkerMessageType，用来定义 发送消息 的类型

   > 每次都靠手写消息类型是不靠谱的，万一手抖拼写错字母了想检查出来都不容易。

7. 此外，虽然本教程一直都是用的是 TypeScript，但是在编写这些类的时候，考虑有些人并不使用 TS，所以我采用的是 .js + JSDoc 的方式，通过 JSDoc 代码注释的形式来定义不同消息的数据结构，没有使用 .ts。

   > 我也是因为这个示例而去学习了 JSDoc，感兴趣可查看我另外一篇学习笔记：[JSDoc的安装与使用.md](https://github.com/puxiao/notes/blob/master/JSDoc%E7%9A%84%E5%AE%89%E8%A3%85%E4%B8%8E%E4%BD%BF%E7%94%A8.md)



<br>

以上仅仅是 “虚构” 的核心代码，除此之外，我们还需要编写对应的 “应用” 层面的代码：

1. index.tsx：JS 主场景 main 代码

2. worker.ts：Worker 场景代码

3. create-world.ts：负责创建 Three.js 3D 场景的核心代码

   > create-world.ts 中的代码并不知道自己将来是运行在 Worker 中还是运行在 主场景中(Main)



<br>

**如果你能坚持看到这里没有被我绕晕，那恭喜你。**

是不是该展示具体代码了？

<br>

### 项目实际代码

原理都讲述完了，但是每个类的代码细节实在是太多，我也不想再细致讲述了，所以在 Github 单独创建了一个项目：

**https://github.com/puxiao/using-orbitcontrols-in-worker**



<br>

> 为了让全世界的人能看到我的这个代码，我竟然写了一个英文版 README.MD

你可以直接查看我写的简体中文介绍文档：

https://github.com/puxiao/using-orbitcontrols-in-worker/blob/main/README-zh_CN.md



<br>



### 说一下我的感受

本节后面的代码实现，花费了我有 1 个月的时间，整个过程即充实有痛苦。

1. 从阅读原版教程，完全看不懂 理解不了
2. 后来可以理解
3. 改为自己的实现方式
4. 不断得修改、优化代码
5. 上传到 Github 编写文档

尽管掌握了本章所写的示例代码，但是这些对实际 Three.js 的使用提升并不会有立竿见影的效果。



<br>

#### 但是！

因为不断的深入去理解，学习，我也有很多收获：

1. 我成功向 Three.js 贡献了自己的一点点代码，尤其是在提交自己的 PR 过程中，用蹩脚英语与 Three.js 代码审查人员的不断沟通，是一次很神奇的体验。

2. 通过 OrbitControls，顺带我也看了其他 轨道控制器的 一些源码，事实上我原本的野心计划是编写出所有轨道控制器的 ControlsProxy 版，但是时间和精力，这个事情只能暂时放下，等待将来或者其他感兴趣的人来编写吧。

   > 目前我的项目中只编写了 OrbitControlsProxy.js，如果你感兴趣，也可以尝试编写出其他 轨道控制器对应的 XxxControlsProxy。
   >
   > 提醒：如果你想编写其他轨道控制器的代理管理类，它需要继承于 ControlsProxy，然后重写 configEventListener() 和 dispose() 这 2 个方法。

3. 学习了 JSDoc 注释规范，也就是即使我们不使用 TypeScript，通过 JSDoc 注释依然可以进行类型定义。

   > 通过 JSDoc 的学习，让我对 TypeScript 有更加深一层的理解。
   >
   > 无论 TypeScript 还是 JSDoc，还是 VSCode IDE 本身的代码提示和自动检查，本质上都仅仅在 开发阶段 对我们编写的代码进行约束，将来 JS 运行阶段就管不了那么多了。

4. 接触了 //@ts-ignore 这个特殊注释



<br>

### 本小节结束

本小节至此结束，同时也意味着本系列教程的 “优化” 篇章讲解完成。

接下来，我们要开始新的阶段学习，下一阶段才决定我们 Three.js 实际应用领域 “质的飞越”。



<br>

**下一阶段，我们要开始学习 Three.js 中常见的各种应用场景解决方案。**

加油，好玩有趣的事情终于要开始了。



<br>