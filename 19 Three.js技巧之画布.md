# 19 Three.js技巧之画布

本文讲解一下 画布 Canvas 的一些实用技巧。

* 创建画布截屏(快照)，并保存图片到本地
* 设置不清除画布内容
* 获取键盘事件
* 设置画布为透明
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

无论采用 canvas.toDataURL() 还是 canvas.toBlob() 都可以，本示例我们采用 toDataURL() 。

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
        const imgurl = canvas.toDataURL('image/jpeg', 0.8)
        const a = document.createElement('a')
        a.href = imgurl
        a.download = 'myimg.jpeg' //我们定义下载图片的文件名
        a.click()


        //采用 toBlob() 方式
        // canvas.toBlob((blob) => {
        //     const imgurl = window.URL.createObjectURL(blob)
        //     const a = document.createElement('a')
        //     a.href = imgurl
        //     a.download = 'myimg.jpeg'
        //     a.click()
        // }, 'image/jpeg', 0.8)
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