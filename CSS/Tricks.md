
`content-visibility: auto`

1. 跳过非可视区域的渲染，元素在视口之外时，浏览器会跳过这些元素的渲染，减少不必要的计算和渲染，提高页面性能
2. 提升初始渲染性能
3. 减少内存使用

一般用在`列表渲染`场景，提升网页的初始渲染性能和整体流畅度



`Flex`

`flex-grow`
设置的是flex容器的`剩余空间`分配规则
并不会影响元素本来就占据的空间
如果flex容器的没有剩余空间，那么这个设置就没有用

```html
<div class="wrap">
	<div class="left"></div>
	<div class="right">
		<div class="right-content"></div>
	</div>
</div>
```

```css
.wrap {
	display: flex;
	width: 300px;
}
.left{
	flex: 0 0 100px;
}
.right {
	flex: 1
}
```

这时候，right 容器的宽度，由right-content元素 和 flex 规则共同决定

如果right-content 宽度 大于200， 那么会撑开right，导致right的宽度也超过200；
这时候flex容器wrap无剩余空间可分配，所以right的宽度就是内容的宽度

如果right-content 宽度 为150， 那么首先right的宽度也是150；
此时flex容器wrap有50剩余空间，全部分给right，
所以最终right宽度为200



当right-content 宽度 大于200； 最终导致right元素的的宽度也大于200，超出了flex容器的边界？

**为什么？**

flex：1 代表 
flex-grow： 1 
flex-shrink：1

为什么没有压缩呢？


因为css规范，默认给 **flex容器的子元素（非滚动元素）** 设置了`min-width: auto`
意味着flex容器的子元素宽度不能小于内容的宽度，所以flex-shrink没有生效

可以通过覆盖设置`min-width: 0` 解决这个问题

同时，对于滚动元素，`min-width: 0`;  因此对应另外一种解决方案：
给flex容器的子元素设置**overflow**不为**visible**的其他值

