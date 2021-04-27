# 02 初始化Three.js项目

本文以 Yarn 而不是 NPM 安装 Three.js。

**第1步：全局安装 create-react-app**

```
yarn global add create-react-app
```



**第2步：初始化React+TypeScript项目**

> 我用的是 create-react-app 4.0.2 版本，对应的是 React 17.0.1

```
yarn create react-app test-threejs --template typescript
```



**第3步：初次修改 tsconfig.json 配置**

1. 修改 TS 编译目标 ES版本为 es2017

   ```
   "target": "es2017"
   ```

2. 添加一些本人 TS 配置偏好

   ```
   "noUnusedLocals": true,
   "noUnusedParameters": true,
   "sourceMap": true,
   "removeComments": false
   ```



**第4步：配置 alias，安装对应模块**

由 create-react-app 创建的 React 项目中，配置 alias(路径映射)，我采用的方案是：react-app-rewired + react-app-rewire-alias

```
yarn add --dev react-app-rewired react-app-rewire-alias
```



**第5步：完善 alias 配置**

1. 在项目根目录，新建文件 tsconfig.paths.json，内容暂时设置为：

   ```
   {
       "compilerOptions": {
           "baseUrl": ".",
           "paths": {
               "@/src/*": ["./src/*"],
               "@/components/*": ["./src/components/*"]
           }
       }
   }
   ```

   > 我们暂时先添加 2 个路径映射 src 和 components，具体路径还会根据将来实际开发过程中所需要创建不同的目录结构进行修改。
   >
   > 补充：根据 typescript 官方更新说明文档，baseUrl 这一项是可以省略的，但是上面代码中还是遵循了之前的配置方式，继续添加上了 baseUrl。

2. 在项目根目录，新建文件 config-overrides.js，内容为：

   ```
   const { alias, configPaths } = require('react-app-rewire-alias')
   
   module.exports = function override(config) {
     alias(configPaths('./tsconfig.paths.json'))(config)
   
     return config
   }
   ```

   > 注意是 .js 文件 而不是 .ts 文件，鼠标放到 require 上后也许会显示提示文字：文件是 CommonJS 模块；它可能会转换为 ES6 模块。
   >
   > 请忽略这个提示，这并不是什么错误信息。
   >
   > require 是 Nodejs 导入模块的方式，TypeScript 导入模块使用的是 import。

3. 在项目根目录，新建文件 global.d.ts，内容为：

   > 注意：本步骤的目的是为了让 TS 忽略对 react-app-rewire-alias 和其他一些非常规格式文件的导入检查。
   >
   > 本步骤是可选的，不是必须的，你可以跳过本步骤。

   ```
   declare module '*.png';
   declare module '*.gif';
   declare module '*.jpg';
   declare module '*.jpeg';
   declare module '*.svg';
   declare module '*.css';
   declare module '*.less';
   declare module '*.scss';
   declare module '*.sass';
   declare module '*.styl';
   declare module 'react-app-rewire-alias';
   ```

4. 修改 tsconfig.json 文件，添加以下一行内容：

   ```
   {
     "extends": "./tsconfig.paths.json",
     "compilerOptions": {
       ...
     }
   }
   ```

   > 请注意 extends 是和 compilerOptions 平级的。

   至此，我们的 tsconfig.json 最终内容如下：

   ```
   {
     "extends": "./tsconfig.paths.json",
     "compilerOptions": {
       "target": "es2017",
       "lib": [
         "dom",
         "dom.iterable",
         "esnext"
       ],
       "allowJs": true,
       "skipLibCheck": true,
       "esModuleInterop": true,
       "allowSyntheticDefaultImports": true,
       "strict": true,
       "forceConsistentCasingInFileNames": true,
       "noFallthroughCasesInSwitch": true,
       "module": "esnext",
       "moduleResolution": "node",
       "resolveJsonModule": true,
       "isolatedModules": true,
       "noEmit": true,
       "jsx": "react-jsx",
       "noUnusedLocals": true,
       "noUnusedParameters": true,
       "sourceMap": true,
       "removeComments": false
     },
     "include": [
       "src"
     ]
   }
   ```

   

**第6步：修改 package.json 中的 scripts 命令**

将命令中 start、build、test 3 条命令中的 react-scripts 修改为 react-app-rewired

```
"scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build",
    "test": "react-app-rewired test",
    "eject": "react-scripts eject"
  },
```



**第7步：安装 scss**

```
yarn add node-sass --dev
```

> 假设你使用的是较早版本的 create-react-app，那么当时还不支持最新版 node-sass 5.0，所以你只能安装： `yarn add node-sass@4.14.1 --dev`



**第8步：安装three.js**

```
//npm install --save three
yarn add three
```



> 以下更新于 2021.04.11

**关于 .d.ts 文件的特别说明：**

写这篇文章的时候是 2020年11月底，当时应该是 r124 版本，当时的 Three.js 版本里内置了 .d.ts 文件，但是随着 Three.js 版本升级，大约在 r126 版本以后官方已经将内置的 .d.ts 文件移除，目前最新的版本是 r128。

**所以，我们现在还需要额外安装对应的 .d.ts 文件包：**

```
//npm install @types/three
yarn add @types/three
```



<br>

本系列教程前面相当一部分示例是基于 Three.js 0.124.0 的，而这些示例有可能会在最新版本 0.127.0 中不能正常运行，特此说明。

> 所谓不能正常运行，多数都是因为某些类在新版本中引入路径发生了变化，可根据 VSCode 中的提示进行修改。



我们会在第 23 小节开始，使用 Three.js 最新版本 r127。

<br>



> 以上更新于 2021.04.11



**第9步：给 package.json 添加 homepage 字段**

```
{
  "name": "test-threejs",
  "homepage": ".",
  ...
}
```

homepage 字段是用来设定 html 中文件资源(编译后的 js 或 css) 的根 URL 地址。

假设不添加 homepage 字段，则默认使用网站根目录作为 资源根目录。

> 我猜测 homepage 的默认值是 "/"

如果你的项目将来并不是发布在网站根目录，那么设置 homepage 字段会非常有用。

举例：

假设将来项目网址为：https://threejs.puxiao.com，那么你完全可以跳过本步骤，不作任何修改。

但若将来项目网址为：https://threejs.puxiao.com/demo/，那么此时所有文件资源存放在 demo 这个目录下(并不是网站根目录)，那么必须给 package.json 添加 "homepage": "."，不然文件资源都会出现 404 状态。



**第10步：清理默认 create-react-app 创建的一些无用内容**

以下为我个人的习惯，你可以根据自己喜好对应选择是否清理。

1. 清除掉 index.tsx 中 <React.StrictMode> 、reportWebVitals() 

   > 目前 React 最新版中对应的 tsconfig.json 默认就使用的是严格模式，因此 <React.StrictMode> 是可以删除掉的

2. 删除掉 src 目录下 setupTests.ts、reportWebVitals.ts、App.test.tsx 文件

   > 这些都是用来做 React 自动单元调试的，暂且用不到

3. 删除掉 public 目录下 logo192.png、logo512.png、manifest.json、robots.txt

4. 清除掉 public 目录下 index.html 中无用的代码，只保留最基础的标签。

   ```
   <!DOCTYPE html>
   <html lang="zh-CN">
   
   <head>
     <meta charset="utf-8" />
     <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
     <meta name="viewport" content="width=device-width, initial-scale=1" />
     <meta name="theme-color" content="#000000" />
     <meta name="description" content="Web site created using create-react-app" />
     <title>Hello Threejs</title>
   </head>
   
   <body>
     <noscript>You need to enable JavaScript to run this app.</noscript>
     <div id="root"></div>
   </body>
   
   </html>
   ```




经过以上 10 个步骤之后，已经创建好了一个基础的开发环境：React + TypeScript + Scss + Alias + Threejs

下一章，终于要开始编写 hello world 了。

