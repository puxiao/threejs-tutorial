# 18 Three.js技巧之调试

本文讲解一些 Three.js 的调试技巧，是其他程序员在开发 Three.js 过程中积累的一些找错、调试经验。

> 其中一些调试经验适用于所有的前端项目



## 调试的几点经验

#### 1、使用浏览器调试

个人推荐使用 谷歌浏览器 或 最新版的微软 Edge 浏览器 调试工具。

以谷歌浏览器 Chrome 为例，打开调试的快捷键为：Ctrl + Shift + I

> 个别测试时候，鼠标放在画布上点击右键不显示菜单，或者不显示 “检查”，此时使用快捷键调出调试工具最为合适。



#### 2、关闭缓存

对于开发阶段，为了确保所加载的各种资源是最新的，而不是缓存的，所以推荐关闭缓存。

关闭缓存的方法：打开调试面板(Toggle Tools) > 网络面板(Network) > 勾选 禁用缓存(Disable cache)



#### 3、善用信息打印 Console

可以通过在代码中添加 console.log(xxx)，或者直接在 Console 面板中 添加打印代码，查看当前 JS 环境中的信息变量。



#### 4、添加debugger

我们可以在代码中，添加 debugger ，给代码执行过程中添加断点，好一步步确认整个执行过程。

1. 直接通过 VSCoder 在某行代码左侧，添加断点(小红点)
2. 在代码中添加 `debugger`，当代码执行到此处时即进入调试状态

> 个人推荐使用第 1 种形式添加断点



#### 5、通过URL来获取参数

假如说我们现在需要在场景上创建一个立方体，URL 参数中包含立方体对应的尺寸。

假设 URL 参数为：

```
https://xxx.com/threejs/xxx.index?width=3&height=2&depth=1
```

`width=3&height=2&depth=1` 即我们需要获取并配置给立方体的参数。

> 为了避免参数缺失或错误而导致立方体创建失败，我们给立方体的 宽、高、厚 设置一个默认值 1

**我们可以使用浏览器新增的 URLSearchParams 来解析 URL 参数：**

```
interface URLParams {
    width: number,
    height: number,
    depth: number
}

const getURLParams = (): URLParams => {
    const params = new URLSearchParams(window.location.search.substring(1))
    const widthStr = params.get('width')
    const heightStr = params.get('height')
    const depthStr = params.get('depth')

    let [width, height, depth] = [0, 0, 0]

    if (widthStr) { width = parseInt(widthStr, 10) || 1 }
    if (heightStr) { height = parseInt(heightStr, 10) || 1 }
    if (depthStr) { depth = parseInt(depthStr, 10) || 1 }

    return { width, height, depth }
}

const TestDebugging = () => {
    const urlParams = getURLParams() //获取 URL 参数
    ...
    
    const geometry = new Three.BoxBufferGeometry(urlParams.width, urlParams.height, urlParams.depth)
    
    ...
}
```



#### 6、把一些参数显示在屏幕上

我们还以 刚才的代码为例，在之前的示例中，我们是将组件直接 return 一个 <canvas /\> 对象，

```
return (
    <canvas ref={canvasRef} className='full-screen' />
)
```

我们可以改造一下：

```
return (
    <div className='full-screen'>
        <canvas ref={canvasRef} className='full-screen' />
        <div className='debug'>
            <span>width:{urlParams.width}</span>
            <span>height:{urlParams.height}</span>
            <span>depth:{urlParams.depth}</span>
        </div>
    </div>
）
```

对应的样式：

```
.full-screen, canvas {
    display: block;
    height: inherit;
    width: inherit;
}

.debug {
    position: fixed;
    top: 20px;
    left: 20px;
    width: 80px;
    padding: 20px;
    background-color: rgba($color: #FFFFFF, $alpha: 0.7);
}

.debug span {
    display: block;
}
```

这样当我们调试网页的时候，就可以直接在左上角，看到 立方体的尺寸具体的值。

> 本示例演示的立方体尺寸是固定的，若通过 useState 来定义尺寸，且尺寸会发生修改，那么左上角的展示的参数也可以对应修改成动态可变动的。

> 依次类推，可以延展成其他参数展示



#### 7、把 window.requestAnimationFrame 添加在靠后位置

在之前的一些示例中，渲染场景的函数，可能如下：

```
const render = () => {
    renderer.render(scene, camera)
    window.requestAnimationFrame(render)
}
window.requestAnimationFrame(render)
```

你是否考虑过，假设我们修改成这样：

```
const render = () => {
    window.requestAnimationFrame(render) //代码顺序改变
    renderer.render(scene, camera)
}
window.requestAnimationFrame(render)
```

代码顺序调整后，会有什么问题吗？

答：这里可能会产生一个隐患——由于 window.requestAnimationFrame 代码在前，renderer.render(scene, camera) 代码在后，那么意味着 即使 renderer.render() 执行发生错误，那么代码依然在下一帧中继续执行。

而我们之前的顺序是 renderer.render() 在前，window.requestAnimationFrame 在后，那么万一 renderer.render() 执行时发生错误，此时 浏览器即报错，JS 停止运行，那么后面的 window.requestAnimationFrame 就不会再执行了。

结论：应该尽量把  window.requestAnimationFrame 添加到靠后位置。



#### 8、检查 Three.js 中的单位

在 Three.js 中，单位并没有统一，具体表现在：

1. 镜头的角度使用的是度数、而其他地方涉及角度时单位使用的是 弧度。
2. 默认情况下，对于距离、尺寸的数值 1 表示 1 米，但是也可以通过配置让数值 1 表示为 1 厘米。

因此，在使用 Three.js 时，关于数值单位请格外留意。



#### 9、添加辅助对象、添加镜头轨道控制器

在开发阶段，可以多添加一些辅助对象，例如：坐标辅助对象、灯光辅助对象、镜头辅助对象。

辅助对象可以比较直观得帮助我们去观察，去调试。

同时也要添加镜头轨道控制器，例如 OrbitControls，可以让我们比较方便操控、查看场景。



#### 10、必要时可以将物体材质设置为 MeshBasicMaterial

假设你设置的物体材质是可反光材料，但是渲染时发现并没有渲染出该物体。

这个时候，你可以先将物体材质修改为不反光的 MeshBasicMaterial，这样可以快速排除一些问题。

假设物体材质不反光，此时若依然渲染不出物体，那么就可以肯定问题没有出在灯光上。

反之，则应该去检查灯光的问题。



#### 11、检查镜头的 near 和 far 配置

有些时候场景没有渲染出物体，那么你也要去检查一下镜头的 near 和 far 的值。

比如我们可以暂时性的将镜头的 far 设置为 10000，或者将 near 设置为 0.001，在这种极端配置下，再去看场景是否渲染出物体。

调试结束后，记得将 near 和 far 调整会合理的精度范围。



#### 12、遇到疑惑的地方，查文档，查源码

遇到某些 Three.js 疑惑的地方，例如某属性，某方法，请记得一定先去查阅官方文档，如果没有解决就直接去看 Three.js 的源码。

Three.js 的源码并不是特别复杂，要敢于查看源码来解决疑惑。



## 使用GUI调试场景中的参数

**图形用户界面( Graphical User Interface ) 简称 GUI。**

> 主要目的是用来帮我们快速搭建可视化调试参数面板。



**针对 JS 的 GUI ——dat.gui**

官网地址：https://github.com/dataarts/dat.gui



**针对 React 的 GUI——react-dat-gui**

官网地址：https://github.com/claus/react-dat-gui



**react-dat-gui 的具体用法，请查看我的另外一篇文章：[React中使用GUI.md](https://github.com/puxiao/notes/blob/master/React%E4%B8%AD%E4%BD%BF%E7%94%A8GUI.md)**



本教程所有示例都是基于 react + typescript 的，所以我们选择使用 react-dat-gui

在我们后续的示例中，就会使用到 react-dat-gui。



## 调试GLSL

**图形库着色语言( Graphic Library Shader Language ) 简称 GLSL。**

> 学不动了，学不动了！

GLSL 的相关介绍，可查阅：

WebGL与GLSL：https://webglfundamentals.org/webgl/lessons/zh_cn/webgl-shaders-and-glsl.html

WebGL2与GLSL：https://webgl2fundamentals.org/webgl/lessons/zh_cn/webgl-shaders-and-glsl.html



由于我个人没有学习过 WebGL和 GLSL，所以暂时先不讨论如何调试 GLSL。

> 谷歌浏览器还有一个专门用来调试着色器的插件：Shader Editor
>
> https://chrome.google.com/webstore/detail/shader-editor/ggeaidddejpbakgafapihjbgdlbbbpob?hl=en



关于 Three.js 的调试技巧，就讲到这里。

接下来讲解 Canvas 的一些常用小技巧。