编译是JavaScript的常态
每当遇到障碍-无论是浏览器特性支持、冗长的语法，还是语言本身的缺陷，我们都会用构建编译器的方式来解决问题

编译和打包成为现代JavaScript应用的核心构建方式，同时也是JavaScript工具链复杂性的根源

编译的好处是巨大的：
类型检查、Linting、Tree Shaking、代码拆分、压缩、同构、宏（Macros）、领域特定语言（DSLs）、单体开发/分布式部署等


打包
浏览器并不支持ES模块，JS也没有提供原生机制让开发者以模块化的方式进行开发
因此，[打包]做的工作就是：使用工具抓取、处理并将我们的源码模块串联成可以在浏览器运行的文件


受SPA影响的同构方法，相同的代码可以同时在客户端和服务端运行


受MPA影响的分层执行，Islands架构 和 服务器组件 Web Component


bundler 打包器，把这些步骤聚合在一块

+ parser
	+ 把代码解析成AST
+ transformer
	+ 修改AST，进行代码转换，输出另一种
+ resolver
+ linter
+ formatter
+ minifier
+ bundler
+ test runner
+ meta framework support


作用：
1. 减少网络请求和瀑布流请求
2. 减少产物体积
3. 提高js的运行性能