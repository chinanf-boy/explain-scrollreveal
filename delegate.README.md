# delegate

`resize`, `scroll` 事件-绑定函数

[绑定事件代码-initalize.js](./scrollreveal/src/instance/functions/initialize.js#L21)

``` js

// 包裹 目标元素 的 容器元素， 添加事件 scroll 划动, resize 窗口变化 
each(this.store.containers, container => {
	const target = container.node === document.documentElement ? window : container.node
	target.addEventListener('scroll', this.delegate)
	target.addEventListener('resize', this.delegate)
})

/**
 * Manually invoke delegate once to capture
 * element and container dimensions, container
 * scroll position, and trigger any valid reveals
 */
this.delegate()
```

---

explain

---

``` js
import animate from './animate'
import sequence from './sequence'

import { each, mathSign, raf } from 'tealight'
import { getGeometry, getScrolled, isElementVisible } from '../../utils/core'

``` 

- `animate`

> 动画实现

- `sequence`

> 动画顺序管理队列

- `each`, `mathSign`, `raf`

> 兼容性函数

---

文件代码

``` js
export default function delegate (event = { type: 'init' }, elements = this.store.elements) {
	raf(() => {
		// 当 触发事件 resize , init
		const stale = event.type === 'init' || event.type === 'resize'

		each(this.store.containers, container => {
			// 窗口变化
			if (stale) {
				container.geometry = getGeometry.call(this, container, true)
				// 如果 窗口大小重载
			}
			const scroll = getScrolled.call(this, container)
			// 滚动数值

			if (container.scroll) { 
				// 偏移量
				container.direction = {
					x: mathSign(scroll.left - container.scroll.left),
					y: mathSign(scroll.top - container.scroll.top),
				}
			}
			container.scroll = scroll
		})

		/**
		 * Due to how the sequencer is implemented, it’s
		 * important that we update the state of all
		 * elements, before any animation logic is
		 * evaluated (in the second loop below).
		 */
		each(elements, element => {
			if (stale) {
				element.geometry = getGeometry.call(this, element)
			}
			element.visible = isElementVisible.call(this, element)
		})

		each(elements, element => {
			if (element.sequence) { 
				// 元素是否需要间隔运行
				sequence.call(this, element)
			} else {
				// 立刻动画
				animate.call(this, element)
			}
		})

		this.pristine = false
	})
}
```

- [this.pristine](./scrollreveal/src/instance/constructor.js#L87)

- `getGeometry`,`getScrolled`,`isElementVisible`

	- [`getGeometry`](#getgeometry)

	> 获取绝对位置

	- [`getScrolled`](#getscrolled)

	> 获取划动

	- [`isElementVisible`](#iselementvisible)

	> 元素是否 visible, 可使用

---

工具函数解释

---

[core.js](./scrollreveal/src/utils/core.js)

## getgeometry 

``` js
export function getGeometry (target, isContainer) {
	/**
	 * We want to ignore padding and scrollbars for container elements.
	 * More information here: https://goo.gl/vOZpbz
	 */
	const height = isContainer ? target.node.clientHeight : target.node.offsetHeight
	const width = isContainer ? target.node.clientWidth : target.node.offsetWidth

	let offsetTop = 0
	let offsetLeft = 0
	let node = target.node

	do {
		if (!isNaN(node.offsetTop)) {
			offsetTop += node.offsetTop
		}
		if (!isNaN(node.offsetLeft)) {
			offsetLeft += node.offsetLeft
		}
		node = node.offsetParent
	} while (node) // 遍历元素 父节点，获取 绝对位置

	return {
		bounds: {
			top: offsetTop, // 顶边 离 顶点 距离
			right: offsetLeft + width,// 右边 离 左边界 距离
			bottom: offsetTop + height, // 底边 离 顶点 距离
			left: offsetLeft, //左边 离 左边界 距离
		},
		height,
		width,
	}
}
```

> 遍历元素 父节点，获取 绝对位置 数据

---

## getscrolled

``` js
export function getScrolled (container) {
	// document.documentElement 是从 html 标签内所有
	// 一般 container.node 是 	container: document.documentElement
	return container.node === document.documentElement
		? {
			top: window.pageYOffset, // 距离顶点的距离
			left: window.pageXOffset, //  距离左边的距离
		}
		: {
			top: container.node.scrollTop,
			left: container.node.scrollLeft,
		}
}
```

> 获取 容器 滚动像素 值

- `default.js`-默认配置

`	container: document.documentElement,`

- [Element.scrollTop 属性可以获取或设置一个元素的内容垂直滚动的像素数](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/scrollTop)

- [Element.scrollLeft 属性可以读取或设置元素滚动条到元素左边的距离](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/scrollLeft)

---

## isElementVisible

``` js
export function isElementVisible (element) {
	const container = this.store.containers[element.containerId]
	// 一般来说: container === document.documentElement
	const viewFactor = Math.max(0, Math.min(1, element.config.viewFactor))
	// 0
	const viewOffset = element.config.viewOffset

	const elementBounds = {
		top: element.geometry.bounds.top + element.geometry.height * viewFactor,
		right: element.geometry.bounds.right - element.geometry.width * viewFactor,
		bottom: element.geometry.bounds.bottom - element.geometry.height * viewFactor,
		left: element.geometry.bounds.left + element.geometry.width * viewFactor,
	}

	const containerBounds = {
		top: container.geometry.bounds.top + container.scroll.top + viewOffset.top,
		right: container.geometry.bounds.right + container.scroll.left - viewOffset.right,
		bottom: container.geometry.bounds.bottom + container.scroll.top - viewOffset.bottom,
		left: container.geometry.bounds.left + container.scroll.left + viewOffset.left,
	}

	return (
		(elementBounds.top < containerBounds.bottom &&
			elementBounds.right > containerBounds.left &&
			elementBounds.bottom > containerBounds.top &&
			elementBounds.left < containerBounds.right) ||
		element.styles.position === 'fixed'
	)
}

```

- `viewFactor`-`viewOffset`

defualts.js
``` js
	viewFactor: 0.0,
	viewOffset: {
		top: 0,
		right: 0,
		bottom: 0,
		left: 0,
	},
```

- `elementBounds` `containerBounds`

> 目标元素 是否位于 容器 可见范围内

> 如果，目标元素 position:'fixed', 一定可见

---