# initalize

[initalize.js](./scrollreveal/src/instance/functions/initialize.js)

``` js
import { each } from 'tealight'
import rinse from './rinse'

export default function initialize () {
	rinse.call(this) // scrollreveal 函数示例

	each(this.store.elements, element => {
		let styles = [element.styles.inline.generated]
		
		// 使用 通过 `functions/style` 函数 计算并缓存的值，赋予 目标元素-style
		if (element.visible) {
			styles.push(element.styles.opacity.computed)
			styles.push(element.styles.transform.generated.final)
		} else {
			styles.push(element.styles.opacity.generated)
			styles.push(element.styles.transform.generated.initial)
		}

		element.node.setAttribute('style', styles.filter(s => s !== '').join(' '))
	})

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

	/**
	 * Wipe any existing `setTimeout` now
	 * that initialization has completed.
	 */
	this.initTimeout = null
}

```

- rinse.call(this)

> [冲洗内存](./rinse.README.md)

- [`element.styles` 的获得,在 reveal.js](./scrollreveal/src/instance/methods/reveal.js#L104)

``` js
	element.styles = style(element)
```

- [`this.store.containers` 的获得, 在 reveal.js]((./scrollreveal/src/instance/methods/reveal.js#L137))

- [this.delegate 的获得](./scrollreveal/src/instance/constructor.js#L89)

``` js
	Object.defineProperty(this, 'delegate', { get: () => delegate.bind(this) })

```

> [`this.delegate` 的 代码 explain](./delegate.README.md)

- [this.initTimeout 的获取与消除](./scrollreveal/src/instance/methods/reveal.js#L158)

----

