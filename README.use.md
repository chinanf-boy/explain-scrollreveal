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