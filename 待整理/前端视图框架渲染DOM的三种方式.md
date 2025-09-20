
在当今的前端开发中，React、Vue、SolidJS 等框架已然成为构建复杂应用的绝对主流。它们通过声明式的编程模型，将开发者从繁琐而易错的直接 DOM 操作中解放出来。我们只需关心“应用的状态是什么”，而无需编写命令式的代码来告诉浏览器“如何一步步更新视图”。框架的渲染引擎接管了**状态到 UI 的转换与同步工作**，保证了应用性能与开发者体验。

然而，这种“解放”也带来了一层“魔法”面纱。我们享受着 `useState` 或 `signals` 带来的响应性，看着 JSX 或模板被神奇地变成屏幕上的像素，却可能对幕后的核心机制——**框架究竟如何以及为何以特定方式渲染和更新 DOM**——感到陌生。

不同的框架在设计哲学上各有千秋，这直接导致了它们在渲染策略上的根本性分歧。那么，主流框架在渲染 DOM 时究竟有哪些不同的心智模型和实现路径？

根据 SolidJS 框架创建者 **Ryan Carniato** 在油管的一个视频分享， 框架渲染DOM基本上可分为三种形式
1. `Render by Replacement`
2. `Dirty Checking`
3. `Virtual DOM`

并且提供了每种形式的实现原理demo


## `Render by Replacement`
```js
// 框架层
export class Framework {
	state = {}
	
	constructor(initSatte) {
		this.state = initSatte
		// 初始化渲染
		this._template()
	}
	
	/**
	* 实例必须要实现 否则会报错
	*/
	template() {
		throw new Error("must be implated")
	}
	
	setState(newState) {
		this.state = newState
		// 数据更新时重新渲染
		this._template()
	}
	
	_template() {
		// 每次渲染都是完整的innerHTML替换
		document.getElementById("app").innerHTML = this.template()
	}

}
```

```js
// 应用层
import { Framework } from "./framework.js"

class App extends Framework {

	constructor() {
		super({ count: 0 })
		setInterval(() => {
			this.setState({ count: this.state.count + 1 })
		}, 1000)
	}
	
	/**
	* 负责整个页面的dom结构 
	*/ 
	template() {
		return `<div>${this.state.count}</div>`
	}
}

  

new App()
```

这种渲染模式，比较简单直接，每次更新都是根据状态重新生成的新的DOM结构，并且使用innerHTMl直接替换
思路简单，但是效率差

## `Dirty Checking` 

