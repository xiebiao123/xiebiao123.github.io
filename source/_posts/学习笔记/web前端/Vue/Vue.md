---
title: Vue
date: 2019-07-19
categories:
    - 学习
tags:
    - vue
---

### 常用指令

* v-html 输出html代码 如 v-html="message"
* v-bind
  * 判断属性中的值 如 v-bind:class="{'class1': use}"
  * v-bind 指令被用来响应地更新 HTML 属性 \<a v-bind:href="url"\>
  * v-bind:style  v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }
  * v-bind:class  v-bind:class="{ active: isActive, 'text-danger': hasError }"
* v-if 条件判断
* v-else 可以用 v-else 指令给 v-if 添加一个 "else" 块
* v-else-if v-else-if 在 2.1.0 新增，顾名思义，用作 v-if 的 else-if 块。可以链式的多次使用
* v-show 根据条件展示元素：
* v-for
* v-module 双向绑定
* v-on:click 如 \<a v-on:click="doSomething"\>

<!-- more -->

#### 指令缩写

``` js
v-bind 缩写
    <!-- 完整语法 -->
    <a v-bind:href="url"></a>
    <!-- 缩写 -->
    <a :href="url"></a>

v-on 缩写
    <!-- 完整语法 -->
    <a v-on:click="doSomething"></a>
    <!-- 缩写 -->
    <a @click="doSomething"></a>
```

### 过滤器

``` js
<!-- 在两个大括号中 -->
{{ message | capitalize }}

<!-- 在 v-bind 指令中 -->
<div v-bind:id="rawId | formatId"></div>

过滤器可以串联：
{{ message | filterA | filterB }}

过滤器是 JavaScript 函数，因此可以接受参数：
{{ message | filterA('arg1', arg2) }}
```

### 事件装饰服

``` js
<!-- 阻止单击事件冒泡 -->
<a v-on:click.stop="doThis"></a>
<!-- 提交事件不再重载页面 -->
<form v-on:submit.prevent="onSubmit"></form>
<!-- 修饰符可以串联  -->
<a v-on:click.stop.prevent="doThat"></a>
<!-- 只有修饰符 -->
<form v-on:submit.prevent></form>
<!-- 添加事件侦听器时使用事件捕获模式 -->
<div v-on:click.capture="doThis">...</div>
<!-- 只当事件在该元素本身（而不是子元素）触发时触发回调 -->
<div v-on:click.self="doThat">...</div>
<!-- click 事件只能点击一次，2.1.4版本新增 -->
<a v-on:click.once="doThis"></a>
```

### 自定义组件

#### 全局组件

``` js
Vue.component(tagName, options)
<tagName></tagName>

<div id="app">
    <runoob></runoob>
</div>
 
<script>
// 注册
Vue.component('runoob', {
  template: '<h1>自定义组件!</h1>'
})
// 创建根实例
new Vue({
  el: '#app'
})
</script>
```

#### 局部组件

``` js
<div id="app">
    <runoob></runoob>
</div>
 
<script>
var Child = {
  template: '<h1>自定义组件!</h1>'
}
 
// 创建根实例
new Vue({
  el: '#app',
  components: {
    // <runoob> 将只在父模板可用
    'runoob': Child
  }
})
</script>
```

#### 动态 Prop

``` js
<div id="app">
    <div>
      <input v-model="parentMsg">
      <br>
      <child v-bind:message="parentMsg"></child>
    </div>
</div>
 
<script>
// 注册
Vue.component('child', {
  // 声明 props
  props: ['message'],
  // 同样也可以在 vm 实例中像 "this.message" 这样使用
  template: '<span>{{ message }}</span>'
})
// 创建根实例
new Vue({
  el: '#app',
  data: {
    parentMsg: '父组件内容'
  }
})
</script>

注意: prop 是单向绑定的：当父组件的属性变化时，将传导给子组件，但是不会反过来。
```

#### 自定义事件

``` js
父组件是使用 props 传递数据给子组件，但如果子组件要把数据传递回去，就需要使用自定义事件！
我们可以使用 v-on 绑定自定义事件, 每个 Vue 实例都实现了事件接口(Events interface)，即：

使用 $on(eventName) 监听事件
使用 $emit(eventName) 触发事件

另外，父组件可以在使用子组件的地方直接用 v-on 来监听子组件触发的事件。

<div id="app">
    <div id="counter-event-example">
      <p>{{ total }}</p>
      <button-counter v-on:increment="incrementTotal"></button-counter>
      <button-counter v-on:increment="incrementTotal"></button-counter>
    </div>
</div>
 
<script>
Vue.component('button-counter', {
  template: '<button v-on:click="incrementHandler">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    incrementHandler: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
})
new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
</script>
```

#### 自定义指令

##### 钩子函数

钩子函数的参数有：

* el: 指令所绑定的元素，可以用来直接操作 DOM 。
* binding: 一个对象，包含以下属性：
  * name: 指令名，不包括 v- 前缀。
  * value: 指令的绑定值， 例如： v-my-directive="1 + 1", value 的值是 2。
  * oldValue: 指令绑定的前一个值，仅在 update 和 componentUpdated 钩子中可用。无论值是否改变都可用。
  * expression: 绑定值的表达式或变量名。 例如 v-my-directive="1 + 1" ， expression 的值是 "1 + 1"。
  * arg: 传给指令的参数。例如 v-my-directive:foo， arg 的值是 "foo"。
  * modifiers: 一个包含修饰符的对象。 例如： v-my-directive.foo.bar, 修饰符对象 modifiers 的值是 { foo: true, bar: true }。
* vnode: Vue 编译生成的虚拟节点。
* oldVnode: 上一个虚拟节点，仅在 update 和 componentUpdated 钩子中可用。

``` js
/* 局部指令 */
<div id="app">
  <p>页面载入时，input 元素自动获取焦点：</p>
  <input v-focus>
</div>
 
<script>
// 创建根实例
new Vue({
  el: '#app',
  directives: {
    // 注册一个局部的自定义指令 v-focus
    focus: {
      // 指令的定义
      inserted: function (el) {
        // 聚焦元素
        el.focus()
      }
    }
  }
})
</script>

/* 全局指令 */
<div id="app">
    <div v-runoob="{ color: 'green', text: '菜鸟教程!' }"></div>
</div>
 
<script>
Vue.directive('runoob', function (el, binding) {
    // 简写方式设置文本及背景颜色
    el.innerHTML = binding.value.text
    el.style.backgroundColor = binding.value.color
})
new Vue({
  el: '#app'
})
</script>
```

### 路由

``` js
<router-link>
```

### 过度动画

``` js
<transition name = "nameoftransition">
   <div></div>
</transition>
```

### Ajax

待续...
