# animate

动画开始

[explain 文件中使用](./delegate.README.md#L208) | 
[文件中使用](./scrollreveal/src/instance/functions/delegate.js#42) | 
[源文件](./scrollreveal/src/instance/functions/animate.js)


---

`animate.call(this, element)`

---

代码 1-20

``` js
import clean from '../methods/clean'

export default function animate (element, force = {}) {
	const pristine = force.pristine || this.pristine // ready
	const delayed =
		element.config.useDelay === 'always' ||
		(element.config.useDelay === 'onload' && pristine) ||
		(element.config.useDelay === 'once' && !element.seen)
    // 执行-形式-选项

	const shouldReveal = element.visible && !element.revealed
	const shouldReset = !element.visible && element.revealed && element.config.reset

	if (shouldReveal || force.reveal) {
		return triggerReveal.call(this, element, delayed)
	}

	if (shouldReset || force.reset) {
		return triggerReset.call(this, element)
	}
}
```

- `this.pristine` 确认准备好-✅-动画

> [constructor.js](./scrollreveal/src/instance/constructor.js)

- `element.visible`

> [delegate.js](./scrollreveal/src/instance/functions/delegate.js#35)

