# reveal

ok,我们进入 ScrollReveal 主要使用函数 reveal

[scrollreveal/src/method/reveal](./scrollreveal/src/instance/methods/reveal.js)

``` js
export default function reveal (target, options, interval, sync)
```

- [target](##target-interval)

- [options](#目标元素数组获取)

- [interval](#target-interval)

- [sync](#同步设置-sync)

---
 
让我们，提示一下我们的例子

``` html
<!-- HTML -->
<div class="foo"> Foo </div>
<div class="bar"> Bar </div>
```

``` js
// JavaScript
window.sr = ScrollReveal();
sr.reveal('.foo'); //<-- target == .foo
sr.reveal('.bar'); //<-- target == .bar
```

---

explain 本目录

- [reveal - 主顺序](#reveal)

- [工具库-tealight](#工具库-tealight)

- [其他工具函数](#工具函数-style)

---

### reveal

- [目标元素数组-获取函数](#目标元素数组获取)

- [目标缓存-elements](#目标缓存-elements)

- [容器缓存-containerBuffer](#容器缓存-containerBuffer)

- [同步设置-sync](#同步设置-sync)

---

[scrollreveal/src/method/reveal](./scrollreveal/src/instance/methods/reveal.js#L14)

代码 14-24

``` js
	/**
	 * The reveal method has optional 2nd and 3rd parameters,
	 * so we first explicitly check what was passed in.
	 */
	if (typeof options === 'number') {
		interval = parseInt(options)
		options = {}
	} else {
		interval = parseInt(interval)
		options = options || {}
	}
```

`规范 第二 和 第三 变量 `

---

### target-interval

代码 26-37

``` js
	/**
	 * To start things off, build element collection,
	 * and attempt to instantiate a new sequence.
	 */
	let nodes
	let sequence
	try {
		nodes = getNodes(target) // <--
		sequence = interval ? new Sequence(interval) : null // <--
	} catch (e) {
		return logger.call(this, 'Reveal failed.', e.stack || e.message)
	}
```

``` js
import { deepAssign, each, getNode, getNodes } from 'tealight'
```

> tealight 是 作者 写的工具函数库

- [getNodes](#工具库(getNodes))

> 在本例子中 ， interval is `undefined` 

> 所以 sequence = null

`or` 你可以看看发生了什么

- [`Sequence`](#第三个变量(interval))

> `import { Sequence } from '../functions/sequence'`

next

---

### 目标元素数组获取

代码 39-64

``` js
	/**
	 * Begin element set-up...
	 */

    // nodes 是 Array 类型



	try {
		const elements = nodes.reduce((elementBuffer, elementNode) => {
			const element = {}
			const existingId = elementNode.getAttribute('data-sr-id')

            if (existingId) { // <-- 是否存在 data-sr-id
            // 存在data-sr-id 
				deepAssign(element, this.store.elements[existingId])

				/**
				 * In order to prevent previously generated styles
				 * from throwing off the new styles, the style tag
				 * has to be reverted to it's pre-reveal state.
				 */
				element.node.setAttribute('style', element.styles.inline.computed)
			} else {
            //  没有data-sr-id 
            //  在本例子 是 没有 运行到这里代码
				element.id = nextUniqueId() // <-- 0
				//纯函数-每运行一次，返回 +1
                // 0,1，2，3，4...
				element.node = elementNode // 附上元素
				element.seen = false 
				element.revealed = false
				element.visible = false
            }
        	const config = deepAssign({}, element.config || this.defaults, options)

			// ...
			
        ,[]) //<-- reduce函数 初始值 
```

>  `reduce() `方法对累加器和数组中的每个元素（从左到右）应用一个函数，将其减少为单个值。

- [reduce()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)

- [nextUniqueId()]()

``` js
export const nextUniqueId = (() => {
	let uid = 0
	return () => uid++
})() //<-- 注意这个 立刻运行
```

- [deepAssign()](#工具库(deepAssign))

> 就像 Object.assign

``` js
Object.assign({},{1:'ddd'},{a:'d',1:'d'})
// {1: "d", a: "d"}
```

next 

---

代码 66 - 116 在上面的 `reduce 函数内`

``` js
/**
 * Verify the current device passes our platform configuration, // 确认 设备平台
 * and cache the result for the rest of the loop.
 * // 缓存 循环的结果
 */
let disabled
{
	if (disabled == null) {
		// 是否 在移动端 或者 桌面端 中的一个就行
		disabled = (!config.mobile && isMobile()) || (!config.desktop && !isMobile())
	}
	if (disabled) { // 无法识别 平台 返回
	// 一般来说这一步，在第一次 reduce 函数 就会识别出来
		if (existingId) {
			clean.call(this, element) // 清除-缓存数组
		}
		return elementBuffer
		// 所以一般，return []
	}
}

const containerNode = getNode(config.container)//document.body

let containerId
{
	if (!containerNode) {
		throw new Error('Invalid container.')
	}
	if (!containerNode.contains(elementNode)) {
		// 
		// config.container 不包含 getNodes(target)[index]
		// 过滤 getNodes(target)[index]

		return elementBuffer // skip elements found outside the container
	}

	containerId = getContainerId(containerNode, containerBuffer, this.store.containers)
	// 从 args[1]打后的变量中找寻，相等containerNode
	// 获取 {id:id,node:containerNode} 
	//  return id

	// - containerBuffer 本地缓存
	// - this.store 	 全局缓存

	if (containerId == null) {
		containerId = nextUniqueId() // <-- 1
		containerBuffer.push({ id: containerId, node: containerNode })
	}
}

element.config = config 
element.containerId = containerId
element.styles = style(element)

if (sequence) { // <-- null
	element.sequence = {
		id: sequence.id,
		index: sequence.members.length,
	}
	sequence.members.push(element.id)
}

elementBuffer.push(element) // 元素缓存数组
return elementBuffer
}, [])
```

- [clean](#工具函数-clean)

> `import clean from '../methods/clean'`

- [getNode](#工具库(getNode))

- [getContainerId](#工具函数(getcontainerid))

- [this.store](./scrollreveal/src/instance/constructor.js#L80)

> 由 `总-scrollreveal` 定义的

``` js
this.store = {
	containers: {},
	elements: {},
	history: [],
	sequences: {},
}
```

- [style](#工具函数-style)

`import style from '../functions/style'`

next

---

代码 118-129

### 目标缓存-elements

``` js
		/**
		 * Modifying the DOM via setAttribute needs to be handled
		 * separately from reading computed styles in the map above
		 * for the browser to batch DOM changes (limiting reflows)
		 */
		each(elements, element => {
			// 全局缓存导入
			this.store.elements[element.id] = element
			// 赋值 每个 用到的 element 
			// <element data-sr-id='id'></element>
			element.node.setAttribute('data-sr-id', element.id)
		})
```

next

---

代码 131-145

### 容器缓存-containerBuffer

``` js
	/**
	 * Now that element set-up is complete...
	 * Let’s commit any container and sequence data we have to the store.
	 */
	// 不要忘记了，元素的爸爸或妈妈，what ever
	{
		each(containerBuffer, container => {
			this.store.containers[container.id] = {
				id: container.id,
				node: container.node,
			}
		})
		if (sequence) {
			this.store.sequences[sequence.id] = sequence
		}
	}
```

next

---

代码 147-163

### 同步设置-sync

``` js
	/**
	 * If reveal wasn't invoked by sync, we want to
	 * make sure to add this call to the history.
	 */
	// 默认异步
	if (!sync) {
		this.store.history.push({ target, options, interval })

		/**
		 * Push initialization to the event queue, giving
		 * multiple reveal calls time to be interpreted.
		 */
		if (this.initTimeout) {
			window.clearTimeout(this.initTimeout)
		}
		this.initTimeout = window.setTimeout(initialize.bind(this), 0)
	}

```

- [initalize-初始化](#工具函数-initalize)

> `import initialize from '../functions/initialize'`

---

总结: `reveal` 函数所需要做得

- 确定用户定义-`config`-对应-用户定义目标元素

- 获取 `目标元素` 真实节点和数据，缓存

- 确定 数据 唯一性

- 获取 `容器元素`，缓存

- 把获得的-缓存-放入全局缓存

---

## 第三个变量-interval

可以用来定义，动画表现的时间

---

### 工具函数-getContainerId

``` js
function getContainerId (node, ...collections) {
	let id = null
	each(collections, collection => {
		each(collection, container => {
			if (id === null && container.node === node) {
				id = container.id
			}
		})
	})
	return id
}
```

- [each](#工具库-each)

---

### 工具函数-style

[style.js](./scrollreveal/src/instance/functions/style.js)

[explain--style](./README.style.md)

---

### 工具函数-clean

[clean.js](./scrollreveal/src/instance/methods/clean.js)

[explain--clean](./README.clean.md)

---


### 工具函数-initalize

[initalize.js](./scrollreveal/src/instance/functions/initalize.js)

[explain--initalize](./README.initalize.md)

---


## 工具库-tealight

因 在本 解释 scrollreveal 源码章节 中

作者 修改了 tealight 的代码 , 去掉了许多内容，

> 代码总是要最新最好的，工具库-tealight 的解释 先暂缓

``` js
import { deepAssign, each, getNode, getNodes } from 'tealight'
```

---

### 工具库-getNodes

``` js
import isNode from './is-node'
import isNodeList from './is-node-list'
export default function getNodes(target) {
	if (target instanceof Array) return target
	if (isNode(target)) return [target]
	if (isNodeList(target)) return Array.prototype.slice.call(target)
	if (typeof target === 'string') {
		try {
			const query = document.querySelectorAll(target)
			return Array.prototype.slice.call(query)
		} catch (err) {
			return []
		}
	}
	return []
}
```

---

### 工具库-deepAssign

``` js


```
---

### 工具库-each

