# delegate

事件，函数

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

---

``` js
export default function delegate (event = { type: 'init' }, elements = this.store.elements) {
	raf(() => {
		const stale = event.type === 'init' || event.type === 'resize'

		each(this.store.containers, container => {
			if (stale) {
				container.geometry = getGeometry.call(this, container, true)
			}
			const scroll = getScrolled.call(this, container)
			if (container.scroll) {
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
				sequence.call(this, element)
			} else {
				animate.call(this, element)
			}
		})

		this.pristine = false
	})
}
```