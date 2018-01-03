# style

[style.js](./scrollreveal/src/instance/functions/style.js)

---

> 被使用在 [reveal.js](./scrollreveal/src/instance/methods/reveal.js#L104)

> 主要是用来获得 `element`-->`style`
## 引用库

代码 1-3
``` js
import { parse, multiply, rotateX, rotateY, rotateZ, scale, translateX, translateY } from 'rematrix'

import { getPrefixedStyleProperty } from '../../utils/browser'
```

- `rematrix`

- `../../utils/browser`

---

本目录

- [生成-inline-style](#生成-inline-style)

- [生成-opacity-style](#生成-opacity-style)

- [生成-transformation-style](#生成-transformation-style)

- [生成-transition-style](#生成-transition-style)

- [返回-结果](#获取结果)

---
## style


> style(element)

- `element` 不是一个节点, `element.node` 才是

- `element` 是一个 node 对象缓存

``` js
            // 在 reveal.js 中的定义
            element.id = nextUniqueId()
            
            element.node = elementNode
            
            element.seen = false
            
            element.revealed = false
            
            element.visible = false			
            
            element.config = config
			
            element.containerId = containerId
			
            element.styles = style(element)
            //...
```        

## 生成-inline-style

代码 5-34

``` js
export default function style (element) {
	const computed = window.getComputedStyle(element.node)
	const position = computed.position
	const config = element.config

	/**
	 * Generate inline styles
	 */
	const inline = {}
	const inlineStyle = element.node.getAttribute('style') || ''
	const inlineMatch = inlineStyle.match(/[\w-]+\s*:\s*[^;]+\s*/gi) || []

	inline.computed = inlineMatch ? inlineMatch.map(m => m.trim()).join('; ') + ';' : ''

	// /visibility\s?:\s?visible/i)
	// find - 'visibility: visible'
	inline.generated = inlineMatch.some(m => m.match(/visibility\s?:\s?visible/i))
		? inline.computed
		: [...inlineMatch, 'visibility: visible'].map(m => m.trim()).join('; ') + ';'
```

---

## 生成-opacity-style

``` js

	/**
	 * Generate opacity styles
	 */
	const computedOpacity = parseFloat(computed.opacity)
	
	// config.opacity --> defaults --> 0
	const configOpacity = !isNaN(parseFloat(config.opacity))
		? parseFloat(config.opacity)
		: parseFloat(computed.opacity)
	// 
	const opacity = {
		computed: computedOpacity !== configOpacity ? `opacity: ${computedOpacity};` : '',
		generated: computedOpacity !== configOpacity ? `opacity: ${configOpacity};` : '',
	}
```

- window.getComputedStyle

- [element.config ](.README.use.md#)

- [/[\w-]+\s*:\s*[^;]+\s*/gi](http://refiddle.com/nr1i)

- [`visibility: visible`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/visibility)

- !isNaN

``` js
!isNaN(1)
// true
!isNaN('123')
// true
!isNaN('1222dd3')
// false
```

- `opacity` 两种情况

	- 一：用户输入正确情况下，

		`opacity.computed == computedOpacity`,

		`opacity.generated == configOpacity`,

		若是 用户输入的，和 元素本身 opacity 一致

		opacity.computed == opacity.generated == ''
		

	
	- 二：用户输入不正确情况下

		`computedOpacity == configOpacity`

		opacity.computed == opacity.generated == ''

next

---

## 生成-transformation-style

代码 36-138

- [config.distance](./README.use.md#L11) 默认为 '0'


``` js
	/**
	 * Generate transformation styles
	 */
	const transformations = []
	//  偏移
	if (parseFloat(config.distance)) {
		const axis = config.origin === 'top' || config.origin === 'bottom' ? 'Y' : 'X'

		/**
		 * Let’s make sure our our pixel distances are negative for top and left.
		 * e.g. { origin: 'top', distance: '25px' } starts at `top: -25px` in CSS.
    	 */

		let distance = config.distance
		if (config.origin === 'top' || config.origin === 'left') {
			// distance 为 负，进来这里
			distance = /^-/.test(distance) ? distance.substr(1) : `-${distance}`
		}



		const [value, unit] = distance.match(/(^-?\d+\.?\d?)|(em$|px$|%$)/g)

		// 三种格式 em,px,%
		switch (unit) {
			case 'em':
				distance = parseInt(computed.fontSize) * value
				break
			case 'px':
				distance = value
				break
			case '%':
				/**
				 * Here we use `getBoundingClientRect` instead of
				 * the existing data attached to `element.geometry`
				 * because only the former includes any transformations
				 * current applied to the element.
				 *
				 * If that behavior ends up being unintuitive, this
				 * logic could instead utilize `element.geometry.height`
				 * and `element.geoemetry.width` for the distaince calculation
				 */
				distance =
					axis === 'Y'
						? element.node.getBoundingClientRect().height * value / 100
						: element.node.getBoundingClientRect().width * value / 100
				break
			default:
				throw new RangeError('Unrecognized or missing distance unit.')
		}

		if (axis === 'Y') {
			transformations.push(translateY(distance))
		} else {
			transformations.push(translateX(distance))
		}
	}

``` 

- [translateY](#工具函数-translatey)

- [translateX](#工具函数-translatex)

``` js
	if (config.rotate.x) transformations.push(rotateX(config.rotate.x)) //<--
	if (config.rotate.y) transformations.push(rotateY(config.rotate.y)) //<-->
	if (config.rotate.z) transformations.push(rotateZ(config.rotate.z)) //<-->
	if (config.scale !== 1) {
		if (config.scale === 0) {
			/**
			 * The CSS Transforms matrix interpolation specification
			 * basically disallows transitions of non-invertible
			 * matrixes, which means browsers won't transition
			 * elements with zero scale.
			 *
			 * That’s inconvenient for the API and developer
			 * experience, so we simply nudge their value
			 * slightly above zero; this allows browsers
			 * to transition our element as expected.
			 *
			 * `0.0002` was the smallest number
			 * that performed across browsers.
			 */
			transformations.push(scale(0.0002))// <-->
		} else {
			transformations.push(scale(config.scale))
		}
	}

	const transform = {}
	if (transformations.length) {
		transform.property = getPrefixedStyleProperty('transform')
		/**
		* The default computed transform value should be one of:
		* undefined || 'none' || 'matrix()' || 'matrix3d()'
		*/
		transform.computed = {
			raw: computed[transform.property],
			matrix: parse(computed[transform.property]), //<-->
		}

		transformations.unshift(transform.computed.matrix)
		const product = transformations.reduce(multiply)

		transform.generated = {
			initial: `${transform.property}: matrix3d(${product.join(', ')});`,
			final: `${transform.property}: matrix3d(${transform.computed.matrix.join(', ')});`,
		}
	} else {
		transform.generated = {
			initial: '',
			final: '',
		}
	}
```

- [rotateX](#工具函数-rotatex)

- [rotateY](#工具函数-rotatey)

- [rotateZ](#工具函数-rotatez)

- [multiply](#工具函数-multiply)

- [parse](#工具函数-parse)

- [getPrefixedStyleProperty](#getprefixedstyleproperty)

- [transfrom playground](https://developer.mozilla.org/en-US/docs/Web/CSS/transform)

> CSS transform 属性允许你修改CSS视觉格式模型的坐标空间。

>使用它，元素可以被转换`（translate）`、`旋转（rotate）`、缩放（scale）`、倾斜（skew）`。 

> ❗️CSS transform 属性 , 只对 block 级元素生效！

next

---

## 生成-transition-style

``` js
	/**
	 * Generate transition styles
	 */
	let transition = {}
	if (opacity.generated || transform.generated.initial) {
		transition.property = getPrefixedStyleProperty('transition')
		transition.computed = computed[transition.property]
		transition.fragments = []

		const { delay, duration, easing } = config

		if (opacity.generated) {
			transition.fragments.push({
				delayed: `opacity ${duration / 1000}s ${easing} ${delay / 1000}s`,
				instant: `opacity ${duration / 1000}s ${easing} 0s`,
			})
		}

		if (transform.generated.initial) {
			transition.fragments.push({
				delayed: `${transform.property} ${duration / 1000}s ${easing} ${delay / 1000}s`,
				instant: `${transform.property} ${duration / 1000}s ${easing} 0s`,
			})
		}

		/**
		 * The default computed transition property should be one of:
		 * undefined || '' || 'all 0s ease 0s' || 'all 0s 0s cubic-bezier()'
		 */
		if (transition.computed && !transition.computed.match(/all 0s/)) {
			transition.fragments.unshift({
				delayed: transition.computed,
				instant: transition.computed,
			})
		}

		const composed = transition.fragments.reduce(
			(composition, fragment, i) => {
				composition.delayed += i === 0 ? fragment.delayed : `, ${fragment.delayed}`
				composition.instant += i === 0 ? fragment.instant : `, ${fragment.instant}`
				return composition
			},
			{
				delayed: '',
				instant: '',
			}
		)

		transition.generated = {
			delayed: `${transition.property}: ${composed.delayed};`,
			instant: `${transition.property}: ${composed.instant};`,
		}
	} else {
		transition.generated = {
			delayed: '',
			instant: '',
		}
	}

	return { // <-- 获取结果
		inline,
		opacity,
		position,
		transform,
		transition,
	}
}

```

- [transition 说明](https://developer.mozilla.org/zh-CN/docs/Web/CSS/transition)

> transition CSS 属性是一个简写属性，

> 用于 `transition-property`, `transition-duration`, `transition-timing-function`, 和 `transition-delay。` 

> ❗️CSS transform 属性 , 只对 block 级元素生效！ 

---

> CSS transitions 提供了一种在更改CSS属性时控制动画速度的方法。 其可以让属性变化成为一个持续一段时间的过程，而不是立即生效的。

> 比如，将一个元素的颜色从白色改为黑色，通常这个改变是立即生效的，使用 CSS transitions 后该元素的颜色将逐渐从白色变为黑色，按照一定的曲线速率变化。

> 这个过程可以自定义。

- [transition 例子说明](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Transitions/Using_CSS_transitions)

next

---

## 获取结果 

``` js
	return { // <-- 获取结果
		inline,
		opacity,
		position,
		transform,
		transition,
	}
```

---

其他

## getPrefixedStyleProperty 

[browser.js](./scrollreveal/src/utils/browser.js)

> 获取 style 属性 前缀 ：-webkit- ..etc...

``` js
export const getPrefixedStyleProperty = (() => {
	let properties = {}
	const style = document.documentElement.style

	function getPrefixedStyleProperty (name, source = style) {
		if (name && typeof name === 'string') {
			if (properties[name]) {
				return properties[name]
			}
			if (typeof source[name] === 'string') {
				return (properties[name] = name)
			}
			if (typeof source[`-webkit-${name}`] === 'string') {
				return (properties[name] = `-webkit-${name}`)
			}
			throw new RangeError(`Unable to find "${name}" style property.`)
		}
		throw new TypeError('Expected a string.')
	}

	getPrefixedStyleProperty.clearCache = () => (properties = {})

	return getPrefixedStyleProperty
})()
```

## 工具库-rematrix

> 作者说：如果我们要写 `transform: rotateZ(45deg)`，我们可以使用

> `Rematrix`在`JavaScript`中创建相同的转换，如下所示：

---

``` js
Rematrix.rotateZ(45)
```

==

``` css
transform: rotateZ(45deg)
```

---

[github](https://github.com/jlmakes/rematrix)

[jsfiddle 实际例子](https://jsfiddle.net/jL4vnh08/)

``` js
import { parse, multiply, rotateX, rotateY, rotateZ, scale, translateX, translateY } from 'rematrix'
```

### 工具函数-parse

### 工具函数-multiply

### 工具函数-rotateX

### 工具函数-rotateY

### 工具函数-rotateZ

### 工具函数-scale

### 工具函数-translateX

### 工具函数-translateY

---

详细 [rematrix explain](https://github.com/chinanf-boy/explain-rematrix)


