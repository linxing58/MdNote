# 🍍(Pinia)不酸，保甜 - 掘金
_**本文正在参加[「金石计划」](https://juejin.cn/post/7207698564641996856/ "https://juejin.cn/post/7207698564641996856/")**_

### 为什么是Pinia

怎么说呢，其实在过往的大部分项目里面，我并没有引入过状态管理相关的库来维护状态。因为大部分的业务项目相对来说比较独立，哪怕自身功能复杂的时候，可能也仅仅是通过技术栈自身的提供的状态管理能力来处理业务场景问题，比如React中的context，基本都能解决我遇到的问题。

针对Redux或者Vuex这类状态管理的库，我认为在较为复杂的大型业务场景下才能发挥他们的真实作用，在场景较为简单单一，数据状态相对不复杂的时候，引入他们可能并不能带来那么多的便捷。

说回Pinia，接触使用到它主要是因为有一次手里的发布功能需要进行重构。虽然仅仅是一个发布页面，但是梳理起来发现，里面涉及几类数据需要维护，包括主表单信息、错误校验信息、公共弹窗信息以及关联用户的各种状态信息等。想起之前有看到过关于小🍍的介绍，抱着试试看的心态接入试试。（不行我就继续Provide/Inject了\[捂脸\]）

### 认识Pinia

> Pinia 是 Vue 的存储库，它允许您跨组件/页面共享状态。

上面这个是官网对Pinia的一个定义，从定义上我们其实可以看出来，它**可能**比Vuex要精炼一些（Vuex 是一个专为 Vue.js 应用程序开发的**状态管理模式 \+ 库**。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。）具体如何我们还是得具体从使用情况来看一下。

### Pinia的常规用法

#### 安装

通过常用的包管理器进行安装：

```csharp
yarn add pinia

npm install pinia

```

安装完成后，我们需要将pinia挂载到Vue应用中，也就是我们需要创建一个根存储传递给应用程式。我们需要修改main.js，引入pinia提供的cteatePinia方法：

```ini
import { createApp } from 'vue'
import { createPinia } from 'pinia'
​
const pinia = createPinia()
const app = createApp(App)
app.use(pinia).mount('

```

上述安装引入基于Vue3，如果使用Vue2的话，轻参照[官网相关](https://link.juejin.cn/?target=https%3A%2F%2Fpinia.web3doc.top%2Fgetting-started.html%23%25E5%25AE%2589%25E8%25A3%2585 "https://pinia.web3doc.top/getting-started.html#%E5%AE%89%E8%A3%85")说明即可。

#### Store

store简单来说就是数据仓库的意思，我们 的数据都放在store里面。当然你也可以把它理解为一个公共组件，只不过该公共组件只存放数据，这些数据我们其它所有的组件都能够访问且可以修改。它有三个概念，**state**、**getters**和**actions**，我们可以将它们等价于组件中的“**数据**”、“**计算属性**”和“**方法**”。

store中应该包含可以在整个应用中访问的数据、全局性数据，我们应该避免将可以管理在具体组件内部的数据放到store中。

我们需要使用pinia提供的`defineStore()`方法来创建一个store，该store用来存放我们需要全局使用的数据。我们可以在项目中创建store目录存储我们定义的各种store：

```javascript

import { defineStore } from 'pinia';
​

const useFormInfoStore = defineStore('formInfo', {
  
})
​
export default useFormInfoStore;

```

defineStore接收两个参数：

*   name：一个字符串，必传项，该store的唯一id。
*   options：一个对象，store的配置项，比如配置store内的数据，修改数据的方法等等。

将返回的函数命名为 _use..._ 是组合式开发的约定，使其符合使用习惯。我们可以根据项目情况定义任意数量的store存储不同功能模块的数据，一个store就是一个函数，它和Vue3的实现思想也是一致的。

##### 使用store

我们可以在任意组件中引入定义的store来进行使用

```xml
<script setup>

import useFormInfoStore from '@/store/formInfo';

const formInfoStore = useFormInfoStore();
</script>

```

store 被实例化后，你就可以直接在 store 上访问 `state`、`getters` 和 `actions` 中定义的任何属性。

##### 解构store

`store` 是一个用`reactive` 包裹的对象，这意味着不需要在getter 之后写`.value`，但是，就像`setup` 中的`props` 一样，**我们不能对其进行解构**，如果我们想要提取store中的属性同时保持其响应式的话，我们需要使用`storeToRefs()`，它将为响应式属性创建refs。

```xml
<script setup>
import { storeToRefs } from 'pinia';

import useFormInfoStore from '@/store/formInfo';

const formInfoStore = useFormInfoStore();
​
const { name, age } = formInfoStore; 
​
const { name, age } = storeToRefs(formInfoStore); 
</script>

```

#### State

store是用来存储全局状态数据的仓库，那自然而然需要有地方能够保存这些数据，它们就保存在state里面。defineStore传入的第二个参数options配置项里面，就包括state属性。

```javascript

import { defineStore } from 'pinia';
​
const useFormInfoStore = defineStore('formInfo', {
   
   state: () => {
      return {
        
        name: 'Hello World',
        age: 18,
        isStudent: false
      }
   }
  
   
   
   
   
   
   
})
​
export default useFormInfoStore;

```

##### 访问state

默认情况下，您可以通过 `store` 实例来直接读取和写入状态:

```xml
<script setup>
import useFormInfoStore from '@/store/formInfo';
const formInfoStore = useFormInfoStore();
​
console.log(formInfoStore.name); 
formInfoStore.age++;  
</script>

```

pinia还提供了几个常见场景的方法供我们使用来操作state：`$reset`、`$patch`、`$state`、`$subscribe`：

```xml
<script setup>
import useFormInfoStore from '@/store/formInfo';
const formInfoStore = useFormInfoStore();
​
console.log(formInfoStore.name); 

formInfoStore.age++;  
​

formInfoStore.$reset();
console.log(formInfoStore.age); 
  

formInfoStore.$patch({
    name: 'hello Vue',
    age: 198
});
  

formInfoStore.$state = {
  name: 'hello Vue3',
  age: 100,
  gender: '男'
}
​

formInfoStore.$subscribe((mutation, state) => {
  
}, {
  detached: true  
})
</script>

```

针对上面示例，有几点需要关注一下：

*   1.同直接修改state中的属性不同，通过$patch方法更新多个属性时，在devtools的timeline中，是合并到一个条目中的。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d02516329a0744928da78033770e38b3~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

*   2.通过实验得知，$state在进行替换时，会更新已有的属性，新增原来state不存在的属性，针对不在替换范围内的，则保持不变。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f0151a4e738b49e79e68bf680e57ab21~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

如上图，针对gender属性，执行的是"add"操作，然后这个替换过程我们没有设置isStudent属性，它仍然保持原状态在state中不变。

*   3.subscribe只会订阅到patches引起的变化，即上面的通过subscribe只会订阅到patches引起的变化，即上面的通过patch方法和$state覆盖时会触发到状态订阅，直接修改state的属性则不会触发。

#### Getters

pinia中的getters可以完全等同于Store状态的`计算属性`，通常在defineStore中的getters属性中定义。同时支持组合多个getter，此时通过`this`访问其他getter。

```javascript
import { defineStore } from 'pinia';
​
const useFormInfoStore = defineStore('formInfo', {
    state: () => {
        return {
            name: 'Hello World',
            age: 18,
            isStudent: false,
            gender: '男'
        };
    },
    getters: {
        
        isMan: (state) => {
            return state.gender === '男';
        },
        isWoman() {
            
            return !this.isMan;
        }
    }
});
​
export default useFormInfoStore;
​

```

在使用时，我们可以直接在store实例上面访问getter:

```xml
<template>
  <div>The person is Man: {{ formInfoStore.isMan }} or is Woman: {{ formInfoStore.isWoman }}</div>
</tempalte>
​
<script setup>
import useFormInfoStore from '@/store/formInfo';
const formInfoStore = useFormInfoStore();
</script>

```

通常getter是不支持额外传参的，但是我们可以从getter返回一个函数的方式来接收参数：

```javascript
import { defineStore } from 'pinia';
​
const useFormInfoStore = defineStore('formInfo', {
    state: () => {
        return {
            name: 'Hello World',
            age: 18,
            isStudent: false,
            gender: '男'
        };
    },
    getters: {
        isLargeBySpecialAge: (state) => {
          return (age) => {
             return state.age > age
          }
        }
    }
});
​
export default useFormInfoStore;

```

在组件中使用时即可传入对应参数，注意，在这种方式时，**getter不再具有缓存性**。

```xml
<template>
  <div>The person is larger than 18 years old? {{ formInfoStore.isLargeBySpecialAge(18) }}</div>
</tempalte>
​
<script setup>
import useFormInfoStore from '@/store/formInfo';
const formInfoStore = useFormInfoStore();
</script>

```

#### Actions

actions相当于组件中的methods，它们定义在defineStore中的actions属性内，常用于定义业务逻辑使用。**action可以是异步的**，可以在其中`await` 任何 API 调用甚至其他操作

```javascript
import { defineStore } from 'pinia';
​
const useFormInfoStore = defineStore('formInfo', {
    state: () => {
        return {
            name: 'Hello World',
            age: 18,
            isStudent: false,
            gender: '男'
        };
    },
    getters: {
        isMan: (state) => {
            return state.gender === '男';
        },
        isWoman() {
            return !this.isMan;
        }
    },
    actions: {
        incrementAge() {
            this.age++;
        },
        async registerUser() {
            try {
                const res = await api.post({
                    name: this.name,
                    age: this.age,
                    isStudent: this.isStudent,
                    gender: this.gender
                });
                showTips('用户注册成功～');
            } catch (e) {
                showError('用户注册失败！');
            }
        }
    }
});
​
export default useFormInfoStore;
​

```

使用起来也非常方便

```xml
<script setup>
import useFormInfoStore from '@/store/formInfo';
const formInfoStore = useFormInfoStore();
  
const registerUser = () => {
  formInfoStore.registerUser();
}
</script>

```

##### $onAction()

可以使用 `store.$onAction()` 订阅 action 及其结果。传递给它的回调在 action 之前执行。 `after` 处理 Promise 并允许您在 action 完成后执行函数，`onError` 允许您在处理中抛出错误。

```javascript
const unsubscribe = formInfoStore.$onAction(
  ({
    name, // action 的名字
    store, // store 实例
    args, // 调用这个 action 的参数
    after, // 在这个 action 执行完毕之后，执行这个函数
    onError, // 在这个 action 抛出异常的时候，执行这个函数
  }) => {
    
    const startTime = Date.now()
    
    console.log(`Start "${name}" with params [${args.join(', ')}].`)
​
    
    
    after((result) => {
      console.log(
        `Finished "${name}" after ${
          Date.now() - startTime
        }ms.\nResult: ${result}.`
      )
    })
​
    
    onError((error) => {
      console.warn(
        `Failed "${name}" after ${Date.now() - startTime}ms.\nError: ${error}.`
      )
    })
  }
)
​

unsubscribe()

```

和$subscribe类似，在组件中使用时，组件卸载，订阅也会被删除，如果希望保留的话，需要传入true作为第二个参数。

```xml
<script setup>
import useFormInfoStore from '@/store/formInfo';
const formInfoStore = useFormInfoStore();
​
formInfoStore.$onAction(callback, true);
</script>

```

Pinia的基础使用我们暂时介绍到这里，其他使用场景大家可以参照官方文档进一步学习。

### Pinia VS Vuex

回过头来，我们再来看一下，Pinia为什么现在这么受到推崇。和我们过往常用的Vuex相比，它到底好在哪里呢？

对于Vuex，我们知道，它的背后基本思想借鉴了Flux。Flux 是 Facebook 在构建大型 Web 应用程序时为了解决**数据一致性**问题而设计出的一种架构，它是一种描述状态管理的设计模式。绝大多数前端领域的状态管理工具都遵循这种架构，或者以它为参考原型。Vuex在它的基础上进行了一些设计上的优化：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6927588a06964294a84744f2896f4ca3~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

Vuex主要有五部分核心内容：

*   📦 **state**：整个应用的状态管理单例，等效于 Vue 组件中的 data，对应了 Flux 架构中的 store。
    
*   🧮 **getter**：可以由 state 中的数据派生而成，等效于 Vue 组件中的计算属性。它会自动收集依赖，以实现计算属性的缓存。
    
*   🛠 **mutation**：类似于事件，包含一个类型名和对应的回调函数，在回调函数中可以对 state 中的数据进行同步修改。
    
    *   Vuex 不允许直接调用该函数，而是需要通过 `store.commit` 方法提交一个操作，并将参数传入回调函数。
    *   commit 的参数也可以是一个数据对象，正如 Flux 架构中的 action 对象一样，它包含了类型名 `type` 和负载 `payload`。
    *   这里要求 mutation 中回调函数的操作一定是同步的，这是因为同步的、可序列化的操作步骤能保证生成唯一的日志记录，才能使得 devtools 能够实现对状态的追踪，实现 time-travel。
*   🔨 **action**：action 内部的操作不受限制，可以进行任意的异步操作。我们需要通过 `dispatch` 方法来触发 action 操作，同样的，参数包含了类型名 `type` 和负载 `payload`。
    
    *   action 的操作本质上已经脱离了 Vuex 本身，假如将它剥离出来，仅仅在用户（开发者）代码中调用 `commit` 来提交 mutation 也能达到一样的效果。
*   📁 **module**：store 的分割，每个 module 都具有独立的 state、getter、mutation 和 action。
    
    *   可以使用 `module.registerModule` 动态注册模块。
    *   支持模块相互嵌套，可以通过设置命名空间来进行数据和操作隔离。

通过和我们上面学习到的Pinia的基础内容对比可以看出，Pinia舍弃了mutation和module两部分，这样我们在使用时就更加的便捷。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/27f1849b2dc44baaa66a4c088051d933~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

与Vuex3.x/4.x对比，主要区别如下：

*   _mutations_ 不再存在。他们经常被认为是 **非常 冗长**。他们最初带来了 devtools 集成，但这不再是问题。
*   无需创建自定义复杂包装器来支持 TypeScript，所有内容都是类型化的，并且 API 的设计方式尽可能利用 TS 类型推断。
*   不再需要注入、导入函数、调用函数、享受自动完成功能！
*   无需动态添加 Store，默认情况下它们都是动态的，您甚至都不会注意到。请注意，您仍然可以随时手动使用 Store 进行注册，但因为它是自动的，您无需担心。
*   不再有 _modules_ 的嵌套结构。您仍然可以通过在另一个 Store 中导入和 _使用_ 来隐式嵌套 Store，但 Pinia 通过设计提供平面结构，同时仍然支持 Store 之间的交叉组合方式。 **您甚至可以拥有 Store 的循环依赖关系**。
*   没有 _命名空间模块_。鉴于 Store 的扁平架构，“命名空间” Store 是其定义方式所固有的，您可以说所有 Store 都是命名空间的。

其实对于我来说，之所以选择Pinia，甚至是喜欢上它，是因为它和Vue3的组合是API形式更加贴合，只需要把它当作一个特殊的数据状态组件来使用就好，不需要那么复杂的流程。

### 小结

通过对Pinia的基本功能的使用介绍，以及和Vuex进行对比，让我们比较清晰的来认识Pinia，使我们能够入门使用。在具体业务场景中，有效的划分Store，合理的组合使用，可以帮助我们完成复杂的业务逻辑。

内容收录于[github 仓库](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Flengxing%2FMyBlog "https://github.com/lengxing/MyBlog")

* * *

参考内容：

[pinia中文文档](https://link.juejin.cn/?target=https%3A%2F%2Fpinia.web3doc.top%2F "https://pinia.web3doc.top/")

[Pinia or Vuex？](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F7dcc6cb99dd2 "https://www.jianshu.com/p/7dcc6cb99dd2")

[Vuex 与 Pinia 的设计实现对比](https://link.juejin.cn/?target=https%3A%2F%2Fjelly.jd.com%2Farticle%2F6347a6613ff25d005bb177fb "https://jelly.jd.com/article/6347a6613ff25d005bb177fb")