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

2. 给每个平面添加一个纹理贴图，并设置正方形平面透明度为 0.5

3. 让这 2 个平面形成 十字交叉 的状态







