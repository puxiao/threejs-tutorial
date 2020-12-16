# 11 Three.js基础之灯光

## 灯光的种类

在场景中添加灯光后，灯光照射在物体上产生明暗、光亮和阴影，从而让物体显得更加立体有光泽。

> 在有些文档或教程中，会把 灯光 称呼为 光源，这只是对 light 这个单词的不同翻译而已

> 向场景中添加灯光 这个行为，在传统 3D 软件制作中通常被称为 “打灯”



### 灯光的种类

**在 Three.js 中，有 6 种类型的灯光，他们都继承于 Three.Light 。**

| 灯光类型(都继承于Light) | 灯光名称           | 是否产生阴影 |
| ----------------------- | ------------------ | ------------ |
| AmbientLight            | 环境光、氛围光     | 否           |
| DirectionalLight        | 平行光             | 是           |
| HemisphereLight         | 半球光源、户外光源 | 否           |
| PointLight              | 点光源可以         | 是           |
| RectAreaLight           | 矩形面光源         | 否           |
| SpotLight               | 聚光灯光源         | 是           |

> 有个别文档或教程中，会把 HemisphereLight 也称呼为 “环境光”。  
> 事实上 AmbientLight 和 HemisphereLight 的作用都是提供环境光，只是 HemisphereLight 的环境光更加真实，当然渲染所需性能也更多。



### 灯光作用与特点

**AmbientLight(环境光、氛围光)：**  

通常仅作为基础光线，一般需要与其他灯光配合使用。不能产生阴影、也无需指定坐标位置，仅需设置颜色和强度。

注意：不会产生阴影。



**DirectionalLight(平行光)：**

最为经常使用的光源，光纤(光芒)都是平行向着一个方向发射。

经常使用 DirectionalLight 来模拟太阳光照射到某个物体上的光照效果。



**HemisphereLight(半球光源)：**

相对 ambientLight，hemisphereLight 更加真实的模拟自然光源，提供 天空 和 地面 漫反射光线。

一共接收 3 个参数：

* 第 1 个参数：天空光线颜色
* 第 2 个参数：地面反射光颜色
* 第 3 个参数：光的反射强度

注意：不会产生阴影。



**PointLight(点光源)：**

类似生活中的灯泡，光纤(光芒)没有固定方向，朝着四周散射。



**RectAreaLight(矩形面光源)：**

与 DirectionalLight 模拟太阳光不同，RectAreaLight 光源形状为一个矩形，可以模拟出明亮的窗口或矩形照明光源。

注意：

1. 不会产生阴影
2. 只有 MeshStandardMaterial 和 MeshPhysicalMaterial 材料才支持 RectAreaLight 光源
3. 场景中，必须加入 RectAreaLightUniformsLib



**SpotLight(聚光灯)：**

类似生活中的聚光灯效果。



**3 种 Probe：**

| probe类型            | 解释说明 |
| -------------------- | -------- |
| AmbientLightProbe    |          |
| HemisphereLightProbe |          |
| LightProbe           |          |



## 灯光的效果

我们将通过创建一个 HelloLight 的例子，直观的观察不同类型灯光的效果。

> 这个示例中，会用到我们上节学习的 纹理 相关知识。



#### 前期准备1：OrbitControls 的简介和用法

本示例中需要用到一个之前从未使用过的类：OrbitControls，先简单介绍一下这个类。

1. orbit 单词的翻译为：轨道
2. controls 单词的翻译为：控制权

OrbitControls 是鼠标镜头轨道控件，可以通过鼠标来配置镜头的运动轨道，例如 缩放、平移、旋转。也就是说在不修改场景的前提下，可以通过鼠标来改变镜头，以便查看不同角度下的场景。

> 在手机端，不是鼠标，而是手势



**OrbitControls的用法：**

```
const controls = new OrbitControls(camera, canvas) //创建一个实例
controls.target.set(0, 5, 0) //controls.target 为镜头的坐标系统
//controls.target.set(0, 5, 0) 的意思是：设置原点 Y 轴的坐标(以高出5米的轨道运行)
controls.update() //使控件使用新目标
```

请注意，在上面代码中，OrbitControls 的构造函数中第 2 个参数为DOM 中的 canvas 节点，实际上当添加过 OrbitControls 之后，鼠标在 canvas 上的 拖拽、鼠标滚轴滚动 等操作都会被捕捉到，并且做出相对应的镜头画面切换。

说直白点，我们终于可以通过鼠标对 3D 场景进行不同角度，距离的切换操作了。



**特别注意：**

OrbitControls 并不是包含在 three 根目录下，而是位于 three/examples/jsm/controls/OrbitControls 中，因此引入代码为：

```
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
```

> 提醒：在 OrbitControls.js 中，分别导出有 OrbitControls 和 MapControls，所以引入 OribitControls 是需要加 大括号 { }。



**补充说明：**

严格意义上讲，OrbitControls 并不是 Three.js 核心包含的代码内容，OrbitControls 是将最常见的鼠标与 Three.js 场景交互内容的一个额外封装。

由于 OrbitControls 实在是过于频繁使用，最终 Three.js 将 OrbitControls 也包含到了 Three.js 代码包中，只不过不在默认的目录中，而是在 Three.js 示例目录中。

> 在 Three.js 早期的版本中，代码包中并未包含 OrbitControls，若想使用还需要 yarn 安装 three-orbitcontrols 这个包。只不过当后来 Three.js 包含了 OrbitControls 之后，才再也无需额外安装了。



#### 前期准备2：制作纹理图片 checker.png

本示例中需要用到一个类似 3D 场景地面黑白网格的纹理，因此我们需要提前准备好这个纹理图片。

在 PhotoShop 中新建一个 宽高都为 2 像素的画布，然后：

1. 左上角和右下角 的那 1 像素中填充一个较黑的颜色
2. 右上角和左下角 的那 1 像素中填充一个较白的颜色
3. 将图片导出为 checker.png，并保存到 src/assets/imgs/ 目录中





