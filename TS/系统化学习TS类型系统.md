

TS = JS + 类型系统

TS类型系统表现出非常明显的编程语言特性


## 关键字/符号
+ 类型：boolean、number、string、null、undefined、unknow、any、never
+ 运算符： &、｜ 、...
+ 声明：type、interface、declare
+ 其他：readonly、keyof、extends、infer、？、-？


## 语法

### 声明
声明类型结构： `interface`  描述静态类型shu

声明类型：`type`

```js
interface Point {
	x: number
	y: number
}

type Points = Point & { z: number } // 计算确切结果
type Intersect<P, Q> = Pick<P, keyof P & keyof Q> // 描述计算过程，类似函数
```



