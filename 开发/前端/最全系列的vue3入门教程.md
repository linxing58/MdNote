# 最全系列的vue3入门教程
| [点击在线阅读，体验更好](https://link.juejin.cn/?target=https%3A%2F%2Fwww.coding-time.cn "https://www.coding-time.cn") | [链接](https://link.juejin.cn/?target=https%3A%2F%2Fwww.coding-time.cn "https://www.coding-time.cn") |
| --- | --- |
| **[现代JavaScript高级小册](https://link.juejin.cn/?target=https%3A%2F%2Fwww.coding-time.cn "https://www.coding-time.cn")** | [链接](https://link.juejin.cn/?target=https%3A%2F%2Fwww.coding-time.cn "https://www.coding-time.cn") |
| **[深入浅出Dart](https://link.juejin.cn/?target=https%3A%2F%2Fwww.coding-time.cn "https://www.coding-time.cn")** | [链接](https://link.juejin.cn/?target=https%3A%2F%2Fwww.coding-time.cn "https://www.coding-time.cn") |
| **[现代TypeScript高级小册](https://link.juejin.cn/?target=https%3A%2F%2Fwww.coding-time.cn "https://www.coding-time.cn")** | [链接](https://link.juejin.cn/?target=https%3A%2F%2Fwww.coding-time.cn "https://www.coding-time.cn") |
| **[linwu的算法笔记📒](https://link.juejin.cn/?target=https%3A%2F%2Fwww.coding-time.cn%2Flc "https://www.coding-time.cn/lc")** | [链接](https://link.juejin.cn/?target=https%3A%2F%2Fwww.coding-time.cn%2Flc "https://www.coding-time.cn/lc") |

Vue 3 简介
--------

Vue 3 是一个流行的开源JavaScript框架，用于构建用户界面和单页面应用。它带来了许多新特性和改进，包括更好的性能、更小的打包大小、更好的TypeScript支持、全新的组合式 API，以及一些新的内置组件。

### 1\. Vue 3 的新特性

Vue 3引入了许多新特性，包括：

*   组合式API：这是Vue 3最重要的新特性之一，它允许更灵活、更逻辑化地组织代码。
*   更好的性能：Vue 3的虚拟DOM重写，提供了更快的挂载、修补和渲染速度。
*   更小的打包大小：由于新的架构和树摇技术，Vue 3的打包大小比Vue 2小。
*   更好的TypeScript支持：Vue 3在内部使用了TypeScript，因此它为开发者提供了更好的TypeScript支持。

### 2\. 与 Vue 2 的区别

Vue 3与Vue 2的主要区别包括：

*   构建：Vue 3使用monorepo架构，更容易管理和维护。
*   API：Vue 3引入了新的组合式API，它提供了更灵活的代码组织方式。
*   性能：Vue 3提供了更好的性能，包括更快的渲染速度和更小的打包大小。
*   TypeScript：Vue 3提供了更好的TypeScript支持。

### 3\. 全新的核心架构

Vue 3的核心架构进行了全面的重写和优化，以提高性能和灵活性。此外，Vue 3还引入了许多新的API和组件，以满足现代web开发的需求。

基础
--

### 1\. 创建 Vue 3 项目

首先，我们需要通过Vite创建一个新的Vue 3项目。你可以通过下面的命令安装Vite：

```bash
pnpm create vite

```

然后，你可以通过下面的命令创建一个新的Vue 3项目：

```bash
pnpm create vite my-vue-app --template vue-ts

```

### 2\. 应用和组件编写

在Vue 3中，我们可以使用新的组合式API创建和管理组件。下面是一个简单的Vue 3组件示例：

```javascript
<template>
  <div>
    <p>{{ count }}</p>
    <button @click="increment">Increment</button>
  </div>
</template>

<script>
import { ref } from 'vue'

export default {
  setup() {
    const count = ref(0)
    const increment = () => { count.value++ }

    return {
      count,
      increment
    }
  }
}
</script>

```

在这个示例中，我们首先导入了 `ref` 函数，然后在 `setup` 函数中使用 `ref` 创建了一个响应式值 `count`。我们还定义了一个 `increment` 函数，用于增加 `count` 的值。最后，我们将 `count` 和 `increment` 返回给模板，以便在模板中使用它们。

响应式系统
-----

### 1\. 响应式原理

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9cfe6a0d0ddd43738241a7b4d04964dc~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)
 Vue的响应式系统是其核心特性之一，它使得数据变更能够自动更新到UI上。在Vue 3中，这个系统基于JavaScript的 `Proxy` 对象重写，提供了更好的性能和更多的功能。

### 2\. ref 和 reactive

Vue 3提供了两个主要的函数来创建响应式数据：`ref` 和 `reactive`。

`ref` 函数创建一个响应式引用。在模板中，你可以直接使用响应式引用的值，而在JavaScript代码中，你需要通过 `.value` 属性来访问或修改它的值。

```javascript
import { ref } from 'vue'

const count = ref(0)
console.log(count.value) 
count.value++ 

```

`reactive` 函数创建一个响应式对象。你可以直接访问和修改它的属性。

```javascript
import { reactive } from 'vue'

const state = reactive({ count: 0 })
console.log(state.count) 
state.count++ 

```

### 3\. 实现机制区别

`ref` 和 `reactive` 的主要区别在于，`ref` 是为了让基本类型（如数字和字符串）可以变为响应式，而 `reactive` 是为了让对象变为响应式。

`ref` 创建的响应式数据需要通过 `.value` 属性进行访问和修改，而 `reactive` 创建的响应式对象可以直接访问和修改其属性。因此，`ref` 更适合于处理基本类型，而 `reactive` 更适合于处理对象。

组合式 API
-------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3fafbfc601e0493aa27c32a3386f1c2f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)
 组合式API是Vue 3的重要新特性，它提供了一种更灵活、更逻辑化的方式来组织和复用代码。

### 1\. setup 函数

在Vue 3中，你可以使用 `setup` 函数来使用组合式API。`setup` 函数是组件的入口点，在组件实例被创建和初始化之后，但在渲染发生之前被调用。

```javascript
export default {
  setup() {
    
  }
}

```

### 2\. 响应式编程

你可以在 `setup` 函数中使用 `ref` 和 `reactive` 来创建响应式数据。

```javascript
import

 { ref, reactive } from 'vue'

export default {
  setup() {
    const count = ref(0)
    const state = reactive({ name: 'Vue' })

    return {
      count,
      state
    }
  }
}

```

### 3\. 计算属性与监视

你可以使用 `computed` 和 `watch` 来创建计算属性和监视响应式数据的变化。

```javascript
import { ref, computed, watch } from 'vue'

export default {
  setup() {
    const count = ref(0)
    const doubled = computed(() => count.value * 2)

    watch(count, (newValue, oldValue) => {
      console.log(`count changed from ${oldValue} to ${newValue}`)
    })

    return {
      count,
      doubled
    }
  }
}

```

### 4\. 生命周期钩子

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2710a3f6633041c3bc39c106a46a75ca~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)
 你可以在 `setup` 函数中使用生命周期钩子，比如 `onMounted`、`onUpdated` 和 `onUnmounted`。

```javascript
import { onMounted, onUpdated, onUnmounted } from 'vue'

export default {
  setup() {
    onMounted(() => {
      console.log('component mounted')
    })

    onUpdated(() => {
      console.log('component updated')
    })

    onUnmounted(() => {
      console.log('component unmounted')
    })
  }
}

```

### 5\. 自定义 hooks

你可以创建自定义的hooks来复用代码。一个自定义hook就是一个函数，它可以使用其他的响应式数据和组合式API。

```javascript
import { ref, onMounted } from 'vue'

function useCounter() {
  const count = ref(0)

  const increment = () => {
    count.value++
  }

  onMounted(() => {
    console.log('counter mounted')
  })

  return {
    count,
    increment
  }
}

export default {
  setup() {
    const counter = useCounter()

    return {
      counter
    }
  }
}

```

高级功能
----

### 1\. 浅层响应式

在某些情况下，你可能想要创建一个浅层的响应式对象，这样其内部的属性不会被转化为响应式的。这可以通过 `shallowReactive` 函数来实现。

```javascript
import { shallowReactive } from 'vue'

const state = shallowReactive({ count: 0 })

```

### 2\. 只读数据

你可以使用 `readonly` 函数来创建一个只读的响应式对象。任何试图修改只读对象的操作都将在开发环境下引发一个错误。

```javascript
import { readonly } from 'vue'

const state = readonly({ count: 0 })

```

### 3\. 自定义 Ref

你可以使用 `customRef` 函数来创建一个自定义的响应式引用。这允许你控制和观察当引用的值发生变化时的行为。

```javascript
import { customRef } from 'vue'

const count = customRef((track, trigger) => {
  let value = 0

  return {
    get() {
      track()
      return value
    },
    set(newValue) {
      value = newValue
      trigger()
    }
  }
})

```

### 4\. toRefs 和 toRef

当我们从 `setup` 函数返回一个响应式对象时，对象的属性将失去响应式。为了防止这种情况，我们可以使用 `toRefs` 或 `toRef` 函数。

```javascript
import { reactive, toRefs } from 'vue'

export default {
  setup() {
    const state = reactive({ count: 0 })

    return {
      ...toRefs(state)
    }
  }
}

```

新组件
---

### 1\. Fragment

在Vue 3中，你可以在一个组件的模板中有多个根节点，这被称为 Fragment。

```vue
<template>
  <div>Hello</div>
  <div>World</div>
</template>

```

### 2\. Teleport

Teleport 组件允许你将子组件渲染到DOM的任何位置，而不仅仅是它的父组件中。

```vue
<teleport to="#modal">
  <div>This will be rendered wherever the #modal element is.</div>
</teleport>

```

### 3\. Suspense

Suspense 组件允许你等待一个或多个异步组件，然后显示一些备用内容，直到所有的异步组件都被解析。

```vue
<Suspense>
  <template #default>
    <AsyncComponent />
  </template>
  <template #fallback>
    <div>Loading...</div>
  </template>
</Suspense>

```

深入编译优化
------

Vue 3在编译优化上做出了很大的提升。在编译过程中，Vue 3对模板进行静态分析，提取出不会改变的部分，预编译为纯JavaScript，这大大提高了运行时的渲染效率。下面详细介绍一下这些优化。

### 静态节点提升

在 Vue 3 中，如果你的模板中有不会改变的部分，例如：

```html
<template>
  <div>
    <h1>Hello, world!</h1>
    <p>Welcome to Vue 3</p>
  </div>
</template>

```

在编译时，"Hello, world!" 和 "Welcome to Vue 3" 这些静态节点将会被提升，避免了每次渲染时都需要重新创建。

### 片段、模板并用

在 Vue 3 中，你可以在一个组件模板中有多个根节点，这就是片段：

```html
<template>
  <header>Header</header>
  <main>Main content</main>
  <footer>Footer</footer>
</template>

```

在这个示例中，我们有三个根节点，这在 Vue 2 中是不允许的。但在 Vue 3 中，这是完全正常的。

### 动态编译

Vue 3 的动态编译可以让我们在运行时编译模板字符串，例如，我们可以动态创建一个`Hello组件`：

```javascript
import { compile, h } from 'vue'

const template = `<h1>{{ greeting }} World!</h1>`

const render = compile(template)

export default {
  data() {
    return {
      greeting: 'Hello',
    }
  },
  render
}

```

在这个例子中，我们在运行时编译了模板字符串，并将结果设置为组件的渲染函数。这种技术在需要动态生成模板的场景中非常有用，比如在一个 CMS 中渲染用户提供的模板。

### 深入组件化

Vue 3在组件化方面也有很多进步，下面详细介绍一下。

### 动态组件

Vue 3支持使用`component`标签来动态加载组件，组件可以是一个组件选项对象，也可以是一个组件的名字。这个特性在根据不同的状态显示不同组件时非常有用。

```vue
<component :is="currentComponent"></component>

```

### 异步组件

Vue 3 支持异步组件，你可以使用`defineAsyncComponent`方法来定义一个异步组件，该方法接收一个返回 Promise 的工厂函数，Promise 需要解析为一个组件。

```javascript
import { defineAsyncComponent } from 'vue'

const AsyncComponent = defineAsyncComponent(() =>
  import('./components/AsyncComponent.vue')
)

```

### 高阶组件

高阶组件（Higher-Order Component，简称 HOC）是一种设计模式，它是接收一个组件并返回一个新组件的函数。在 Vue 3 中，你可以使用组合式 API 来更容易地创建高阶组件。

```javascript
import { ref } from 'vue'

export default function withHover(Component) {
  return {
    setup(props, { slots }) {
      const isHovered = ref(false)

      return () => (
        <div
          onMouseenter={() => (isHovered.value = true)}
          onMouseleave={() => (isHovered.value = false)}
        >
          <Component {...props} isHovered={isHovered.value} v-slots={slots} />
        </div>
      )
    }
  }
}

```

在这个例子中，`withHover` 函数接收一个组件，并返回一个新的组件，新的组件有一个 `isHovered` 属性，表示鼠标是否悬停在组件上。这种模式可以帮助我们在不同的组件间复用逻辑。

其它组合API
-------

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d05799744a6341fd908ec03e5916d7b6~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### computed

`computed` 函数用于创建一个响应式的计算属性。这个属性的值是由提供的 getter 函数计算得出，并且只有当它的依赖项改变时才会重新计算。

```javascript
import { ref, computed } from 'vue'

export default {
  setup() {
    const count = ref(0)
    const doubleCount = computed(() => count.value * 2)

    return {
      count,
      doubleCount
    }
  }
}

```

在这个示例中，我们创建了一个 `doubleCount` 计算属性，它的值始终是 `count` 的两倍。

### watch 和 watchEffect

`watch` 和 `watchEffect` 函数用于观察一个或多个响应式引用或函数，并在其值改变时执行副作用。

```javascript
import { ref, watch, watchEffect } from 'vue'

export default {
  setup() {
    const count = ref(0)

    watch(count, (newCount, oldCount) => {
      console.log(`Count changed from ${oldCount} to ${newCount}`)
    })

    watchEffect(() => {
      console.log(`Count is ${count.value}`)
    })

    return {
      count
    }
  }
}

```

在这个示例中，我们使用 `watch` 函数观察 `count`，并在其值改变时打印消息。我们还使用 `watchEffect` 函数创建了一个副作用，它会在 `count` 改变时立即执行。

### lifecycle hooks

在 Vue 3 中，你可以在 `setup` 函数中直接使用生命周期钩子函数，例如 `onMounted`，`onUpdated` 和 `onUnmounted`。

```javascript
import { onMounted, onUpdated, onUnmounted } from 'vue'

export default {
  setup() {
    onMounted(() => {
      console.log('Component is mounted')
    })

    onUpdated(() => {
      console.log('Component is updated')
    })

    onUnmounted(() => {
      console.log('Component is unmounted')
    })
  }
}

```

在这个示例中，我们在组件挂载、更新和卸载时打印消息。

这些是 Vue 3 中提供的更为高级的响应式 API，让我们逐一了解它们。

### shallowReactive 与 shallowRef

`shallowReactive` 和 `shallowRef` 允许我们创建一个浅层的响应式对象。对于 `shallowReactive`，只有对象的第一层属性会变为响应式的，对象的更深层次的属性不会被转换。`shallowRef` 是 `ref` 的浅层版本，它不会自动解包内部的值。

```javascript
import { shallowReactive, shallowRef } from 'vue'

const obj = shallowReactive({ a: { b: 1 } })
obj.a.b 

const num = shallowRef(1)
num.value 

```

### readonly 与 shallowReadonly

`readonly` 和 `shallowReadonly` 允许我们创建一个只读的响应式对象。对于 `readonly`，对象的所有属性（包括嵌套属性）都会变为只读。`shallowReadonly` 是 `readonly` 的浅层版本，只有对象的第一层属性会变为只读。

```javascript
import { readonly, shallowReadonly } from 'vue'

const obj = readonly({ a: { b: 1 } })
obj.a = 2 

const shallowObj = shallowReadonly({ a: { b: 1 } })
shallowObj.a.b = 2 

```

### toRaw 与 markRaw

`toRaw` 和 `markRaw` 允许我们逃避 Vue 的响应式系统。`toRaw` 可以返回一个对象的原始版本，而 `markRaw` 可以防止一个对象被转换为响应式的。

```javascript
import { reactive, toRaw, markRaw } from 'vue'

const obj = reactive({ a: 1 })
const rawObj = toRaw(obj) 

const nonReactiveObj = markRaw({ a: 1 }) 

```

### customRef

`customRef` 允许我们创建一个自定义的 ref，我们可以控制它何时触发依赖追踪和更新。

```javascript
import { customRef } from 'vue'

const myRef = customRef((track, trigger) => ({
  get() {
    track()
    return someValue
  },
  set(newValue) {
    someValue = newValue
    trigger()
  }
}))

```

### provide 与 inject

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c7f14cb3e5945c183a6b4939d9d893f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)
 `provide` 和 `inject` 是 Vue 3 的依赖注入 API，可以用于在组件树中传递值，而不必一层一层地通过 props 传递。

```javascript
import { provide, inject } from 'vue'


provide('myValue', 123)


const myValue = inject('myValue') 

```

### 响应式判断

Vue 3 提供了 `isReactive` 和 `isRef` 函数，用于检查一个值是否是响应式的或者一个 ref。

```javascript
import { reactive, ref, isReactive, isRef } from 'vue'

const obj = reactive({ a: 1 })
isReactive(obj) 

const num = ref(1)
isRef(num) 

```

这些 API 提供了更多的控制和灵活性，使我们能够根据需要来选择如何使用 Vue 的响应式系统。

深入响应式系统
-------

Vue 3 的响应式系统构建在一个名为 `effect` 的函数基础之上，它被用来收集依赖项（依赖追踪）和触发副作用。当一个响应式对象的属性被访问时，`effect` 将其收集为依赖项；当一个响应式对象的属性被修改时，它将触发关联的副作用。

### effect、reactive、ref

`reactive` 和 `ref` 是 Vue 3 的两种基本响应式 API。它们都使用 `effect` 来跟踪依赖项和触发更新。

```javascript
import { effect, reactive, ref } from 'vue'


const state = reactive({ a: 1 })

effect(() => {
  console.log(state.a)
})

state.a = 2 


const count = ref(0)

effect(() => {
  console.log(count.value)
})

count.value++ 

```

在这个示例中，我们创建了一个响应式对象和一个 ref，然后使用 `effect` 创建了两个副作用，它们分别打印出对象和 ref 的值。当这些值被改变时，副作用就会被触发。

### track、trigger

`track` 和 `trigger` 是 Vue 3 的底层 API，它们分别被用来收集依赖项和触发更新。

```javascript
import { reactive, effect, track, trigger } from 'vue'

const state = reactive({ a: 1 })

effect(() => {
  
  track(state, 'a')
  console.log(state.a)
})

state.a = 2 


trigger(state, 'a')

```

在这个示例中，我们使用 `track` 手动收集了 `state.a` 作为依赖项，然后使用 `trigger` 手动触发了更新。

### 嵌套结构处理

Vue 3 的响应式系统可以处理嵌套的响应式对象。

```javascript
import { reactive, effect } from 'vue'

const state = reactive({
  user: {
    name: 'Alice'
  }
})

effect(() => {
  console.log(state.user.name)
})

state.user.name = 'Bob' 

```

在这个示例中，我们创建了一个嵌套的响应式对象，并使用 `effect` 创建了一个副作用，它打印出用户的名字。当用户的名字被改变时，副作用就会被触发。

Render 函数
---------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/adb7b08090dc443587709e30abd5ad41~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)
 在 Vue 3 中，Render 函数是一种提供了更大灵活性的高级功能。虽然 Vue 的模板系统已经足够强大，但在某些情况下，直接使用 JavaScript 编写渲染逻辑会更加方便。

Render 函数的工作原理是通过返回一个虚拟节点（VNode）来告诉 Vue 如何渲染界面。Vue 3 提供了 `h` 函数用于创建 VNode。

```javascript
import { h } from 'vue'

export default {
  render() {
    return h('div', {}, 'Hello, world!')
  }
}

```

在这个例子中，我们使用 `h` 函数创建了一个 `div` 元素，然后在 Render 函数中返回它。

### 编译优化

> 参考上一章节

Vue 3 的编译器在编译时做了许多优化，例如静态节点提升和动态节点绑定，从而在运行时减少了不必要的工作。静态节点提升可以将不会改变的节点从渲染函数中提取出来，从而避免在每次渲染时都重新创建它们。动态节点绑定则是对那些可能改变的节点进行优化，只有当这些节点的绑定值发生变化时，才会重新渲染节点。

### 手动编写渲染逻辑

有时，我们可能需要手动编写渲染逻辑。比如，当我们需要根据一组数据动态生成一个列表时，我们可以在 Render 函数中使用 JavaScript 的数组方法。

```javascript
import { h } from 'vue'

export default {
  data() {
    return {
      items: ['Apple', 'Banana', 'Cherry']
    }
  },
  render() {
    return h('div', {}, this.items.map(item => h('div', {}, item)))
  }
}

```

在这个例子中，我们使用 `map` 方法动态生成了一个列表的元素，然后在 Render 函数中返回它。

Render 函数提供了一种强大的方式来控制 Vue 应用的渲染过程，使得我们能够更好地控制和优化应用的性能。

vue生态配套
-------

### 状态管理

Pinia 是 Vue 3 提供的一种新型状态管理库，它提供了 Vuex 的核心功能，但在 API 设计上更加简洁且易用。

Pinia 的主要优点包括：

1.  它有更简洁的 API，减少了模板代码的数量。
2.  它通过 TypeScript 提供了更好的类型支持。
3.  它提供了基于组件的状态存储，只在需要时加载状态。

下面是一个 Pinia 使用示例：

首先，安装 Pinia：

```bash
npm install pinia

```

然后，创建一个 Pinia store：

```javascript

import { defineStore } from 'pinia'

export const useCounterStore = defineStore({
  id: 'counter',
  state: () => ({ count: 0 }),
  actions: {
    increment() {
      this.count++
    }
  }
})

```

然后，在你的 main.js 或 main.ts 文件中创建 Pinia 插件，并将其添加到你的 Vue 应用中：

```javascript
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

const app = createApp(App)

app.use(createPinia())
app.mount('#app')

```

然后，你可以在组件中使用该 store：

```vue
<template>
  <button @click="increment">{{ count }}</button>
</template>

<script>
import { useCounterStore } from '@/stores/counter'

export default {
  setup() {
    const counter = useCounterStore()

    return {
      count: counter.count,
      increment: counter.increment
    }
  }
}
</script>

```

以上就是 Pinia 的基本用法，它提供了一种更简洁、更灵活的方式来管理 Vue 应用的状态。

### 路由管理 \- Vue Router

Vue Router 是 Vue.js 的路由库，你可以使用它构建单页面应用程序，如下面的例子所示：

首先，创建一个 router：

```javascript
import { createRouter, createWebHistory } from 'vue-router'
import Home from './components/Home.vue'
import About from './components/About.vue'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    { path: '/', component: Home },
    { path: '/about', component: About }
  ]
})

```

然后，在 Vue 应用中使用这个 router：

```javascript
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

createApp(App).use(router).mount('#app')

```

最后，在组件中使用 `<router-link>` 和 `<router-view>` 来导航和渲染路由：

```vue
<template>
  <router-link to="/">Home</router-link>
  <router-link to="/about">About</router-link>
  <router-view></router-view>
</template>

```

### UI 框架 - Element UI

Element UI 是一个为 Vue.js 开发的 UI 框架，它提供了一套丰富多样的组件，可以帮助我们更容易地构建出漂亮的界面：

```vue
<template>
  <el-button type="primary">主要按钮</el-button>
  <el-button type="success">成功按钮</el-button>
  <el-button type="warning">警告按钮</el-button>
</template>

```

### 测试方案 \- Vitest

Vitest 是一个由 Vite 提供支持的极速单元测试框架。

```javascript
import { assert, describe, it } from 'vitest'

describe.skip('skipped suite', () => {
  it('test', () => {
    
    assert.equal(Math.sqrt(4), 3)
  })
})

describe('suite', () => {
  it.skip('skipped test', () => {
    
    assert.equal(Math.sqrt(4), 3)
  })
})

```

其他改变
----

在 Vue 3 中，开发者会注意到一些重要的变化，主要体现在全局 API 的转移等，以及对 TypeScript 的更好支持上。

### 全局 API 转移

在 Vue 3 中，一些全局 API 已经被转移到了 `globalProperties` 上，例如 `Vue.prototype` 在 Vue 3 中变成了 `app.config.globalProperties`。这样做是为了更好地隔离全局 API，并为未来可能的更改提供更大的灵活性。

例如，原本在 Vue 2 中我们可能会这样添加一个全局的方法：

```javascript
Vue.prototype.$myGlobalMethod = function () {
  return 'Hello World!'
}

```

在 Vue 3 中，我们需要这样做：

```javascript
const app = createApp(App)
app.config.globalProperties.$myGlobalMethod = function () {
  return 'Hello World!'
}

```

然后我们就可以在任何组件中使用这个方法：

```javascript
this.$myGlobalMethod()

```

### 删除的 API

Vue 3 为了简化框架并避免未来的维护负担，删除了一些在 Vue 2 中已经被废弃的 API，例如 `Vue.set`、`Vue.delete` 和 `Vue.observable`。

### TypeScript 支持

Vue 3 从一开始就在内部使用了 TypeScript 重写，因此在 TypeScript 的支持上有了显著的提升。这包括更好的类型推断、自动补全，以及更强大的类型安全性。

例如，在 Vue 2 中，我们可能需要使用 Vue.extend() 或者 `@Component` 装饰器来确保 TypeScript 类型正确，但在 Vue 3 中，我们可以直接使用 `defineComponent` 方法，它能正确地推断出组件的类型：

```typescript
import { defineComponent } from 'vue'

export default defineComponent({
  
})

```