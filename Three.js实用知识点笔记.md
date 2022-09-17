# Three.js实用知识点笔记

从今天开始，在本文中记录实际 Three.js 开发过程中所遇到的知识点。



<br>

**关于示例代码的一般约定说明：**

1. 举例的时候，很多都是伪代码
2. 为了保证代码简洁，所以在实用某些类时没有添加 `Three.` 前缀
3. 绝大多数时候都使用箭头函数
4. 使用 Xxx 泛指同一类型，例如 XxxCamer 泛指各类相机
5. 由于我们讲解的是 Three.js，所以全部使用的是 右手坐标系



<br>

#### 01、每一个Object3D对象都只能有一个父级

这里说的 Object3D 实际上包括所有继承于 Object3D 的子类，例如 Mesh、Camera、Group 等

举例说明：

```
const mesh = new Mesh(geometry,material)

const sceneA = new Scene()
const sceneB = new Scene()

sceneA.add(mesh)
sceneB.add(mesh)
```

由于 mesh 只能有一个父类，所以当 sceneB 也执行 .add(mesh) 后，sceneA.children 中会自动删除掉 mesh。



<br>

#### 02、克隆或复制 Mesh 不会在内存中真正复制出一份 顶点(geometry)和材质(material)，它们使用的是引用，而不是复制

Object3D 拥有 .clone() 和 .copy() 两个方法，Mesh 继承于 Object3D，所以也拥有这两个方法。

在 Mesh 类中扩展了 .copy() 方法，但是针对 Mesh 内部的属性 顶点和材质，实用的是引用而不是真正内存中的复制。

```
const meshB = meshA.clone()
```

上述代码中新复制得到的 meshB 仅仅复制了 meshA 的一些变换相关的属性，例如 matrix 等，但是对于占内存大头的 顶点和材质 这两项实用的是引用。

也就是说此时 meshA 和 meshB 它们共用了一份 geometry 和 material。

> 不用担心因为多复制了几份 mesh 而增加很多内存。



<br>

#### 03、添加场景(scene)或其他Object3D渲染之前和渲染之后的回调函数

场景 scene 继承于 Object3D，而 Object3D 可以配置 2 个渲染之前或之后的回调函数：

```
scene.onBeforeRender = () => { ...}
scnet.onAfterRende = () => { ... }
```

使用场景举例：假设我们希望不渲染场景上的某一类元素，那么我们可以在 renderer.render() 之前通过上面 2 个回调函数进行设置

```
scene.onBeforeRender = () => {
    scene.children.forEach(item => {
        if(item.type === 'Points'){
            item.visible = false
        }
    })
}

scene.onAfterRender = () => {
    scene.children.forEach(item => item.visible = true)
}

renderer.render(scene,camera)
```



<br>

#### 04、通过 .layers 控制物体是否被渲染

在 Three.js 中 .layers 对应的是 Layers 这个类，Three.js 规定 Layers 级别的值取值范围为 0 - 32。

> 你可以把 layers 翻译成 “级别”，也可以称呼为 “层级”

任何继承于 Object3D 的类，例如 相机、物体 等都具有 .layers 属性。

它们的 .layers 默认级别都为 0。

不能通过直接给 .layers 赋值的方式修改级别，而是应该通过 .set(value) 这种形式。

```
mymesh.layers.set(1)
camera.layers.set(1)
```



<br>

对于相机而言，它只能渲染出同一级别的物体元素。

```
const meshA = new Mesh(...)
//meshA.layers.set(0) //默认就是 0

const meshB = new Mesh(...)
meshB.layers.set(1)

const scene = new Scene()
scene.add(meshA)
scene.add(meshB)

const cameraA = new XxxCamera()
//cameraA.layers.set(0) //默认就是 0

const cameraB = new XxxCamera()
cameraB.layers.set(1)
```

在上面代码中：

1. 我们按照默认的形式添加了 meshA、cameraA，它们默认层级为 0
2. 手动修改了  meshB、cameraB 的 .layers 层级为 1

<br>

那么当执行下面的代码：

```
renderer.render(scene, cameraA)
renderer.render(scene. cameraB)
```

1. cameraA 只会渲染出场景中同一级别的 meshA
2. cameraB 只会渲染出场景中同一级别的 meshB



<br>

也可以选择随时修改 meshA 的 .layers 值，这样 cameraB 就可以渲染到它了。

```
meshA.layers.set(1)
renderer.render(secen, cameraB)
```



<br>

换句话说，假设我们希望控制是否渲染场景中某些元素，那么有 2 种途径：

1. 设置其 .visible 的值来决定是否渲染
2. 设置其 .layers 的值来决定只被同一层级的相机渲染



<br>

#### 05、手工修改Object3D实例的.matrix时切记要设置 .matrixAutoUpdate=false

默认情况下 Object3D 实例的 .matrixAutoUpdate 的值为 true，也就是说当通过 .applyMatrix4()、.applyQuaternion() 等修改实例的变换时，默认会自动更新其他所有相关属性值，例如 position、quaternion、scale、rotation。

但是，如果直接通过修改 Object3D 实例的 .matrix 值时，生效的前提是：

1. 先把 .matrixAutoUpdate 设置为 false
2. 不去调用 .updateMatrix()

```
const mesh = new Mesh(...)

mesh.matrixAutoUpdate = false
mesh.matrix.copy(otherMatrix)
```



<br>

但是上面的代码存在另外一个问题：尽管 .matrix 值更新了，可是 mesh 的其他属性值 例如 .position，.quaternion，scale，rotation 却没有自动更新。

解决方式很简单，可以通过 Matrix 的 .decompose() 方法优雅更新它们。

举例说明：假设现在有 meshA、meshB 两个对象，需要将 meshB 的各种变换属性值设置成和 meshA 完全相同

```
meshB.matrixAutoUpdate = false
meshB.matrix.copy(meshA.matrix)
meshB.matrix.decompose(meshB.position, meshB.quaternion, meshB.scale)
```

> 当修改 meshB.quaternion 值后会自动修改 meshB.rotation 的值



<br>

#### 06、绘制三角形的顶点顺序决定了该三角形是正面(顺时针)还是反面(逆时针)

一个三角形有 3 个顶点，假定为 a、b、c，那么：

1. 假定 a b c 连接顺序为 逆时针，那么最终形成的三角形为 正面
2. 假定 a b c 连接顺序为 顺时针，那么最终形成的三角形为 反面(背面)

另外一种判定形式是：右手握住沿着两个顶点添加顺序的连接线，此时大拇指指示方向即为正面



<br>

#### 07、保持外观和位置的前提下，将立方体的顶点坐标 "归一化"

这里说的立方体是指基于 BoxGeometry 而创建的立方体。

这里说的 顶点坐标 “归一化” 是指将立方体顶点坐标修改成 1x1x1 规格的立方体顶点坐标。

这里说的 保持外观和位置 是指通过修改其 变换矩阵 .matrix 来实现。

实现思路：

1. 凡是基于 BoxGeometry 的立方体，其顶点坐标信息都是统一规范的，尽管其值可能不同
2. 所以我们就根据其值来确定这个立方体与 1x1x1 立方体的 宽、高、深 比例
3. 将这个缩放比例应用到立方体本身的矩阵中即可
4. 同时将这个立方体的顶点信息修改成 1x1x1 立方体的顶点信息

```
const boxGeometryNormalize = (box) => {
    const originX = box.geometry.attributes.position.array[0]
    const originY = box.geometry.attributes.position.array[1]
    const originZ = box.geometry.attributes.position.array[2]

    const scaleX = originX / 0.5
    const scaleY = originY / 0.5
    const scaleZ = originZ / 0.5
    
    box.geometry = new BoxGeometry()
    box.matrixAutoUpdate = false
    box.matrix.makeScale(scaleX, scaleY, scaleZ)
    box.matrix.decompose(box.position, box.quaternion, box.scale)
}
```



<br>



