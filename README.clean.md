# clean 

衔接 [README.reveal.md](./README.reveal.md#L413)

`reveal` [函数-代码使用](./README.reveal.md#191)

## 作用在于清除 元素数组缓存

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
            // <------
		}
		return elementBuffer
		// 所以一般，return []
	}
}
// ...
```

---

## `clean.call(this,element)`

- this == reveal 函数

- element == target

[clean.js](./explain-scrollreveal/scrollreveal/src/instance/methods/clean.js)
``` js
import { each, getNodes } from 'tealight'
import { logger } from '../../utils/core'
import rinse from '../functions/rinse'

export default function clean (target) {
	let dirty
	try {
		each(getNodes(target), node => {
			const id = node.getAttribute('data-sr-id')
			if (id !== null) {
                dirty = true
                // this.store 是 全局缓存
                const element = this.store.elements[id]
                // 动画函数-执行-去除
				if (element.callbackTimer) {
					window.clearTimeout(element.callbackTimer.clock)
                }
                // 拆卸 id ，安装 原style
				node.setAttribute('style', element.styles.inline.generated)
                node.removeAttribute('data-sr-id')
                // 删除 目标元素 在 全局缓存
				delete this.store.elements[id]
			}
		})
	} catch (e) {
		return logger.call(this, 'Clean failed.', e.stack || e.message)
	}

	if (dirty) {
        // 残留数据
		try {
			rinse.call(this) //
		} catch (e) {
			return logger.call(this, 'Clean failed.', e.stack || e.message)
		}
	}
}

```

- element.callbackTimer 

> ?

- rinse.call(this)

> 冲洗内存
