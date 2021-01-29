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

https://github.com/gfxfundamentals/threejsfundamentals/blob/master/threejs/resources/threejs/r122/examples/jsm/controls/OrbitControls.js



我把源码中和 Dom 元素相关的一些关键代码摘录出来：

```
//设置 scope = this
var scope = this; 

var OrbitControls = function ( object, domElement ) {
  //下面这行代码相当于 scope.domElement = domElement
  this.domElement = domElement;
}

//添加鼠标右键(上下文)菜单事件侦听
scope.domElement.addEventListener( 'contextmenu', onContextMenu, false );

//添加触控笔摁下事件侦听
scope.domElement.addEventListener( 'pointerdown', onPointerDown, false );

//添加鼠标滚轴事件侦听
scope.domElement.addEventListener( 'wheel', onMouseWheel, false );

//添加触摸开始、结束、移动事件侦听
scope.domElement.addEventListener( 'touchstart', onTouchStart, false );
scope.domElement.addEventListener( 'touchend', onTouchEnd, false );
scope.domElement.addEventListener( 'touchmove', onTouchMove, false );

//添加键盘事件侦听
scope.domElement.addEventListener( 'keydown', onKeyDown, false );
	
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



**重点来了，请听好：**

1. 既然 OrbitControls 需要 DOM 元素的目的是为了获取并添加各种事件侦听
2. 而 Worker 中无法获取 DOM 元素
3. 那么有没有可能我们虚拟出来一个对象，让该对象拥有和 DOM 元素相同的事件 API
4. 然后我们让 OrbitControls 去侦听这个虚拟对象所发出的各种事件
5. 也就是说让这个虚拟对象代替真实的 DOM 元素，以此来解决我们目前的困境



**思路有了，那该具体怎么做呢？**

首先我们要明白一件事，原生 DOM 对应的事件其实是非常复杂的、拥有众多庞杂的属性。

而 OrbitControls 并不是每一个事件的每一个属性都使用到了，也就是说我们所谓的 “模拟”不需要 100% 一模一样，我们只需要提供 OrbitControls 需要的即可。

举例说明：

1. 对于鼠标滚轴滚动来说，OrbitControls 只需要使用到该事件的 deltaY 值

2. 对于键盘事件来说，OrbitControls 只需要使用到该事件的 ctrlKey、metaKey、shiftKey、keyCode 值

   > meta 键？
   >
   > 这个 meta 键在 Windows 键盘上相当于 windows 键、在苹果键盘上是一个四瓣的小花。

   > 在 OrbitControls 内部并未使用到 altKey 值

3. 对于鼠标事件，OrbitControls 需要使用到该事件的 ctrlKey、metaKey、shiftKey、button、clientX、clientY、pageX、pageY 值

4. 等等...

我们需要让 “虚拟对象” 可以抛出事件，并且事件参数拥有上面那些属性值。



**所谓的 “虚拟对象” 更加贴切的说法应该是：**

1. “监听 DOM 元素的各种事件，然后自己抛出该事件供 Worker 监听使用”

2. “虚拟对象” 实际上就是一个 “事件代理对象”

   > 有一些教程中，会将该对象称呼为 “事件派发器”



**除了模拟出 DOM 元素的各种事件之外，还需要模拟出其他一些内容：**

> 你可以先想一下常见 DOM 元素还有哪些常用的属性或方法？

1. event.preventDefault、event.stopPropagation：事件冒泡相关设置
2. focus()：设置 DOM 元素为当前焦点
3. clientWidth、clientHeight：对象原始宽高
4. getBoundingClientRect()：获取元素显示大小、相对浏览器视窗的位置



思路理清之后，剩下的就容易了。



有 2 个思路途径：

1. 使用 Three.js 提供的 EventDispatcher

   ```
   import { EventDispatcher } from 'three'
   ```

2. 使用 原生 JS 提供的 EventTarget 



额~ 因为最近接了一个设计的活，本文先暂停更新几天。

