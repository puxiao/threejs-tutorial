# 17 Three.js技巧之按需渲染

**灵魂拷问：什么叫按需渲染？按哪个需？**

答：就是字面意思——需要的时候才渲染，不需要的时候不渲染。

在 基础篇 中，我们所有的示例中，都有以下代码：

```
const render = () =>{
    ...
    renderer.render(scene,camera)
    window.requestAnimationFrame(render)
}
window.requestAnimationFrame(render)
```

也就是意味着，无论任何时候，我们都会在每一帧上进行场景渲染。

假设场景本身就是静止的，没有任何物体变化，此时依然进行不间断的循环渲染，其实是对客户端设备性能、电量的一种浪费。



## 按需渲染示例：RenderingOnDemand

> 在基础篇中，所有示例都是以 HelloXxxx 来命名 React 组件的，但是以后我们不会继续使用这种命名方式，而是会根据实际讲解内容来定义 React 组件名。

#### 只渲染一次的一个示例：

scr/components/rendering-on-demand/index.tsx

```
import { useEffect, useRef } from 'react'
import * as Three from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

import './index.scss'

const RenderingOnDemand = () => {
    const canvasRef = useRef<HTMLCanvasElement>(null)
    useEffect(() => {
        if (canvasRef.current === null) { return }

        const renderer = new Three.WebGLRenderer({ canvas: canvasRef.current })
        const scene = new Three.Scene()
        const camera = new Three.PerspectiveCamera(45, 2, 1, 100)
        camera.position.z = 20
        const light = new Three.DirectionalLight(0xFFFFFF, 1)
        light.position.set(5, 5, 10)
        scene.add(light)

        const colors = ['blue', 'red', 'green']
        const cubes: Three.Mesh[] = []
        colors.forEach((color, index) => {
            const material = new Three.MeshPhongMaterial({ color })
            const geometry = new Three.BoxBufferGeometry(4, 4, 4)
            const mesh = new Three.Mesh(geometry, material)
            mesh.position.x = (index - 1) * 6
            scene.add(mesh)
            cubes.push(mesh)
        })

        const render = () => {
            renderer.render(scene, camera)
        }
        window.requestAnimationFrame(render)

        const controls = new OrbitControls(camera, canvasRef.current)
        controls.update()

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
    }, [canvasRef])
    return (
        <canvas ref={canvasRef} className='full-screen' />
    )
}

export default RenderingOnDemand
```



**请注意上面代码中的这一段：**

```
const render = () => {
    renderer.render(scene, camera)
}
```

以往示例中，我们还会在 render 里添加：`window.requestAnimationFrame(render)` 不停的循环渲染场景。

当我们这次没有添加这行代码后，实际运行，会得到以下结果：

1. 场景只在初始化时，渲染一次
2. 尽管添加有 OrbitControls，但是任何鼠标操作，场景并不会进行更新渲染
3. 尽管添加有浏览器窗口尺寸变化监听，但是浏览器只会针对 canvas 进行变形拉伸，场景并不会进行更新渲染



**我们肯定是需要当 OrbitControl发生改变、浏览器窗口发生改变时，重新渲染场景。**

**如何实现这 2 个需求呢？**



**当 OrbitControls 发生变化时，我们添加对应事件处理函数，调用 render 函数即可。**

```diff
  const controls = new OrbitControls(camera, canvasRef.current)
+ controls.addEventListener('change',render) //添加事件处理函数，触发重新渲染
  controls.update()
```



**当浏览器窗口尺寸发生变化时，我们在 handleResize 函数中调用 render 函数即可。**

```diff
const handleResize = () => {
    if (canvasRef.current === null) { return }
    const width = canvasRef.current.clientWidth
    const height = canvasRef.current.clientHeight
    camera.aspect = width / height
    camera.updateProjectionMatrix()
    renderer.setSize(width, height, false)
+   window.requestAnimationFrame(render) //触发重新渲染
    //注意，这里并不建议直接调用 render()，而是选择执行 window.requestAnimationFrame(render)
}
```

> 就这么简单，就这么 easy 。

实际修改后的代码，调试运行后，就做到了 按需渲染。



**为什么不建议直接调用 render() ？**

答：因为直接 render() 是在当前帧中执行的代码，这样可能会让浏览器 卡顿一下，而选择执行 window.requestAnimationFrame(render) 则明确告知浏览器，在下一帧中执行，确保用户体验流畅一些。



#### 完美？

仔细观察你会发现，当我们拖拽鼠标会触发重新渲染，鼠标拖拽和重新渲染几乎是 同时发生又同时结束的。

停止鼠标拖拽，场景变化(渲染)戛然而止。

你想象一下这个场景：

1. 你手拉着一个绳子，绳子另外一头拴在一个比较重的铁球上面，此时你拉着绳子拽着铁球前进。

2. 假如说你突然停止脚步，那么应该发生什么？

   A、铁球和你同时停止(分秒不差)

   B、尽管你停下了脚步，但是铁球由于惯性，依然会往前移动一点点

   > 哪怕铁球特别沉，多少总会表现出往前一点点的移动的迹象的

我们都相信，B 选项更加符合我们日常的感知预期。

把话题拉回到 鼠标控制轨道 上面来，事实上当我们拖拽鼠标移动停止后，不应该立即停止场景渲染，而是应该让场景继续往后渲染一点点。



#### 如何实现"惯性"？

答：开启 轨道控制器的 enableDamping 属性。

> damping 单词的意思是 阻尼，也就是 惯性

```
controls.enableDamping = true
```

> enableDamping 默认值为 false



但是，设置 enableDamping 为 true 之后，会引发新的问题。

1. enableDamping 设置为 true 之后，需要继续调用 OrbitControls 实例 的 update() 函数，以便相机能够 “靠着惯性继续往前移动轨道”。

2. 但是虽然我们已停止了鼠标拖拽，但是由于惯性，controls 会继续触发 change 事件。

3. 而 change 事件又会调用 render 函数

4. 最终演变成了一个 无限循环 的状况

   > 尽管是无限循环渲染，但是请放心，并不会因此造成客户端崩溃，因为在之前的示例中，我们本身就是不断的无限循环调用 render 函数的。



**如何解决惯性引发的无限循环渲染？**

答：我们可以添加一个 Boolean 类型的参数，用来区分出究竟是 惯性引发的渲染，还是我们主动鼠标拖拽引发的渲染。

请注意，不要用 useState 来创建 这个 Boolean 参数，因为 useState 是异步的，并且每次执行 useState 改变 boo 的值都会引发重新渲染。

 我们采用的是在 组件外部声明 的方式来定义 boo。

具体的做法是：

```diff
+ let boo = false
const RenderingOnDemand = () => {
    useEffect(() =>{
    
      ...
    
      const render = () => {
+           boo = false
+           controls.update()
            renderer.render(scene, camera)
        }
      window.requestAnimationFrame(render)

+     const handleChange = () => {
+        if (boo === false) {
+            boo = true
+            window.requestAnimationFrame(render)
+         }
+      }
        
      const controls = new OrbitControls(camera, canvasRef.current)
-     controls.addEventListener('change', render)
+     controls.addEventListener('change', handleChange)
+     controls.enableDamping = true
      controls.update()
    
      return () =>{
        window.removeEventListener('resize', handleResize)
      }
    },[canvasRef])
}
```



由于存在 “惯性”，所以会在 惯性 期间继续不断调用 controls.update()，直至惯性消失，不再触发 change 时间，此时才会停止调用 controls.update()，从而中断了 无限循环渲染。



**完整的示例代码：**

```
import { useEffect, useRef } from 'react'
import * as Three from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

import './index.scss'

let boo = false

const RenderingOnDemand = () => {
    const canvasRef = useRef<HTMLCanvasElement>(null)

    useEffect(() => {
        if (canvasRef.current === null) { return }

        const renderer = new Three.WebGLRenderer({ canvas: canvasRef.current })
        const scene = new Three.Scene()
        const camera = new Three.PerspectiveCamera(45, 2, 1, 100)
        camera.position.z = 20
        const light = new Three.DirectionalLight(0xFFFFFF, 1)
        light.position.set(5, 5, 10)
        scene.add(light)

        const colors = ['blue', 'red', 'green']
        const cubes: Three.Mesh[] = []
        colors.forEach((color, index) => {
            const material = new Three.MeshPhongMaterial({ color })
            const geometry = new Three.BoxBufferGeometry(4, 4, 4)
            const mesh = new Three.Mesh(geometry, material)
            mesh.position.x = (index - 1) * 6
            scene.add(mesh)
            cubes.push(mesh)
        })

        const render = () => {
            boo = false
            controls.update()
            renderer.render(scene, camera)
        }
        window.requestAnimationFrame(render)

        const handleChange = () => {
            if (boo === false) {
                boo = true
                window.requestAnimationFrame(render)
            }
        }
        
        const controls = new OrbitControls(camera, canvasRef.current)
        controls.addEventListener('change', handleChange)
        controls.enableDamping = true
        controls.update()

        const handleResize = () => {
            if (canvasRef.current === null) { return }
            const width = canvasRef.current.clientWidth
            const height = canvasRef.current.clientHeight
            camera.aspect = width / height
            camera.updateProjectionMatrix()
            renderer.setSize(width, height, false)

            window.requestAnimationFrame(render)
        }
        handleResize()
        window.addEventListener('resize', handleResize)

        return () => {
            window.removeEventListener('resize', handleResize)
        }
    }, [canvasRef])
    return (
        <canvas ref={canvasRef} className='full-screen' />
    )
}

export default RenderingOnDemand
```





## 补充一个 OrbitControls 的知识

我也就是今天才知道，原来 OrbitControls 除了鼠标可改变轨迹之外，还可以通过键盘上的 4 个方向键(上下左右)来更改视图。

但是我在 React 中试验，发现键盘事件并不触发。于是我自己新建一个  React 组件，进一步测试：

```
import { useRef, useEffect } from 'react'

const TestKeydown = () => {
    const canvasRef = useRef<HTMLCanvasElement>(null)

    const handleKeydown = (eve: KeyboardEvent) => {
        console.log(eve)
    }

    useEffect(() => {
        if (canvasRef.current === null) { return }
        canvasRef.current.addEventListener('keydown', handleKeydown, false)
    }, [canvasRef])

    return (
        <canvas ref={canvasRef} style={{ display: 'block', width: '100%', height: '100%' }} />
    )
}

export default TestKeydown
```

实际运行发现，确实根本不会触发键盘事件。

后经过查阅资料，才知道对于 React 来说，更加倾向于使用 React 合成事件，例如 <imput onKeydown={ xxx } /\> 而不是 通过 addEventListener('keydown',xxx)。

上面的代码若想触发键盘事件，需要修改成：

```diff
import { useRef, useEffect } from 'react'

const TestKeydown = () => {
    const canvasRef = useRef<HTMLCanvasElement>(null)

    const handleKeydown = (eve: KeyboardEvent) => {
        console.log(eve)
    }

    useEffect(() => {
        if (canvasRef.current === null) { return }
+        canvasRef.current.focus()
        canvasRef.current.addEventListener('keydown', handleKeydown, false)
    }, [canvasRef])

    return (
+        <canvas ref={canvasRef} tabIndex={0} style={{ display: 'block', width: '100%', height: '100%' }} />
    )
}

export default TestKeydown
```

1. 添加 canvas 自动获取焦点
2. 给 canvas 添加 tabIndex 值，该值是 -1、0、1 都可以，但是必须添加



但是对于 OrbitControls 来说，若想在 React 中也使用键盘事件，则只能依此修改。

不过当 canvas 失去焦点后，则键盘事件就会失效。

**Three.js 官方示例使用的是原生的 html + js，是完全支持键盘事件的。**

**React 则对原生键盘事件支持度不高。**

> 不过我们完全不必介意这件事情，我们继续使用鼠标来控制场景变换角度好了。



下一节，我们学习如何调试Three.js。

