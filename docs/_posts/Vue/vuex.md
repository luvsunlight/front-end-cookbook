## 应用

### 定位和作用

vuex是一个专门为Vue.js开发的状态管理工具，平时的应用场景比较多的大型单页应用中组件间数据的同步和管理，本质上是一个全局的单例模式

首先在`new Vue`中注入Vue实例，然后Vue实例包括组件（组件可以认为是根Vue实例新建的）都可以访问到这个全局唯一的$store对象

Vuex的使用过程中包含五个核心概念

* state
* getter
* muatation
* actions
* module

### state

state属性即为单例模式的状态，它是最为核心的内容，全部操作都围绕着它展开

### getter

getter属性用于获取可复用的复杂state状态，它默认会有两个参数，一个是state的引用，一个是getters

### mutation

它是唯一改变单例模式中状态的途径，它的参数是一个state（可能还有payload），正常情况下，我们无法直接使用mutation，而是要通过`store.commit('mutation_name')`的方法

* Mutation必须是同步函数（不然的话debug快照就没有意义了）
* store中的属性最好事先声明，不然没有响应式
* 用常量来代替Mutation类型

### Actions

它和Mutation相比，有两个特点，一个是它不是直接改变状态，二是它可以包含异步操作。它的参数是一个`context`，和前面不一样的是并不是单指当前的store实例，作用方式是context.commit('mutation',payload)

### Modules

当项目的模块比较多的时候，单一状态管理起来比较臃肿也复杂，我们需要根据功能对状态进行划分，比如登录模块，购物车模块，商品模块等等。

我们可以定义不同的module，这些对象和正常的store对象一样具有state,getter,mutation,actions等属性，最后我们在主store对象上，赋予modules属性（这也是为什么actions里是context）

* getter中除了state，getters还有rootState，rootGetters
* 正常情况下，module里注册的getters，mutations，actions都是注册在全局作用域中的，需要我们设置namespace:true，子模组也会继承这个namespace

### 插件

vuex提供插件机制，使用方式就是在store定义中加入plugins选项。在编写插件的时候，它暴露的是store实例自身，插件也是发布-订阅的模式

```
const myPlugin = store => {
  // 当 store 初始化后调用
  store.subscribe((mutation, state) => {
    // 每次 mutation 之后调用
    // mutation 的格式为 { type, payload }
  })
}
```

## 原理

### Vuex实例是如何注入Vue实例中的

我们首先来看Vuex正常用法

```
# store.js

Vue.use(Vuex)
export default Vuex.Store({...})

# main.js

import store from './store'

new Vue({
    ...,
    store
})
```

可以看出来Vuex的注入有三个步骤

* Vue.use(Vuex).即调用Vuex对象的install方法
* 使用Vuex中的Store类构造实例
* 将Store实例注入Vue实例

```
export function install (_Vue) {
  if (Vue && _Vue === Vue) {
    if (process.env.NODE_ENV !== 'production') {
      console.error(
        '[vuex] already installed. Vue.use(Vuex) should be called only once.'
      )
    }
    return
  }
  Vue = _Vue
  applyMixin(Vue)
}
```

_Vue即Vue在调用install时将自己传进来，在运行完`Vue = _Vue`这段代码之后，该模块的全局变量中Vue的值就等于Vue了，然后是applyMixin

```
# mixin.js

export default function (Vue) {
  const version = Number(Vue.version.split('.')[0])

  if (version >= 2) {
    Vue.mixin({ beforeCreate: vuexInit })
  } else {
    // override init and inject vuex init procedure
    // for 1.x backwards compatibility.
    const _init = Vue.prototype._init
    Vue.prototype._init = function (options = {}) {
      options.init = options.init
        ? [vuexInit].concat(options.init)
        : vuexInit
      _init.call(this, options)
    }
  }

  /**
   * Vuex init hook, injected into each instances init hooks list.
   */

  function vuexInit () {
    const options = this.$options
    // store injection
    if (options.store) {
      this.$store = typeof options.store === 'function'
        ? options.store()
        : options.store
    } else if (options.parent && options.parent.$store) {
      this.$store = options.parent.$store
    }
  }
}

```

可以看出来applyMixin()函数主要作用就是在Vue实例的生命周期为`beforeCreate时`通过调用`Vue.mixin`或者低版本下自己绑定，实现每个Vue实例都有一个$store属性，并且指向的都是同一个store，指向方法是Vue实例init时的store方法

总结一下，Vue.use(vuex)时就会默认调用vuex的install方法，这个方法一个是将Vue绑定到模块里的全局变量中，另一个是调用applyMixin方法，而这个方法的作用是设置了Vue实例在beforeCreate阶段时的钩子，此时Vue实例会试图访问init时的options里的store属性，如果有的话，则将本实例的$store属性指向那个store实例，如果没有则从parent对象中找，这样做就实现了所有注册的Vue实例都拥有访问store实例的权利了