# 28 Three.js解决方案之选中、拾取某物体

在日常的 Three.js 应用中，我们除了要渲染出 3D 场景外，还需要大量用户鼠标或笔触(手指触摸)操作。

在讲解本文正式内容之前，我们先讲解一下 Three.js 官方帮我们写好的控制器。

> 在之前示例中我们经常使用的是 OrbitControls，除了这个以外还有其他几种控制器。



### 镜头轨道控制器

> 再次重申，在本系列文章中 “镜头” 和 “摄像机” 是同一个意思，我个人更习惯使用 “镜头” 这个词。

在 Three.js 中，官方提供了以下几种控制器：

| 控制器                    | 作用           | 补充说明                                                     |
| ------------------------- | -------------- | ------------------------------------------------------------ |
| DeviceOrientationControls | 设备朝向控制器 | 这个控制器只能应用在手机端，<br />监听手机设备朝向变化，进而改变镜头朝向 |
| DragControls              | 拖放控制器     | 该控制器实例化时，需要传入可拖放的元素数组                   |
| FirstPersonControls       | 第一人称控制器 | 像游戏中人物行走一样，来切换场景视角<br />该控制器是 FlyControls 的另一种实现 |
| FlyControls               | 飞行控制器     | 像鸟飞行一样的视角，来切换场景                               |
| OrbitControls             | 轨道控制器     | 我们日常使用最为频繁的一个控制器                             |
| PointerLockControls       | 指针锁定控制器 | Pointer，可以让我们脱离鼠标，无限移动使用                    |
| TrackballControls         | 轨迹球控制器   | 与 OrbitControls 类似，但又不相同，<br />它不能恒定保持镜头的 up 向量 |



<br>

**补充说明：**

1. 在使用 DeviceOrientationControls 时，需要在每次渲染函数中执行 .update()
2. 在使用 DragControls 时，需要给控制器实例添加 .addEventListener('drag', render)
3. 在使用 FirstPersonControls 时，当浏览器窗口尺寸发生变化时，需要执行 .handleResize()，已更新交互范围
4. 在使用 FlyControls 时，当浏览器窗口尺寸发生变化时，需要适当修改 .movementSpeed 和 执行 .update( delta:number)
5. 在使用 OrbitControls 时，若要获得阻尼惯性效果，要设置 .enableDamping =  true
6. 在使用 PointerLockControls 时，锁定(隐藏)鼠标光标的是执行 .lock()，退出(显示)鼠标光标的是执行 .unlock
7. 在使用 TrackballControls 时，可以通过键盘 A/S/D 键来控制场景视角



<br>

以上这些控制器，都是用户与场景之间的交互操作。

那么接下来，我们讲解一下在场景中，一些我们自定义的一些鼠标交互操作：拾取元素



<br>

#### 抱歉，本系列教程暂停更新。

接下来一段时间，我打算去开发一个 3D 云点标注工具。

还是从实战中去学习 Three.js 吧，或许有一天会重新继续更新本系列教程。