
+ 组件状态
+ 页面状态
+ 页面缓存状态
+ 路由栈状态
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
