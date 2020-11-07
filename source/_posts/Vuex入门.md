---
title: Vuex入门
date: 2020-11-04 15:29:35
description: Vuex笔记
categories: Vue
tags: 
	- Vue
	- Vuex
---

![Centralized State Management for Vue.js.](https://raw.githubusercontent.com/vuejs/vuex/dev/docs/.vuepress/public/vuex.png)

### 项目初始化

初始化项目，项目目录为：

```txt
│  .gitignore
│  babel.config.js
│  package-lock.json
│  package.json
│  README.md
│  
├─node_modules       
├─public
│      favicon.ico
│      index.html
│      
└─src
    │  App.vue
    │  main.js
    │  
    ├─assets
    │      logo.png
    │      
    ├─components
    │      HelloWorld.vue
    │      
    └─store
            index.js
```

初始`App.vue`

```vue
<template>
  <div id="app">
    <HelloWorld />
  </div>
</template>

<script>
import HelloWorld from "./components/HelloWorld.vue";

export default {
  name: "App",
  components: {
    HelloWorld,
  },
};
</script>

<style>
</style>
```

初始`HelloWorld.vue`

```vue
<template>
  <div class="hello">
    <h1>Vuex 示例</h1>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
};
</script>

<style scoped>
</style>

```

初始化的`Vuex`在`/src/store`目录下的`index.js`:

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
    state: {},
    mutations: {},
    actions: {},
    modules: {}
})
```

此时页面显示效果：

![image-20201104165246857](https://cdn.jsdelivr.net/gh/Wanfengyueluo/images/image-20201104165246857.png)

### state

在组件中引入`Vuex`中的状态

1. 直接引入

   直接修改`HelloWorld.vue`的h标签

   ```html
   <h1>{{ this.$store.state.msg }}</h1>
   ```

   在`index.js`中加入

   ```js
   state: {
           msg: 'this is vuex --- Vuex'
       },
   ```

   显示结果为：

   ![image-20201105084551745](https://cdn.jsdelivr.net/gh/Wanfengyueluo/images/image-20201105084551745.png)

2. 计算属性引入

   在`HelloWorld.vue`中增加计算属性，h标签和上面保持一致

   ```js
   export default {
       name: "HelloWorld",
       computed: {
           msg() {
               return this.$store.state.msg
           }
       }
   };
   ```

   显示结果为：

   ![image-20201105085912982](https://cdn.jsdelivr.net/gh/Wanfengyueluo/images/image-20201105085912982.png)

3. `mapState`辅助函数

   修改`HelloWorld.vue`

   ```vue
   <template>
   <div class="hello">
       <h1>{{ msg }}</h1>
       <h1>{{ msgAlias }}</h1>
       <h1>{{ msgAddLocation }}</h1>
   </div>
   </template>
   
   <script>
   import {
       mapState
   } from 'vuex'
   export default {
       name: "HelloWorld",
       data(){
         return{
           localMsg:'local'
         }
       },
       computed: mapState({
           msg: state => state.msg,
           msgAlias: 'msg',
           msgAddLocation(state) {
             return state.msg+this.localMsg
           }
       })
   };
   </script>
   
   ```

   显示结果为：

   ![image-20201105091214317](C:\Users\wan\AppData\Roaming\Typora\typora-user-images\image-20201105091214317.png)
   
   当映射的计算属性名称和`state`的子节点名称相同，可以给`mapState`传一个字符串数组
   
   ```js
   computed: mapState([
           'msg'
       ])
   ```
   
   此时显示结果为：
   
   ![image-20201105091719949](C:\Users\wan\AppData\Roaming\Typora\typora-user-images\image-20201105091719949.png)


4. 对象展开运算符`...mapState`

   ```js
   computed:{
       ...mapState(['msg']),
       ...mapState({
           message:'msg'
       })
   }
   ```

   

### Getter

`getter`可以视为`store`的计算属性，其第一个参数为`state`

修改`index.js`：

```js
export default new Vuex.Store({
    state: {
        msg: 'this is vuex --- Vuex by mapState',
        name: 'wonderful '
    },
    getters: {
        getMsg: state => state.msg,
        getName: (state, getters) => state.name + getters.getMsg
    },
    //......
})
```

修改`HelloWorld.vue`

```js
computed: {
    msg() {
        return this.$store.getters.getName
    }
}
```

显示结果为：

![image-20201105093838536](https://cdn.jsdelivr.net/gh/Wanfengyueluo/images/image-20201105093838536.png)

`mapGetters`辅助函数

修改`HelloWorld.vue`：

```vue
<template>
<div class="hello">
    <h1>{{ msg }}</h1>
    <h1>{{ name }}</h1>
</div>
</template>

<script>
import {
    mapGetters
} from 'vuex'
export default {
    name: "HelloWorld",
    computed: mapGetters({
      msg: 'getMsg',
      name: 'getName'
    })
};
</script>

<style scoped>
</style>

```

使用对象展开运算符`...mapGetters`：

```vue
<template>
<div class="hello">
    <h1>{{ getMsg }}</h1>
    <h1>{{ getName }}</h1>
</div>
</template>

<script>
import {
    mapGetters
} from 'vuex'
export default {
    name: "HelloWorld",
    computed: {
        ...mapGetters(['getMsg', 'getName'])
    }
};
</script>

<style scoped>
</style>

```

上面两种写法的显示结果为：

![image-20201105095102215](https://cdn.jsdelivr.net/gh/Wanfengyueluo/images/image-20201105095102215.png)

### Mutation

更改`Vuex`的`store`中状态的唯一方法是提交`mutation`

修改的`HelloWorld.vue`

```vue
<template>
<div class="hello">
    <h1 @mouseover="onmouseover">{{ count }}</h1>
</div>
</template>

<script>
import {
    mapGetters
} from 'vuex'
export default {
    name: "HelloWorld",
    computed: mapGetters({
      msg: 'getMsg',
      name: 'getName',
      count: 'getCount'
    }),
    methods:{
      onmouseover(){
        this.$store.commit('increment')
      },
    }
};
</script>

<style scoped>
</style>
```

修改后的`index.js`

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
    state: {
        msg: 'this is vuex --- Vuex by mapState',
        name: 'wonderful ',
        count: 1
    },
    getters: {
        getMsg: state => state.msg,
        getName: (state, getters) => state.name + getters.getMsg,
        getCount: state => state.count
    },
    mutations: {
        increment(state) {
            state.count++
        }
    },
    actions: {},
    modules: {}
})
```

此时鼠标在数字上移动时，数字会递增。

在提交`commit`时也可以传入额外的参数，即`mutation`的载荷(`payload`)

对上述例子进行少许改动：

`HelloWorld.vue`：

```js
methods:{
    onmouseover(){
        this.$store.commit('increment',10)
    },
}
```

`index.js`：

```js
mutations: {
    increment(state, payload) {
        state.count += payload
    }
},
```

对于`payload`，也可以使用对象风格：

```vue
<template>
<div class="hello">
    <h1 @mouseover="onmouseover">{{ count+' '+name }}</h1>
</div>
</template>

<script>
import {
    mapGetters
} from 'vuex'
export default {
    name: "HelloWorld",
    computed: mapGetters({
        msg: 'getMsg',
        name: 'getName',
        count: 'getCount'
    }),
    methods: {
        onmouseover() {
            this.$store.commit('increment', {
                count: 10,
                msg: 'new',
                name: 'thanks'
            })
        },
    }
};
</script>

<style scoped>
</style>

```

`index.js`：

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
    state: {
        msg: 'msg',
        name: 'name ',
        count: 1
    },
    getters: {
        getMsg: state => state.msg,
        getName: (state, getters) => state.name + getters.getMsg,
        getCount: state => state.count
    },
    mutations: {
        increment(state, payload) {
            state.count += payload.count
            state.msg += payload.msg
            state.name += payload.name
        }
    },
    actions: {},
    modules: {}
})
```

`mapMutations`辅助函数

修改`HelloWorld.vue`

```vue
<template>
<div class="hello">
    <h1 @mouseover="onmouseover">{{ count }}</h1>
</div>
</template>

<script>
import {
    mapGetters,
    mapMutations
} from 'vuex'
export default {
    name: "HelloWorld",
    computed: mapGetters({
        msg: 'getMsg',
        name: 'getName',
        count: 'getCount'
    }),
    methods: {
        onmouseover() {
            this.add({
                count: 10
            })
        },
        ...mapMutations(['increment']),
        ...mapMutations({
          add:'increment'//将increment映射为add
        })
    }
};
</script>

<style scoped>
</style>

```

> 在 `Vuex` 中，**`mutation `都是同步事务**,`store.commit('increment')`中，任何由`increment`导致的状态变更都应该在此刻完成。

### Action

`Action`类似于`mutation`，不同在于：

- `Action`提交的是`mutation`，而不是直接变更状态
- `Action`可以包含任意**异步**操作

注册一个`Action`：

```js
actions: {
    increment(context) {
        context.commit('increment')
    }
}
```

上述写法等同于以下写法(**ES2015 的 [参数解构](https://github.com/lukehoban/es6features#destructuring)**)：
```js
actions: {
    increment({commit}) {
        commit('increment')
    }
}
```

`Action`函数接受一个与`store`实例具有相同方法和属性的`context`对象，因此可以调用`context.commit`提交`mutation`，或者通过`context.state`和`context.getters`来获取`state`和`getters`

> `context`对象并不是`store`实例本身

`Action`通过`store.dispatch`方法触发：

`store.dispatch('increment')`

修改`HelloWorld.vue`中的`mouseover`函数：

```js
onmouseover() {
	this.$store.dispatch('increment')
}
```

不同于`mutation`，我们可以在`Action`中执行异步操作：

```js
increment(context) {
    setTimeout(() => {
        context.commit('increment')
    }, 1000)
}
```

`Actions `支持同样的载荷方式和对象方式进行分发：

```js
// 以载荷形式分发
store.dispatch('increment', {
  amount: 10
})

// 以对象形式分发
store.dispatch({
  type: 'increment',
  amount: 10
})
```

`mapActions`辅助函数

修改`HelloWorld.vue`

```js
import {
    mapGetters,
    mapMutations,
    mapActions
} from 'vuex'
export default {
    name: "HelloWorld",
    computed: mapGetters({
        msg: 'getMsg',
        name: 'getName',
        count: 'getCount'
    }),
    methods: {
        onmouseover() {
            this.increment()
            //this.add()
        },
        ...mapActions(['increment']),
        ...mapActions({
          add: 'increment'
        })
    }
};
```

`Action`组合

### Module

