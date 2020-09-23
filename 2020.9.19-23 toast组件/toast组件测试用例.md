### 1. 验证 autoClose 属性
```
describe('props', function(){
  // 因为是异步，所以加上done
  it('接收 autoClose', (done)=>{
    // 因为要验证body里是否还含有$el, 所以需要先挂载到div上
    let div = document.createElement('div')
    document.body.appendChild(div)
    // 常规套路
    const Constructor = Vue.extend(Toast)
    const vm = new Constructor({
      // 一定要记得包裹在 propsData 里
      propsData: {        
        autoClose: 3000
      }
    }).$mount(div)
    vm.$on('close', ()=>{
      // 判断node里是否含有某个元素 用 node.contains($el)
      expect(document.body.contains(vm.$el).to.eq(false))
      done();
    })
  })
})
```

### 2. 验证 closeButton
```
// 判断close div里的 文字是不是等于传入的
it('接收 closeButton ', ()=>{
  const callback = sinon.fake()
  const Constructor = Vue.extend(Toast)
  const vm = new Constructor({
    propsData: {
      closeButton: {
        text: '关闭了',
        callback,
      }
    }
  }).$mount()
  let closeButton = vm.$el.querySelector('.close')
  // $el.textContent 获取文字内容
  expect(closeButton.textContent.trim()).to.eq('关闭了')
  closeButton.click()
  expect(callback).to.have.been.called
})
```

### 3. 验证 enableHtml
```
it('接收 enableHtml ', ()=>{
  const Constructor = Vue.extend(Toast)
  const vm = new Constructor({
    propsData: {
      enableHtml: true
    }
  })
  // 加入默认slot内容这一步需要在mount之前
  vm.$slots.default = '<strong id="strongtest">hi<strong>'
  vm.$mount()
  let strong = vm.$el.querySelector('#strongtest')
  expect(strong).to.exist
})
```

### 3. 验证 position
it('接收 position ', ()=>{
  const Constructor = Vue.extend(Toast)
  const vm = new Constructor({
    propsData: {
      position: 'bottom'
    }
  }).$mount()
  // 判断元素的class里是否包含某个class
  expect(vm.$el.classList.contains('position-bottom').to.eq(true))
})
