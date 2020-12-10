# sticky组件
### 1. 如何获取元素至页面顶部的距离（含滚动的距离）？ 
![](1sticky组件1.png)
```
let distance =  window.scrollY + el.getBoundingClientRect().top
```