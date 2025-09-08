1. Vue框架要解决的问题
2. vdom
3. vdom+编译器 优化
4. vapor mode
5. vapor mode应用场景

## 现代视图框架要解决的问题

在Web开发中，现代视图框架众多，Vue、React、Solid等，这些框架有不同的设计思想，但是它们要解决的问题都是一样的： Web开发中**状态与UI的同步问题**。

比如如下一个计数器和操作历史的功能
![[Pasted image 20250906205239.png]]


在传统的开发中，需要的代码如下：
```html
<div class="approach-content">
	<div class="counter-display" id="counterValue">0</div>
    <div class="buttons">
	    <button class="decrement" onclick="decrement()">-1</button>
        <button class="reset" onclick="reset()">重置</button>
        <button class="increment" onclick="increment()">+1</button>
        </div>
    <div class="history">
        <h3>操作历史</h3>
        <div class="history-list" id="historyList"></div>
    </div>
</div>
```

```js
let count = 0;
let history = [];

// 更新UI的函数
function updateUI() {
    const counterEl = document.getElementById('counterValue');
    counterEl.textContent = count;
    
    // 手动更新样式
    counterEl.className = 'counter-display';
    if (count > 0) counterEl.classList.add('positive');
    if (count < 0) counterEl.classList.add('negative');
    
    // 手动更新历史
    const historyEl = document.getElementById('historyList');
    historyEl.innerHTML = history.map(item => 
        `&lt;div class="history-item"&gt;${item}&lt;/div&gt;`
    ).join('');
}

// 业务逻辑
function increment() {
    count++;
    history.push(`增加 -> ${count}`);
    if (history.length > 5) history.shift();
    updateUI(); // 需要手动更新UI
}

function decrement() {
    count--;
    history.push(`减少 -> ${count}`);
    if (history.length > 5) history.shift();
    updateUI(); // 需要手动更新UI
}

function reset() {
    count = 0;
    history.push(`重置 -> ${count}`);
    if (history.length > 5) history.shift();
    updateUI(); // 需要手动更新UI
}
```

而在框架的加持下，以Vue为例：
```html
<div id="app" class="approach-content">
    <!-- 声明式UI：只描述UI应该是什么样子 -->
    <div class="counter-display" 
      :class="{ positive: count > 0, negative: count < 0 }"
    >
        {{ count }}
    </div>
    <div class="buttons">
        <button class="decrement" @click="decrement">-1</button>
        <button class="reset" @click="reset">重置</button>
        <button class="increment" @click="increment">+1</button>
    </div>
    <div class="history">
        <h3>操作历史</h3>
        <div class="history-list">
            <div class="history-item" 
               v-for="(item, index) in history" 
               :key="index"
            >
                {{ item }}
            </div>
        </div>
    </div>
</div>
```

```js
// 响应式状态
const count = ref(0);
const history = ref([]);

// 业务逻辑
const increment = () => {
  count.value++;
  history.value.push(`增加 -> ${count.value}`);
  if (history.value.length > 5) history.value.shift();
  // 不需要手动更新UI，Vue会自动处理
};

const decrement = () => {
  count.value--;
  history.value.push(`减少 -> ${count.value}`);
  if (history.value.length > 5) history.value.shift();
};

const reset = () => {
  count.value = 0;
  history.value.push(`重置 -> ${count.value}`);
  if (history.value.length > 5) history.value.shift();
};
```

对比两者的实现，可以发现有两个关键的不同
1. 在点击事件之后，传统开发需要手动去操作dom更新，而Vue只需要更新数据
2. 在【操作历史】这个模块，Vue直接**声明**了此处的dom结构，而传统开发需要开发者使用dom api去手动去填充

这两个现象其实就对应现代开发中的两个常见概念：**数据驱动** + **声明式编程**。

所谓**数据驱动** 就是开发者更偏重于状态的维护，以数据的状态或结构来主导程序的控制流，代码是围绕分析和处理数据来组织的。 比如上述Vue代码中，用户点击事件的响应逻辑就是在更新数据。

而**声明式编程**  在前端范畴内，可以理解为声明式描述视图，描述网页界面的期望结果，而不是命令式的在js逻辑中操纵整个视图的变化过程。 比如上述【操作历史】模块，在数据之前即声明了这个模块是一系列操作历史的呈现。

就是这两个概念指导下，现代框架通过各种形式，实现了状态和UI的自动同步，而不需要开发者手动操作。


## VDOM
那了解了框架的目标，以及指导思想， 那具体如何去实现呢？
Vue的选择是virtual dom 即VDOM
所谓VDOM， 简单来说就是用JS对象来描述DOM结构。 这个JS对象内存储有DOM渲染所需要的所有信息，比如属性、样式、事件绑定以及子元素等。

比如上述代码片段中按钮区域对应的vdom示意如下：

```js
const vnode = {
  tag: 'div',
  props: { class: 'buttons' },
  children: [
    {
      tag: 'button',
      props: {
        class: 'decrement',
        onClick: decrement // 这里decrement是函数引用
      },
      children: '-1'
    },
    {
      tag: 'button',
      props: {
        class: 'reset',
        onClick: reset // reset函数引用
      },
      children: '重置'
    },
    {
      tag: 'button',
      props: {
        class: 'increment',
        onClick: increment // increment函数引用
      },
      children: '+1'
    }
  ]
}
```

有了VDOM之后，页面结构信息就以数据（状态）的形式存在了，
可以将VDOM数据转换成真实的DOM结构并挂载在指定DOM节点中。

```js
render(vdom, container) 

// 渲染vdom到真实dom节点
function render() {

}
```

在引入VDOM之后，以SFC模式为例，Vue页面的基本流程如下：
1. 开发过程中，编写sfc vue组件
2. 打包编译过程中，vue-loader将模板转换成渲染函数fn，编译完成之后产物主要内容就是包含多个渲染函数的js代码
3. 页面加载时，执行对应模块的渲染函数，生成vdom
4. 根据vdom渲染出真实dom结构，同时在运行时缓存对应的vdom
5. 页面需要更新时，先产生一个新页面结构的vdom
6. 
7. 再对比新旧vdom，找到两者最小化差异
8. 调用真实dom api，更新页面


**VDOM带来的优点**
1. 提高了**页面dom更新**的效率， 注意不是dom操作的效率，是dom更新的效率
	1. 因为通过dom diff 找到了最小化的dom操作集合，减少了中间态无用的dom操作
	2. 但并不是提高了真实dom操作的效率，这个效率提升是浏览器本身要做的优化


**对应的缺点**
1. 首屏渲染效率问题，dom结构都是从vdom转换来的，在首评的时候会有更多的白屏时间
2. 额外的运行时内存，因为始终在内存中维护一份页面结构对应的vdom数据
3. 复杂页面更新问题，dom diff以组件为单位，组件内任何细小的更新，需要遍历整个组件的vdom，确认最终的更新点


> 需要注意的是，VDOM并不是声明式编程和数据驱动的唯一实现方式，
> VDOM只是其中的一个实现方案
> 除此之外，它还可以应用于跨平台渲染场景，dom结构所需要的数据全部在vdom中，完全可以调用不同的平台api，渲染出对应平台的界面


## Vue3针对VDOM的优化
针对上述VDOM的缺陷，Vue从编译层面做了相应的优化
因为Vue本身是掌控了编译流程，因此一些编译时可以确定的信息就可以传递到渲染层，，帮助渲染器更高效的渲染

比如组件内的静态节点，可以跳过diff流程，直接复用
又比如，整个diff过程是，对于节点的比较，是先比较节点自身的的属性、事件等看是否有更新，再递归比较节点的children。编译时其实已经可以确定当前节点具体哪些信息是动态的可能变化的，那就可以给节点加上flag信息，在diff的过程中直接去确认对应可能变化的信息，而跳过不会变化部分的对比，一次提高diff效率

总的来说，vue做了如下两个主要的优化

### Static Hoist
### Patch Flag



## Vapor Mode
一种不使用vdom的编译模式

编译之后，直接将模板渲染成真实node节点，
并且建立响应式数据与相关node的映射关系
在数据变更时，直接调用真实DOM API 更新node，实现细粒度更新

不需要diff操作，在找到需要的dom更新行为


更小的包体积
更高效的dom更新
更少的运行时内存占用


应用场景：
1. 大组件
2. 极致的性能要求


