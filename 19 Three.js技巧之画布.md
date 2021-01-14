# 19 Three.js技巧之画布

本文讲解一下 画布 Canvas 的一些实用技巧。

* 创建画布截屏(快照)，并保存图片到本地
* 设置不清除画布内容
* 获取键盘事件
* 设置画布透明度
* 设置画布为背景



## 示例基本代码：HelloCanvas

#### 先制作一个简单的、带动画的Three.js场景

为了演示各个功能，我们先创建一个基础的 Three.js 动画场景：场景上有 3 个不同颜色、不停旋转的立方体。

这个场景在之前多个示例中已经创建过多次，但是这次和之前的略微不同。

**不同点1：**由于本文是讲解 canvas 的，本身和 Three.js 没太大的关联，所以这次我们会创建一个 useCreateScene 的自定义 hook，用来专门创建 3D 场景，这样我们的 index.tsx 代码可以更加简洁。

> 所谓 `自定义 react hook`，本质上就是包含有 hook 的普通函数

**不同点2：**由于讲解过程中需要用到一些按钮，所以我们这次将引入 react-dat-gui 这个组件，来添加一些示例相关按钮。

关于如何使用 react-dat-gui 这个，请参考我写的 [React中使用GUI.md](https://github.com/puxiao/notes/blob/master/React%E4%B8%AD%E4%BD%BF%E7%94%A8GUI.md) 这篇教程。

> 请务必学习一下 react-dat-gui 这个组件，在后续示例中我们会经常使用这个组件来作为调试面板。



#### 具体的代码

我们创建一个专门存放本示例的目录 src/components/hello-canvas/

**use-create-scene.ts：**

```
import { useEffect } from 'react'
import * as Three from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

const useCreateScene = (canvasRef: React.RefObject<HTMLCanvasElement>) => {
    useEffect(() => {
        if (canvasRef.current === null) { return }

        const renderer = new Three.WebGLRenderer({ canvas: canvasRef.current })
        const scene = new Three.Scene()
        scene.background = new Three.Color(0x222222)
        const camera = new Three.PerspectiveCamera(45, 2, 0.1, 100)
        camera.position.set(0, 5, 10)

        const light = new Three.DirectionalLight(0xFFFFFF, 1)
        light.position.set(5, 10, 0)
        scene.add(light)

        const controls = new OrbitControls(camera, canvasRef.current)
        controls.update()

        const colors = ['blue', 'red', 'green']
        const cubes: Three.Mesh[] = []
        colors.forEach((color, index) => {
            const mat = new Three.MeshPhongMaterial({ color })
            const geo = new Three.BoxBufferGeometry(2, 2, 2)
            const mesh = new Three.Mesh(geo, mat)
            mesh.position.x = (index - 1) * 4
            scene.add(mesh)
            cubes.push(mesh)
        })

        const render = (time: number) => {
            time *= 0.001
            cubes.forEach((cube) => {
                cube.rotation.x = cube.rotation.y = time
            })
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
    }, [canvasRef])
}

export default useCreateScene
```



**index.scss：**

```
.full-screen,
canvas {
    display: block;
    height: inherit;
    width: inherit;
}

.dat-gui {
    top: 16px !important;
    font-size: 18px !important;
}
```



**index.tsx：**

```
import { useRef, useState } from 'react'
import DatGUI, { DatButton } from 'react-dat-gui'
import useCreateScene from './use-create-scene'

import './index.scss'
import 'react-dat-gui/dist/index.css'

const HelloCanvas = () => {
    const canvasRef = useRef<HTMLCanvasElement>(null)
    const [date, setDate] = useState<any>({})

    useCreateScene(canvasRef)

    const handleGUIUpdate = (newDate: any) => {
        setDate(newDate)
    }

    const handleSaveClick = () => {
        //编写点击之后的代码
    }

    return (
        <div className='full-screen'>
            <canvas ref={canvasRef} className='full-screen' />
            <DatGUI data={date} onUpdate={handleGUIUpdate} className='dat-gui' >
                <DatButton label='点击保存画布快照' onClick={handleSaveClick} />
            </DatGUI>
        </div>
    )
}

export default HelloCanvas
```

**补充说明：**

1. 所有创建 3D 场景的代码都转移到了 use-create-scene.ts 中，index.tsx 的代码终于看上去非常简洁了。

2. 我们使用了 <DatGUI \> 标签，但是由于我们本身只使用 按钮(<DatButton \>)，并未用到任何其他变量，所以

   ` <DatGUI data={date} onUpdate={handleGUIUpdate} >`这行属性配置只是为了不让 DatGUI 报缺省错误，并无其他作用。

   > DatGUI 的 date、onUpdate 为必填属性



至此，本示例所用到的基础场景代码已搭建好，接下来开始讲解 canvas 的使用技巧。



## 创建画布截屏(快照)，并保存图片到本地

### 先说一下如何创建画布截屏(快照)

对于 HTML5 中的 canvas 来说，创建画布截屏(快照)有 2 种方式：canvas.toDataURL()、canvas.toBolb()

> 所谓截屏和快照，更加准确的说法应该是：获取画布当前图片的内容

#### 第1种：canvas.toDataURL()

canvas.toDataURL() 可以创建一个临时的图片地址，该图片地址可以作为当前页面中的 <image \>标签中的 src 属性值。或者可以创建一个下载链接，点击下载这个图片。



**toDataURL()用法：**

```
canvas.toDataURL(type?: string, quality?: any): string;
```

1. type：图片格式类型，值只能是 "image/png" 或 "image/jpeg"

   > 除了上面 2 个固定值，若你填写其他值则不启作用也不报错，最终会使用 "image/png" 来作为默认值

   > 对于 谷歌浏览器 Chrome ，还额外支持一个类型 “image/webp”

2. quality：jpeg 图片的压缩质量，取值范围 0 - 1，默认值为 0.92

   > quality 的值越大，图片清晰度越高，文件体积越大

   > 如果 quality 的值不在 0-1 范围内，则会使用默认值 0.92



**创建 PNG 图片：**

```
const imgurl = canvas.toDataURL('image/png')
console.log(imgurl)

//输出以下内容
data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAB4AAAAEUCAYAAADK...
```

**创建 JPEG 图片：**

```
const imgurl = canvas.toDataURL('image/jpeg',quality)
console.log(imgurl)

//输出以下内容
data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD/2wBDAAYEBQY...
```

> 请注意上面输出内容中，均包含了 ”base64“，这种格式的图片，是可以自动下载的
>
> 关于图片自动下载保存到本地我们会稍后讲解



#### 第2种：canvas.toBlob()

canvas.toBlob() 可以创建 Blob 对象，该对象包含图片的数据内容。

> 注意：canvas.toDataURL() 是获得一个临时的图片地址，而 canvas.toBlob() 是获得图片数据内容。



**canvas.toBlob()用法：**

```
toBlob(callback: BlobCallback, type?: string, quality?: any): void;
```

1. callback：获得 Blob 对象的回调函数

   > 通常为：canvas.toBlob( (blob: Blob|null) => {} )

2. type：图片的格式类型，默认值为 "image/png"

3. quality：若图片格式类型为 "image/jpeg"，quality 表示 JPEG 的压缩质量

   > type 的取值和默认值、quality 的取值范围和默认值 与 canvas.toDataURL() 完全相同



**使用示例：**

```
canvas.toBlob((blob) => {
    console.log(blob)
})

或

canvas.toBlob((blob) => {
    console.log(blob)
}, 'image/jpeg', 0.8)
```

> 注意：不同于 canvas.toDataURL()，canvas.toBlob() 这个函数是没有返回值的

> 另外，假设使用 jpeg  压缩质量为 0.8，文件体积有可能只有 png 格式的 1/3 。



#### 补充说明：

**关于分辨率：**

按照 MDN 文档，无论哪种方式保存的图片分辨率都是 96，但是我在 PC 机上试验，将下载的图片保存到本地，并在 PhotoShop 软件中查看，发现图片分辨率依然是 72。

我怀疑 保存图片的分辨率其实是和 当前系统一致的。假设在手机上，有可能图片分辨率就是 96 了。

> 稍后我会在手机上验证一下 图片分辨率 这个问题。



最后，建议你去 MDN 上看 canvas 保存图片 的相关讲解作为本小节的补充。

https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Pixel_manipulation_with_canvas



### 再说一下如何自动将图片下载到本地

**图片自动下载到本地的思路：在JS中创建一个 a 链接，并且模拟出 a 点击事件**



**使用toDataURL()函数：**

```
const canvas = canvasRef.current
const imgurl = canvas.toDataURL('image/jpeg', 0.8)

const a = document.createElement('a')
a.href = imgurl
a.download = 'myimg.jpeg'
a.click()
```

> 注意：我原本以为还需要将 a 标签插入到网页 body 中才可以实现自动下载，但是经过试验发现根本不需要这样，以下为我原本写的代码：
>
> ```
> const a = document.createElement('a')
> document.body.appendChild(a) //根本无需此行代码
> a.style.display = 'none' //由于不需要添加到 body 中，因此也无需此行代码
> a.href = imgurl
> a.download = 'myimg.jpeg'
> a.click()
> document.body.removeChild(a) //根本无需此行代码
> ```



**使用toBlob()函数：**

```
const canvas = canvasRef.current
canvas.toBlob((blob) => {
    const imgurl = window.URL.createObjectURL(blob)
    const a = document.createElement('a')
    a.href = imgurl
    a.download = 'myimg.jpeg'
    a.click()
}, 'image/jpeg', 0.8)
```



**小总结：**

1. 若使用 canvas.toDataURL()，则可以直接将得到的图片临时地址 赋值给 a.href
2. 若使用 canvas.toBlob()，则需要通过 window.URL.createObjectURL() 这个函数将 Blob 数据转化得到对应的地址，然后再赋值给 a.href



好了，关于如何获取画布图片数据、如何保持图片到本地讲解完毕，来实践吧。

无论采用 canvas.toDataURL() 还是 canvas.toBlob() 都可以，本示例我们采用 toBlob() 。

我们将 index.stx 的代码修改如下：

```
import { useRef, useState } from 'react'
import DatGUI, { DatButton } from 'react-dat-gui'
import useCreateScene from './use-create-scene'

import './index.scss'
import 'react-dat-gui/dist/index.css'

const HelloCanvas = () => {
    const canvasRef = useRef<HTMLCanvasElement>(null)
    const [date, setDate] = useState<any>({})

    useCreateScene(canvasRef)

    const handleGUIUpdate = (newDate: any) => {
        setDate(newDate)
    }

    const handleSaveClick = () => {
        if (canvasRef.current === null) { return }
        const canvas = canvasRef.current

        //采用 toDataURL() 方式
        // const imgurl = canvas.toDataURL('image/jpeg', 0.8)
        // const a = document.createElement('a')
        // a.href = imgurl
        // a.download = 'myimg.jpeg' //我们定义下载图片的文件名
        // a.click()

        //采用 toBlob() 方式
        canvas.toBlob((blob) => {
            const imgurl = window.URL.createObjectURL(blob)
            const a = document.createElement('a')
            a.href = imgurl
            a.download = 'myimg.jpeg'
            a.click()
        }, 'image/jpeg', 0.8)
    }

    return (
        <div className='full-screen'>
            <canvas ref={canvasRef} className='full-screen' />
            <DatGUI data={date} onUpdate={handleGUIUpdate} className='dat-gui' >
                <DatButton label='点击保存画布快照' onClick={handleSaveClick} />
            </DatGUI>
        </div>
    )
}

export default HelloCanvas
```

实际运行，点击右上角的 按钮，就会给画布创建图片快照，并且自动下载到本地。

然后你可以查看刚刚下载到本地的 myimg.jpeg 这个文件，打开它——你会发现？？？

**怎么图片啥内容都没有？纯色的？3 个立方体呢？**

**what ？why ？**

呵，马上讲解为什么。



**问题出在了哪里？**

首先我们容易想到，在 use-create-scene.ts 的 render() 函数中，不停的运行着每一帧都进行画布重新渲染的代码，莫非是我们截图那一瞬间刚好画布还未渲染完成？

好，我们先把那行代码删除掉，看是否就可以截图显示有内容了。

```diff
const render = (time: number) => {
    time *= 0.001
    cubes.forEach((cube) => {
        cube.rotation.x = cube.rotation.y = time
    })
    renderer.render(scene, camera)
-   window.requestAnimationFrame(render)
}
window.requestAnimationFrame(render)
```

再次运行，3 个立方体是静止状态，此时点击按钮保存截图。

查看该图，竟然依然是空白，没有内容的。

看来问题并不出在上面一行代码中，我们恢复刚才删除的 `window.requestAnimationFrame(render)`，再去想其他原因。



**真实的原因是：**

1. 我们所谓的针对画布截屏 创建快照，实际上是获取 canvas 中的数据
2. 但这个数据并不是针对 DOM 中已显示的 canvas，而是针对 canvas 对象中缓冲区的数据
3. 关键在于当 canvas 渲染完成后(DOM中已显示出内容)，默认会清空 缓冲区中的数据
4. 所以，这就是我们为什么去 “获取 canvas 图像数据时得到是空白内容” 的原因



**canvas从计算到显示的过程：**

1. canvas 根据相应的 JS 规则，开始创建、计算画布内容数据
2. canvas 将计算得到的画布内容数据填充到 canvas 缓冲区
3. 当 canvas 画布内容计算完成，此时 canvas 缓冲区已有完整的画布内容数据后，将画布内容显示到 DOM 中



**再说一遍：**

我们之前的示例中，渲染并显示 canvas 内容的函数 render 和 创建画布快照 的函数是相互独立的，这就造成了当我们去获取 canvas 缓冲区数据时，canvas 已经将画布内容显示到 DOM 中并且清空了缓冲区。



**解决办法：**

解决办法就是当我们要创建画布快照，获取 canvas 缓冲区内容之前，在同一个函数体内，额外调用一次 render 函数，确保此时 canvas 缓冲区内是有内容的。



**实际代码：**

第1：由于我们示例代码中，render 函数本身位于 useCreateScene 函数内部，因此我们需要创造一个 renderRef  的钩子(hook)，将 renderRef 对外 return 出去，以便 index.stx 中可以获取 render 函数的引用。

```
type RenderType = () => void
...

const renderRef = useRef<RenderType | null>(null)
...

renderRef.current = render
...

return renderRef
```



第2：这样做引申出另外一个问题，就是我们的 render 函数其实是有参数 time 的：

```
const render = (time: number) => {
    time *= 0.001
    cubes.forEach((cube) => {
        cube.rotation.x = cube.rotation.y = time
    })
    renderer.render(scene, camera)
    window.requestAnimationFrame(render)
}
window.requestAnimationFrame(render)
```

而我们希望 index.tsx 中调用 render() 是不传参数 time 的。因为 index.tsx 中根本不存在 time 这个变量，所以我们需要对 渲染 进行适当的改造。

我们将原本的 渲染函数 render() 拆分成 2 个函数：

1. 单纯负责渲染的 render 函数
2. 负责修改物体属性从而产生动画的 animate 函数

```
const render = () => {
    renderer.render(scene, camera)
}
renderRef.current = render

const animate = (time: number) => {
    time *= 0.001
    cubes.forEach((cube) => {
        cube.rotation.x = cube.rotation.y = time
    })
    render() //这样 render() 就是一个不需要参数的函数
    window.requestAnimationFrame(animate)
}
window.requestAnimationFrame(animate)
```

经过这样改造后，完整的 use-create-screen.ts 代码如下：

```
import { useEffect, useRef } from 'react'
import * as Three from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

type RenderType = () => void

const useCreateScene = (canvasRef: React.RefObject<HTMLCanvasElement>) => {
    const renderRef = useRef<RenderType | null>(null)

    useEffect(() => {
        if (canvasRef.current === null) { return }

        const renderer = new Three.WebGLRenderer({ canvas: canvasRef.current })
        const scene = new Three.Scene()
        scene.background = new Three.Color(0x222222)
        const camera = new Three.PerspectiveCamera(45, 2, 0.1, 100)
        camera.position.set(0, 5, 10)

        const light = new Three.DirectionalLight(0xFFFFFF, 1)
        light.position.set(5, 10, 0)
        scene.add(light)

        const controls = new OrbitControls(camera, canvasRef.current)
        controls.update()

        const colors = ['blue', 'red', 'green']
        const cubes: Three.Mesh[] = []
        colors.forEach((color, index) => {
            const mat = new Three.MeshPhongMaterial({ color })
            const geo = new Three.BoxBufferGeometry(2, 2, 2)
            const mesh = new Three.Mesh(geo, mat)
            mesh.position.x = (index - 1) * 4
            scene.add(mesh)
            cubes.push(mesh)
        })

        const render = () => {
            renderer.render(scene, camera)
        }
        renderRef.current = render

        const animate = (time: number) => {
            time *= 0.001
            cubes.forEach((cube) => {
                cube.rotation.x = cube.rotation.y = time
            })
            render() //这样 render() 就是一个不需要参数的函数
            window.requestAnimationFrame(animate)
        }
        window.requestAnimationFrame(animate)

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

    return renderRef
}

export default useCreateScene
```

index.tsx 完整代码如下：

```
import { useRef, useState } from 'react'
import DatGUI, { DatButton } from 'react-dat-gui'
import useCreateScene from './use-create-scene'

import './index.scss'
import 'react-dat-gui/dist/index.css'

const HelloCanvas = () => {
    const canvasRef = useRef<HTMLCanvasElement>(null)
    const [date, setDate] = useState<any>({})

    const renderRef =  useCreateScene(canvasRef) //获取自定义 hook 返回的 renderRef

    const handleGUIUpdate = (newDate: any) => {
        setDate(newDate)
    }

    const handleSaveClick = () => {
        if (canvasRef.current === null || renderRef.current === null) { return }
        const canvas = canvasRef.current

        renderRef.current() //此时调用 render()，进行一次渲染，确保 canvas 缓冲区有数据

        //采用 toDataURL() 方式
        // const imgurl = canvas.toDataURL('image/jpeg', 0.8)
        // const a = document.createElement('a')
        // a.href = imgurl
        // a.download = 'myimg.jpeg' //我们定义下载图片的文件名
        // a.click()

        //采用 toBlob() 方式
        canvas.toBlob((blob) => {
            const imgurl = window.URL.createObjectURL(blob)
            const a = document.createElement('a')
            a.href = imgurl
            a.download = 'myimg.jpeg'
            a.click()
        }, 'image/jpeg', 0.8)
    }

    return (
        <div className='full-screen'>
            <canvas ref={canvasRef} className='full-screen' />
            <DatGUI data={date} onUpdate={handleGUIUpdate} className='dat-gui' >
                <DatButton label='点击保存画布快照' onClick={handleSaveClick} />
            </DatGUI>
        </div>
    )
}

export default HelloCanvas
```

调试运行，这次保存的画布快照图片，就不会再是空白，而是有具体内容了。



## 设置不清除画布内容

上面刚讲到 HTML5 中的 Canvas 每次渲染都存在一个 数据缓冲区的概念，而 Three.js 的渲染器 WebGLRenderer 也同样存在 数据缓冲区 这个概念。

WebGLRenderer 缓冲区内为每次渲染场景得到的画面数据，默认情况下每一次渲染都会清空(释放)上一次的渲染画面数据。

Canvas 数据缓冲区每次清空 这个我们没有办法修改，只能调用渲染函数，重新渲染一次。

但是 WebGLRenderer 的数据缓冲区却是可以通过设置让默认不清除的。

> 所谓不清除上一次数据缓冲区的内容，本质上就是保留上一次渲染画面内容

> 所谓不清除画布内容，本质上是让渲染器不清除之前的渲染内容



**设置 WebGLRenderer 保留数据缓冲区中的历史数据：**

我们只需要将 WebGLRender 的配置修改如下：

```
const renderer = new Three.WebGLRenderer({
    canvas: canvasRef.current,
    preserveDrawingBuffer: true,
    alpha: true
})
renderer.autoClearColor = false
```

经过以上的修改之后，每次渲染都会继续保留之前渲染历史画面。

调试运行代码，你就能感受到和之前渲染的不一样效果了。



**但是，存在一个问题：当浏览器窗口尺寸改变后，由于执行了 renderer.setSize()，则此时 渲染器中过往的渲染内容将会被清空。**

> 渲染器中的渲染历史内容被清空后，画面就好像第一次刚开始那样，重新开始渲染。

> 补充说明：当用户在手机上浏览时，手机从竖屏变为横屏时，也会触发重新绘制。



**真正的解决方案：离屏渲染**

例如使用 WebGLRenderTarget，具体请回顾我们之前讲解的内容：[15 Three.js基础之离屏渲染.md](https://github.com/puxiao/threejs-tutorial/blob/main/15%20Three.js%E5%9F%BA%E7%A1%80%E4%B9%8B%E7%A6%BB%E5%B1%8F%E6%B8%B2%E6%9F%93.md)



### 补充一个示例：PreserveDrawingBuffer

src/components/hello-canvas/preserve-drawing-buffer.tsx

#### 示例目标：

1. 创建一个由 6 个立方体，做相互缠绕运动的一个物体
2. 创建一个正交镜头 OrthographicCamera
3. 给 canvas 添加鼠标滑动监听、以及 手指滑动监听
4. 当 鼠标或手指滑动画布时，更新 物体 在镜头中的位置
5. 设置渲染器不清除历史画面

最终呈现出的效果：类似一个 画笔在画板上 画画 的效果。



**PreserveDrawingBuffer 代码：**

```
import { useEffect, useRef } from 'react'
import * as Three from 'three'

import './index.scss'

const state = { x: 0, y: 0, z: 0 }

const PreserveDrawingBuffer = () => {
    const canvasRef = useRef<HTMLCanvasElement | null>(null)

    useEffect(() => {
        if (canvasRef.current === null) { return }

        const renderer = new Three.WebGLRenderer({
            canvas: canvasRef.current,
            preserveDrawingBuffer: true,
            alpha: true
        })
        renderer.autoClearColor = false

        const camera = new Three.OrthographicCamera(-2, 2, 1, -1, -1, 1)

        const scene = new Three.Scene()
        scene.background = new Three.Color(0xFFFFFF)

        const light = new Three.DirectionalLight(0xFFFFFF, 1)
        light.position.set(-1, 2, 3)
        scene.add(light)

        const geometry = new Three.BoxBufferGeometry(1, 1, 1)
        const base = new Three.Object3D()
        scene.add(base)
        base.scale.set(0.1, 0.1, 0.1)

        const colors = ['#F00', '#FF0', '#0F0', '#0FF', '#00F', '#F0F']
        const numArr = [-2, 2] //同一坐标轴上，对称 2 个立方体的坐标
        colors.forEach((color, index) => {
            const material = new Three.MeshPhongMaterial({ color })
            const cube = new Three.Mesh(geometry, material)

            const col = Math.floor(index / numArr.length)
            const row = index % numArr.length
            let result = [0, 0, 0]
            result[col] = numArr[row]

            cube.position.set(result[0], result[1], result[2])

            base.add(cube)
        })

        const temp = new Three.Vector3()
        const updatePosition = (x: number, y: number) => {
            if (canvasRef.current === null) { return }

            // const rect = canvasRef.current.getBoundingClientRect()
            // const newX = (x - rect.left) * canvasRef.current.width / rect.width
            // const newY = (y - rect.top) * canvasRef.current.height / rect.height

            // const resX = newX / canvasRef.current.width * 2 - 1
            // const resY = newY / canvasRef.current.height * -2 + 1

            const resX = x / canvasRef.current.width * 2 - 1
            const resY = y / canvasRef.current.height * -2 + 1

            temp.set(resX, resY, 0).unproject(camera)
            state.x = temp.x
            state.y = temp.y
        }

        const handleMouseMove = (eve: MouseEvent) => {
            updatePosition(eve.clientX, eve.clientY)
        }
        const handleTouchMove = (eve: TouchEvent) => {
            eve.preventDefault()
            const touche = eve.touches[0]
            updatePosition(touche.clientX, touche.clientY)
        }

        canvasRef.current.addEventListener('mousemove', handleMouseMove)
        canvasRef.current.addEventListener('touchmove', handleTouchMove, { passive: false })

        const render = (time: number) => {
            time = time * 0.001
            base.position.set(state.x, state.y, state.z)
            base.rotation.x = time
            base.rotation.y = time * 1.11
            renderer.render(scene, camera)
            window.requestAnimationFrame(render)
        }
        window.requestAnimationFrame(render)

        const handleResize = () => {
            if (canvasRef.current === null) { return }
            const width = canvasRef.current.clientWidth
            const height = canvasRef.current.clientHeight
            camera.right = width / height
            camera.left = - camera.right
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

export default PreserveDrawingBuffer
```

**补充说明：**

上面这段代码略微复杂、陌生。因为这段代码中有几个地方是我们之前示例中从来未接触过的：

1. 使用的是正交镜头，而不是透视镜头

2. 当窗口尺寸发生变化时，更新正交镜头

   > 更新方式和我们之前习惯使用的 透视镜头(PerspectiveCamera) 大不同

3. 监听鼠标滑动、手指滑动事件

4. 更新 物体 在正交镜头中的 “投影位置”



**额外补充：**

上述代码中，6 个立方体 他们分别是：

1. 在 x 轴对称的 2 个立方体

   > 在示例中，对应的坐标分别为 (-2,0,0)、(2,0,0)

2. 在 y 轴对称的 2 个立方体

   > 在示例中，对应的坐标分别为 (0,-2,0)、(0,2,0)

3. 在 z 轴堆成的 2 个立方体

   > 在示例中，对应的坐标分别为 (0,0,-2)、(0,0,2)

这 6 个立方体他们依次对应的坐标，没有提前写死，而是通过一段特殊的 forEach 循环来计算得出的。

可以阅读下面这段通用的代码，帮助你理解 整个 forEach 循环是如何得到每个立方体坐标的。

```
//遍历出 目标长度为N，特殊值为 [xx, xx, ...] 的 多维数组
const numArr = [-2, 2] //定义特殊位置上出现的数字
const arrLength = 3 //定义目标数组长度
const total = arrLength * numArr.length //根据目标数组长度以及特殊数字的个数，计算得出目标数组的总个数
for (let i = 0; i < total; i++) {
    const col = Math.floor(i / numArr.length) //计算出特殊位置的索引
    const row = i % numArr.length //计算出特殊位置上数字值对应的索引
    let result = new Array(arrLength) //得到一个 长度为 arrLenght 的数组
    result.fill(0) //将数组每一项填充为 0
    result[col] = numArr[row] //修改特殊位置上的值
    console.log(result)
}
```



如果你对 useCreateScene 这个示例不太理解也没有关系，因为毕竟这个示例中出现了一些我们之前示例中从未用到的一些类，你可以先跳过这个示例，继续后面的学习。

> 随着日后对于 正交镜头 的多次使用，终归会熟练并理解的。





## 获取键盘事件

#### 让 Canvas 获取键盘事件

必须同时满足以下  2 个条件后，canvas 才可以获得键盘事件。

1. canvas 当前获得焦点

2. `<canvas \>` 标签中必须添加 tabIndex属性，

   > 属性值是 -1、0、1 都无所谓，建议设置为 0



补充说明：当 canvas 获得当前焦点后，会在四周出现一个蓝色边框，可以通过定义 css 样式来取消这个样式。

```
canvas:focus {
    outline: none;
}
```



**简单示例：**

```
import { useEffect, useRef } from 'react'

import './index.scss'

const CanvasKeyboard = () => {
    const canvasRef = useRef<HTMLCanvasElement | null>(null)
    useEffect(() => {
        if (canvasRef.current === null) { return }

        canvasRef.current.focus() //自动获取焦点

        const handleKeydown = (event: KeyboardEvent) => {
            console.log(event)
        }

        canvasRef.current.addEventListener('keydown', handleKeydown)

        return () => {
            if (canvasRef.current === null) { return }
            canvasRef.current.removeEventListener('keydown', handleKeydown)
        }
    }, [canvasRef])
    return (
        <canvas ref={canvasRef} className='full-screen' tabIndex={0} />
    )
}

export default CanvasKeyboard
```



#### 让 OrbitControl 获取键盘事件

默认 OrbitControl 对象就包含键盘方向键侦听。

键盘上的 上下左右 方向键 均可操控改变 镜头轨道视图。

但是我们之前的代码中，经常是这样写的：

```
const controls = new OrbitControls(camera, canvasRef.current)
```

这样存在的问题是，当 canvas 失去焦点后，就无法再获得键盘事件。

最简单的解决办法就是将代码修改为：

```
const controls = new OrbitControls(camera, document.body)
```

这样键盘事件就不容易丢失。





## 设置画布透明度

设置画布透明度，你可能会疑惑，这有什么好讲的，直接通过 css 给 canvas 添加透明度样式即可：

```
canvas {
    opacity: 0.4;
}
```

这样做肯定没有问题，但是这里说的 “设置画布透明度” 实际上是指 给不同物体设置透明度。

例如我们之前示例中的立方体，那么所有的示例中立方体都不是半透明的。



**给材质设置透明度：**

1. 需要给材质设置透明度

   ```
   const mat = new Three.MeshPhongMaterial({
       color,
       opacity: 0.4
   })
   ```

   

2. 渲染器需要开启透明度渲染

   ```
   const renderer = new Three.WebGLRenderer({ 
       canvas: canvasRef.current,
       alpha:true,
       premultipliedAlpha:false
   })
   ```

   > alpha：canvas 是否包含透明度，默认为 false
   >
   > premultipliedAlpha：renderer 是否假设颜色有 premultiplied alpha (预乘alpha)，默认为 true

   

**针对 预乘Alpha 的补充说明：**

premultiplied alpha：颜色值 预乘 alpha

这是传统 3D 绘制中的一个重要概念，你可以简单理解成如下：

假设我们要表示一个 透明度为 60% 的纯红色，采用 RGBA 的方式为 (255,0,0,0.6)，通过  预乘 alpha，我们可以得到透明度为 60% 的纯红色如果放置在纯白色底上，实际上最终呈现出来的颜色和 RGB ( 255,102,102) 是完全相同的。

假设一个颜色使用 RGBA 来表示，透明度为 A、rgb 颜色为 C、纯白色(255,255,255)为 F，那么把 RGBA 转化为 RGB 的公式为：

C*A  + ( 1-A ) * F

也就是说 rgba(255,0,0,0.6) 转化为对应的 rgb 过程为：

(255,0,0)*0.6 + (1-0.6) * (255,255,255) = (255,  255 x 0.4, 255 x 0.4) = (255,102,102)



**暂时看不懂没有关系，只需记住若想让渲染器将物体渲染出半透明，除了物体本身材质配置透明度以外，还需要将渲染器中的 alpha 设置为 true 、premultipliedAlpha 设置为 false**

 

## 设置画布为背景

将 canvas 设置为网页背景，事实上也很简单，对应的样式：

```
canvas {
  position: fixed;
  top: 0;
  left: 0;
  z-index: -1;
}
```

上述 CSS  样式就让 canvas 位置固定，且层级最低，这样就成为当前网页背景了。

但是实际项目中，更加建议将 canvas 包含在一个 iframe 中后，再作为 网页的背景。

这样做有几个理由：

1. 使用 iframe 后，可以将  canvas、Three.js 的相关代码独立出来
2. 可以多个页面都引用这个  iframe 



**iframe的相关示例：**

```
<iframe id='background' src='xxx.html' >
<div>
    Hello Three.js
</div>
```

```
#background {
    position: fixed;
    width:100%;
    height:100%;
    left:0;
    top:0;
    z-index:-1;
    border:none;
    pointer-events:none;
}
```

上述 css 样式中：

1. position: fixed;  可以让 iframe 位置固定
2. z-index: -1; 可以让 iframe 层级最低
3. border: none; 可以让 iframe 不显示边框
4. pointer-events: none; 让 iframe 永远不会成为鼠标事件的  target，意味着让 iframe 不接受鼠标交互事件



关于更多 canvas 的相关用法，建议阅读 MDN 上关于 canvas 的相关文档：

https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API



至此，关于 Three.js 的一些常用技巧讲解完毕。

接下来开始讲解 Three.js 的一些性能优化。