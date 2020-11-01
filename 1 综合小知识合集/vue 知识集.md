### 1. 如何判断dom元素内部的子元素标签？
```
<div class="parent" ref="parent">
  <span class="child">儿子</span>
</div>


let childrenEl = this.$refs['parent'].children
for (let node of childrenEl) {           // 注意这里用的是let of因为children是数组元素
  console.log(node.nodeName.toLowerCase())   // span
}

```

如果要判断组件内部的子元素标签：
```
for(let node of this.$el.children) {}
```

**实例应用可以看group-button组件**

> 综上我们可以知道: 
> 1. 获取组件实例的元素可以使用 `this.$el`
> 2. 获取dom元素 可以使用 `this.$ref.xxx`

### 2. 如何判断父组件内部的有哪些子组件？
步骤： 
1. 在每个子组件中定义好 `name`
2. 在父组件中通过 `this.$children` 直接获取子组件
```
this.$children.forEach((vm)=>{
  if(vm.$options.name === 'xxxx'){
    doth...
  }
})
```
![1vue的dom和vm](./1vue的dom和vm.png)
> 其实`this、  this.$parent、 this.$children`都是`vm`（实例），
> - 可以通过`vm`获取组件实例`data`和方法
> - 可以通过`vm.$el`获取dom元素


### 3. vue component组件的prop类型验证
![2](./2prop类型验证.png)
```
props: {
  name: {
    type: String|Number,  
    // 或者
    // type: [String, Number]
  }
}
```

### 4. vue 插槽

#### 4.1 插槽内容
插槽的作用，顾名思义就是在父组件中将某些内容插入到子组件的特定位置上（一不小心就开车了呢）。
```
// 子组件
<div class="child">
  <header>
    <slot name="header></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>

// 父组件
<child>
  // 将这里插入到子组件的 <slot name="header></slot>插槽里
  <template #header>
    <h1>header</h1>
  </template>

  // 没有写插槽的默认 是 v-slot:default 会默认放到子组件的 <slot></slot> 里
  <p>default-content</p>

  <template v-slot:footer>
    <p>footer</p>
  </template>
</child>
```
**以上`v-slot:` 可以简写为`#`**


#### 4.2 作用域插槽

作用域插槽即在子组件的slot标签上，绑上特定属性，然后在父组件上使用 slotProps来获取到这个属性
```
// 子组件
<div class="child">
  <header>
    // 绑定在 <slot> 元素上的 attribute 被称为插槽 prop, 比如这里将user作为属性传给插槽
    <slot name="header :user="user></slot>
  </header>
</div>

// 父组件
<child>
  // 有时候我们想要拿到子组件中传过来的内容，可以使用 slotProps
  <template #header="slotProps">
    <h1>header {{slotProps.user}}</h1>
  </template>

  // 可以用解构语法
  <template #header="{ user }">
    <h1>header {{ user }}</h1>
  </template>
</child>
```

#### 4.3 slot特性
```
<div class="child">

  // 绑定的class是无效的
  <slot class="xxx"></slot>

  // 使用 $refs['xxx'] 也是无法获取到的
  <slot ref="xxx"></slot>

  // 事件xxx也不会执行
  <slot @click="xxx"></slot>

  // 介于如此可以在slot上绑上一个div元素, 这里的xxx都会生效
  <div ref="xxx" @click="xxx" class="xxx">
    <slot></slot>
  </div>

  // v-if是有效的 
  <slot v-if="false"></slot>
</div>
```

### 5. 组件prop传递规则
我们给一个组件定义prop时：
![3prop](3prop规则.png)

在父组件中:
![3prop](3prop规则2.png)

**如果父组件中使用 `popoverClassName`传递prop, 在子组件中将会是`undefined`**

这是因为在html中 attribute 的名是大小写不敏感的，所以浏览器会把所有大写字符解释为小写字符。

![3prop](3prop规则3.png)


### 6. 自定义指令
**指令就是为了操纵dom的。**

[自定义指令案例](https://jsbin.com/geboqabuye/edit?html,js,console,output)

![](7vue自定义指令.png)

如果给指令传入函数，直接 `binding.value()` 就能执行该函数，`binding.value`即绑定的值。





