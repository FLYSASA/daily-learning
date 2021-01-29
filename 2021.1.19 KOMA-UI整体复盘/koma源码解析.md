# koma-ui
此篇文章主要用于koma-ui搭建过程中的项目复盘以及源码解析。

### 1. 初始化项目
```bash
git init 
git add .
git commit -m 'init'
git remote add origin 'xxx' // xxx为项目在github上的地址
git push origin master
```

### 2. 引入并使用vue
```
npm i vue  // 项目下安装vue
```

#### ① 直接引入
```js
// index.html
<body>
  <div id="app">
    <k-button>按钮</k-button>
  </div>
</body>
<script src="./node_modules/vue/dist/vue.min.js"></script>
<script src="./button.js"></script>
<script>
  new Vue({
    el: '#app'
  })
</script>

// button.js
Vue.component('k-button', {
  template: '<button class="koma-button">按钮</button>'
})
```
这样虽然可以直接使用vue，但是js引入的方式还是很不方便。

#### ② 使用构建工具使用vue
vue的独特之处是可以使用vue单文件的形式去写组件，这需要构建工具做打包编译处理，这里引入 **parcel**。
```
npm i parcel-bundler -D
```
```js
// 代码重构
// index.html
<body>
  <div id="app">
    <k-button>按钮</k-button>
  </div>
</body>
<script src="./src/app.js"></script>

// app.js
import Vue from 'vue';
import Button from 'button.vue'

Vue.component('k-button', Button)

new Vue({
  el: '#app'
})

// button.vue
<template>
  <div>
    <button class="koma-button">按钮</button>
  </div>
</template>
```

运行index.html:
```bash
// 命令行执行
./node_modules/.bin/parcel index.html
或者
npx parcel index.html --no-cache // --no-cache不走缓存
```
到这里就可以正常预览到我们写的按钮了。


---
### 3. Icon 组件
icon组件比较简单，主要是使用了 iconfont symbol引入的方式。
```
<svg class="k-icon">
  <use :xlink:href="`#i-${name}`"></use>
</svg> 
```

- 项目里并没有使用iconfont的在线链接，而是本地静态引入了svg内容，以保证项目的稳定性。
---
### 4. Button 组件
核心思路：
> button组件主要是使用了 `vue slot`。大部分是css相关的，使用`flex`布局, 根据`iconPosition`的改变，来调整元素的`order`。

代码层面上，在`k-button-group`中需要校验其子元素是否为 `button` 元素。
```js
// this.$el.children 是HTMLCollection元素集合，是一个数组
for( let node of this.$el.children) {
  if(node.NodeName.toLowerCase() !== 'button') {
    console.warn(`k-button-group 的子元素应该全是 k-button, 但是你写的是 ${ node.nodeName }`)
  }
}
```

**for...of 是用来遍历数组的（es6引入的新特性）， for...in是用来遍历对象的。**

---
### 5. Grid 组件
核心思路：
> row组件代表一行，采用最基础的24分栏。 内部的col组件通过设置span栏数，来决定自己的宽度。

grid组件的重要属性实现：
- row:
  + `gutter`: 栏之间的间隔。gutter 会由父传给子，然后决定子的左右`padding`值为`gutter`的一半，父的左右`margin`为`﹣1/2gutter`保证整体不会超出容器。
  + `align`: 对齐方式。row是以flex布局的，align的`left, right, center`对应`flex-start, flex-end, flex-center`。

- col:
  + `span`: col占据的栏数。这个决定了其宽度的百分比。
  + `offset`: col偏移多少栏。这个决定了当前栏左margin的宽度百分比。
  + `ipad、narrowPc、pc、widePc`: 这几个可以设置对应不同大小屏占据的分栏数。实现原理就是通过传入不同的属性，来追加不同的class。然后再在不同的class下写不同的样式。越大屏的尺寸样式应该越写在下面，为了不被其他尺寸样式覆盖。

  ```js
  // 大致代码思路
  // html
  <div class="koma-col" :class="colClass"></div>

  // js
  computed: {
    colClass() {
      const { span, offset, ipad, narrowPc, pc, widePc } = this
      return [
        this.createClass({span, offset}),
        this.createClass(ipad, 'ipad-'),
        this.createClass(narrowPc, 'narrow-pc-'),
        this.createClass(pc, 'pc-'),
        this.createClass(widePc, 'wide-pc-'),
      ]
    }
  },
  methods: {
    createClass(obj, str) {
      if(!obj) { return }
      let arr = []
      if(obj.span) {
        arr.push(`col-${str}${obj.span}`)
      }
      if(obj.offset) {
        arr.push(`offset-${str}${obj.offset}`)
      }
      return arr;
    },
  }
  
  // css
  @media (min-width: 577px) {
    .koma-col {
      @class: col-ipad-;
      .col-loop(@n) when (@n>0){
        &.@{class}@{n}{
            width: @n/24*100%;
        }
        .col-loop((@n)-1);
      }
      .col-loop(24);

      @offset: offset-ipad-;
      .offset-loop(@n) when (@n>0){
        &.@{offset}@{n}{
            margin-left: @n/24*100%;
        }
        .offset-loop((@n)-1);
      }
      .offset-loop(24)
    }
  }
  ```
---
### 6. Layout 组件
  核心思路：
> layout组件里如果有 `sider` 就直接`flex-direction: row`. 没有就是 `flex-direction: column`。

```js
<div class="koma-layout" :class="layoutClass">
  <slot></slot>
</div>

export default {
  data() {
    return {
      layoutClass: {
        hasSider: false
      }
    }
  },
  mounted () {
    // 关键点在于判断layout组件下有没有sider组件
    this.$children.forEach((vm) => {
      if(vm.$options.name === 'KomaSider') {
        this.layoutClass.hasSider = true
      }
    })
  }
}
```

---
### 7. Input组件
核心思路：
> input组件主要是实现 v-model 双向绑定，它其实就是一个语法糖，它利用了名为 `value` 的 `prop`，和名为 `input` 的事件。

```html
// v-model实现
<k-input v-model="value" ></k-input>

// 相当于
<k-input :value="value" @input="value=$event.target.value"></k-input>

// k-input
<input type="text"  :value="value" @input="$emit('input', $event.target.value)"><input>
```

另外一个小知识点是 `v-bind=$attrs` 的用法， 当我们子组件想继承父组件的属性时，但是又不想在props里挨个定义，就可以直接使用 $attrs , 包含了父作用域中不作为 prop 被识别 (且获取) 的 attribute 绑定 (class 和 style 除外)

另外一个类似的是 `v-on=$listeners"`, 包含父作用域中的不含 `.native` 修饰器的 `v-on` 事件监听器。
### 8. Toast组件
toast一般调用方法如： `this.$toast('我知道');`  看起来需要绑定在`vue.protoType` 上。但是直接绑定会有vue风险，如：
```
// 用户引入vue,  此时就应该绑定在 vue2.prototype上
import vue2  from Vue;
```

vue官方提供了插件的方法，支持你对Vue添加全局功能。示例如下：

```js
// 1.创建plugin.js
// plugin要做的事并不复杂，只需要导出一个install方法即可。
export default {
  install(Vue, options) {
    // 这里面就可以添加全局方法、全局指令、全局混入项等等
    Vue.protoType.$toast = function(message, toastOptions) {
      // dosth...
    }
  }
}


// 2. 使用插件
// 可以在一个公共js的地方引入
// common.js

import plugin from './plugin.js'
Vue.use(plugin);    // 用户自己使用Vue调用插件，避免了上面vue指向问题
```

核心思路：
> 1. toast要考虑的是其位置，这里主要在toast div上又包裹一层 wrapper，这个wrapper作用很简单就是 横向定位，这么做的目的在于，因为内部的toast的还有上下的动画效果，由于定位都是使用的transform，如果放在一个div上，会重置掉横向定位。
> 2. toast需要考虑唯一性，因为用户可能重复点击，会导致多个toast同时出现，所以就需要每次清除掉上次出现的toast。


```js
// plugin.js
import Toast from './components/toast';

let currentToast;
export default {
  install(Vue, options) {
    // 全局调用的实例方法
    Vue.prototype.$toast = function (message, toastOptions) {
      // 如果存在，就清掉
      if(currentToast) {
        currentToast.close();
      }
      currentToast = createToast({
        Vue,
        message,
        propsData: toastOptions,
        // 插入一个回调，监听toast组件的close事件，然后将currentToast清掉
        onClose: ()=>{
          currentToast = null;
        }
      })
    }
  }
}

function createToast ({ Vue, message, propsData, onClose}) {
  const Constructor = Vue.extend(Toast);
  const toast = new Constructor({
    propsData
  })
  toast.$slots.default = message;
  toast.$mount()
  toast.$on('close', onClose)
  document.body.appendChild(toast.$el);
  return toast;
}

// toast.vue
// 主要关注点在于点击关闭后的清除dom操作
close() {
  // 清除元素
  this.$el.remove();
  this,$emit('close');
  // 销毁实例
  this.$destroy();
}

```

### 9. Tabs组件
核心思路：
> 采用最经典的发布订阅模式，引入了eventBus，因为只要某一项被激活需要通知其他兄弟组件的此时的选中状态。

```js
// tabs.vue 注入eventBus
<div class="koma-tabs">
  <slot></slot>
</div>

export default {
  props: {
    selected: {
      type: String,
      required: true
    }
  }, 
  data() {
    return {
      eventBus: new Vue,
    }
  },
  provide() {
    return {
      eventBus: this.eventBus
    }
  },
  mounted() {
    this.eventBus && this.eventBus.$emit('update:selected', this.selected)
  }
}
```

```js
// tabs-head.vue
<template>
  <div ref="tabsHead" class="koma-tabs-head">
    <!-- 独立line出来是为了做动画 -->
    <div ref="activeLine" class="active-line"></div>
    <slot></slot>
  </div>
</template>

export default {
  inject: [ 'eventBus' ],
  mounted() {
    if(this.eventBus) {
      this.eventBus.$on('update:selected', (name)=>{
        this.$children.forEach((vm) => {
          if(vm.name === name) {
            const { width } = vm.$el.getBoundingClientRect()
            // 注意这里使用的是offsetLeft， 而不是left，因为line相对parent定位的所以获取相对parent的left值
            const left = vm.$el.offsetLeft
            this.$ref['activeLine'].style.width = width
            this.$ref['activeLine'].style.left = left
          }
        })
      })
    }
  },
}
```


```js
// tabs-item.vue 
// 负责发布 点击事件后的name
<template>
  <div class="koma-tabs-item" @click="onClick" :class="{ active }"></div>
</template>


export default {
  props: {
    name: {
      type: String | Number,
      required: true
    }
  },
  data() {
    return {
      active: new Vue,
    }
  },
  inject: [ 'eventBus' ],
  mounted() {
    if(this.evnentBus) {
      this.eventBus.$on('update:selected', (name) => {
        this.active = name === this.name
      })
    }
  },
  methods: {
    onclick() {
      this.eventBus && this.eventBus.$emit('update:selected', this.name)
    }
  }
}
```

### 10. Popover组件

核心思路：
> popover组件实现起来有点麻烦，但是核心思路还是很简单，在组件内部预备两个插槽，一个是trigger触发器（这也是默认插槽），另外一个是 content 内容插槽。通过监听触发器的click或hover事件，触发content的显示。 
> 显示：content的显示需要重新定位，因为放在popover下面的话，如果父容器 `overflow：hidden; ` ，也会导致content显示问题。所以content visible时，如果用户传入指定挂载容器就挂载至改容器下，没有的话就直接挂载到body下。
> 隐藏：content的隐藏，需要监听document的点击事件，当点击popover或者content的时候都是不需要隐藏的，所以触发关闭条件时需要排除掉点击这些时的情况。

```js
<template>
  <div class="koma-popover" ref="popover">
    <div ref="contentWrapper" class="koma-popover-content-wrapper" v-if="visible"
      :class="{[`position-${position}`]:true}">
      <slot name="content" :close="close"></slot>
    </div>
    <span ref="triggerWrapper" style="display: inline-block;">
      <slot></slot>
    </span>
  </div>
</template>

export default {
  name: 'KomaPopover',
  props: {
    position: {
      type: String,
      default: 'top',
      validator(val){
        return ['top', 'bottom', 'left', 'right'].indexOf(val) >= 0
      }
    },
    trigger: {
      type: String,
      default: 'click',
      validator(val){
        return ['click', 'hover'].indexOf(val) >= 0
      }
    },
    container: {
      type: Element
    }
  },
  data() {
    return {
      visible: true,
    }
  },
  mounted() {
    // 为什么不直接在div里绑定click事件？
    // 因为这里支持两种click和mouseenter的方式，如果直接绑定在div上将无法区分
    this.addPopoverListener()
  },
  beforeDestroy() {
    this.recoverElcontent();
    this.removePopoverListener();
  },
  methods: {
    removePopoverListener() {
      let popoverEL = this.$refs.popover;
      if(!popoverEL) { return; }
      if(this.trigger === 'click') {
        popoverEl.removeEventListener('click', this.clickPopover)
      } else {
        popoverEl.removeEventListener('mouseenter', this.open)
        popoverEl.removeEventListener('mouseleave', this.close)
      }
    },
    addPopoverListener() {
      let popoverEl = this.$refs.popover
      if(this.popoverEl) {
        if(this.trigger === 'click') {
          popoverEl.addEventListener('click', this.clickPopover)
        } else {
          popoverEl.addEventListener('mouseenter', this.open)
          popoverEl.addEventListener('mouseleave', this.close)
        }
      }
    },
    recoverElcontent() {
      const {contentWrapper, popover} = this.$refs
      if(!contentWrapper){return}
      popover.appendChild(contentWrapper)
    },
    clickPopover(e) {
      if(this.$refs.triggerWrapper.contains(e.target)) {
        if(this.visible) {
          this.close()
        } else {
          this.open()
        }
      }
    },
    open() {
      this.visible = true
      // content显示后，记得给document绑定click事件，点击popover以外的地方都要关闭content
      document.addEventListener('click', this.listenToDocument)
      this.updatePopoverPosition()
    },
    close() {
      this.visible = false
      this.$emit('close')
      document.removeEventListener('click', this.listenToDocument)
    },
    updatePopoverPosition() {
      const { popover, triggerWrapper } = this.$refs;
      if(!popover){return;}
      // 需要先挂载到页面上，才能设样式
      (this.container || document.body).appendChild(popover)
      const { left, top, height, width } = triggerWrapper.getBoundingClientRect()
      const { height: height2 } = contentWrapper.getBoundingClientRect()

      let positions = {
        top: {
          top: top + window.scrollY,
          left: left + window.scrollX
        },
        bottom: {
          top: top + height + window.scrollY,
          left: left + window.scrollX,
        },
        left: {
          top: top + window.scrollY + (height - height2)/2,
          left: left + window.scrollX
        },
        right: {
          top: top + window.scrollY + (height - height2)/2,
          left: left + width + window.scrollX
        }
      }
      contentWrapper.style.left = positions[this.position].left + 'px'
      contentWrapper.style.top = positions[this.position].top + 'px';
    },
    listenToDocument(e) {
      if(this.$refs.popover === e.target || this.$refs.popover.contains(e.target)){
        return;
      }
      if(this.$refs.triggerWrapper === e.target || this.$refs.triggerWrapper.contains(e.target)){
        return;
      }
      this.close()
    },

  }
  
}
```

### 11. Collapse组件
核心思路：
> collapse组件的实现，很好的体现了单向数据流，在子组件中数据发生变化但不改变数据，父组件中监听事件来进行数据操作，再通过广播事件改变相关组件的选中态。

广播事件自然想到了eventBus,代码如下：
```js
// collapse.vue
<template>
  <div class="koma-collapse">
    <slot></slot>
  </div>
</template>

export default {
  name: 'KomaCollapse',
  props: {
    selected: {
      type: Array,
      default() {
        return []
      }
    },
    single: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      eventBus: new Vue()
    }
  },
  provide() {
    return {
      eventBus: this.eventBus
    }
  },
  mounted() {
    let selected = JSON.parse(JSON.stringfy(this.selected))
    this.eventBus.$emit('change', selected)
    this.eventBus.$on('removeChange', (name)=>{
      let index = selected.indexOf(name)
      selected.splice(index, 1)
      this.broadcast(selected)
    })
    this.eventBus.$on('addChange', (name)=>{
      if(this.single) {
        selected = []
      }
      this.selected.push(name)
      this.brodacast(selected)
    })
  },
  methods: {
    broadcast(selected){
      this.eventBus.$emit('change', selected)
      this.$emit('update:selected', selected)
    }
  }
}


// collapse-item.vue
<template>
  <div class="koma-collapse-item">
    <div class="title" @click="click">
      <span>{{title}}</span>
      <k-icon name="right" class="collapse-icon" :class="{'expand-icon': open}"></k-icon>
    </div>
    <collapse-transition>
      <div class="content" v-show="active">
        <slot></slot>
      </div>
    </collapse-transition>
  </div>
</template>

export default {
  name: 'KomaCollapse',
  props: {
    name: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      active: false
    }
  },
  inject: ['eventBus'],
  mounted() {
    this.eventBus && this.eventBus.$on('change', (selected)=>{
      if (selected.indexOf(this.name) > -1) {
        this.active = true
      } else {
        this.active = false
      }
    })
  },
  methods: {
    onClick() {
      if(this.active) {
        this.eventBus &&  this.eventBus.$emit('removeChange', this.name)
      } else {
        this.eventBus &&  this.eventBus.$emit('addChange', this.name)
      }
    }
  }
}
```

### 12. Cascader 组件和 directives 指令

#### 12.1 directives 指令

在复盘cascader前，首先复习下指令 `directives` 的用法。
在popover组件时，我们在open时，会给document绑定一个click事件，它的主要作用是点击popover外部时，直接关闭popover。这个场景在很多组件都会运用到，比如： 常用的选择器（下拉选择，日期选择等），这里我们就可以引入指令去做这件事：

指令你可以理解为：操作dom。官方解释是：需要对普通DOM元素进行底层操作，这时就可以使用自定义指令。

```js
// 注册全局指令 'v-focus'
Vue.directive('focus', {
  inserted: function(el) {
    el.focus()
  }
})

// 组件内部指令选项, 与全局不同的是要加 s
directives: {
  focus: {
    inserted: function(el) {
      el.focus()
    }
  }
}

```

定义的指令就是一个对象，对象里都是一些钩子函数。
> bind: 只调用一次，指令第一次绑定到元素时调用。
> inserted: 被绑定元素插入父节点时调用（仅保证父节点存在，但不一定已插入到文档中）
> update: 所在组件的 VNode 更新时调用，但是可能发生在其子 VNode 更新之前。
> componentUpdated：指令所在组件的 VNode 及其子 VNode 全部更新后调用。
> unbind: 只调用一次，指令与元素解绑时调用。

钩子函数的参数：
```JS
<div id="hook-arguments-example" v-demo:foo.a.b="message"></div>

Vue.directive('demo', {
  bind: function (el, binding, vnode) {
    console.log(binding.name)   // demo
    console.log(binding.arg)    // foo
    console.log(binding.modifiers)  // 修饰： {a: true, b: true}
    console.log(binding.expression) // 字符串表达式  message
    console.log(binding.value)  // hello
  }
})

new Vue({
  el: '#hook-arguments-example',
  data: {
    message: 'hello!'
  }
})


```


具体例子：
```js 
// 这里封装了一个 v-click-outside 的指令，用于监听是否点击了除元素本身外的dom
// click-outside.js
// 当引入这个js后，会自上而下的执行所有语句
// 如果直接在bind里监听document，会在每个绑定指令的元素上都生成一个监听器，
// 如果同时有10个元素绑定了v-click-outside, 那么会生成10个监听器，这显然不合理。
// 所以我们利用js自上而下的特点，使用一个监听器。

document.addEventListener('click', onClickDocument)
let callbacks = [];
export default {
  bind: function(el, binding) {
    callbacks.push({el, callback: binding.value})
  }
}
let onClickDocument = (e) => {
  let { target } = e;
  callbacks.forEach((i) => {
    if(target === i.el || i.el.contains(target)) {
      return;
    } else {
      i.callback()
    }
  })
}

let removeListenter = () => {
  document.removeEventListener('click', onClickDocument)
}

export { removeListenter }



// cascader.vue 
<div class="koma-cascader" ref="cascader" v-click-outside="close"></div>

import ClickOutside from '../directives/click-outside';

export default {
  name: 'KomaCacader',
  directives: {
    ClickOutside
  },
  methods: {
    close() {
      this.popoverVisible = false
    },
  }
}
```

#### 12.2 Cascader 组件
核心思路：
> cascader随着数据源的嵌套，所以需要循环递归组件。递归的时候需要一个属性判断到了哪一层，这里就使用了level, 通过level来给selected赋值。

```js
// cascader.vue 简化版
<template>
  <div class="koma-cascader" v-click-outside="close">
    <div class="trigger" @click="toggle"></div>
    <div class="popover-wrapper" v-if="popoverVisible">
      <cascader-item class="popover"
      :selected="selected"
      @update:selected="onUpdateSelected"
      :items="datas"
      ></cascader-item>
    </div>
  </div>
</template>

<script>
import cascaderItem from './cascader-item';
import ClickOutside from '../directives/click-outside'

export default {
  name: 'KomaCascader',
  components: { cascaderItem },
  directives: {
    ClickOutside
  },
  props: {
    datas: {
      type: Array,
      default: () = > []
    },
    selected: {
      type: Array,
      default: () = > []
    }
  },
  data() {
    return  {
      popoverVisible: false
    }
  },
  methods: {
    open() {
      this.popoverVisible = true
    },
    close() {
      this.popoverVisible = false
    },
    toggle() {
      if(this.popoverVisible) {
        this.close()
      } else {
        this.open()
      }
    },
    // 因为是单向数据流，所以还是由这里来处理数据，子组件选中后都会触发这个方法
    onUpdateSelected() {
      this.$emit('update:selected', val)
    },
  }
}
</script>

```

```js
// cascader-item.vue
<template>
  <div class="koma-cascader-item" :style="{ height }">
    <div class="left">
      <div class="label" v-for="(item, index) in items"
        :key="index"
        @click="onclickLable(item)">
        <span class="name">{{ item.name }}</span>
        <span class="icons">
            <icon class="icon" v-if="rightArrowVisible(item)" name="right"></icon>
        </span>
      </div>
    </div>
    <div class="right" v-if="rightItems">
      <cascader-item
      :loading-item="loadingItem"
      :load-data="loadData"
      :selected="selected"
      :level="level+1"
      :items="rightItems"
      @update:selected="onUpdateSelected"
      :height="height"></cascader-item>
    </div>
  </div>
</template>

<script>
export default {
  // 这里的name必须定义，这样就能递归调用
  name: 'cascader-item',
  props: {
    items: {
      type: Array,
    }
    selected: {
      type: Array,
      default: () => []
    },
    // 很关键
    level: {
      type: Number,
      default: 0
    }
  },
  computed: {
    // 嵌套的数据源一定是个计算熟悉，需要根据level来匹配其对应的children
    rightItems () {
      // 之前的会存在bug，当selected和level不更新时，不会重新计算该值
      // 现在改为检测items的变化
      if(this.selected && this.selected[this.level]) {
        let item = this.items.filter(i => i.name === this.selected[this.level].name)
        if(item[0] && item[0].children && item[0].children.length) {
          return item[0].children;
        }
      }
      return null;
    },
  },
  methods:{
    rightArrowVisible(item) {
      return item.children
    },
    onUpdateSelected(val){
      this.$emit('update:selected', val)
    },
    // 点击后更新selected, 按照level索引去赋值selected选中的值
    onclickLable(item) {
      let copy = JSON.parse(JSON.stringify(this.selected))
      copy[this.level] = item
      // 单向数据流, 不建议子直接修改父传入的值，所以这里copy一下
      this.$emit('update:selected', copy)
    }
  }
}
</script>

```

基本的cascader的写法如上， 如果想加入懒加载的功能，稍微麻烦点，
核心思路： 
> cascader-item触发onclickLable事件后，父组件 cascader 在监听的事件里主要做了两件事： 
> 1. 拿到更新后的selected值后再次触发 this.$emit('update:selected', val)；
> 2. 用户通过懒加载的callback，回传更新后的值，父组件将数据源对应的项的children更新成回传过来的值，然后触发this.$emit('update:datas', copy)

```js
// cascader.vue
// 在子组件触发onclickLable 方法后，父组件监听事件进行懒加载操作：

onUpdateSelected(val) {
  this.$emit('update:selected', val)
  let lastItem = val[val.length - 1]
  let callback = (res) => {
    // 用户把数据回传后就不需要显示loading了，所以清空掉
    this.loadingItem = {};
    let copyData = JSON.parse(JSON.stringify(this.datas))
    let toUpdate = this.getTreeItemById(copyData, lastItem.id)
    toUpdate.children = res
    this.$emit('update:datas', copyData)
  }
  if(!lastItem.isLeaf && this.loadData) {
    // 这个loadingItem为了显示loading图标
    this.loadingItem = lastItem;
    this.loadData(lastItem, callback)
  }
}

getTreeItemById(datas, id) {
  let hasfound = false, result = null;
  let fn = (datas, id) => {
    if(Array.isArray(datas) && !hasfound) {
      datas.forEach((i) => {
        if(i.id === id) {
          hasfound = true
          result = i
        } else if( i.children ) {
          fn(i.children, id)
        }
      })
    }
  }
  fn(datas, id)
  return result;
}


// example.vue
<k-cascader 
  :selected.sync="selected"
  :datas.sync="lazyDatas"
  :load-data="loadData">
</k-cascader>

data(){
  return {
    selected: [],
    lazyDatas: [],
  }
},
created() {
  ajax(0).then(res => {
    this.lazyDatas = res
  })
},
methods: {
  loadData({id}, callback) {
    ajax(id).then(res => {
      setTimeout(() => {
        callback(res)
      }, 2000)
    })
  }
}
```


### 13. Carousel组件
核心思路：
> 很简单，`carousel-item` 通过selected值，判断是否与自身的name相等，相等即显示。需要加入切换效果。

```js
// carousel.vue 
<template>
  <div class="koma-crousel">
    <div class="koma-crousel-window">
      <div class="koma-crousel-wrapper">
        <slot></slot>
      </div>
    </div>
    <div class="koma-crousel-dots">
      <span @click="select(selectedIndex - 1)">
        <k-icon name="left"></k-icon>
      </span>
      <!-- 不能直接使用$children, 因为他不是响应式的所以不会刷新视图 -->
      <span v-for="n in childrenLength"
      :key="n"
      @click="select(n-1)"
      :data-index="n-1"
      :class="{active: n - 1 === selectedIndex}">
        {{ n }}
      </span>
      <span @click="select(selectedIndex + 1)">
        <k-icon name="right"></k-icon>
      </span>
    </div>
  </div>
</template>

<script>
export default {
  name: 'KomaCarousel',
  props: {
    selected: {
      type: String | Number
    },
    autoPlay: {
      type: Boolean,
      default: true
    },
    autoPlayDelay: {
      type: Number,
      default: 3000
    }
  },
  data () {
    return {
      childrenLength: 0,
      // 上次选中的项
      lastSelectedIndex: null,
      timerId: null
    };
  },
  computed: {
    selectedIndex(){
      let index = this.names.indexOf(this.selected)
      return index === -1 ? 0 : index
    },
    names(){
      return this.carouselItems.map(i => i.name) || []
    },
    carouselItems(){
      return this.$children.filter(vm => vm.$options.name === 'KomaCarouselItem')
    }
  },
  mounted() {
    // 因为使用$children.length不是响应式的
    this.childrenLength = this.carouselItems.length
    this.updateChildren()
    this.playAutomatically()
  },
  // 这个钩子也很重要，外面selected值更新后, 需要重新通知每一个子组件新的值
  updated(){
    this.updateChildren()
  },
  methods: {
    // 点击下标切换选中
    select(newIndex){
      if( newIndex === -1 ) {
        newIndex = this.names.length - 1
      }
      if (newIndex === this.names.length) {
        newIndex = 0
      }
      // 记录上一次的选中，以便判断是否reverse
      this.lastSelectedIndex = this.selectedIndex
      this.$emit('update:selected', this.names[newIndex])
    },
    // 这里用 setTimeout 模拟了 setInterval
    playAutomatically() {
      if(!this.autoPlay){return;}
      if(this.timerId) {return;}
      // setInterval在极端情况下，会有问题
      // 用 setTimeout 模拟 setInterval
      let run = ()=>{
        let index = this.names.indexOf(this.getSelected())
        let newIndex = index + 1
        this.select(newIndex)
        this.timerId = setTimeout(run, this.autoPlayDelay)
      }
      this.timerId = setTimeout(run, this.autoPlayDelay)
    },
    getSelected() {
      let first = this.carouselItems[0]
      return this.selected || first.name
    },
    // 主要做了两件事： 1. 确认当前选中的值，并赋值给子组件  2. 确定每一个carousel-item的左右切换动画
    updateChildren() {
      // ① 首先确定好selected值，外部没有传的话取第一个children的name
      this.selected = this.getSelected()
      // ② 这里主要是确定每一个carousel的左右切换动画
      this.carouselItems.forEach((vm) => {
        // 通过比对当前选中的index的值和上一次的选中的值的大小，来判断是左滑还是右滑
        let reverse = this.selectedIndex > this.lastSelectedIndex ? false : true
        // 还需要考虑 首转末 和 末转尾的情况
        if(this.lastSelectedIndex === 0 && this.selectedIndex === this.carouselItems.length - 1) {
          reverse = true
        }
        if(this.lastSelectedIndex === this.carouselItems.length - 1 && this.selectedIndex === 0) {
          reverse = false
        }
        vm.reverse = reverse
        // 这里加nextTick的原因是保证 reverse 在新的选中时是对的
        this.$nextTick(()=>{
          vm.selected = selected
        })
      })
    }
  }
}
</script>
```

```js
// carousel-item.vue
// 因为大部分选中态的处理都是在父组件里处理
// 这里只需要根据selected 来显示隐藏和处理动画效果
<template>
  <transition name="slide">
    <div class="koma-crousel-item" v-if="visible" :class="{ reverse }">
      <slot></slot>
    </div>
  </transition>
</template>

<script>
export default {
  name: 'KomaCarouselItem',
  components: {},
  props: {
    name: {
      type: String | Number,
      required: true
    }
  },
  data () {
    return {
      selected: '',
      reverse: false,
    };
  },
  computed: {
    visible() {
      return this.selected === this.name;
    }
  },
}

</script>
<style lang='less' scoped>
.koma-crousel-item {
  width: 100%;
}
// 动画
.slide-enter-active, .slide-leave-active {
  transition: all 0.5s;
}
// 这里绝对定位的目的是为了让走的那个不占位置
.slide-leave-active {
  width: 100%;
  height: 100%;
  position:absolute;
  left: 0;
  top: 0;
}
.slide-enter {
  transform: translateX(100%);
  // opacity: 0;
}
.slide-enter.reverse {
  transform: translateX(-100%);
  // opacity: 0;
}

.slide-leave-to {
  transform: translateX(-100%);
  // opacity: 0;
}
.slide-leave-to.reverse {
  transform: translateX(100%);
  // opacity: 0;
}
</style>

```

### 14. Nav组件
核心思路：
> nav组件还是有点难度的，我们需要知道nav下有多少 `nav-item`，并且获取到其集合，使用`$children`，然后遍历 `$options.name === 'KomaNavItem'` 的方式虽然可以，但是如果被 `sub-nav` 嵌套，那就无法拿到嵌套的`nav-item`了，所以我们需要换一种方式。

> 这里有一种比较巧妙的方式，通过nav父组件，`provide` 一个 `root` ，然后子组件`mounted`的时候通过注入，来调用 `root` 的 `addItem` 方法，来将vm告知父组件。

```js
// nav.vue
data() {
  items: [],
},
provide() {
  return {
    root: this
  }
},
methods: {
  addItems(vm) {
    this.items.push(vm)
  }
},

// nav-item.vue
inject: ['root'],
mounted() {
  this.roote.addItems(this)
}
```

上面的nav组件因为是slot方式去插入子组件，无法在其上面使用 @事件监听，我们可以通过遍历上面的items然后使用 $on 去监听：

```js
// nav.vue 完整代码
<template>
  <div class="koma-nav" :class="{ vertical }">
    <slot></slot>
  </div>
</template>

<script>
export default {
  name: 'KomaNav',
  components: {},
  provide(){
    return {
      root: this,
    }
  },
  props: {
    selected: {
      type: String,
      default: ''
    },
  },
  data() {
    return {
      items: [],
      namePath: [],  // 记录多层级选中的值
    };
  },
  updated(){
    // 因为父组件props更新后，需要重新触发此事件去判断子组件是否选中
    this.updateChildren()
  },
  mounted() {
    this.updateChildren()
    this.listenToChildren()
  },
  methods: {
    updateNamePath() {},
    // 使用注入的好处是，让所有子组件自己告诉我它的存在，而不需要我去遍历找子组件
    addItem(vm){
      this.items.push(vm);
    },
    updateChildren(){
      // 更新选中的值
      this.items.forEach(vm => {
        if(this.selected === vm.name) {
          vm.selected = true
        } else {
          vm.selected = false
        }
      })
    },
    listenToChildren(){
      // 监听子组件的点击事件
      this.items.forEach(vm => {
        // 这里遍历vm去监听它，虽然看起来很别扭，感觉是自己监听了自己，但是确实可以监听
        vm.$on('update:selected', (name)=>{
          if(this.selected !== name ) {
            // 不要直接修改传入的prop
            this.$emit('update:selected', name)
          }
        })
      })
    }
  },
};
</script>
```

在subNav里，需要注意的是纵向的时候的展开动画，同时也要注意如果subNav一直重复嵌套，我们如何拿到已选中的name集合？

```js
// 在单向数据流中，最底层的子组件数据处理往往是最简单的。
// nav-item.vue
<template>
  <div class="koma-nav-item" :class="{active: selected, vertical}" @click="onclick" :data-name="name">
    <slot></slot>
  </div>
</template>

<script>
export default {
  name: 'KomaNavItem',
  components: {},
  props: {
    name: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      selected: false
    };
  },
  inject: ['root', 'vertical'],
  created() {
    this.root.addItem(this)
  },

  methods: {
    onclick(){
      // 激活父组件
      this.root.namePath = []
      // 这两句就是用来更新namePath的
      this.$parent.updateNamePath && this.$parent.updateNamePath()
      // 子组件简单处理，只需要通知点击的name就行，数据交给父组件组处理
      this.$emit('update:selected', this.name)
    }
  },
};

```
```js
// sub-nav.vue
<template>
  <div class="koma-sub-nav" :class="{ active, vertical }" v-click-outside="close">
    <span class="koma-sub-nav-label" @click="onClick">
      <slot name="title"></slot>
      <span class="koma-sub-nav-icon" :class="{ open, vertical }">
        <k-icon name="right"></k-icon>
      </span>
    </span>
    <template v-if="vertical">
      <transition @enter="enter" @leave="leave" @after-leave="afterLeave" @after-enter="afterEnter">
        <div class="koma-sub-nav-popover" v-show="open" :class="{ vertical }">
          <slot></slot>
        </div>
      </transition>
    </template>
    <template v-else>
      <div class="koma-sub-nav-popover" v-show="open" :class="{ vertical }">
        <slot></slot>
      </div>
    </template>
  </div>
</template>

<script>
import KIcon from '../icon'
import ClickOutside from '../directives/click-outside';
export default {
  name: 'KomaSubNav',
  components: {
    KIcon
  },
  inject: ['root', 'vertical'],
  directives: {
    ClickOutside
  },
  props: {
    name: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      open: false
    };
  },

  computed: {
    active(){
      return this.root.namePath.indexOf(this.name) >= 0
    }
  },


  methods: {
    enter(el, done) {
      // 先拿到auto真实高度，存到变量height
      let {height} = el.getBoundingClientRect()
      // 高度先为0 -> 然后转换auto，这样高度会有变化，可以响应 transition: height 1s;
      el.style.height = 0
      // 这里加一句计算高度的原因是因为浏览器会让多次多样的样式赋值合并
      // 如果中间进行一个进行元素高度计算的操作，就不会发生合并现象
      el.getBoundingClientRect()
      el.style.height = `${height}px`
      el.addEventListener('transitionend', ()=>{
        done();
      })
    },
    // 重新设为auto，不然因为overflow:hidden; 子级会看不到
    afterEnter(el) {
      el.style.height = 'auto';
    },
    leave(el, done) {
      let {height} = el.getBoundingClientRect()
      el.style.height = `${height}px`
      el.getBoundingClientRect()
      el.style.height = 0;
      // 如果直接调用done的话，动画会直接结束，直接display none。这里监听动画结束事件然后再去调用done
      el.addEventListener('transitionend', ()=>{
        done();
      })
    },
    afterLeave(el){
      el.style.height = 'auto';
    },
    close(){
      this.open = false
    },
    onClick(){
      this.open = !this.open;
    },
    // 子组件是激活状态的话，自己也激活
    updateNamePath(){
      this.root.namePath.unshift(this.name)
      // 如果还有父级就一直调用
      if(this.$parent.updateNamePath) {
        this.$parent.updateNamePath()
      } else {
        this.root.namePath = []
      }
    },
  },
};
</script>
```

### 15. Validator验证
首先要了解什么是类，类是'特殊的函数'，可以理解为一个构造工厂。
如下：
```js
class Validator {
  constructor (height, width) {
    this.height = height
    this.width = width
  }
}
```

为什么要将 `Validator` 以类的形式构造？ 那是因为，为了避免许多重复的代码，构造函数可以实现代码复用。
当你需要大批量的写对象的时候，就需要用到构造函数，它可以方便创建多个对象的实例，并且创建的对象可以被标识为特定的类型，可以通过继承扩展代码。

如何构建一个验证类 `Validator`?
验证顾名思义，我们给validator传入数据和验证规则，然后其返回验证结果error。
```js
// 简单的调用示例：
import Validate from './validate';

let validator = new Validate()
let data = {
  email: '1@aa.com',
  password: '123'
}
let rules = {
  { key: 'email', pattern: 'email', required: true},
  { key: 'password', minLength: 6, required: true}
}
this.error = validator.validate(data, rules)

```

从上面可知，我们需要给类一个validate方法，方法结果会返回一个error对象。
```js
// validate.js
class Validator {
  constructor (){}

  validate(data, rules) {
    let errors = {}
    rules.forEach(rule => {
      let value = data[rule.key]
      if(rule.required) {
        // this就是类Validator
        let error = this.required(value)
        if(error) {
          ensureObject(errors, rule.key)
          // error = { email: {required: '必填'}}
          errors[rule.key].required = error
          return;
        }
      }
      let validatorsKeys = Object.keys(rule).filter(key => key!== 'key' && key!=='required')
      validatorsKeys.forEach(k => {
        if(this[k]){
          let error = this[k](value, rule[k])
          if(error) {
            ensureObject(errors, rule.key)
            errors[rule.key][k] = error;
          }
        } else {
          throw `不存在的校验器: ${k}`
        }
      })
    })
    // {email: {required: '必填', minLength: '太短'}}
    return errors;
  }
  required(value){
    if(!value && value !== 0){
      return '必填';  
    }
  }
  pattern(value, pattern){
    if(pattern === 'email') {
      pattern = /^.+@.+$/
    }
    if(!pattern.test(value)){
      return  '格式不正确'
    }
  }
  minLength(value, minLength){
    if(value.length < minLength) {
      return '太短'
    }
  }
  maxLength(value, maxLength){
    if(value.length > maxLength) {
      return '太长'
    }
  }
}
function ensureObject (obj, key) {
  if(typeof obj[key] !== 'object'){
    obj[key] = {}
  }
}
```

上面就是Validator的类，当我们new 一个类时，如果扩展了类的方法，并不会影响其它的实例，如：
```js
// validate.spec.js
let validator1 = new Validator()
let validator2 = new Validator()
validator1.hasNumber = (value) => {
  if(!/\d/.test(value)){
    return '必须含有数字'
  }
}
let rules = [
  { key:'email', required: true, hasNumber: true}
]
expect(()=>{
  validator1.validate(data, rules)
}).to.not.throw();
expect(()=>{
  validator2.validate(data, rules)    // 这个测试用例是无法通过的，因为hasNumber是validator1扩展的
}).to.throw();
```

如何添加一个类的公共方法？
```js
class Validator {
  // 精髓
  // 注意实例是不能调用add的
  static add (name, fn) {
    Validator.prototype[name] = fn
  }
}

// 无法用实例调用
let valid = new Validator()
valid.add  // undefined

// 需要类自己去调用
Validator.add('hasNumber', (value) => {
  if(!/\d/.test(value)){
    return '必须含有数字'
  }
})

expect(()=>{
  validator1.validate(data, rules)    // pass
}).to.not.throw();
expect(()=>{
  validator2.validate(data, rules)    // pass
}).to.not.throw();

```

### 16. Form组件
比较简单，就直接放源码了。
```js
// form.vue
<template>
  <form class="koma-form" @submit.prevent v-bind="$attrs">
    <slot></slot>
  </form>
</template>

<script>
import Validate from './validate';
function debounce(fn, delay) {
  let _delay = delay || 200;
  return function (args) {
    let _this = this
    let _args = args
    fn.id && clearTimeout(fn.id)
    fn.id = setTimeout(function () {
      fn.call(_this, _args)
    }, _delay)
  }
}
export default {
  name: 'KomaForm',
  components: {},
  provide(){
    return {
      root: this
    }
  },
  props: {
    model: {
      type: Object,
      required: true
    },
    rules: {
      type: Array
    }
  },
  data() {
    return {
      validator: new Validate(),
      childItems: [],
      errors: {}
    };
  },

  watch: {
    model: {
      handler(val) {
        this.checkError()
      },
      deep: true
    }
  },
  methods: {
    addItem(item){
      this.childItems.push(item)
    },
    checkError: debounce( function () {
      const rules = this.rules
      this.errors = this.validator.validate(this.model, rules)

      this.childItems.forEach((i)=>{
        if(this.errors[i.prop]) {
          i.error = this.errors[i.prop]
        } else {
          i.error = {}
        }
      })
    }),
    validate(fn){
      this.checkError()
      const valid = Object.keys(this.errors).length === 0
      fn(valid, this.errors);
    }
  },
};
</script>
```

```js
// form-item.vue
<template>
  <div class="koma-form-item">
    <label>{{label}}</label>
    <div class="controls">
      <slot></slot>
      <div class="error-text">
        {{ error && Object.values(error)[0]}}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'KomaFormItem',
  components: {},
  props: {
    label: {
      type: String,
      required: true
    },
    prop: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      error: {}
    };
  },
  watch: {},
  inject: ['root'],

  computed: {},

  created() {
    this.root.addItem(this)
  },
};
</script>
<style lang='less' scoped>
@import '../../css/_var';
.koma-form-item {
  margin-bottom: 8px;
  display: flex;
  label {
    margin-right: 1em;
  }
  .error-text {
    font-size: @small-font-size;
    color: @red;
  }
}
</style>

```

### 17. Pager组件
核心思路：
> pager组件核心就是操控数组，数组元素固定显示 
`[1, currentPage - 2, currentPage - 1, currentPage, currentPage + 1, currentPage + 2, totalPage]`

```js
// pager.vue
<template>
  <div class="koma-pager" v-if="!(hideIfOnePage && totalPage <= 1)">
    <span class="koma-pager-nav prev" :class="{disabled: currentPage === 1}"
      @click="change(currentPage - 1)">
      <k-icon name="left"></k-icon>
    </span>
    <template v-for="(page, index) in pages">
      <template v-if="page === currentPage">
        <span :key="index" class="koma-pager-item active">{{page}}</span>
      </template>
      <template v-else-if="page === '...'">
        <k-icon name="more" :key="index" class="separator"></k-icon>
      </template>
      <template v-else>
        <span :key="index" class="koma-pager-item other" @click="change(page)">{{page}}</span>
      </template>
    </template>
    <span class="koma-pager-nav next"  :class="{disabled: currentPage === totalPage}"
      @click="change(currentPage + 1)">
      <k-icon name="right"></k-icon>
    </span>
  </div>
</template>

<script>
function unique(array) {
  // return [...new Set(array)]
  // 考虑兼容性写法, 应用对象key唯一的特性
  let obj = {}
  array.forEach((i)=>{
    obj[i] = true
  })
  // 注意这里返回的都是字符串，不再是number
  return Object.keys(obj).map(i => parseInt(i))
}
import KIcon from '../icon';
export default {
  name: 'KomaPager',
  components: { KIcon },
  props: {
    totalPage: {
      type: Number,
      required: true
    },
    currentPage: {
      type: Number,
      required:true
    },
    hideIfOnePage: {
      type: Boolean,
      default: true
    }
  },
  data () {
    return {

    }
  },
  computed: {
    pages(){
      let pages = unique([1, this.totalPage, this.currentPage, 
      this.currentPage - 1, 
      this.currentPage - 2, 
      this.currentPage + 1, 
      this.currentPage + 2])
      .filter((n)=> n > 0 && n <= this.totalPage)
      .sort((a, b)=> a - b)                     // 升序排序
      .reduce((prev, current, index, array)=>{  // 页码间隔大于1时，中间显示...
        if(array[index - 1] && array[index] - array[index - 1] > 1){
          prev.push('...')
        }
        prev.push(current)
        return prev;
      }, [])
      return pages;
    }
  },
  created () {},
  methods: {
    change(page) {
      if(page >= 1 && page <= this.totalPage) {
        this.$emit('update:current-page', page)
      }
    },
  }
}

</script>
<style lang='less' scoped>
@import "../../css/_var";
@width: 25px;
@height: 25px;
.koma-pager {
  display: flex;
  align-items: center;
  user-select: none;
  .separator{
    width: @width;
    font-size: @font-size;
    cursor: default;
  }
  .koma-pager-nav {
    display: inline-flex;
    justify-content: center;
    align-items: center;
    height: @height;
    width: @width;
    margin: 0 4px;
    cursor: pointer;
    &.disabled {
      cursor: not-allowed;
      svg {
        fill: darken(@gray, 30%);
      }
    }
  }
  &-item {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    border: 1px solid @border-color;
    border-radius: @border-radius;
    font-size: @font-size;
    padding: 0 4px;
    margin: 0 4px;
    min-width: @width;
    height: @height;
    &:hover {
      border-color: @primary-color;
      color: @primary-color;
    }
    cursor: pointer;
    &.active {
      border-color: @primary-color;
      color: @primary-color;
    }
  }
}
</style>
```

### 18. Table组件
table组件就比较复杂了，需要考虑列的渲染，操作列，固定表头等。
核心思路：
> table组件是基于原生table的，所以要考虑th表头 td行内容。这两个都需要一个数组columns遍历去绑定数据源，所以column需要提供基本属性，如： `text(列名)、prop(列的标识)、width(列宽)`。所以需要考虑的重点是如何构造这样的columns。

columns和我们看到的列的是一一对应的，有多少koma-table-column，columns就有多少元素。

```js
// table.vue 
mounted() {
  // 我们需要知道如果slot内容有作用域，$slots是不会统计到的，如<template #action="scope"></template>
  // 利用这个特点可以找出所有插槽内容为k-table-column的
  // 因为会有tag undefined的无意义值，将其过滤掉
  const komaTableColumns = this.$slots.default.filter(i => i.tag)
  this.columns = komaTableColumns.map(node => {
    // 精髓，也是使用k-table-column中转的原因，方便拿到prop
    // 需要在k-table-column 组件里定义该prop才能拿到
    let { text, prop, width } = node.componentsOptions.propsData
    // 如果用户自己写了dom，可以直接拿到对应的渲染表达式做渲染
    let render = node.data.scopedSlots &&  node.data.scopedSlots.default
    return { text, prop, width, render}
  })
}
```

columns 拿到后，就可以渲染html了：
```js
// table.vue html骨架
<table class="koma-table" ref="table">
  <thead>
    <tr>
      <th :style="{width: `${column.width}px`}" v-for="(column, idx) in columns" :key="column.prop">
        <div class="kama-table-name">
          {{column.text}}
        </div>
      </th>
    </tr>
  </thead>
  <tbody>
    <template v-for="(item, index) in dataSource">
      <!-- 主体内容 -->
      <tr :key="item.id">
        <template v-for="column in columns">
          <td :style="{width: `${column.width}px`}" :key="column.prop">
            <template v-if="column.render">
              <!-- 使用render的方式，可以将孙组件里的html直接渲染到父组件上，并且可以通过传递参数到slot-scope上 -->
              <vnodes :vnodes="column.render({row: item})"></vnodes>
            </template>
            <template v-else>
              {{item[column.prop]}}
            </template>
          </td>
        </template>
      </tr>
    </template>
  </tbody>
</table>

export default {
  name: 'KomaTable',
  components: {
    // 我有特殊的渲染技巧
    Vnodes: {
      functional: true,
      render: (h, ctx) => ctx.props.vnodes
    }
  }
}
```

<font color="red">实现固定表头：</font>
如何做到滚动内容，而保证表头固定不动？
核心思路：
> 首先浅拷贝table壳，还真是壳，table里的内容（如tbody,thead）全部不要。然后将原来table的thead取出塞到拷贝的table里，然后将table塞到wrapper里。

代码如下：
```js
mounted() {
  this.$nextTick(() => {
    this.fixedHeader()
  })
},
beforeDestroy() {
  this.cloneTable.remove()
},
methods: {
  fixedHeader() {
    let cloneTable = this.$refs.table.cloneNode(false)
    this.cloneTable = cloneTable
    cloneTable.classList.add('clone-table')
    // this.$refs.table.children: HTMLCollection: [thead, tbody] 
    let thead = this.$refs.table.children[0]
    let {height} = thead.getBoundingClientRect()
    // 因为多了一个表格，所以需要将原来的表格高度减掉新的表格高，保证总体高度不变
    this.$refs.tableWrapper.style.marginTop = height + 'px';
    this.$refs.tableWrapper.style.height = this.height - height + 'px';
    cloneTable.appendChild(thead)
    this.$refs.wrapper.appendChild(cloneTable)
  }
}
```

勾选功能：
```js
// table.vue html骨架
<table class="koma-table" ref="table">
  <thead>
    <tr>
    <th v-if="checkable" :style="{width: '50px'}" class="table-center">
      <input class="pointer" ref="allCheckBox" type="checkbox" @change="onChangeAllItems" :checked="isAllItemSelected">
    </th>
      <th :style="{width: `${column.width}px`}" v-for="(column, idx) in columns" :key="column.prop">
        <div class="kama-table-name">
          {{column.text}}
        </div>
      </th>
    </tr>
  </thead>
  <tbody>
    <template v-for="(item, index) in dataSource">
      <tr :key="item.id">
        <td v-if="checkable" :style="{width: '50px'}" class="table-center">
          <!-- 这里不用 selectedItems.indexOf(item) 是因为， selectedItems里的对象都是经过深拷贝追加的，已经不再是原来的元素，他们是不等的 -->
          <input type="checkbox"
          :checked="inselectedItems(item)"
          @change="onChangeItem(item, index, $event)">
        </td>
        <template v-for="column in columns">
          <td :style="{width: `${column.width}px`}" :key="column.prop">
            <template v-if="column.render">
              <vnodes :vnodes="column.render({row: item})"></vnodes>
            </template>
            <template v-else>
              {{item[column.prop]}}
            </template>
          </td>
        </template>
      </tr>
    </template>
  </tbody>
</table>

computed: {
  isAllItemSelected() {
    const a = this.dataSource.map(i => i.id).sort()   
    const b = this.selectedItems.map(i => i.id).sort()
    if(a.length !== b.length) {
      return false
    }
    let equal = true
    for(let i = 0; i < a.length; i++){
      if(a[i] !== b[i]){
        equal = false;
        break;
      }
    }
    return equal;
  }
},
watch: {
  selectedItems(val) {
    // 半选中态
    if(val.length > 0 && val.length < this.dataSource.length) {
      this.$refs['allCheckBox'].indeterminate = true
    } else if (val.length === this.dataSource.length){
      this.$refs['allCheckBox'].indeterminate = false
    } else {
      this.$refs['allCheckBox'].indeterminate = false
    }
  }
}
methods: {
  inselectedItems(item){
    return this.selectedItems.filter(i => i.id === item.id).length > 0
  },
  onChangeAllItems(e){
    let selected = e.target.checked;
    // 依然是单向数据流的方式
    this.$emit('update:selectedItems', selected ? this.dataSource : [])
  },
  onChangeItem(item, index, e){
    let selected = e.target.checked;
    let copy = JSON.parse(JSON.stringify(this.selectedItems))
    if(selected){
      copy.push(item)
    }else{
      // 这个就很巧妙了，不是使用的splice而是用的filter
      copy = copy.filter(i => i.id !== item.id)
    }
    this.$emit('update:selectedItems', copy)
  },
}
```

### 19. Uploader组件
核心思路：
> uploder需要一个trigger触发器，然后通过监听trigger的点击事件，创建一个隐藏的type为file的input，监听该 input 的 change 事件来开始uploadFile操作。

```js
<template>
  <div class="koma-uploader">
    <div @click="onClickUpload" ref="trigger">
      <slot></slot>
    </div>
    <div ref="temp" style="display: none;"></div> 
  </div>
</template>

<script>
export default {
  methods: {
    onClickUpload() {
      let input = this.createInput()
      input.addEventListener('change', e => {
        let rawFiles = input.files
        this.uploadFile(rawFiles)
        input.remove()
      })
    },
    createInput() {
      this.$refs.temp.innerHtml = ''
      let input = document.createElement('input')
      input.type = 'file'
      input.accept = this.accept
      input.multiple = this.multiple
      this.$refs.temp.appendChild(input)
      return input;
    },
    uploadFile(rawFiles) {
      let newName = [];
      for(let i=0; i < rawFiles.length; i++){
        let rawFile = rawFiles[i]
        let {name} = rawFile
        // 重复文件名称叠加
        let newName = this.generateName(name)
        // 不改变真实的rawFiles， 将新的name用数组存起来
        newNames[i] = newName
      }
      if(!this.beforeUploadFile(rawFiles, newNames)) {return;}
      // 通过上传前的验证后，开始上传
      for(let i=0; i<rawFiles.length; i++){
        let rawFile = rawFiles[i]
        let {name, type, size} = rawFile
        let newName = newNames[i]

        let formData = new FormData()
        formData.append(this.name, rawFile)
        this.doUploadFile(formData, 
          // success
          (res)=>{
            // imgUrl是用户回传的
            this.imgUrl = this.parseResponse(res)
            // 传newName是为了找到之前那个文件以更新文件状态
            this.afterUploadFile(newName, this.imgUrl)
          }, 
          // error
          (xhr)=>{
            this.uploadError(newName, xhr)
          }
        )
      }
    },
    // 上传失败
    uploadError(newName, xhr){
      let copyFileList = JSON.parse(JSON.stringify(this.fileList))
      copyFileList.some((i)=>{
        if(i.name === newName){
          i.status = 'fail'
          return true;
        }
      })
      this.$emit('update:fileList', copyFileList)
      let error = ''
      if(xhr.status === 0) {
        error = '网络无法连接'
      }
      this.$emit('error', error)
    },
    // 上传中
    doUploadFile(formData, success, fail) {
      http[this.method.toLowerCase()](this.action, { success, fail, formData })
    },
    // 上传成功后
    afterUploadFile(newName, url){
      // 找到上传成功之前的那个file，更新它的状态
      let copyFileList = JSON.parse(JSON.stringify(this.fileList))
      copyFileList.some((i)=>{
        if(i.name === newName){
          i.url = url
          i.status = 'success'
          return true;
        }
      })
      this.$emit('update:fileList', copyFileList)
      this.$emit('uploaded')
    },
    // 上传前
    // rawFiles 注意这里的这个是个文件对象需要转一下
    beforeUploadFile(rawFiles, newNames) {
      rawFiles = Array.from(rawFiles)
      for(let i=0; i<rawFiles.length; i++){
        let {type, size} = rawFiles[i]
        if(size > this.sizeLimit){
          this.$emit('error', '文件大于2MB')
          return false;
        }
      }
      // 在上传前给每个文件加入name和loading状态
      let arr = rawFiles.map((rawFile, index)=> {
        return {
          name: newNames[index], 
          type: rawFile.type, 
          size: rawFile.size, 
          status: 'uploading'
        }})
      this.$emit('update:fileList', [...this.fileList, ...arr])
      return true;
    }
    generateName(name) {
      // 简单的循环判断已上传的文件中是否有重复名字
      while(this.fileList.filter(f=> f.name === name).length > 0) {
        let dotIndex = name.lastIndexOf('.')
        let nameWithoutExtension = name.substring(0, dotIndex)
        let extension = name.substring(dotIndex)
        name = nameWithoutExtension + '(1)' + extension
      }
      return name;
    },
  }
}
</script>
```