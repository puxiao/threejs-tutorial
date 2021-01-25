# 20 Three.js优化之合并对象

前面学习了 Three.js 入门、基础、技巧，今天开始学习 Three.js 的性能优化。

关于性能优化有很多方式，最基础也是最常见的方式就是——合并几何对象。

> 在 “Three.js 基础之图元” 那篇文章中，我们将几何体称呼为 图元，现在我们修改一下这个称呼，本文以后，绝大多数情况下我们都使用 “几何体” 来代替 “图元”。



既然谈到性能优化，就不能使用简单的示例，要不然根本无法体现出 优化前和优化后 的区别。

激动人心的时刻到了。

#### 本文我们的目标：制作并优化一个显示地球人口人数分布的可视化3D地球

你可以先访问以下网址，先感受一下我们本文要模仿的效果：

https://globe.chromeexperiments.com/

> 补充说明：这个网站，是谷歌浏览器为了向大众演示 WebGL 技术而制作的一个演示网页。

我相信你第一次看到这种基于浏览器的 3D 地球数据展示，一定会被震撼到的。

接下来我们就要逐步分析，找出实现方式。

**我们先考虑怎么把这个场景实现出来，然后再考虑优化的事。**



#### 核心模块分析

我们要先搞明白这个 3D 数字地球的核心模块。

1. 3D 地球

   > 就是一个球体，添加一个地球纹理图片
   >
   > > 图片为一个矩形地球展开图，本示例使用的地球纹理图片资源：  
   > > https://threejsfundamentals.org/threejs/resources/images/world.jpg

2. 表示人口多少的柱状物

   > 某个地区人口多则柱状物就比较高，反之人口少则柱状物比较低

3. 鼠标可交互

   > 这个直接使用 OrbitControls 就可以
   > 只不过本示例只允许左右、上下拖拽查看，但不允许修改镜头轨道的远近距离

上面的核心模块 1 、3 都很容易实现，重点我们进一步拆解一下 “表示人口多少的柱状物”。



#### 如何实现“表示人口多少的柱状物”？

我们通过以下 3 个灵魂追问，来梳理思路。

**第1问：人口数据从哪里来？**

原网页提供 3 个年份的人口数量统计，分别是 1990、1995、2000年。

美国国家航天局(NASA)提供的统计结果介绍页：
https://sedac.ciesin.columbia.edu/data/set/gpw-v4-admin-unit-center-points-population-estimates-rev11/data-download

请注意，该页面还提供最近年份的统计结果，但是下载时候提示需要注册。

我们选择不使用最新的 2020 年数据，而是使用 2010 年的结果。

为了方便你获得到 2010 年男性人口统计结果，你可以直接点击下面这个地址，直接下载：

https://threejsfundamentals.org/threejs/resources/data/gpw/gpw_v4_basic_demographic_characteristics_rev10_a000_014mt_2010_cntm_1_deg.asc

> 反正我们本文的重点是模仿效果，至于数据时效性不必纠结

> 该人口统计数据文件格式为 .asc，至于如何解析该文件，我们会稍后讲解

> 为啥是男性人口统计？女性人口呢？为什么不是全部人口统计呢？  
> 因为在下一篇文章中，就会有女性人口统计，然后做出同一个地区 男女人口数量比较 的动画  
> 为了简化，本文下面文字中，将忽略 “男性人口数量” 这个概念，统一称呼为 “人口数据”



**第2问：人口数据和地区的对应关系，如何表现在地球上？**

我们把刚才下载得到的人口统计数据文件，重命名为 gpw_v4_014mt_2010.asc，然后将该文件移动到：src/assets/data/ 目录中。

点击该文件，用记事本查看该文件内容，你会发现里面大致为以下内容：

```
ncols         360
nrows         145
xllcorner     -180
yllcorner     -60
cellsize      0.99999999999994
NODATA_value  -9999
-9999 -9999 -9999 -9999 -9999 -999...
...
...
```

这里面的数据内容为：**矩形地球地图上，不同点(经纬度)对应的数值(人口数量)**。

> 补充：这里面的人口数量值并不是具体人口数量(比如 45932551 个人)，而是具有一定比例单位的值(例如 458.6)
>
> 具体单位值对应的人口我还不清楚，或许是 万，也或许是百万，不过不影响我们本示例，你只需把他当成数字即可

我们需要将数据与地图纹理图片进行点对点的位置匹配。



> .asc 后缀的文件有特别多种用途和场景，我们这里提到的 .asc 文件是指：以 `PGP (Pretty Good Privacy) ASCII Armored File`形式存在的栅格化结构的数据文件。

> **.asc 栅格化结构的数据文件说明：**
>
> | 关键字                       | 对应含义                 |
> | ---------------------------- | ------------------------ |
> | ncols(number colos)          | 表示该数据内容有多少列   |
> | nrows(number rows)           | 表示该数据内容有多少行   |
> | xllcorner(x-low-left-corner) | 栅格的左下角坐标 x 的值  |
> | yllcorner(y-low-left-corner) | 栅格的左下角坐标 y 的值  |
> | cellsize(cell size)          | 每个单元格元的尺寸       |
> | NODATA_value                 | 单元格内没有值时对应的值 |
>
> > 你可以把 .asc文件 想象成一个数据表格，每个单元格为一项，nrows 行 ncols 列 个单元格构建成了一个 数据网格。
> >
> > 除了 .asc 开头的属性值键对外，后面的就是依次填入数据单元网格中的数据。





**第3问：根据人口多少，如何创建对应的柱状物？**

当得到地球某个经纬度(地球纹理图片上的某个坐标)上对应的人口数据后，就可以根据人口数量按照一定比例，创建柱状物。



原理讲过后，那接下来就是实际操作了。



## 基础示例：HelloEarth

接下来，将通过以下几个步骤，逐步实现我们的目标示例。



**特别说明：**

以下几个步骤中的代码，重点是向你讲解具体的功能和思路，并不是最终的代码。

最终完整的示例代码中，会对这些每个步骤中的代码进行新的组织。



#### 第1步：加载人口数据文件(gpw_v4_014mt_2010.asc)

1. **数据文件路径为 ./src/assets/data/gpw_v4_014mt_2010.asc**

2. **由于我们使用 alias 来得到 .asc 文件编译后的路径，所以请记得：**

   1. tsconfig.pahts.json 的 paths 中添加 `"@/assets/*": ["./src/assets/*"]`

   2. global.d.ts 中添加 `declare module '*.asc';`

   3. 以上 2 处均配置正确后，才可以让我们在代码中方便使用 `require('@/assets/xx/xx.asc').default` 来获取 .asc 资源的路径

      > 假设你并不是使用 react + typescript + alias，那么你可以忽略我提到的配置，直接请求一个固定的网络资源(.asc文件)就好了。

3. **通过 window.fetch() 这个函数来获取 .asc 文件内容**

   > 我们这里没有使用 xhr 或 axios 来请求获取文件资源，而是使用了 fetch 这个 Web API
   >
   > 关于 fetch 的用法，请参考：https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API

**具体的代码：**

```
const loadDataFile = async (url: string) => {
    try {
        const res = await window.fetch(url)
        const text = await res.text() // text 就是 .asc 文件里的内容
    } catch (error) {
        console.log('加载数据出错')
    }
}

const ascURL = require('@/assets/data/gpw_v4_014mt_2010.asc').default
loadDataFile(ascURL)
```



> 额外说一个事情，本文对应的是 Three.js 官方教程 https://threejsfundamentals.org/threejs/lessons/threejs-optimize-lots-of-objects.html
>
> 我在阅读英文原文时，当时他代码中使用的是：
>
> ```
> async function loadFile(url) {
>   const req = await fetch(url);
>   return req.text();
> }
> ```
>
> 我认为不应该将返回值使用变量 req(request)，而应该是 res(response)，于是我就提交了一个合并请求(PR)，然后很快就得到 [greggman](https://github.com/greggman) 的回应，我的 PR 已被合并到 master 中。
>
> 呵，我也顺带成为了这个项目中的一名 贡献者(contributor)。



#### 第2步：解析人口数据

1. **为了方便我们以后代码提示，我们先使用 TypeScript 定义解析 .asc 数据后的格式**

   ```
   type DataType = (number | undefined)[][]
   type ASCData = {
       data: DataType,
       ncols: number,
       nrows: number,
       xllcorner: number,
       yllcorner: number,
       cellsize: number,
       NODATA_value: number,
       max: number,
       min: number,
   }
   ```

   > data 为栅格化的世界人口数据，一共 nrows 条，每一条是由 ncols 个数字构成
   >
   > 假设某个点对应有人口数据则值为具体的数字，若没有人口则值为 undefined。
   >
   > 请记得没有人口数据的值为 undefined，而不是 0。
   >
   > max、min 分别为我们添加的自定义属性，用来记录所有地区人口数据中最多和最少的人口数量，以此我们方便计算出 柱状高度比例

2. **开始解析 .asc 文件内容，大体步骤如下：**

   1. 首先我们知道 .asc 中每一行对应一条数据，那么就可以使用换行符 '\n' 来分隔出每一条数据，然后针对每一条数据进行解析

      ```
      text.split('\n').forEach((line) => { ... })
      ```

   2. 被分隔出来的每一行数据，再进一步转化和分析：

      1. 由于可能存在多个连续空格，因此我们对每一条数据，再通过正则表达式 `/\s+/` 进一步分隔

         ```
         // 在正则表达式 ‘/\s+/’ 中 s 表示为空格，+ 表示 1个或多个
         const parts = line.trim().split(/\s+/)
         ```

      2. 位于 .asc 文件开头，描述栅格化数据的一些属性，例如 ncols、nrows...，这些数据的结构为：`属性名 + 空格 + 值` 构成的

         ```
         if (parts.length === 2) { ... }
         ```

      3. 位于 .asc 文件中间，一行行，一条条具体的数据值，这些数据的结构为：`数字 + 空格 + 数字 + ...`

         ```
         if (parts.length > 2) { ... }
         ```

      4. 位于 .asc 文件尾部，可能存在的、无用的空白换行，这些空白换行是需要被我们通过条件判断来忽略掉的

         ```
         由于前面已经进行了 length === 2 或 > 2 的判断，那么剩下的就肯定是空白无用的换行，我们什么也不做处理就好。
         ```

   3. 在解析所有人口数据的过程中，我们要不断记录、得出 人口最大数值和最小数值

   4. 最终将解析好的数据结果对象，通过 TS 的 as 断言，对外返回出结果

   5. 补充一点：由于我们从 text 中读取到的 “数字” 其实是 字符串，所以在解析过程中都需要使用 parseFloat() 这个函数将 string 转化为 number

   6. 再补充一个细节，在初始化 max 和 min 时：

      1. 让 max 初始化值为 0，因为我们知道有人口数据的值一定是大于 0 的
      2. 让 min 初始化值为 99999，因为我们知道一定有人口数据的值一定是小于 99999 的，且人口数量一定不会是负数

   **具体的代码：**

   ```
   const parseData = (text: string) => {
       const data: (number|undefined)[][] = []
       const settings: { [key: string]: any } = { data }
       let max:number = 0
       let min:number = 99999
       text.split('\n').forEach((line) => {
           const parts = line.trim().split(/\s+/)
           if (parts.length === 2) {
               settings[parts[0]] = parseFloat(parts[1])
           } else if(parts.length > 2) {
               const values = parts.map((item) => {
                   const value = parseFloat(item)
                   if (value === settings['NODATA_value']) {
                       return undefined
                   }
                   max = Math.max(max, value)
                   min = Math.min(min, value)
                   return value
               })
               data.push(values)
           }
       })
       return { ...settings, ...{ max, min } } as ASCData
   }
   ```

   

#### 第3步：加载地球纹理图片

```
const loader = new Three.TextureLoader()
const texture = loader.load(require('@/assets/imgs/world.jpg').default,render)
const material = new Three.MeshPhongMaterial({
    map: texture
})
const geometry = new Three.SphereBufferGeometry(2, 32, 32)
const earth = new Three.Mesh(geometry, material)
scene.add(earth)
```

> 请注意上述代码中，loader.load(xxx, render)，我们希望当纹理图片加载完成后，才执行 render 渲染



#### 第4步：将人口数据与地球纹理图片进行位置上的匹配

**先不考虑球体，假设我们仅仅想获得一张显示人口数量分布、平面的世界地图，该如何做呢？**

**代码思路：**

1. 我们通过 第 2 步骤已经拿到了栅格化后的世界人口分布数据
2. 并且我们知道栅格化的数据是由 nrow(145) 行、ncols(360) 列组成
3. 假设 1 个数据点 对应 1 像素，那么栅格化的数据实际上对应的是一个 高 145 像素、宽 360 像素的图形
4. 假设 数据点最小(人口最少)的地方，我们用黑色来填充，而数据点最大(人口最多)的地方用红色填充，处于中间数量的点按照比例依次进行颜色变化，那么就可以得到我们想要的图形了。
   1. 关于某个点填充的颜色，我们使用 HSL(色相、饱和度、亮度)，其中当 人口少时 L 的值越接近于 0 (黑色)、人口多时 L 的值越接近 1 (红色)
   2. 向画布(canvas) 某个点填充颜色，需要用到 canvas 一些相关知识，请自行先学习了解一下 canvas 相关知识

**对应的代码：**

```
const hsl = (h: number, s: number, l: number) => {
    return `hsl(${h * 360 | 0},${s * 100 | 0}%,${l * 100 | 0}%)`
}

const drawData = (ascData: ASCData) => {
    if (canvasRef.current === null) { return }
    const ctx = canvasRef.current.getContext('2d')
    if (ctx === null) { return }

    const range = ascData.max - ascData.min
    ctx.canvas.width = ascData.ncols
    ctx.canvas.height = ascData.nrows
    ctx.fillStyle = '#444'
    ctx.fillRect(0, 0, ctx.canvas.width, ctx.canvas.height)
    ascData.data.forEach((row, rowIndex) => {
        row.forEach((value, colIndex) => {
            if (value === undefined) { return }
            const amount = (value - ascData.min) / range
            const hue = 1
            const saturation = 1
            const lightness = amount
            ctx.fillStyle = hsl(hue, saturation, lightness)
            ctx.fillRect(colIndex,rowIndex,1,1)
        })
    })
}
```

为了让你比较直观看清，这里贴出目前我们已经写出来的代码。

> 请注意下面的代码并不是我们真正示例的代码，你可以实际运行以下，查看效果

```
import { useEffect, useRef } from 'react'

const loadDataFile = async (url: string) => {
    const res = await window.fetch(url)
    const text = await res.text()
    return text
}

type DataType = (number | undefined)[][]
type ASCData = {
    data: DataType,
    ncols: number,
    nrows: number,
    xllcorner: number,
    yllcorner: number,
    cellsize: number,
    NODATA_value: number,
    max: number,
    min: number,
}

const parseData = (text: string) => {
    const data: DataType = []
    const settings: { [key: string]: any } = { data }
    let max: number = 0
    let min: number = 99999
    text.split('\n').forEach((line) => {
        const parts = line.trim().split(/\s+/)
        if (parts.length === 2) {
            settings[parts[0]] = parseFloat(parts[1])
        } else if (parts.length > 2) {
            const values = parts.map((item) => {
                const value = parseFloat(item)
                if (value === settings['NODATA_value']) {
                    return undefined
                }
                max = Math.max(max, value)
                min = Math.min(min, value)
                return value
            })
            data.push(values)
        }
    })
    return { ...settings, ...{ max, min } } as ASCData
}

const hsl = (h: number, s: number, l: number) => {
    return `hsl(${h * 360 | 0},${s * 100 | 0}%,${l * 100 | 0}%)`
}

const HelloEarth = () => {
    const canvasRef = useRef<HTMLCanvasElement>(null)

    const drawData = (ascData: ASCData) => {
        if (canvasRef.current === null) { return }
        const ctx = canvasRef.current.getContext('2d')
        if (ctx === null) { return }

        const range = ascData.max - ascData.min
        ctx.canvas.width = ascData.ncols
        ctx.canvas.height = ascData.nrows
        ctx.fillStyle = '#444'
        ctx.fillRect(0, 0, ctx.canvas.width, ctx.canvas.height)
        ascData.data.forEach((row, rowIndex) => {
            row.forEach((value, colIndex) => {
                if (value === undefined) { return }
                const amount = (value - ascData.min) / range
                const hue = 1
                const saturation = 1
                const lightness = amount
                ctx.fillStyle = hsl(hue, saturation, lightness)
                ctx.fillRect(colIndex,rowIndex,1,1)
            })
        })

    }

    useEffect(() => {
        if (canvasRef.current === null) { return }
        const ascURL = require('@/assets/data/gpw_v4_014mt_2010.asc').default
        const doSomthing = async () => {
            try {
                const text = await loadDataFile(ascURL)
                const ascData = parseData(text)
                drawData(ascData)
            } catch (error) {
                console.log(error)
            }
        }
        doSomthing()

        return () => {

        }
    }, [canvasRef])
    
        return (
        <canvas ref={canvasRef} />
    )
}

export default HelloEarth
```

实际运行后，就会看到一张 世界人口分布的地图

> 请注意这个 “看似是世界地图”的图片并不是真正的世界地理位置地图，而是人口数量分布图。



**如果你已经看懂了上面的代码，那么接下来就可以真正去制作 3D 立体地球示例了。**

> 本示例是我们做过的最复杂的例子，尽管我们已经进行了详细的思路解读，你一定要多看，多敲几遍，否则接下来的代码你可能更加难以理解。



**我们需要将之前的 drawData() 修改 为 addBoxes()，并且不再是在 canvas 中绘制点，而是在球体上添加柱状物：**

```
const addBoxes = (ascData: ASCData, scene: Three.Scene) => {
    const geometry = new Three.BoxBufferGeometry(1, 1, 1)
    geometry.applyMatrix4(new Three.Matrix4().makeTranslation(0, 0, 0.5))

    const lonHelper = new Three.Object3D()
    scene.add(lonHelper)

    const latHelper = new Three.Object3D()
    lonHelper.add(latHelper)

    const positionHelper = new Three.Object3D()
    positionHelper.position.z = 1
    latHelper.add(positionHelper)

    const range = ascData.max - ascData.min
    const lonFudge = Math.PI * 0.5
    const latFudge = Math.PI * -0.135
    ascData.data.forEach((row, latIndex) => {
        row.forEach((value, lonIndex) => {
            if (value === undefined) { return }
            const amount = (value - ascData.min) / range
            const material = new Three.MeshBasicMaterial()
            const hue = Three.MathUtils.lerp(0.7, 0.3, amount)
            const saturation = 1
            const lightness = Three.MathUtils.lerp(0.1, 1, amount)
            material.color.setHSL(hue, saturation, lightness)
            const mesh = new Three.Mesh(geometry, material)
            scene.add(mesh)

            lonHelper.rotation.y = Three.MathUtils.degToRad(lonIndex + ascData.xllcorner) + lonFudge
            latHelper.rotation.x = Three.MathUtils.degToRad(latIndex + ascData.yllcorner) + latFudge

            positionHelper.updateWorldMatrix(true, false)
            mesh.applyMatrix4(positionHelper.matrixWorld)
            mesh.scale.set(0.005, 0.005, Three.MathUtils.lerp(0.001, 0.5, amount))
        })
    })
}
```

> 上面代码中牵扯到了非常多新的、之前从未使用过的一些函数或属性。

**解释说明：**

1. 栅格化数据 和 纹理图片 均可看作是 2D 矩形坐标，最终需要转化为 3D 球体坐标，转化过程中 lonFudge、latFudge 具体作用机理，暂时还没搞明白。

   > 先记住转化公式，以后再慢慢研究

   > 栅格化数据 为 360 * 145、纹理图片为 2048 * 1024

2. Three.Matrix4：WebGL 中的矩阵库

3. Three.MathUtils：Three.js 中内置的一些计算函数

   > 关于这些新的对象具体详细介绍，请查阅 Three.js 官方文档

4. lonHelper用于赤道上的经度旋转、latHelper用于维度旋转、positionHelper用于 Z 轴(地球地面)上的偏移。

5. lonFudge 的值为 Math.PI * 0.5，也就是相当于 1/4 个圆(地球 1/4 圈)

6. latFudge 的值为 Math.PI * -0.135，这里的 -0.135 不太清楚是怎么得出来的，但是大概率推测它是用来将柱状物与纹理图片对齐的



**示例所需其他代码块：**

1. 创建 3D 地球、以及加载纹理图片
2. 添加 OrbitControls 控制，并且开启 “弹性结束控制”
3. 添加场景渲染函数 render，并且添加 “按需渲染” 相关代码



**最终完整的示例代码：**

```
import { useEffect, useRef } from 'react'
import * as Three from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

import './index.scss'

const loadDataFile = async (url: string) => {
    const res = await window.fetch(url)
    const text = await res.text()
    return text
}

type DataType = (number | undefined)[][]
type ASCData = {
    data: DataType,
    ncols: number,
    nrows: number,
    xllcorner: number,
    yllcorner: number,
    cellsize: number,
    NODATA_value: number,
    max: number,
    min: number,
}

const parseData = (text: string) => {
    const data: DataType = []
    const settings: { [key: string]: any } = { data }
    let max: number = 0
    let min: number = 99999
    text.split('\n').forEach((line) => {
        const parts = line.trim().split(/\s+/)
        if (parts.length === 2) {
            settings[parts[0]] = parseFloat(parts[1])
        } else if (parts.length > 2) {
            const values = parts.map((item) => {
                const value = parseFloat(item)
                if (value === settings['NODATA_value']) {
                    return undefined
                }
                max = Math.max(max, value)
                min = Math.min(min, value)
                return value
            })
            data.push(values)
        }
    })
    return { ...settings, ...{ max, min } } as ASCData
}

// const hsl = (h: number, s: number, l: number) => {
//     return `hsl(${h * 360 | 0},${s * 100 | 0}%,${l * 100 | 0}%)`
// }

let renderRequested = false

const HelloEarth = () => {
    const canvasRef = useRef<HTMLCanvasElement>(null)

    // const drawData = (ascData: ASCData) => {
    //     if (canvasRef.current === null) { return }
    //     const ctx = canvasRef.current.getContext('2d')
    //     if (ctx === null) { return }

    //     const range = ascData.max - ascData.min
    //     ctx.canvas.width = ascData.ncols
    //     ctx.canvas.height = ascData.nrows
    //     ctx.fillStyle = '#444'
    //     ctx.fillRect(0, 0, ctx.canvas.width, ctx.canvas.height)
    //     ascData.data.forEach((row, rowIndex) => {
    //         row.forEach((value, colIndex) => {
    //             if (value === undefined) { return }
    //             const amount = (value - ascData.min) / range
    //             const hue = 1
    //             const saturation = 1
    //             const lightness = amount
    //             ctx.fillStyle = hsl(hue, saturation, lightness)
    //             ctx.fillRect(colIndex, rowIndex, 1, 1)
    //         })
    //     })
    // }

    const addBoxes = (ascData: ASCData, scene: Three.Scene) => {
        const geometry = new Three.BoxBufferGeometry(1, 1, 1)
        geometry.applyMatrix4(new Three.Matrix4().makeTranslation(0, 0, 0.5))

        const lonHelper = new Three.Object3D()
        scene.add(lonHelper)

        const latHelper = new Three.Object3D()
        lonHelper.add(latHelper)

        const positionHelper = new Three.Object3D()
        positionHelper.position.z = 1
        latHelper.add(positionHelper)

        const range = ascData.max - ascData.min
        const lonFudge = Math.PI * 0.5
        const latFudge = Math.PI * -0.135
        ascData.data.forEach((row, latIndex) => {
            row.forEach((value, lonIndex) => {
                if (value === undefined) { return }
                const amount = (value - ascData.min) / range
                const material = new Three.MeshBasicMaterial()
                const hue = Three.MathUtils.lerp(0.7, 0.3, amount)
                const saturation = 1
                const lightness = Three.MathUtils.lerp(0.1, 1, amount)
                material.color.setHSL(hue, saturation, lightness)
                const mesh = new Three.Mesh(geometry, material)
                scene.add(mesh)

                lonHelper.rotation.y = Three.MathUtils.degToRad(lonIndex + ascData.xllcorner) + lonFudge
                latHelper.rotation.x = Three.MathUtils.degToRad(latIndex + ascData.yllcorner) + latFudge

                positionHelper.updateWorldMatrix(true, false)
                mesh.applyMatrix4(positionHelper.matrixWorld)
                mesh.scale.set(0.005, 0.005, Three.MathUtils.lerp(0.001, 0.5, amount))
            })
        })
    }

    useEffect(() => {
        if (canvasRef.current === null) { return }

        const canvas = canvasRef.current
        const renderer = new Three.WebGLRenderer({ canvas })
        const camera = new Three.PerspectiveCamera(45, 2, 0.1, 100)
        camera.position.z = 4
        const scene = new Three.Scene()
        scene.background= new Three.Color(0x000000)

        const controls = new OrbitControls(camera, canvas)
        controls.enableDamping = true
        controls.enablePan =false
        controls.update()

        const render = () => {
            renderRequested = false
            controls.update()
            renderer.render(scene, camera)
        }

        const handleChange =() =>{
            if(renderRequested === false){
                renderRequested = true
                window.requestAnimationFrame(render)
            }
        }
        controls.addEventListener('change',handleChange)

        const loader = new Three.TextureLoader()
        const texture = loader.load(require('@/assets/imgs/world.jpg').default, render)
        const material = new Three.MeshBasicMaterial({
            map: texture
        })
        const geometry = new Three.SphereBufferGeometry(1, 64, 32)
        const earth = new Three.Mesh(geometry, material)
        scene.add(earth)

        const handleResize = () => {
            const width = canvas.clientWidth
            const height = canvas.clientHeight
            camera.aspect = width / height
            camera.updateProjectionMatrix()
            renderer.setSize(width, height, false)

            window.requestAnimationFrame(render)
        }
        handleResize()
        window.addEventListener('resize', handleResize)

        const ascURL = require('@/assets/data/gpw_v4_014mt_2010.asc').default
        const doSomthing = async () => {
            try {
                const text = await loadDataFile(ascURL)
                const ascData = parseData(text)
                //drawData(ascData)
                addBoxes(ascData, scene)
                render()
            } catch (error) {
                console.log(error)
            }
        }
        doSomthing()

        return () => {
            controls.removeEventListener('change',handleChange)
            window.removeEventListener('resize', handleResize)
        }
    }, [canvasRef])

    return (
        <canvas ref={canvasRef} className='full-screen' />
    )
}

export default HelloEarth
```

调试运行，首先就会看到一个 3D 立体地球，等待 1 秒左右，待 .asc 数据加载并解析、添加地球上的柱状物后，就会看到本示例所想演示的最终效果。

> 终于终于到这一步了

不过当你鼠标拖动地球时，会感受到略微卡顿，或者说不够流畅。

那么接下来，就到了本文的核心内容：通过 合并对象 来达到优化场景的目的。



#### 补充：启用浏览器 调试工具 DevTool 的 Rendering 查看渲染性能

除了浏览器本身的 性能(Performance) 面板外，还有另外一个重要的、方便我们查看页面渲染性能的工具——Rendering。

**通过谷歌调试工具 Rendering，查看当前页面渲染性能情况：**

1. 打开浏览器调试工具 DevTool

2. 点击右侧 3 个小圆点

3. 鼠标移动到 More tools

4. 点击 Rendering

5. 在新出现的 Rendering 面板中，勾选 Frame Rendering Stats

   > 备注：在旧的谷歌浏览器中，应该勾选的是 Show FPS meter

这样就可以在网页左上角，实时看到当前渲染性能状况。



**性能数据解读：**

性能展示的数据，主要 2 个模块：Frames 和 GPU

**GPU 相关：**

1. GPU raster ：on  表示 GPU 光栅化已开启

2. GPU memory：GPU 已用大小、GPU 最大可用大小

   > 在本示例中，通常是当修改浏览器尺寸时，此时需要大量计算，会显示出 GPU memory
   >
   > 在普通的 鼠标拖拽 改变地球视角时，不会显示 GPU memory

**Frames相关：**

假设某一时刻，渲染性能结果为 Frames：63% 1082(0m) dropped of 2737

对应的解读为：

第1个数字 63% —— 63% 的帧按时渲染完成

第2个数字 1082 —— 有 1082 个合成帧丢失(未渲染)

第3个数字 0m —— 有 0 个帧丢失

第4个数字 2737 —— 原本计划渲染 2737 个帧

数字之间的计算关系为 63% ≈ 1 - (1082 + 0 )/ 2737

也就是说 第2个数字(丢失的合成帧)越小，那么整体按时完成渲染帧的百分比(第1个数字)越大，意味着此刻网页越流畅。



## 优化代码：合并对象

#### 核心代码分析

在上面的示例代码中，lonHelper用于赤道上的经度旋转、latHelper用于维度旋转、positionHelper用于 Z 轴(地球地面)上的偏移。

> 默认 Three.js 中物体是有 1/2 位于 Z 轴之下的，通过Z轴的偏移让柱状物可以完全出现在地面上

每一个数据点(柱状物)都创建了一个 MeshBasicMaterial 和 Mesh。

我们的数据点一共为 145行、360列，那么就意味着假设全部数据点都有数据，那么数据点总数量为：145 * 360 = 52200，但是考虑到有非常多的数据点的值为 -9999(NODATA_value)，也就是没有值，不需要绘制，那么减去这些没有数据点，**最终需要绘制的数据点(柱状物)大约为 19000 个**。

**柱状物19000个，再加上对应的 3 个辅助对象(lonHelper、latHelper、positionHelper)，相当于总绘制数量为 19000 * 4 = 76000。**

也就是说每一次场景更新，大约需要绘制 7.6 万个对象，所以这才造成了卡顿现象。




**如何解决卡顿？减少需要渲染对象的数量！**

还记得我们刚才统计的渲染对象数量吗？

1. 柱状体 约 19000个
2. 每个柱状体对应 3 个辅助对象 19000 * 3

我们需要做的就是把所有的柱状体合并成一个物体，也就是说原本需要渲染 19000 个柱状体，合并之后只需渲染 1 个，让柱状体数量减少 18999 个。



**修改 addBoxes 函数代码：**

```
import { BufferGeometryUtils } from 'three/examples/jsm/utils/BufferGeometryUtils'

const addBoxes = (ascData: ASCData, scene: Three.Scene) => {
    //const geometry = new Three.BoxBufferGeometry(1, 1, 1)
    //geometry.applyMatrix4(new Three.Matrix4().makeTranslation(0, 0, 0.5))

    const lonHelper = new Three.Object3D()
    scene.add(lonHelper)

    const latHelper = new Three.Object3D()
    lonHelper.add(latHelper)

    const positionHelper = new Three.Object3D()
    positionHelper.position.z = 1
    latHelper.add(positionHelper)

    const originHelper = new Three.Object3D()
    originHelper.position.z = 0.5
    positionHelper.add(originHelper)

    const range = ascData.max - ascData.min
    const lonFudge = Math.PI * 0.5
    const latFudge = Math.PI * -0.135
    const geometries: Three.BoxBufferGeometry[] = []
    const color = new Three.Color()
    ascData.data.forEach((row, latIndex) => {
        row.forEach((value, lonIndex) => {
            if (value === undefined) { return }
            const amount = (value - ascData.min) / range
            //const material = new Three.MeshBasicMaterial()
            //const hue = Three.MathUtils.lerp(0.7, 0.3, amount)
            //const saturation = 1
            //const lightness = Three.MathUtils.lerp(0.1, 1, amount)
            //material.color.setHSL(hue, saturation, lightness)
            //const mesh = new Three.Mesh(geometry, material)
            //scene.add(mesh)

            const geometry = new Three.BoxBufferGeometry(1, 1, 1)

            lonHelper.rotation.y = Three.MathUtils.degToRad(lonIndex + ascData.xllcorner) + lonFudge
            latHelper.rotation.x = Three.MathUtils.degToRad(latIndex + ascData.yllcorner) + latFudge

            //positionHelper.updateWorldMatrix(true, false)
            //mesh.applyMatrix4(positionHelper.matrixWorld)
            //mesh.scale.set(0.005, 0.005, Three.MathUtils.lerp(0.001, 0.5, amount))

            positionHelper.scale.set(0.005, 0.005, Three.MathUtils.lerp(0.01, 0.5, amount))
            originHelper.updateWorldMatrix(true, false)
            geometry.applyMatrix4(originHelper.matrixWorld)

            const hue = Three.MathUtils.lerp(0.7, 0.3, amount)
            const saturation = 1
            const lightness = Three.MathUtils.lerp(0.1, 1, amount)
            color.setHSL(hue, saturation, lightness)
            const rgb = color.toArray().map((value) => {
                return value * 255
            })

            const numVerts = geometry.getAttribute('position').count
            const itemSize = 3
            const colors = new Uint8Array(itemSize * numVerts)

            //这里有一个稍微奇葩点的写法，就是使用下划线 _ 来起到参数占位的作用
            colors.forEach((_, index) => {
                colors[index] = rgb[index % 3]
            })

            const normalized = true
            const colorAttrib = new Three.BufferAttribute(colors, itemSize, normalized)
            geometry.setAttribute('color', colorAttrib)

            geometries.push(geometry)
        })
    })
    const mergedGeometry = BufferGeometryUtils.mergeBufferGeometries(geometries)
    //const material = new Three.MeshBasicMaterial({ color: 'red' })
    const material = new Three.MeshBasicMaterial({
        vertexColors: true
    })
    const mesh = new Three.Mesh(mergedGeometry, material)
    scene.add(mesh)
}
```

> 上述代码中，注释部分为之前 addBoxes() 函数的代码，除了 //const material = new Three.MeshBasicMaterial({ color: 'red' }) 这一行

**代码解析：**

1. 合并所有的柱状物，使用到了一个新的函数 BufferGeometryUtils.mergeBufferGeometries()

   > 注意：BufferGeometryUtils 并非来自 Three，而是来自 'three/examples/jsm/utils/BufferGeometryUtils'

2. 柱状物的颜色，不再使用 color 设定，而是启用了 “顶点着色”。

> 关于这 2 个大的知识点，可以去阅读 Three.js 官方文档

> 需要恶补官方文档，如果只是看了本教程，那么还会有大量的知识点未曾接触。



经过合并优化后的场景，在浏览器中运行，比之前的流畅非常多，没有卡顿的现象了。

> 我本机电脑硬件配置比较高，我分别记录了 Rendering 面板中 优化前后的 Frames 值。
>
> 优化前：顺利渲染帧的百分比约为 60%
>
> 优化后：顺利渲染帧的百分比约为 90%
>
> 可见网页流畅度确实提高了很多



#### 本文小结：

**在 Three.js 大型的场景中，绝大多数都需要采用合并对象的策略来优化渲染性能。**

合并对象可以减少需要渲染的对象数量，并且还可以将有一些根本不可见的面进行删除，减少渲染面，提高渲染性能。



你以为就这样可以结束了？

事实上还有优化空间，本文先到这里结束。

下一篇将继续优化这个场景。