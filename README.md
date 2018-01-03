# scrollreveal

[![explain](http://llever.com/explain.svg)](https://github.com/chinanf-boy/Source-Explain)

[github库](https://github.com/jlmakes/scrollreveal)

[https://scrollrevealjs.org/](https://scrollrevealjs.org/)

简单的网页和移动浏览器的滚动动画。

[使用例子](./README.use.md)

---

本目录

- [ScrollReveal-constructor](#正主(constructor.js))

- [下一节-reveal](./README.reveal.md)

其他

- [验证支持](#验证支持)

- [修葺平台](#修葺平台)

- [debug](#debug)

- [logger 错误日志](#logger)

- [noop 错误返回](#noop)

---

本次例子

``` html
<!-- HTML -->
<div class="foo"> Foo </div>
<div class="bar"> Bar </div>
```

``` js
// JavaScript
window.sr = ScrollReveal();
sr.reveal('.foo');
sr.reveal('.bar');
```

---

找正主

[package.json](./scrollreveal/package.json)

`"main": "dist/scrollreveal.js",`

[构建工具 rollup.config.js](./scrollreveal/rollup.conf.js)

`input: 'src/index.js',`

[scrollreveal/src/index.js](scrollreveal/src/index.js)

``` js
import Constructor from './instance/constructor'

Constructor()

export default Constructor
```

[scrollreveal/src/instance/constructor](scrollreveal/src/instance/constructor#L21)

``` js
export default function ScrollReveal (options = {})
```

ok ,正主找到了

---

 explain

## 正主(constructor.js)

[scrollreveal/src/instance/constructor](scrollreveal/src/instance/constructor)

代码 1- 15 引用

``` js
import defaults from './defaults'
import noop from './noop'

import clean from './methods/clean'
import destroy from './methods/destroy'
import reveal from './methods/reveal'
import sync from './methods/sync'

import delegate from './functions/delegate'

import { isMobile, transformSupported, transitionSupported } from '../utils/browser'
import { logger } from '../utils/core'
import { deepAssign, getNode } from 'tealight'

import { version } from '../../package.json'
```

next

代码 17-19

`let _config
let _debug
let _instance`

next

代码 21-32

``` js
export default function ScrollReveal (options = {}) {
    // ------ 「
    // 有没有用 new 新建这个实例
	const invokedWithoutNew =
		typeof this === 'undefined' || Object.getPrototypeOf(this) !== ScrollReveal.prototype

    // 如果没有
	if (invokedWithoutNew) {
		return new ScrollReveal(options)
	}
//------- 」

    // 是否支持，这个后面再聊 ？
	if (!ScrollReveal.isSupported()) {
		logger.call(this, 'Instantiation aborted.', 'This browser is not supported.')
		return noop
    }
```

-->现在就要知道[ScrollReveal.isSupported](#验证支持(isSupported))

next

代码 34-60

配置config

``` js
	/**
	 * Here we use `buffer` to validate our configuration, before
	 * assigning the contents to the private variable `_config`.
	 */
    /* 
    我们使用 buffer 来 序列化我们的配置，私有变量获得
    _config
    */

	let buffer
	{
		try {
            // 深度复制 第一次 _config flase
            buffer = _config ? deepAssign({}, _config, options) : deepAssign({}, defaults, options)
            
            //所以 buffer = deepAssign({}, defaults, options) 
            // 但是第二次时候就会使用
            // buffer = deepAssign({}, _config, options) 
            
		} catch (e) {
            // 失败记录
			logger.call(this, 'Instantiation failed.', 'Invalid configuration.', e.message)
			return noop
		}

		try {
            // 获得节点元素
			const container = getNode(buffer.container)
			if (!container) {
				throw new Error('Invalid container.')
			}
		} catch (e) {
            // 失败记录
			logger.call(this, 'Instantiation failed.', e.message)
			return noop
		}

		_config = buffer // 
	}

    // 通过 ScrollReveal.defaults 
    // 获得 _config
    Object.defineProperty(this, 'defaults', { get: () => _config })
```

next

代码 62-78

检查平台信息

``` js
	/**
	 * Now that we have our configuration, we can
	 * make our last check for disabled platforms.
	 */
    // 修葺 不能运作的平台 , 具体原因，未知
	if (this.defaults.mobile === isMobile() || this.defaults.desktop === !isMobile()) {
		/**
		 * Modify the DOM to reflect successful instantiation.
		 */
		document.documentElement.classList.add('sr')
		if (document.body) {
			document.body.style.height = '100%'
		} else {
			document.addEventListener('DOMContentLoaded', () => {
				document.body.style.height = '100%'
			})
		}
	}


```

[修葺原因](#修葺平台)

next

代码 80-99

整顿-函数

``` js

	this.store = {
		containers: {},
		elements: {},
		history: [],
		sequences: {},
	}

	this.pristine = true

    // 整顿，记得我们要使用例子的，reveal 函数
	Object.defineProperty(this, 'delegate', { get: () => delegate.bind(this) })
    Object.defineProperty(this, 'destroy', { get: () => destroy.bind(this) })
    
	Object.defineProperty(this, 'reveal', { get: () => reveal.bind(this) }) // <---
	Object.defineProperty(this, 'clean', { get: () => clean.bind(this) })
	Object.defineProperty(this, 'sync', { get: () => sync.bind(this) })

	Object.defineProperty(this, 'version', { get: () => version })
	Object.defineProperty(this, 'noop', { get: () => false })

    // 返回本身 
	return _instance ? _instance : (_instance = this)
}

```

next

我们需要进入 reveal 函数

`import reveal from './methods/reveal'
`

[GO reveal](./README.reveal.md)


---

## 验证支持(isSupported)

``` js
/**
 * Static members are available immediately during instantiation,
 * so debugging and browser support details are handled here.
 */
ScrollReveal.isSupported = () => transformSupported() && transitionSupported()

```

`import { isMobile, transformSupported, transitionSupported } from '../utils/browser'
`

- transformSupported

[scrollreveal/src/util/browser.js](./scrollreveal/src/utils/browser.js#L29)

``` js
export function transformSupported () {
	const style = document.documentElement.style
	return 'transform' in style || 'WebkitTransform' in style
}
```

- transitionSupported

``` js
export function transitionSupported () {
	const style = document.documentElement.style
	return 'transition' in style || 'WebkitTransition' in style
}
```

## 修葺平台

先验证函数

- isMobile

`import { isMobile, transformSupported, transitionSupported } from '../utils/browser'
`

[scrollreveal/src/util/browser.js](./scrollreveal/src/utils/browser.js#L26)

``` js
export function isMobile (agent = navigator.userAgent) {
	return /Android|iPhone|iPad|iPod/i.test(agent)
}
```
## debug

``` js
Object.defineProperty(ScrollReveal, 'debug', {
	get: () => _debug || false,
	set: value => {
		if (typeof value === 'boolean') _debug = value
	},
})
```

## logger

在 代码中如此使用

`logger.call(this, 'Instantiation aborted.', 'This browser is not supported.')`

``` js
export function logger (message, ...details) {
	if (this.constructor.debug && console) {
		let report = `%cScrollReveal: ${message}`
		details.forEach(detail => (report += `\n — ${detail}`))
		console.log(report, 'color: #ea654b;') // eslint-disable-line no-console
	}
}

```

## noop

``` js
export default {
	clean () {},
	destroy () {},
	reveal () {},
	sync () {},
	get noop () {
		return true
	},
}
```
---

## 总结 

[jsbin--](http://jsbin.com/mofazez/edit?html,js,console)

`scrollreveal` 通过 - 传递变量

``` js
// 实例化
new function scrollreveal(){
	this.store // 可以看成全局变量 
	Object.defineProperty(this, 'delegate', { get: () => delegate.bind(this) }) // 传递全局变量
}

```

---