# 22 Three.js优化之OffscreenCanvas

我们知道 JS 是单线程，可以通过 WebWorker 将一些复杂计算执行命令从主线程中分离出来。

关于 WebWorker 的使用方法，请参考：

https://developer.mozilla.org/zh-cn/docs/web/api/web_workers_api/using_web_workers



一些比较新的浏览器(例如谷歌浏览器) 还有另外一个和 WebWorker 类似作用，但是只针对画布的 API 接口：OffscreenCanvas。



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



#### OffscreenCanvas的用法

通常情况下 OffscreenCanvas 必须搭配 Web Worker 一起使用。

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



接下来，我们将通过一个实际的例子，来演示一遍。



## 离屏画布渲染示例：HelloOffscreenCanvas