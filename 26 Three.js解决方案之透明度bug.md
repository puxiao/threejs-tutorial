# 26 Three.js解决方案之透明度bug

透明度(transparency) 在 Three.js 中很容易实现，但是透明度又存在一个 "Bug"，解决起来又比较难。

> 请注意，这里说的 “Bug” 是加了引号的，具体原因我们稍后讲解。



<br>

我们先从一个简单的示例开始。

### 示例1：渲染一个半透明的立方体

所有材质的基类 Three.Material 有 2 个属性和设置透明度有关：

1. transparent：设置材质是否透明
2. opacity：设置材质透明度，取值范围 0 - 1

> 由于 Material 是所有材质的基类，也就意味着所有材质都拥有上述  2 个属性



<br>

假设我们想创建一个半透明的立方体，代码如下：

```
const geometry = new Three.BoxBufferGeometry(2, 2, 2)
const material = new Three.MeshBasicMaterial({
    color: 'red',
    transparent: true,
    opacity: 0.5
})
const cube = new Three.Mesh(geometry, material)
scene.add(cube)
```



<br>

上面我们只是给材质设置了一个 红色，当我们运行程序的时候会发现：单独一个半透明立方体，似乎看不出任何半透明的意思。

> 即使给材质添加 `side: Three.DoubleSide`

我们需要添加多个半透明立方体，才更容易看出来彼此半透明。



我们继续修改示例。

<br>

### 渲染8个小立方体

我们的渲染目标：

1. 渲染8个不同颜色，透明度都为 0.5 的立方体

2. 这 8 个立方体分布在一个 2 x 2 x 2 的空间中

   > 这种分布方式，在魔方玩具中被称为 “二阶魔方”

3. 每个立方体 里外 2 个面都要进行渲染



<br>

具体实现代码：

```
const colors = ['red', 'blue', 'darkorange', 'darkviolet', 'green', 'tomato', 'sienna', 'crimson']
const cube_size = 1 //立方体尺寸
const cube_margin = 0.6 //立方体间距空隙
colors.forEach((color, index) => {
    const geometry = new Three.BoxBufferGeometry(cube_size, cube_size, cube_size)
    const material = new Three.MeshPhongMaterial({
        color,
        transparent: true,
        opacity: 0.5,
        side: Three.DoubleSide
    })

    const cube = new Three.Mesh(geometry, material)
    cube.position.x = (index % 2 ? 1 : -1) * cube_size * cube_margin
    cube.position.y = (Math.floor(index / 4) ? -1 : 1) * cube_size * cube_margin
    cube.position.z = ((index % 4) >= 2) ? 1 : -1 * cube_size * cube_margin / 2

    scene.add(cube)
})
```

> 在目前最新的 Three.js r127 版本中，对于预置颜色的单词，只支持全小写，例如：
>
> 1. 红色 只可以写 `red`，不可以写成 `Red`
> 2. 再或者 暗桔色 只可以写 `darkorange`，不可以写成`DarkOrange`
>
> 这是因为在 Color 源码中，记录内置颜色值的对象 key 都是小写，我已针对这个问题提交了自己的 pr：
>
> https://github.com/mrdoob/three.js/pull/21687
>
> 我修改了一点代码，让取值时对颜色值的字符串执行 .toLowerCase()，这样即使颜色值字符串有大写可以最终实际被转化为小写。



<br>

实际运行后，就会看到 8 个半透明的小立方体。通过 OrbitControls 旋转视角，看着感觉挺好的呀。



<br>

<!-- 如果我不告诉你，可能你一直也发现不了这里面存在着一个 “bug” -->

当你尝试不断变换视角查看立方体时，在某些特殊的视角下，我们看不到立方体左侧后表面。

> 也就是说，原本立方体左侧后表面应该也被渲染，但是实际上并未被渲染。



<br>

如果你实在是没有看出来什么问题，暂时相信我一下，就好像你已真的发现了那样。

下面听一下关于出现这个 "bug" 的解释。



<br>

### Three.js绘制 3D 对象的方式

上面提到的渲染 “Bug”，是由于 Three.js 绘制 3D 对象的方式造成的。

<br>

对于每一个几何图形，每个三角形一次只绘制一个。

> 立方体的 1 个面是一个 正方形，而这个正方形是由 2 个三角形构成的。
>
> 在绘制 1 个面(正方形)，Three.js 会先后绘制 2 个三角形，最终拼接成 1 个正方形。



<br>

每次绘制一个三角形时，会记录 2 个事情：

1. 三角形的颜色

2. 三角形的像素深度

   > 像素深度是指存储每个像素所用的位数，用来度量图像的分辨率。

3. 当绘制下一个三角形时，对于每一个像素，如果深度比之前记录的深度还要深，则不会绘制任何像素

   > 这其实是 Three.js  绘制物体时采取的一种节省性能的策略



<br>

这套策略对于不透明的物体来说非常有用，但是对于透明的物体却不起作用。



<br>

一个立方体有 6 个面，每个面 2 个三角形，也就意味着一个立方体需要绘制 12 个三角形。

而这 12 个三角形究竟先绘制哪个，他们绘制的顺序是什么呢？

答：绘制顺序取决于我们的视角，越接近相机的三角形越优先被绘制。

> 这就是为什么我们上面提到的绘制 bug 只有在某些特定角度下才会出现的原因。

> 越靠近相机的三角形越先被绘制，这也意味着在某些角度下，远离摄像机的某个面(立方体背面或侧背面)有可能不会被绘制。



这种情况不仅会出现在立方体身上，球体上也会出现。



<br>

**针对以上情况，有一种解决方案：**

1. 将每个立方体添加 2 次到场景中
2. 第 1 次添加的立方体设置只让渲染 背面(Three.BackSide)
3. 第 2 次添加的立方体设置只让渲染 前面(Three.FrontSide)

这样一番操作过后，确保 Three.js 可以将每个立方体的前面、后面都会渲染，拼合一下 2 次的渲染结果，将 “正确的” 结果渲染出来。



<br>

**补充说明**

1. 上面的解决方案实际上需要绘制 2 次，这也许会造成性能上的浪费。
2. 如果你并不是特别在乎那一点点渲染 “Bug”，你完全可以忽视它。



<br>

接下来我们再通过另外一个示例，讲解另外一种有针对性的解决方案。



<br>

### 绘制2个中心交叉的平面正方形

我们的示例目标是：

1. 绘制 2 个平面的正方形

   > 创建平面 在 Three.js 中使用的是 Three.PlaneBufferGeometry

2. 给每个平面添加一个颜色和纹理贴图，两面都渲染，并设置正方形平面透明度为 0.5

   > 贴图我们直接使用网上的 2 张图片资源
   >
   > 图片都是由背景色的，并非背景透明的 PNG

3. 让这 2 个平面形成 十字交叉 的状态

   > 也就是说让其中一个平面的 y 轴旋转 180度



<br>

**实现的代码：**

```
const planeDataArr = [
    {
        color: 'red',
        ratation: 0,
        imgsrc: 'https://threejsfundamentals.org/threejs/resources/images/happyface.png'
    },
    {
        color: 'yellow',
        ratation: Math.PI * 0.5,
        imgsrc: 'https://threejsfundamentals.org/threejs/resources/images/hmmmface.png'
    }
]

planeDataArr.forEach((value) => {
    const geometry = new Three.PlaneBufferGeometry(2, 2)
    const textureLoader = new TextureLoader()
    const material = new Three.MeshBasicMaterial({
        color: value.color,
        map: textureLoader.load(value.imgsrc),
        opacity: 0.5,
        transparent: true,
        side:Three.DoubleSide
    })
    const plane = new Three.Mesh(geometry, material)
    plane.rotation.y = value.ratation
    scene.add(plane)
})
```



<br>

这次，我们将很容易看到，当红色平面一侧完全覆盖住黄色平面一侧时，会完全看不到黄色平面那一侧。

> 实际运行效果我就不贴图了，你可以将上面代码实际运行一下。

> 你就假装此刻你看到了。



<br>

用我们上面讲过的理论可解释这个现象：即 红色平面一侧颜色深度大于黄色平面一侧，当完全覆盖住之后黄色平面那一侧就不会再进行渲染，所以我们就看不到了。



<br>

**解决方案：将上面 2 个平面拆分成 4 个平面，这样可以确保每个平面都会被渲染。**

由于我们这个场景 2 个平面十字交叉，所以我们就直接创建 4 个小的平面，然后将这 4 个小平面组合成 “2 个十字相交的平面”。

具体实现的方式是：

1. 将原本 较大的 1 个平面拆分成 2 个小平面

2. 设置这 2 个小平面的纹理贴图偏移，各自占一半

   > 这样可以最终让 2 个小平面贴合成 1 个完整的平面(纹理)

3. 为了方便我们计算旋转，好让他们形成十字交叉，所以我们可以将 每组小平面放置在同一个空间中

   > 这需要使用到 Three.Object3D



<br>

**实际代码：**

```
const planeDataArr = [
    {
        color: 'red',
        ratation: 0,
        imgsrc: 'https://threejsfundamentals.org/threejs/resources/images/happyface.png'
    },
    {
        color: 'yellow',
        ratation: Math.PI * 0.5,
        imgsrc: 'https://threejsfundamentals.org/threejs/resources/images/hmmmface.png'
    }
]

planeDataArr.forEach((value) => {
    const base = new Three.Object3D()
    base.rotation.y = value.ratation
    scene.add(base)

    const plane_size = 2
    const half_size = plane_size / 2
    const geometry = new Three.PlaneBufferGeometry(half_size, plane_size)
    const arr = [-1, 1]
    arr.forEach((x) => {
        const textureLoader = new TextureLoader()
        const texture = textureLoader.load(value.imgsrc)
        texture.offset.x = x < 1 ? 0 : 0.5
        texture.repeat.x = 0.5
        const material = new Three.MeshBasicMaterial({
            color: value.color,
            map: texture,
            transparent: true,
            opacity: 0.5,
            side: Three.DoubleSide
        })
        const plane = new Three.Mesh(geometry, material)
        plane.position.x = x * half_size / 2
        base.add(plane)
    });
})
```

>假设我们目标平面宽高均为 2，那么：
>
>1. 我们将该目标平面拆分成 2 个 宽 1、高 2 的平面
>2. 获取并设置纹理贴图，并分别设置纹理的 offset.x 、repeat.x 各占 一半，也就是 0.5
>3. 我们知道拆分出的 2 个小平面他们 x 轴相差 1 个小平面的宽度，由于我们设置的内部循环数组为 [-1,1]，所以 2 个小平面的 x 值应该是 正负宽度一半的一半。



<br>

这一次，我们再运行就会发现，无论任何视角下，红色平面不再会这该黄色平面了。



<br>

**这是第 2 种解决透明 "bug" 的方案：将对象进行拆分**

**但是请注意，该解决方式只适合那些简单，且位置相对固定的 3D 对象。**

> 若物体本身就比较复杂，面比较多，还要再拆分，那么就太消耗渲染性能
>
> 并且位置必须相对固定，若不固定会增加我们拼接的难度



<br>

接下来讲解第 3 种解决方案。

### 启用alphaTest来避免遮挡问题

首先我们回顾一下上面的例子，当时示例中 2 个平面的纹理贴图背景是不透明的。

那我们可以尝试另外 2 个图片贴图，他们是背景透明的 PNG 图片。

我们要使用材质(Three.Material) 的一个属性 .alphaTest。



<br>

**.alphaTest属性介绍**

.alphaTest 是一个透明度检测值，值得类型是 Number，取值范围为 0 - 1。

若透明度低于该值，则不会进行渲染。

> 反之，只有某个点透明度高于该值的才会进行渲染

.alphaTest 默认值为 0。

> 也就是说默认情况下即使透明度为 0 也会进行渲染
>
> 假设我们给材质设置有 color，那么肯定就会渲染出内容



<br>

我们在最初 2 个平面的代码基础上进行修改。

1. 修改纹理贴图资源，这次使用背景透明的 PNG 图片
2. 材质不再设置 .opacity 属性，改设置 .alphaTest 属性

<br>

修改后的代码如下：

```
const planeDataArr = [
    {
        color: 'red',
        ratation: 0,
        imgsrc: 'https://threejsfundamentals.org/threejs/resources/images/tree-01.png'
    },
    {
        color: 'yellow',
        ratation: Math.PI * 0.5,
        imgsrc: 'https://threejsfundamentals.org/threejs/resources/images/tree-02.png'
    }
]

planeDataArr.forEach((value) => {
    const geometry = new Three.PlaneBufferGeometry(2, 2)
    const textureLoader = new TextureLoader()
    const material = new Three.MeshBasicMaterial({
        color: value.color,
        map: textureLoader.load(value.imgsrc),
        alphaTest: 0.5,
        transparent: true,
        side:Three.DoubleSide
    })
    const plane = new Three.Mesh(geometry, material)
    plane.rotation.y = value.ratation
    scene.add(plane)
})
```

> 请注意，上面代码中我们取了一个透明度的中间值，将 .alphaTest 属性值设置为 0.5。
>
> 你可以尝试将 .alphaTest 分别设置为 0.2 或 0.8 看看结果会有什么变化。
>
> > 最终边缘清晰度取决于贴图图片中抠图的精细程度。若边缘越不清晰(也就是越模糊)，最终呈现出的白边会越严重。



<br>

实际运行后就会发现，2 棵不同颜色的树木，彼此十字交叉，可以透过前面树的枝叶看到另外一颗树。



### 本文小节

我们讲解了为什么会在某些视角下，某些半透明的物体个别地方三角面不会被渲染的原因。

通过几个示例，讲解了 3 种解决方案：

1. 将物体添加 2 份，1份负责渲染前面，另外一份负责渲染后面

   > 缺点：增加渲染工作量

2. 将物体(或平面)进行拆分，已确保每 1 份均会有机会被渲染

   > 缺点：只适合简单的物体，且位置固定容易拼凑

3. 通过设置 .alphaTest，以实现透明渲染

   > 缺点：若贴图抠图不够精细，容易出现白边



<br>

就像上面我们提到的，每一种解决方案都有各自的使用场景和缺点。

我们今后在实际的项目中，一定要根据实际情况来作出选择，看使用哪种方案。

> 说白了，无非就是在性能、复杂度、精细化方面进行取舍，最终找出合适的方案。



<br>

> 你可能会注意到本章节我并没有贴出完整的示例代码，而仅仅贴出了核心的代码。
>
> 我是这样认为的，如果到了今天你依然无法自己写出完整的代码，还需要靠复制我完整的示例代码，那你干脆别学 Three.js了，放弃吧。



<br>

至此，本章结束。

目前我们所有的示例都是基于 1 个 画布(canvas) 和 1 个 镜头(camera)，下一节我们讲解同一个网页中渲染多个 画布 和多个镜头。

