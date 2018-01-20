# sequence

间隔运行 队列

[sequence.js](./scrollreveal/src/instance/functions/sequence.js)

---

- [小结](#小结)

---
当使用了

``` js

sr.reveal(".seq-1", block, 200) // 200 间隔时间
```

我们回到-`reveal.js`-从一开始-`interval==200`

``` js
	sequence = interval ? new Sequence(interval) : null
```

---

## Sequence-interval

- [Sequence(interval)](./scrollreveal/src/instance/functions/sequence.js#L69)

``` js
export function Sequence (interval) {
	if (typeof interval === 'number') {
		if (interval >= 16) {
			/**
			 * Instance details.
			 */
			this.id = nextUniqueId()
			this.interval = interval
			this.members = []

			/**
			 * Flow control for sequencing animations.
			 */
			this.headblocked = true
			this.footblocked = true

			/**
			 * The last successful member indexes,
			 * and a container for DOM models.
			 */
			this.lastReveal = null
			this.lastReset = null
			this.models = {}
		} else {
			throw new RangeError('Sequence interval must be at least 16ms.')
		}
	} else {
		return null
	}
}
```

从 `sequence = new Sequence(interval)`

``` js
//
// sequence : Object  ==
Sequence{
id : nextUniqueId()
footblocked:true
headblocked:true
interval:200
lastReset:null
lastReveal:null
members:[]
models:{}
}

```

---

## element-sequence

下一个关键点 [也是在-`reveal.js`](./scrollreveal/src/instance/methods/reveal.js#L106)

``` js
			if (sequence) { // 我们知道是个 Object 类型
				element.sequence = {
					id: sequence.id,
                    index: sequence.members.length, 
                    // 队列排列的顺序 

                    // 比如开始是 index == 0
				}
                sequence.members.push(element.id)
                // 这里-sequence.members.length == 1
			}
```

## store-sequence

全局缓存-`store`-关键点 [在-`reveal.js` 142](./scrollreveal/src/instance/methods/reveal.js#L142)

``` js
		if (sequence) {
			this.store.sequences[sequence.id] = sequence
		}
```

> 全局缓存-一个元素集`elements`-队列信息

## initialize

下一步关键点 [在-`reveal.js` 161](./scrollreveal/src/instance/methods/reveal.js#L161)

``` js
		this.initTimeout = window.setTimeout(initialize.bind(this), 0)

```

这个时候进入-[初始化阶段-initialize.js](./scrollreveal/src/instance/functions/initialize.js)

> 完成了 [`样式的置换`](./README.style.md) - `scroll`+`resize`事件的绑定

同时引出了，事件函数
[this.delegate](./scrollreveal/src/instance/functions/initialize.js#L21)

``` js
target.addEventListener('scroll', this.delegate)
target.addEventListener('resize', this.delegate)
```

## delegate

当进入到事件函数后, 

> 关于，偏移量的判定 - 元素集是否需要队列

[sequence | animate](./scrollreveal/src/instance/functions/delegate.js#38)

``` js
		each(elements, element => {
			if (element.sequence) {
				sequence.call(this, element) // 进入队列
			} else {
				animate.call(this, element) // 动画
			}
		})
```

> `sequence` <-- `import sequence from './sequence'`

---

## sequence-element

> [sequence.js](./scrollreveal/src/instance/functions/sequence.js) 主函数 

代码 5-12

``` js
export default function sequence (element, pristine = this.pristine) {
	const seq = this.store.sequences[element.sequence.id] // id
	const i = element.sequence.index // 顺序

	if (seq) { 
		const visible = new SequenceModel('visible', seq, this.store)
        const revealed = new SequenceModel('revealed', seq, this.store)
        seq.models = { visible, revealed }

```

- [SequenceModel](#sequencemodel)

> 找出 是否 visible 分为 3类

`new SequenceModel('visible', seq, this.store)`

``` js
visible = {
    head: [] // 不可见 :中: 比-可见-前-的顺序
    body: [] // 可见的顺序
    foot: [] // 不可见 :中: 比-可见-后-的顺序
}

```

---

## 第一可见元素-动画

代码 15-36

``` js
		/**
		 * If the sequence has no revealed members,
		 * then we reveal the first visible element
		 * within that sequence.
		 *
		 * The sequence then cues a recursive call
		 * in both directions.
		 */
		if (!revealed.body.length) { // 如果没有运行过
			const nextId = seq.members[visible.body[0]] // 可见的元素id
			const nextElement = this.store.elements[nextId] // 可见元素

			if (nextElement) {
				cue.call(this, seq, visible.body[0], -1, pristine) 
				cue.call(this, seq, visible.body[0], +1, pristine)
                // 剔除 nextElement
                seq.lastReveal = visible.body[0]
                // 历史记录
                return animate.call(this, nextElement, { reveal: true, pristine })
                // 动画效果
			} else {
				return animate.call(this, element)
			}
		}

```

- [cue](#cue)

> 引导-队列走向 -1 向上走 +1 向下走

---

## 是否重置

？？一些问题

代码 38-67

``` js
		/**
		 * Assuming we have something visible on screen
		 * already, and we need to evaluate the element
		 * that was passed in...
		 *
		 * We first check if the element should reset.
		 */

        // 通过上面我们有了，可见元素的动画了，那么

        // 我们首先检查元素是否应该重置。
		if (!element.visible && element.revealed && element.config.reset) {
			seq.lastReset = i
			return animate.call(this, element, { reset: true })
		}

```

---

## 方向约束

那么接下来

``` js
//seq
Sequence{
id : nextUniqueId()
footblocked:true
headblocked:true
interval:200
lastReset:null
lastReveal:null
members:[]
models:{}
}
```

``` js
		/**
		 * If our element isn’t resetting, we check the
		 * element sequence index against the head, and
		 * then the foot of the sequence.
		 */

        // 如果-上面-没有堵塞，并且，顺序来到
		if (!seq.headblocked && i === [...revealed.head].pop() && i >= [...visible.body].shift()) {
			cue.call(this, seq, i, -1, pristine)
			seq.lastReveal = i
			return animate.call(this, element, { reveal: true, pristine })
		}

        // 如果-下面-没有堵塞，并且，顺序来到
		if (!seq.footblocked && i === [...revealed.foot].shift() && i <= [...visible.body].pop()) {
			cue.call(this, seq, i, +1, pristine)
			seq.lastReveal = i
			return animate.call(this, element, { reveal: true, pristine })
		}
	}
}
```



---


## SequenceModel

``` js
		const visible = new SequenceModel('visible', seq, this.store) // 找出 是否可见，分为3类
        const revealed = new SequenceModel('revealed', seq, this.store) // 找出 是否 reveal 分为 3类
```

好了我们有了，输入变量了

``` js

export function SequenceModel (prop, sequence, store) {
	this.head = [] // Elements before the body with a falsey prop.
	this.body = [] // Elements with a truthy prop.
	this.foot = [] // Elements after the body with a falsey prop.

	each(sequence.members, (id, index) => {
		const element = store.elements[id] // 从全局缓存中-获取元素缓存
		if (element && element[prop]) { // 元素-可见
			this.body.push(index) // 顺序
		}
	})

	if (this.body.length) {
		each(sequence.members, (id, index) => {
			const element = store.elements[id]
			if (element && !element[prop]) { // 元素-不可见
				if (index < this.body[0]) {
					this.head.push(index) // 顺序在 body 前
				} else {
					this.foot.push(index) // 顺序在 body 后
				}
			}
		})
	}
}
```

>

---
## cue

``` js
    cue.call(this, seq, visible.body[0], -1, pristine) 
    // --- 1
    cue.call(this, seq, visible.body[0], +1, pristine) 
    // --- 2
```

``` js
function cue (seq, i, direction, pristine) {
    const blocked = ['headblocked', null, 'footblocked'][1 + direction]
    // 第一个 
    const nextId = seq.members[i + direction]
    // direction == -1
    // nextId 其实是 上一个
    // -------------
    // direction == +1
    // nextId 就是 下一个
	const nextElement = this.store.elements[nextId]

	seq[blocked] = true

	setTimeout(() => {
		seq[blocked] = false // 
		if (nextElement) { //
			sequence.call(this, nextElement, pristine)
		}
	}, seq.interval)
}

```

- `blocked = headblocked = false`

> 说明: 队列走向可以向 上, 上面没有堵塞  

- `blocked = footblocked = false`

> 说明: 队列走向可以向 下, 下面没有堵塞

- `seq[blocked[0|2]] = false `

说明: 队列开始运作

---

## 小结

队列的走向

交给了 [cue-函数](#cue) 

[从-第一个可见元素-动画](#第一可见元素-动画) 用 `cue函数`

分流-两个方向-一`上`一`下`

``` js 
// -1 为 上
    cue.call(this, seq, visible.body[0], -1, pristine) 
    cue.call(this, seq, visible.body[0], +1, pristine)
// +1 为 下
```

[我们首先检查元素是否应该重置。](#是否重置)

然后

[方向约束](#方向约束)

---