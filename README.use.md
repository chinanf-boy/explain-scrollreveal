# scrollreveal 

- [使用](#使用)

- [默认配置](#defau)
## 使用

``` js
let block = {
        origin   : "top", // 位置
        distance : "24px", // 偏移
        duration : 1500, // 速度
        scale    : 1.05, // 从多大开始
        delay    : 1000, // 发生时间
        reset    : true, // 是否重来动画, 重新出现在视窗时
        viewOffset : { top: 64px;} // 视窗偏移量，默认为 0, 比如-快划出 -视窗-60px，消失动画，超过元素 60px,开始动画 + delay time
      }
      
      //使用
      sr.reveal(".demo .block", block)
```

``` js
let two = {
            reset    : true, // 是否重来动画, 重新出现在视窗时
        viewOffset : { top: 64px;} // 视窗偏移量，默认为 0, 比如-快划出 -视窗-60px，消失动画，超过元素 60px,开始动画 + delay time
      }
    //
      sr.reveal(".seq-1", two, 200) // 200 => delay time
```

## defaults

[defaults.js](./scrollreveal/src/instance/defaults.js)

``` js
export default {
	delay: 0,
	distance: '0',
	duration: 600,
	easing: 'cubic-bezier(0.6, 0.2, 0.1, 1)',
	opacity: 0,
	origin: 'bottom',
	rotate: {
		x: 0,
		y: 0,
		z: 0,
	},
	scale: 1,
	container: document.documentElement,
	desktop: true,
	mobile: true,
	reset: false,
	useDelay: 'always',
	viewFactor: 0.0,
	viewOffset: {
		top: 0,
		right: 0,
		bottom: 0,
		left: 0,
	},
	afterReset () {},
	afterReveal () {},
	beforeReset () {},
	beforeReveal () {},
}
```

``` js
//'bottom', 'left', 'top', 'right'
origin: 'bottom',

// Can be any valid CSS distance, e.g. '5rem', '10%', '20vw', etc.
distance: '20px',

// Time in milliseconds.
duration: 500,
delay: 0,

// Starting angles in degrees, will transition from these values to 0 in all axes.
rotate: { x: 0, y: 0, z: 0 },

// Starting opacity value, before transitioning to the computed opacity.
opacity: 0,

// Starting scale value, will transition from this value to 1
scale: 0.9,

// Accepts any valid CSS easing, e.g. 'ease', 'ease-in-out', 'linear', etc.
easing: 'cubic-bezier(0.6, 0.2, 0.1, 1)',

// `<html>` is the default reveal container. You can pass either:
// DOM Node, e.g. document.querySelector('.fooContainer')
// Selector, e.g. '.fooContainer'
container: window.document.documentElement,

// true/false to control reveal animations on mobile.
mobile: true,

// true:  reveals occur every time elements become visible
// false: reveals occur once as elements become visible
reset: false,

// 'always' — delay for all reveal animations
// 'once'   — delay only the first time reveals occur
// 'onload' - delay only for animations triggered by first load
useDelay: 'always',

// Change when an element is considered in the viewport. The default value
// of 0.20 means 20% of an element must be visible for its reveal to occur.
viewFactor: 0.2,

// Pixel values that alter the container boundaries.
// e.g. Set `{ top: 48 }`, if you have a 48px tall fixed toolbar.
// --
// Visual Aid: https://scrollrevealjs.org/assets/viewoffset.png
viewOffset: { top: 0, right: 0, bottom: 0, left: 0 },

// Callbacks that fire for each triggered element reveal, and reset.
beforeReveal: function (domEl) {},
beforeReset: function (domEl) {},

// Callbacks that fire for each completed element reveal, and reset.
afterReveal: function (domEl) {},
afterReset: function (domEl) {}
```