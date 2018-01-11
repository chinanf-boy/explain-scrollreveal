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
    // 延迟执行-形式-选项

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

> [位于-constructor.js](./scrollreveal/src/instance/constructor.js)

- delayed

``` 
`always` - 延迟显示所有动画`
`once` - 只延迟第一次显示发生`
'onload' - 只对第一次加载的动画延迟
```

- `element.visible`

> [位于-delegate.js](./scrollreveal/src/instance/functions/delegate.js#35)

``` js
element.visible = isElementVisible.call(this, element)
```

- [triggerReveal](#triggerreveal)

>  根据[作者在 README.md 的描述](https://github.com/jlmakes/scrollreveal#42-improve-user-experience)

``` html
在大多数情况下，您的元素将从opacity：0开始，所以它们可以淡入。

但是，由于JavaScript在页面开始渲染之后加载，因此您可能会在元素被

ScrollReveal的JavaScript隐藏之前开始渲染时看到闪烁。

理想的解决方案是将您的揭密元素的可见性设置为隐藏在页面的<head>中，

以确保在您的JavaScript加载时呈现为隐藏：
```

> 所以一开始，元素最好 visibility == hidden

``` css
      .sr .fooReveal { visibility: hidden; }
```

> 

- [triggerReset](#triggerreset)

> 重置动画

---

## triggerReveal

``` js
function triggerReveal (element, delayed) {
	// 动画运行
	const styles = [
		element.styles.inline.generated,
		element.styles.opacity.computed,
		element.styles.transform.generated.final, //最终动画
	]
		
	if (delayed) { // 延迟
	// 通过 style.js 计算 得到的
		styles.push(element.styles.transition.generated.delayed)
	} else {
	// 通过 style.js 计算 得到的		
		styles.push(element.styles.transition.generated.instant)
	}
	element.revealed = element.seen = true
	element.node.setAttribute('style', styles.filter(s => s !== '').join(' '))
	registerCallbacks.call(this, element, delayed)
}
```

- [registerCallbacks](#registercallbacks)

---

## triggerReset

``` js
function triggerReset (element) {
	// 重置
	const styles = [
		element.styles.inline.generated,
		element.styles.opacity.generated,
		element.styles.transform.generated.initial,//初始动画
		element.styles.transition.generated.instant,
	]
	element.revealed = false
	element.node.setAttribute('style', styles.filter(s => s !== '').join(' '))
	registerCallbacks.call(this, element)
}
```

- [registerCallbacks](#registercallbacks)

---

## registerCallbacks

``` js

function registerCallbacks (element, isDelayed) {
	// 是否延迟了
	const duration = isDelayed
		? element.config.duration + element.config.delay
		: element.config.duration

	// 动画前函数
	const beforeCallback = element.revealed
		? element.config.beforeReveal
		: element.config.beforeReset

	// 动画后函数
	const afterCallback = element.revealed ? element.config.afterReveal : element.config.afterReset

	// 两次触发，获取上次时间点，
	let elapsed = 0
	if (element.callbackTimer) {
		elapsed = Date.now() - element.callbackTimer.start
		window.clearTimeout(element.callbackTimer.clock)
	}


	beforeCallback(element.node)

	// 定义 定时函数
	element.callbackTimer = {
		start: Date.now(),
		clock: window.setTimeout(() => {
			afterCallback(element.node)
			element.callbackTimer = null // 清除
			if (element.revealed && !element.config.reset) {
				// 不需要，重复动画时，说明此 目标元素 完成任务了
				// 清除相关内容
				clean.call(this, element.node)
			}
		}, duration - elapsed),
	}
}

```

动画-触发-注册函数，通过 `setTimeout` 控制

- `duration`

> 延迟时间

- `beforeCallback` `afterCallback`

``` js
	afterReset () {},
	afterReveal () {},
	beforeReset () {},
	beforeReveal () {},
```