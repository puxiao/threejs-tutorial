# threejs-tutorial
从今天 2020年11月27日 开始学习和探索 Three.js 。  




## 我的学习资料
我刚开始学习 three.js，目前主要看 Three.js 官方出的 教程 和文档：
* [threejsfundamentals.org：官方教程](https://threejsfundamentals.org/threejs/lessons/zh_cn/) (该教程只有前几篇是有中文翻译的)
* [threejs.org：官方中文文档](https://threejs.org/docs/index.html#manual/zh/introduction/Creating-a-scene)

除此之外，还有其他几个值得推荐的、国内博主写的 Three.js 系列教程：

* [wjceo.com：暮志未晚写的three.js教程](https://www.wjceo.com/blog/threejs/)
* [hewebgl.com：Three.js基础教程](http://www.hewebgl.com/article/articledir/1)
* [webgl3d.cn：Three.js教程](http://www.webgl3d.cn/Three.js/)

特别说明 hewebgl.com 和 webgl3d.cn 的教程存在问题就是：

1. 教程内容版本有些老化，使用的并不是最新版 three.js
2. 教程基于网页，而不是基于 React，更不是基于 React + TypeScript

但是这两个网站教程作者编写的时候，非常用心，里面讲述的大量关于 Three.js 理论知识是值得反复学习阅读的。

**综上所述**

1. 我会以官方教程(https://threejsfundamentals.org/threejs/lessons/zh_cn/) 为主线。
2. 我会在以上教程、文档，以及我搜集到的其他相关教程基础上，来编写本系列 Three.js 教程。
3. 我会以一个新手的视角，心路历程，来编写本系列 Three.js 教程。



## WebGL相关教程

首先说明一下，如果学想对 Three.js 有更深层次的修炼，那么你一定要去学习一下 WebGL。

> WebGL又分为：WebGL1、WebGL2

Three.js 本身就是针对 WebGL 的封装。

WebGL 教程：https://webglfundamentals.org/webgl/lessons/zh_cn/

WebGL2 教程：https://webgl2fundamentals.org/webgl/lessons/zh_cn/

**假设你不想学习 WebGL 也没有关系，直接学习 Three.js 也是完全没有问题的。**



## 你还需要掌握的技术栈

* **JS、ES6**
* **CSS、SCSS**
* **React、hooks**
* **TypeScript**
* **包管理工具Yarn 或 NPM**

以上是本系列文章使用的技术栈。

若将来要将开发的项目发布到线上，你可能还需要掌握：

* **Git 代码管理**
* **Koa 创建简单web服务器**
* **Nginx 配置静态服务器**
* **Docker 创建容器服务**



## 关于3D建模

Three.js 内置了很多基础模型，也支持内部自定义图形。

但是，**建模并不是 Three.js 最核心和擅长的，Three.js 最核心功能是进行 浏览器 3D 场景渲染和交互**。

**因此学习 Three.js 的核心应该放在 渲染和交互 上，而不是建模。**

> 以上纯粹目前个人观点，仅供参考



#### 传统 3D 软件

多数场景下 3D 建模这个工作还应该在传统的 3D 软件中完成，例如 3D Max、C4D、Blender 等。

因此，若想学好、用好 Three.js，你还需要掌握一门 3D 软件，我个人强烈推荐以下 2 个软件：

**第1推荐(强烈推荐)：C4D**

优点：轻量级 3D 建模软件、支持简体中文、国内中文教程、资源非常多

缺点：软件收费，当然你可以自己网上搜到 ** 版

**第2推荐：Blender**

优点：开源免费、也属于轻量级

缺点：国内使用人群数量较少，教程和资源较少



#### 补充说明

即使在你的项目团队中，有专门的人负责 3D 建模并导出 Three.js 支持的 文件给你使用，我也非常建议你要学习一下 3D 软件。

如果你不曾使用过 3D 软件，那么你会对 Three.js 中的很多概念感到陌生，甚至是无法想象为什么会是这样。

> 例如：场景、网格、材质、灯光 等等。



## 本教程的缺点

#### 1、是Three.js教程，但不是Three.js文档

我们只是从一个初学者的角度来讲解 Three.js，但是不会讲解每一个讲解对象、每一个类的全部属性或方法，如果想了解某个类的全部属性和方法，建议你直接去看 Three.js 的官方文档。

本教程可以带你入门，但你依然需要不断地查阅官方文档，来弥补本教程中没有提及的属性或方法。



#### 2、没有配图

无论是相关知识点，纹理、示例运行效果，都没有配图。

没有别的原因，就是因为我懒，打字已经够占用时间了，真的没更多精力去配图。

不过我的每个示例都有详细完整的代码，你只需要复制到本地，实际调试一下就能看到效果。



#### 3、所有的示例基本上都是独立的，没有抽离出公共的类或组件

在实际的项目中一定会把某些创建过程、处理函数、逻辑进行抽离，单独成为一个类、函数或组件。

但是本系列教程中，为了避免比较绕，每个示例基本上都是完全独立的，包括样式 scss 文件。

这既是优点，也是缺点。

优点是你在查看某个示例时，代码独立而完整。

缺点是由于没有代码抽离，所以代码量会比较多，阅读起来略显麻烦。

> 我只在刚开始的几个示例中添加了代码注释，后面的示例中就因为懒，所以没有添加代码注释。



**大家都是 Three.js 小白新手，一起加油！**

