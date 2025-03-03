

TS = JS + 类型系统

TS类型系统表现出非常明显的编程语言特性


## 关键字/符号
+ 类型：boolean、number、string、null、undefined、unknow、any、never
+ 运算符： &、｜ 、...
+ 声明：type、interface、declare
+ 其他：readonly、keyof、extends、infer、？、-？


## 语法

### 声明
声明类型结构： `interface`  描述静态类型<数据>结构

声明类型：`type` 描述类型之间的计算关系

> 有点类似Vue中，状态和计算属性的关系

```js
interface Point {
	x: number
	y: number
}

type Points = Point & { z: number } // 计算确切结果

// 描述计算过程，类似函数 计算输入的两个类型之间的共有属性
type Intersect<P, Q> = Pick<P, keyof P & keyof Q> 
```


### 条件分支
```js
declare function distance<P extends Point>(p1: P, p2: P): number

declare function promisify<T>(p: T): T extends Promise<any> ? T : Promise<T>
```


### 递归
