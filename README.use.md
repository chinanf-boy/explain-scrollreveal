# scrollreveal 

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