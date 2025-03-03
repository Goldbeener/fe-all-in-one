

TS = JS + 类型系统

TS类型系统表现出非常明显的编程语言特性


## 关键字/符号
+ 类型：boolean、number、string、null、undefined、unknow、any、never
+ 运算符： &、｜ 、...
+ 声明：type、interface、declare
+ 其他：readonly、keyof、extends、infer、？、-？、+？

-？ 用于移除在TS interface 或 type中的可选标记？； 确保属性在新类型中是必选的
+？ 用于显示添加可选属性，让属性在新类型中是可选的

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
循环不是编程语言的必要语法，循环的应用场景可以被递归替代

```js
type GenArr<
	N extends number,
	Arr extends any[] = []
> = Arr["length"] extends N ? Arr : GenArr<N, [...Arr, 1]>;

type ThreeItemArr = GenArr<3> // [1, 1, 1] 指定长度的元组，每一项都是1

```

拓展，创建指定长度、指定类型的元组

```js
type GenArr<
	N extends number,
	T,
	Arr extends any[] = []
> = Arr["length"] extends N ? Arr : GenArr<N, T, [...Arr, T]>
```

### 标准库
标准库是以语言自身基础能力实现的通用工具函数

```js
type Partial<T> = {
	[P in keyof T]?: T[P];
}
// Partial<{ x: string}>
// { x?: string }


type Required<T> = {
	[P in keyof T]-?: T[P];
}
// Required<{ x?: string }>
// { x: string }


type Readonly<T> = {
	readonly [P in keyof T]: T[P];
}
// Readonly<{ x: string }>
// { readonly x: string }

type Exclude<T, U> = T extends U ? never : T
// Exclude<"a" | "b" | "c", "a">
// "b" | "c"

type Extract<T, U> = T extends U ? T : never;
// Exclude<"a" | "b" | "c", "a">
// "a"

type Pick<T, K extends keyof T> = {
	[P in K]: T[P]
}
// Pick<{ x: string, y: number }, "x">
// { x: string }

type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>
// Omit<{ x: string, y: number }, 'x'>
// { y: number }

type Record<K extends keyof any, T> = {
	[P in K]: T
}
// Record<string, string >
// { [key: string]: string }

type NonNullable<T> = T extends null | undefined ? never : T
// NonNullable<null | 1 | 2>
// 1 | 2

// 提取函数的参数类型
type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never
// Parameters<(a: string, b: number) => Promise<string>>
// [string, number]

// 提取函数的返回值类型
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any
// ReturnType<(a: string, b: number) => Promise<string>>
// Promise<string>
```


### 练习
柯里化函数类型描述
```js
type Curr<Args, R> = Args extends [infer First, ...infer Rest] ? (arg: First) => Curr<Rest, R> : R

declare function curry<Fn extends (...args: any[]) => any> (fn: Fn): Fn extends (...args: infer Args) => infer R ? Curr<Args, R> : never

function add(a: number, b: number): number {
	return a + b
}
const curryAdd =  curry(add)
curryAdd(1)(2)
```

获取readonly字段
```js
type Equal<First, Second> = (<T>() => T extends First ? true : false) extends (<T>() => T )
```
