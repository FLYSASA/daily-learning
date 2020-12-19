# scroll组件

缓动函数速查表：
https://www.xuanfengge.com/easeing/easeing/

#### 1.  使用drag api （无法实现，放弃）
scroll纵向拖动时，要保证只能Y方向的移动，使用drag api的话是无法限制在Y方向移动的, 即使给了left 0的限制，依然不影响其drag，只是最后出现回拉。

#### 2. 使用 mouseenter、 mousedown、 mouseup 事件
限制 x 方向偏移值始终为0即可。


#### 3. selectstart 事件
阻止默认文本选中。
```js
@selectstart="selectstart"

methods: {
  selectstart(e) {
    e.prevent()
  }
}
```