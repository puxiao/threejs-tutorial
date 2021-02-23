# 22 Three.js优化之OffscreenCanvas与WebWorker

我们知道 JS 是单线程，可以通过 WebWorker 将一些复杂计算执行命令从主线程中分离出来。

关于 WebWorker 的使用方法，请参考：

https://developer.mozilla.org/zh-cn/docs/web/api/web_workers_api/using_web_workers

或者查看我写的另外一篇文章：[WebWorker学习笔记.md](https://github.com/puxiao/notes/blob/master/WebWorker%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0.md)



一些比较新的浏览器(例如谷歌浏览器) 还有另外一个和 WebWorker 搭配使用、针对画布的类：OffscreenCanvas。

**OffscreenCanvas 的作用就是将 canvas 的控制权转让给 Web Worker。**

> 因此 OffscreenCanvas 必须搭配 Web Worker 一起使用。



> 补充一个知识点：
>
> 和 OffscreenCanvas 类似的还有 ArrayBuffer、 MessagePort、ImageBitmap，他们都可以与 WebWorker 搭配使用



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



**如何创建 OffscreenCanvas ？**

不可以使用 new OffscreenWorker() 的方式来创建 OffscreenCanvas，而是使用 canvas.transferControlToOffscreen() 来获得 canvas 对应的 OffscreenCanvas。



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



**实际差异：**

刚才将的是理论上大体步骤，但是由于我们本教程的示例代码，实际上是运行在 React + TypeScript 环境上的，也就是说我们需要编写的是 worker.ts 而不是 worker.js。

> 当然你也可以采取在编写 worker 时使用 .js 而不是 .ts，只不过这样就失去了 TypeScript 的便利性。



我们推荐的解决方案是使用 webpack 的插件：worker-loader 来解决 react + typescript 环境中编写 worker。

 具体的配置步骤，请参考我写的另外一篇文章：[React内嵌WebWorker代码](https://github.com/puxiao/notes/blob/master/WebWorker%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0.md#React%E5%86%85%E5%B5%8CWebWorker%E4%BB%A3%E7%A0%81)



接下来，我们将通过一个实际的例子，来演示一遍 OffscreenCanvas + Worker。



## 离屏画布渲染示例：HelloOffscreenCanvas

假设你已经配置好了 worker-loader，那么我们开始本示例。



#### 示例目标：

1. 场景上有 3 个不同颜色，不断旋转的立方体
2. 我们将场景中的渲染工作，从主程序中抽离出去，让 Web Worker 来负责场景的渲染工作，依次来减轻主程序的运算负担。



**补充说明：**

1. 我们场景中动画渲染本身计算量并不是很大，即使我们不使用 web worker 浏览器也不会卡顿，但本示例只是为了演示如何使用 OffscreenCanvas + Worker。
2. 我们先假设你的浏览器是支持 OffscreenCanvas 的。



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



#### 其他补充：

由于 worker 本身无法获取 DOM ，以及无法获取浏览器某些事件，例如浏览器的窗口大小变动事件。

所以当浏览器窗口尺寸发生变化后，我们要让 index.tsx 及时通知 worker.ts 新的浏览器窗口宽高，以便让 worker.ts 作出相应的调整。

> 与窗口尺寸改变相似的还有鼠标移动事件，也可以通过传递当前鼠标坐标位置传递给 Worker 以便做出相应的处理。后期我们会学习如何做场景物体拾取效果，就是鼠标放到某个物体上时物体做出相应变化，这种场景就会需要用到鼠标坐标。



接下来，就开始具体编写代码吧。



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



**接下来我们要解决 2 个问题：**

1. 控制 3D 场景用到的 OrbitControls 类，在新建时需要传递 HTML DOM 元素，交互的过程中需要 DOM 元素的鼠标事件和键盘事件，但是 worker 内部又不能访问 DOM 元素，那该如何解决？

2. 假设浏览器不支持 OffscreenCanvas ，那又该如何拆分我们的代码可以做到兼容？

   > 在软件开发术语中，会使用 “鲁棒性或健壮性” 来指代码的兼容性和容错性。



## 模拟并添加OrbitControls

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



#### 那么究竟该怎么办呢？

我们先研究一下 OrbitControls 的源码：

https://github.com/mrdoob/three.js/blob/dev/examples/jsm/controls/OrbitControls.js



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

//添加触控笔摁下事件侦听
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
    //添加触控笔移动事件侦听
    scope.domElement.ownerDocument.addEventListener( 'pointermove', onPointerMove, false );
    //添加触控笔离开事件侦听
    scope.domElement.ownerDocument.addEventListener( 'pointerup', onPointerUp, false );
    scope.dispatchEvent( startEvent );
}
```

从上面可以看出，我们初始化传递给 OrbitControls 的 DOM 元素主要是用来添加各种事件侦听。

> 当然 OrbitControls 代码中也有对应的 removeEventListener 移除事件侦听。

> 补充一点：contextmenu 事件虽然目前部分浏览器支持(主要是火狐浏览器)，但是在 MDN 的文档中已经明确该事件即将被废除。



**重点来了，请听好：**

1. 既然 OrbitControls 需要 DOM 元素的目的是为了获取并添加各种事件侦听

2. 而 Worker 中无法获取 DOM 元素

3. 那么有没有可能我们虚拟出来一个对象，让该对象拥有和 DOM 元素相同的事件 API

   > 补充一点，这里说的 DOM 事件其实有 2 种：
   >
   > 1. DOM 元素上的各种用户交互 5 种事件：鼠标事件、滚轴事件、键盘事件、触摸事件、右键菜单事件
   > 2. 浏览器窗口尺寸发生变化引发 DOM 元素尺寸发生变化的 window resize 事件
   > 3. 我们需要做的就是分别模拟出以上 6 种事件

4. 然后我们让 OrbitControls 去侦听这个虚拟对象所发出的各种事件

5. 也就是说让这个虚拟对象代替真实的 DOM 元素，以此来解决我们目前的困境



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

3. 对于鼠标事件，OrbitControls 需要使用到该事件的 ctrlKey、metaKey、shiftKey、button、clientX、clientY、pageX、pageY 值

4. 对于触摸(touch)事件，OrbitControls 需要使用每一个 触摸点的 pageX、pageY 属性值

5. ...

我们需要让 “虚拟对象” 抛出 “虚拟事件”，并且虚拟事件拥有上面那些属性值。

> 补充说明：这里所说的 “抛出事件” 暗含 2 种事件：
>
> 1. 用户交互事件：鼠标事件、滚轴事件、键盘事件...
>
> 2. 浏览器窗口变化引发 “DOM 元素” 内部尺寸属性相关的修改事件
>
>    > 除了 window 拥有 resize 事件之外，普通 DOM 元素是不具备 resize 事件的，我们可以将这个原本不存在的 DOM 元素 resize 追加到 我们的消息中，对于 用户交互事件 我们选择直接抛出、对于 window resize 引发的尺寸修改事件，我们选择内部直接处理。



**所谓的 “抛出事件” 实现的途径是由 我们虚构出的一个 事件派发器 来实现的。**

**该事件派发器的 3 个功能责任：**

1. 监听功能：监听真实DOM元素事件：用户交互事件、浏览器尺寸变化事件
2. 事件转化：将监听到的事件内容，转化为一条数据，该数据包含该事件中 OrbitControls 所需要的各种属性值
3. 消息传递：将事件转化后的数据，通过 web worker 的 postMessage() 传递给 web worker



**所谓的“模拟的DOM元素”，也就是在 web worker 中工作的 DOM 元素。**

**该“模拟的DOM元素”的 2 个功能责任：**

1. 模拟 DOM 元素的属性和方法
2. 处理 DOM 元素需要处理的各种事件

**该“模拟的DOM元素”的生命工作流程为：**

1. index.tsx 中创建该 “DOM元素”

2. index.tsx 将该元素发送给 web worker

   > 请注意，这里所谓的发送，本质上其实是 web worker 接收到之后会重新克隆一份该 “模拟的DOM元素”

3. worker.ts 中接收并克隆一份该 “DOM元素”，并且把他赋给 OrbitControls

4. index.tsx 中创建 事件派发器，并开始监听真实 DOM 元素的各种事件

5. index.tsx 中的 事件派发器 监听到真实 DOM 元素的各种事件后，将事件转化为消息并发送给 web worker。

6. worker.ts 中接收到事件消息后，将该消息转发给 “模拟的DOM元素”

7. “模拟的DOM元素” 接收该消息，然后将该消息(包含事件类型、事件属性)抛出，供 OrbitControls 使用

   > 我们抛出的事件，也是模拟出来的事件，并不是原始 DOM 事件，只不过我们模拟出来的事件中恰好包含 OrbitControls 需要的所有属性和方法。
   
   > 在 web worker 中工作的 OrbitControls，从始至终都不知道自己操作的其实是一个假的 DOM 元素，以及监听的事件也是假的 DOM 事件。



**整个事件的流程为：**

**真实DOM事件 > 转化为消息 > 发送给 worker > 传递给“模拟的DOM元素” > 抛出该消息(相当于抛出该事件)**

通过 事件 > 消息 > 消息 > 事件 这样一波操作过后，可以让 OrbitControls 就像监控真实 DOM 元素一样监控运行在 web worker 中的那个 “模拟的DOM元素”。

> 我故意一直使用 “模拟的DOM元素”这个词，而没有使用 “虚拟DOM”，就是为了让我们避免和 react 中 虚拟DOM 的一词弄混淆。



**关于 事件抛出 的补充说明：**

有 2 个已经定义好，可供我们使用的类：

1. 使用 Three.js 提供的 EventDispatcher

   ```
   import { EventDispatcher } from 'three'
   ```

2. 使用 原生 JS 提供的 EventTarget 



我们这里使用 Three.js 提供的 EventDispatcher。

> 原生 JS 中内置的 EventTarget 本质上也是一种内置的对象而已，和我们使用 Three.js 提供的 EventDispatcher 区别并不大。



**整体解决思路回顾：为什么我们可以实现？**

回顾一下本小节开头的困境问题：web worker 本身不可以访问真实 DOM 元素，当然也包括 DOM 事件。

**那么我们究竟是怎么做到的？以及为什么我们可以做到？**

**答：机缘巧合**

**第1种机缘巧合：**虽然 web worker 无法访问真实的 DOM 元素，但是 canvas 元素对应的 OffscreenCanvas 却是一个例外，通过主线程让出 canvas 绘制控制权，让 web worker 拥有了可以操作并绘制 canvas 元素的能力。

**第2种机缘巧合：**虽然 web worker 无法直接监听真实 DOM 元素事件，但是 web worker 内部却可以运行 抛出事件 这个操作。于是我们通过 事件 > 消息 > 消息 > 事件 的操作让 web worker 内部模拟出的 DOM 元素拥有了像真实 DOM 元素一样的各种事件抛出机制。

> 再次提醒一下：在 web worker 中，不光 DOM 元素是我们模拟出来的，就连抛出的 DOM 元素事件也是我们模拟出来的。

最终我们让 web worker 中 OrbitControls 正常运行起来了。



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



接下来开始讲解具体代码如何实现。

> 由于我使用的是 React + TypeScript，加上我有一些自己代码理解和划分，所以我下面讲述的代码和原教程略微不同。

> 过程有点复杂，希望我能讲清楚。



#### 代码模块规划

1. 定义一个类 ElementProxyReceiver 继承于 EventDispatcher，用这个类的实例来模拟 “DOM元素本身”

   > 请注意：ElementProxyReceiver 只具备(模仿、模拟) “DOM元素” 本身的一些属性和方法(例如 width、height、left、top、getBoundingClientRect() 等)，但并不具备可以直接和 web worker 通信的能力

   > 在模拟一些 “无用” 的方法时，例如 focus()、preventDefault()、stopPropagation()，将该方法代码体内不添加任何代码，只是保证有这个方法但无需有实际执行内容。

2. 定义一个类 ProxyManager 用来创建和管理所有 ElementProxyReceiver 实例

   > 事实上我认为这一步骤不是必须的，因为实际项目中，绝大多数情况下网页中都只会有一个 用户交互元素(通常为 canvas 或 document.body)，并不会有太多元素需要我们管理。但是本文对应的 [原版教程](https://threejsfundamentals.org/threejs/lessons/threejs-offscreencanvas.html) 中有这一环节(管理层)，那么我们也继续遵循。

   > 为了区别不同的管理对象，我们将在内部给每一个 ElementProxyReceiver 添加一个对应的 id

3. 定义一个类 ElementProxy 用来负责 真实DOM元素与 Web Worker 之间通信。

   > 当浏览器窗口尺寸发生变化时，我们模拟的 "DOM元素本身"也应该会发生尺寸变化，而这个变化的触发并不是由 ElementRroxyReceiver 完成的，而是由 ElementProxy 来完成的。 

4. 定义一些 通信数据结构类型，方便我们在 TypeScript 中使用(类型约束和自动检查)

   > 例如 DOMEvent、MainMessage、EventMessage

   以上的所作所为 都是为了让我们在 index.tsx 和 worker.ts 中使用。

5. index.tsx 中需要处理的事情有：

   1. 初始化 “虚拟DOM”，也就是 ElementProxy 实例
   2. 真实DOM元素的各种事件进行添加，并在事件处理函数中将变化传递出去
   3. 初始化 web worker

6. worker.ts 中需要处理的事情有：

   1. 初始化场景、初始化 OrbitControls
   2. OrbitControls 监听 “虚拟DOM” 发送过来的各种事件
   3. 监听 网页 resize 事件

> 以上每个步骤具体实现时还都挺复杂的，多点耐心继续往下看吧。



### 模拟DOM事件实例结构类型：DOMEvent

使用 typescript 先定义我们需要模拟的 DOM 事件类型结构。

DOMEvent 的主要内容有：

1. type：交互事件类型名称，有 resize、contextmenu、mousedown、mousemove、mouseup、touchstart、touchmove、wheel、keydown
2. preventDefault、stopPropagation：阻止事件冒泡函数
3. `[key: string]: any`：这里虚拟定义了一些属性值，例如鼠标事件对应的一些属性值 clientX/clientY/pageX....，或 触摸事件对应的 pageX/pageY 等。

**具体代码：**

```
export interface DOMEvent {
    type: string
    preventDefault: () => void
    stopPropagation: () => void
    [key: string]: any
}
```

> 我们假定需要模拟出的 DOM 事件，即包含 resize 事件，又包含用户各种交互事件。

> 像真实 DOM 事件实例 event 还有很多其他属性，例如 event.target 等，这些我们就都不模拟了，因为 OrbitControls 根本用不到。



### 负责主线程与worker通信的数据类型：WorkerMessage

WorkerMessage 是由 3 种消息类型组成的：

1. 主场景通知 worker 初始化的消息：MainMessage

2. 主场景通知 worker 被监控的 DOM 元素尺寸发生变化的消息：ResizeMessage

   > 请注意，ResizeMessage 携带的 DOM 元素尺寸等属性值，不光 OrbitControls 需要使用，Three.js 3D 场景中的 镜头和渲染器也需要使用到。

3. 主场景通知 worker 被监控的 DOM 元素用户各种交互事件的消息：EventMessage

每一种消息实体都拥有 type 属性 和 消息本身需要的特有属性。



**具体代码：**

```
import { DOMEvent } from "./dom-event"

export enum MessageType {
    main = 'main',
    event = 'event',
    resize = 'resize'
}

export class MainMessage {
    type: MessageType.main
    canvas: OffscreenCanvas
    canvasId: string
    element: HTMLElement
    constructor(canvas: OffscreenCanvas, canvasId: string, element: HTMLElement) {
        this.type = MessageType.main
        this.canvas = canvas
        this.canvasId = canvasId
        this.element = element
    }
}

export class ResizeMessage {
    type: MessageType.resize
    top: number
    left: number
    width: number
    height: number
    constructor(top: number, left: number, width: number, height: number) {
        this.type = MessageType.resize
        this.top = top
        this.left = left
        this.width = width
        this.height = height
    }
}

export class EventMessage {
    type: MessageType.event
    id: number
    event: DOMEvent
    constructor(id: number, event: DOMEvent) {
        this.type = MessageType.event
        this.id = id
        this.event = event
    }
}

export type WorkerMessage = MainMessage | EventMessage | ResizeMessage
```

> **补充一下相关的 TypeScript 知识：**
>
> 像 WorkerMessage 这种形式，在 TypeScript 中被称为 “可辨识类型”，总类(父类) 其实是由 子类 联合起来后定义(推理)得出的。
>
> 在传统的面对对象编程语言中，一定是先定义父类，再定义子类。但是在 TS 中却可以先定义 “子类”，然后 若干种 “子类” 联合起来可以组成了 “父类”。
>
> TypeScript会将各个 “子类” 中相同的元素进行提取分析，然后将这些相同的属性归类到“父类”中。
>
> 在上面的代码中 MainMessage、EventMessage、ResizeMessage 这 3 个类本身都是相互独立的，但是由于他们都有 type 这个属性，且该属性类型不同，那么在其他地方使用 WorkerMessage 时 TS 可以根据 type 的值来推理出究竟该消息实例是哪一种消息，进而得到 TS 相关代码提示和检查。



### 模拟DOM元素：ElementProxyReceiver

ElementProxyReceiver 的主要内容有：

1. 模拟真实 DOM 元素的属性：width、height、left、right、top、bottom

2. 模拟真实 DOM 元素的方法：get clientWidht()、get clientHeight()、getBoundingClientReact()、focus()

   > 由于 OrbitControls 中使用到了 focus()，所以我们才需要模拟出 focus()，OrbitControls 中并未使用到 blur()，所以我们无需模拟该方法

   > 注意：我们在模拟  focus() 函数时，仅仅声明该函数即可，不需要具体执行任何代码 
   
   > 假设有一天我们需要 bulr() 或者其他函数或事件，例如 click 事件，到时候再继续扩展 ElementProxyReceiver
   
3. 模拟用户交互事件对象本身的 2 个方法：preventDefault、stopPropagation

   > 这 2 个方法原本是 OrbitControls 用来阻止 DOM 元素事件继续冒泡的，我们模拟 2 个方法时也无需做任何处理，因为在 web worker 中实际上根本不存在真实的 DOM 元素，阻止不阻止没有任何意义。

   > 这一点和 focus() 函数一样，尽管实际上不根本需要、调用之后也不起作用，但是我们还要定义他们，以便 OrbitControls 调用时不报错。因为 OrbitControls 并不知道原来他一直在操作一个假的DOM元素、监听假的 DOM 事件。

4. 添加供 web worker 调用的函数：handleEvent()

   > handleEvent() 函数需要处理 2 种事件，以及他们的处理方式：
   >
   > 1. DOM 元素尺寸发生变化的事件(resize)：需要内部处理，但无需再次抛出该事件，因为 OrbitControls 无需处理 resize 事件
   > 2. DOM 元素用户交互事件(鼠标事件、键盘事件... )：不需要内部处理，但需要抛出该事件，因为 OrbitControls 需要处理这些事件

**具体代码：**

```
import { EventDispatcher } from 'three'
import { WorkerMessage, MessageType } from './worker-message'

const noOperate = () => {
    //no operate
}

class ElementProxyReceiver extends EventDispatcher {

    width: number
    height: number
    left: number
    right: number
    top: number
    bottom: number

    constructor() {
        super()

        this.width = 0
        this.height = 0
        this.left = 0
        this.right = 0
        this.top = 0
        this.bottom = 0
    }

    get clientWidth() {
        return this.width
    }

    get clientHeight() {
        return this.height
    }

    getBoundingClientRect() {
        return {
            left: this.left,
            top: this.top,
            width: this.width,
            height: this.height,
            right: this.left + this.width,
            bottom: this.top + this.height,
        };
    }

    focus() {
        //no operate
    }

    handleEvent(message: WorkerMessage) {
        switch (message.type) {
            case MessageType.resize:
                this.left = message.left
                this.top = message.top
                this.width = message.width
                this.height = message.height
                break;
            case MessageType.event:
                message.event.preventDefault = noOperate
                message.event.stopPropagation = noOperate
                this.dispatchEvent(message.event)
                break
            default:
                throw new Error(`ElementProxyReceiver: Can't find ${message.type} handler.`)
                break
        }
    }
}

export default ElementProxyReceiver
```



