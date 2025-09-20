
在当今的前端开发中，React、Vue、SolidJS 等框架已然成为构建复杂应用的绝对主流。它们通过声明式的编程模型，将开发者从繁琐而易错的直接 DOM 操作中解放出来。我们只需关心“应用的状态是什么”，而无需编写命令式的代码来告诉浏览器“如何一步步更新视图”。框架的渲染引擎接管了**状态到 UI 的转换与同步工作**，保证了应用性能与开发者体验。

然而，这种“解放”也带来了一层“魔法”面纱。我们享受着 `useState` 或 `signals` 带来的响应性，看着 JSX 或模板被神奇地变成屏幕上的像素，却可能对幕后的核心机制——**框架究竟如何以及为何以特定方式渲染和更新 DOM**——感到陌生。

不同的框架在设计哲学上各有千秋，这直接导致了它们在渲染策略上的根本性分歧。那么，主流框架在渲染 DOM 时究竟有哪些不同的心智模型和实现路径？

根据 SolidJS 框架创建者 **Ryan Carniato** 的一个视频分享， 框架渲染DOM基本上可分为三种形式：
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

```js
// 框架层
export class Framework {
  state = {}

  constructor(initState) {
    this.state = initState

    // 初始化 首次渲染
    document.getElementById("app").append(this.template())
  }

  template() {
    throw new Error("must be implemented")
  }

  setState(newState) {
    this.state = newState
    // 数据更新时 重新渲染
    this.template()
  }
}

const cache = new Map()

export function html(template, ...holes) {
  let prev

  if (!(prev = cache.get(template))) {
    const t = document.createElement("template")
    // 将模板字符串内的插值用注释替换
    t.innerHTML = template.join("<!>")

    cache.set(
      template,
      (prev = {
        node: t.content.firstChild,
        bindings: createBindings(t.content.firstChild),
      })
    )
  }

  diff(holes, prev.bindings)
  return prev.node
}

/**
* 遍历模板字符串
* 转换成注释节点占位符 
* 并且这个bindings的子项和dom中使用的插值变量位置时一一对应的
* 后面数据更新时，找到变化的数据，就可以根据位置信息定位到需要更新的DOM节点  
* */
function createBindings(element) {
  const bindings = []

  let tw = document.createTreeWalker(
    element,
    NodeFilter.SHOW_COMMENT,
    null
  )

  let comment

  while ((comment = tw.nextNode())) {
    bindings.push({
      type: "insert",
      ref: comment,
      value: undefined,
    })
  }

  return bindings
}

/**
* 将变量值 替换到 模板内的对应位置 
*/
function diff(holes, bindings) {
  for (let i = 0; i < holes.length; i++) {
    const binding = bindings[i]

    if (holes[i] !== binding.value) {
      if (binding.type === "insert") {
        if (binding.value == null) {
          // 首次渲染在注释节点之前插入真实的变量值
          binding.ref.parentNode.insertBefore(
            document.createTextNode(holes[i]),
            binding.ref
          )
        } else {
          /**
			* 非首次渲染 之需要更新对应节点的值即可
			* 不需要重新创建节点
			* */
          binding.ref.previousSibling.nodeValue = holes[i]
        }
      }

      binding.value = holes[i]
    }
  }
}

```

```js
// 应用层
import { html, Framework } from "./framework.js"

class App extends Framework {
  constructor() {
    super({ count: 0 })

    setInterval(() => {
      this.setState({
        count: this.state.count + 1,
      })
    }, 1000)
  }

  template() {
    return html`<div>${this.state.count}</div>`
  }
}

new App()

```

这种是现实路只关注模板中的**变量部分**， 在内存中存储了一份当前界面使用的状态数据缓存，以及**该数据关联的DOM节点位置**
每次数据更新时，遍历一遍缓存数据，有更新的话，就定向的更新这个状态数据关联的那个节点信息


## `Virtual DOM`
```js
// 框架层

/**
* vnode 数据结构设计
*
* {
*   type: string; // 节点类型
*   attrs?: Record<string, string>; // 属性
*   children?: VNode[]; // 子节点
*   value?: string; // 节点值
*    _el?: Node; // 对应的真实dom对象
* }
*/

export class Framework {
  state = {}
  _node = undefined

  constructor(initSatte) {
    this.state = initSatte
    // 初始化渲染
    this._diff(this.template())
  }

  /**
  * 子实例必须要实现 否则会报错
  */
  template() {
    throw new Error("must be implated")
  }

  setState(newState) {
    this.state = newState
    // 数据更新时重新渲染
    this._diff(this.template())
  }

  _diff(newNode) {
    diff(this._node, newNode, document.getElementById("app"))
    this._node = newNode
  }
}

/**
* 渲染函数
* 返回vdom
*/
export function h(...args) {
  let node = null

  function item(value) {
    if (value == null) {
    } else if (typeof value === "string") {
      if (!node) {
        node = {
          type: value,
          attrs: {},
          children: [],
        }
      } else {
        node.children.push({
          type: "#text",
          value,
        })
      }
    } else if (
      typeof value === "number" ||
      typeof value === "boolean" ||
      value instanceof Date
    ) {
      node.children.push({
        type: "#text",
        value: value.toString(),
      })
    } else if (typeof value === "object") {
      if (value.type) {
        node.children.push(value)
      } else {
        for (var k in value) {
          node.attrs[k] = value[k]
        }
      }
    }
  }

  while (args.length) {
    item(args.shift())
  }

  return node
}

/**
* diff 过程
* 对比新旧vdom 找出其中需要更新的地方
* 对比主要是三个步骤
* 1. 确认新旧元素是否是相同类型，是的话复用，不是的话删除老的，新建新的
* 2. 在第一步复用基础上，对比属性是否有变化，有变化更新
* 3. 在前两步基础上，对比元素children是否有变化，有变化对每一个子元素重复上述步骤
*/

function diff(node, newNode, root) {
  let element

  if (!node || node.type !== newNode.type) {
    /**
    * 初次渲染 或者 新旧节点不是相同的类型
    * 无法复用，直接清理老节点
    * 创建新节点
    * */
    if (node && node._el) {
      node._el.remove()
    }

    element =
      newNode.type === "#text"
        ? document.createTextNode(newNode.value)
        : document.createElement(newNode.type)

    root.append(element)
  } else {
    /**
    * 新旧节点是相同类型，直接复用
    * 将老vdom上的真实dom对象赋值
    * */
    element = node._el
  }

  // 把新vdom与真实节点关联起来
  newNode._el = element

  // 至此确认了新节点的类型

  /**
  * 判断新节点是否是文本节点
  * 是的话直接更新文本内容
  * */
  if (newNode.type === "#text") {
    element.textContent = newNode.value
    return
  }

  /**
  * diff 属性变化
  * 之需要关注新的属性对象
  * 如果老的有并且值不一样就更新属性
  * */
  for (var k in newNode.attrs) {
    if (node.attrs[k] !== newNode.attrs[k]) {
      element.setAttribute(k, newNode.attrs[k])
    }
  }

  /**
 * diff 子元素
 * 直接递归调用
 * */
  if (newNode.children) {
    for (let i = 0; i < newNode.children.length; i++) {
      diff(node?.children?.[i], newNode?.children?.[i], element)
    }
  }
}

```

```js
// 应用层
import { h, Framework } from "./framework.js"

class App extends Framework {
  constructor() {
    super({ count: 0 })

    setInterval(() => {
      this.setState({ count: this.state.count + 1 })
    }, 1000)
  }

  template() {
    return h("div", this.state.count)
  }
}

new App()

```

Virtual-DOM方案是现在比较流行的一个解决方案
主要思路就是将DOM结构使用JS对象来描述，一个真实的DOM单元对应一个VNode
在数据更新时，新旧VNode进行diff对比，找出其中变化的部分并更新，复用不变的部分
因为每轮更新只是做了必要的最小化的DOM操作，所以效率相对较高。

