### 简单拖曳实现两种方式




### 方法一：
使用trag api:
https://developer.mozilla.org/zh-CN/docs/Web/API/HTML_Drag_and_Drop_API
[简单的drag示例](https://jsbin.com/coqiwotaru/1/edit?html,js,output)

```html
<div id="app">
  <div id="test" draggable="true"></div>
</div>
```
```js
let test = document.querySelector('#test')
let startPosition;
let endPosition;

test.addEventListener('dragstart', (e)=>{
  test.classList.add('hide')
  let {screenX: x, screenY: y} = e
  startPosition = [x, y]
})

test.addEventListener('dragend', (e)=>{
  test.classList.add('hide')
  let {screenX: x, screenY: y} = e
  endPosition = [x, y]
  let deltaX = endPosition[0] - startPosition[0]
  let deltaY = endPosition[1] - startPosition[1]
  // 注意通过style.left获取到的是内联样式，所以初始为空，parseInt会出现NaN
  // test.style.left = parseInt(test.style.left) + deltaX + 'px'
  // test.style.top = parseInt(test.style.top) + deltaY + 'px'

  // 使用getComputedStyle，不管内联还是非内联样式都能拿到
  let {left, top} = window.getComputedStyle(test)
  test.style.left = parseInt(left) + deltaX + 'px'
  test.style.top = parseInt(top) + deltaY + 'px'
})
```

#### 移动过程中隐藏原来的
```js
test.addEventListener('dragstart', (e)=>{
  test.classList.add('hide')
  let {screenX: x, screenY: y} = e
  startPosition = [x, y]
  setTimeout(()=>{
    test.style.visibility = 'hidden'
  })
})

test.addEventListener('dragend', (e)=>{
  let {screenX: x, screenY: y} = e
  endPosition = [x, y]
  let deltaX = endPosition[0] - startPosition[0]
  let deltaY = endPosition[1] - startPosition[1]
  // 注意通过style.left获取到的是内联样式，所以初始为空，parseInt会出现NaN
  // test.style.left = parseInt(test.style.left) + deltaX + 'px'
  // test.style.top = parseInt(test.style.top) + deltaY + 'px'

  // 使用getComputedStyle，不管内联还是非内联样式都能拿到
  let {left, top} = window.getComputedStyle(test)
  test.style.left = parseInt(left) + deltaX + 'px'
  test.style.top = parseInt(top) + deltaY + 'px'
  test.classList.remove('hide')
  test.style.visibility = 'visible'
})
```


### 方法二：
使用mouseenter事件：
https://jsbin.com/huhexapegu/1/edit?html,js,output
```js
let test = document.querySelector('#test')
let startPosition;
let endPosition;
let top;
let left;

// 只有mousedown时才去判断为move
let isMoving = false

test.addEventListener('mousedown', (e)=>{
  isMoving = true
  let {screenX: x, screenY: y} = e
  startPosition = [x, y]
  
  // 和drag不同，move时去获取left top时并不是初始的位置，所以要在这里获取
  // drag时，本身的位置left top是不会变化的
  top = window.getComputedStyle(test).top
  left = window.getComputedStyle(test).left
})

test.addEventListener('mousemove', (e) => {
  if(!isMoving) {return;}
  console.log(1)
  let {screenX: x, screenY: y} = e
  endPosition = [x, y]
  let deltaX = endPosition[0] - startPosition[0]
  let deltaY = endPosition[1] - startPosition[1]
  
  test.style.left = parseInt(left) + deltaX + 'px'
  test.style.top = parseInt(top) + deltaY + 'px'
})

test.addEventListener('mouseup', (e)=>{
  isMoving = false
  let {screenX: x, screenY: y} = e
})
```

上面的方式虽然达到了移动效果，但是移动过快的话会丢失目标。

解决办法： 改为监听document

