### 分页器
分页器的展示，主要是控制一个数组, 数组元素可以简单分为 :
```js
[
  1, 
  this.currentPage - 1,   
  this.currentPage - 2, 
  this.currentPage,     // 当前页
  this.currentPage + 1, 
  this.currentPage + 2,
  this.totalPage        // 总页数
]
```

### 1. 将数组去重
```js
function unique(arr) {
  // 简单写法
  // return [...new Set(arr)]
  // 兼容性写法： 应用对象key唯一的特性
  let obj = {}
  arr.forEach((i)=>{
    obj[i] = true
  })
  // 注意这里返回的都是字符串，不再是number， 所以需要parseInt一下
  return Object.keys(obj).map(i => parseInt(i))

  // 以上不要偷懒写成
  // var arr = ['1', '2', '3']
  // return Object.keys(obj).map(parseInt)  // [1, NaN, NaN]
  // 相当于 parseInt(i, index, arr)  
  // parseInt('1', 0, arr)    // 转成0进制  0   进制是 2-36 之间
  // parseInt('2', 1, arr)    // 转成1进制  NaN
  // parseInt('3', 2, arr)    // 转成2进制  NaN
}
```