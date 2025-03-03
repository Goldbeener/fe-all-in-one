## 背景
浏览器并不支持ES模块，开发者不能直接以模块化的方式进行开发和部署
并且开发中也使用到不同格式的文件模块，js、css、svg、png、json等等
这个文件都被按照模块处理
所以有了各种各样的打包器

**打包器的工作**：抓取、处理开发中使用的源码模块，并将其串联成可以在浏览器中运行的文件

随着JS构建应用规模的扩大，目前市面上的打包器**存在的问题**
1. 冷启动缓慢， 打包器需要抓取并构建整个应用，才能提供服务
2. 热替换缓慢 HMR缓慢

Vite实现的基础依据：
1. 浏览器开始原生支持ES模块
2. 越来越多的JS工具使用编译型语言编写

## Vite的解决方案：

**缓慢的服务器启动**
一开始，将应用中的模块区分为 **依赖** 和 **源码**

> 通过import/export支持的静态分析，识别出裸模块，判定为依赖？

`依赖`，开发时不会变动的纯JS，使用esbuild预构建依赖，将CommonJS/UMD转换成ESM格式

`源码`，开发者编写的需要转换的非JS文件，JSX、CSS、Vue组件等；这部分内容基于路由拆分，按需加载，并不是直接处理、加载应用的所有模块

Vite以原生ESM方式提供源码，Vite只需要在浏览器请求源码时进行转换并按需提供源码，
也就是只需要提供页面展示内容对应的模块，并且只需要做转换工作

在开发环境，所有的模块都是以ESM形式提供的


**缓慢的更新**
Vite的HMR是在原生ESM上执行的，
一个ESM更新时，Vite精准的**使已编辑的模块与其最近的HMR边界之间的链失活**

同时利用HTTP缓存加速页面加载
依赖模块使用强缓存持久存储，一旦缓存不需要再次请求
源码模块会使用304协商缓存

**开发环境和生产环境区别**
开发环境，ESBuild + ESM
生产环境，使用rollup打包输出产物


## Vite组成
+ 一个开发服务器
+ 一套构建指令

生产构建，默认情况下目标浏览器支持原生ES模块、原生ESM动态导入、import.meta、nullish coalescing、BigInt

开发期间vite是一个服务器，
index.html 是vite项目的入口文件，
因此index.html是在项目根目录下

根目录
入口文件、vite配置所在


### 功能
+ npm依赖解析和预构建
	+ esbuild 预构建 速度快
	+ 重写裸模块url，方便后续使用原生esm
	+ 依赖强缓存 
+ 模块热替换
+ TS支持
	+ 仅转译，不校验
+ HTML处理，会作为应用的一部分进行处理和打包，也就是修改html内容
+ Vue文件支持
+ JSX/TSX  开箱即用
	+ 但是vue项目需要额外加上 `@vitejs/plugin-vue-jsx` 插件
+ CSS
+ PostCSS
+ CSS Modules
+ CSS 预处理器
+ Lightning CSS
+ 静态资源处理
+ JSON
	+ 整个对象导入
	+ 根字段具名导入
+ Glob 导入
	+ 多模块批量导入
+ 动态导入
+ WebAssembly
+ Web Workers
+ CSP

css预处理器
css modules
PostCSS 
三者的关系以及作用，是否可以搭配使用，以及在整个编译周期所处的位置和前后顺序

预处理器
提供增强的CSS语法，如CSS变量、嵌套、函数等

CSS Modules
让CSS作用域只对当前组件有效，避免全局污染

PostCSS
处理CSS兼容性、插件化扩展（自动添加铅坠、嵌套、压缩等）

执行顺序：
预处理器 > CSS Modules > PostCSS

### 构建优化
默认请款下开箱即用，不要显式配置，除非需要禁用
+ 代码分割
+ 预加载指令生成
+ 异步chunk加载优化


### 命令行接口
CLI命令
+ vite
+ vite build
+ vite preview


### 依赖预构建
时机：首次启动Vite
1. CommonJS和UMD兼容，开发阶段，Vite作为开发服务器，提供的所有模块都是按照ESM规范处理的，所以必须要把依赖的CommonJS和UMD模块，转换成ESM模块
2. 性能，将依赖包分散的ESM模块，合并成一个模块，减少请求次数


自动依赖搜寻
Monorepo和链接依赖
自定义行为

缓存
+ 文件系统缓存，预构建的依赖项
+ 浏览器缓存，已预构建的依赖在浏览器中强缓存


### 静态资源处理
+ 将资源引入为URL
+ 显式URL引入
+ 显式内联处理
+ 将资源引入为字符串
+ 导入脚本作为Worker
+ public目录
+ new URL + import.meta.url
	+ 通过字符串模版支持动态URL

### 构建生产版本

默认构建产物，chrome >= 87

`@vitejs/plugin-legacy `
包含语法转译和polyfill



公共基础路径
base 配置
影响静态资源产物路径覆写


相对基础路径

自定义构建

产物分块策略
处理加载报错

多页面应用模式

库模式

高级基础路径
入口文件、静态资源可能想要部署在不同的路径，以便设置不同的缓存策略


### 环境变量和模式

**内置常量**
import.meta.env.MODE
import.meta.env.BASE_URL
import.meta.env.PROD
import.meta.env.DEV
import.meta.env.SSR

**环境变量** 
env相关文件提供的信息
+ VITE_SOME_KEY
+ .env文件 使用及优先级
+ HTML环境变量替换
+ 环境变量的智能提示

**模式**
--mode  xxx 
影响的是  import.meta.env.MODE 的值

NODE_ENV  process.env.NODE_ENV


+ [ ] process.env.NODE_ENV 和 mode 在Vite中的使用场景？

## 插件
插件默认在vite核心插件之后调用该插件，
默认在开发和生产模式下都会调用
+ 强制插件排序  可以使用enforce指明插件调用时机
+ 按需应用，可以使用apply属性指明，仅在开发或生产调用

- [ ] 创建一个自定义插件