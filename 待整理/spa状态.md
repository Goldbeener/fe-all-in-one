
+ 组件状态
+ 页面状态
	+ 同一个页面，跨组件读写， provide/inject
+ 页面缓存状态 KeepAlive
	+ 缓存的是组件实例，不是单数据
		+ 以组件name进行匹配，必须给组件显式声明一个name
	+ 条件缓存
		+ 路由栈状态缓存，某些页面根据实际情况，动态缓存或者不缓存
+ 应用级状态
+ 会话级状态
+ 持久化状态

```
生命周期短
│
├── 组件级状态
│      ref/reactive
│
├── 页面级状态
│      provide/inject
│
├── 页面缓存状态
│      KeepAlive
│
├── 路由栈状态
│      KeepAlive + CacheManager
│
├── 应用级状态
│      Pinia
│
├── 会话级状态
│      sessionStorage
│
└── 持久化状态
       localStorage / IndexedDB

生命周期长
```
