## **主要的编程逻辑**

通过在根目录创建一个 **composition**文件夹

里面放置的是关于各个模块的辅助函数（用于**封装函数**，给页面上要对数据进行操作的标签使用）

通过函数的**返回值（函数）**，在**App.vue组件setup函数的return中调用**获取到**需要用到的函数**，如：

```js
<button
          @click="removeCompleted"
          class="clear-completed"
          v-show="completedRef > 0"
>
//上面用到的函数或数据都是在辅助函数中封装好的，只不过是在这边setup的return中调用函数获取，从而能够使用

setup() {
    const { todosRef } = useTodoList();
    return {
      todosRef,
      ...useNewTodo(todosRef),//展开对象拿到里面封装的函数，然后在页面上就能够使用了
      ...useFilter(todosRef),
      ...useEditTodo(todosRef),
      ...useRemoveTodo(todosRef),
    };
  },
```

setup函数返回对象的属性，能够在组件中使用（相当于vue2中data的return）

# 入门------------------------------------------

## 搭建工程

#### 使用vue-cli构建

前提是安装了 vue-cli

**在终端输入**：   vue create vue3-app-cli

**启动工程**: npm run serve

#### 使用vite 构建（推荐使用）

**对比vue-cli的优点**：运行速度快

vite是一个构建工具，可以构建多种框架，如：vue、react等待

**使用方式**： npm init vite-app  工程名

**工程构建完成后，需要进入到工程文件夹**

**安装需要的依赖**：  npm i 

**启动工程：** npm run dev

**注意：**在vite搭建的工程中，**导入文件的位置，必须加上文件后缀名**（node_modules里面的模块除外）

**与vue-cli工程的一点小区别**：页面文件不在public目录下，而是在工程的根目录中

**打包**： vite build

## Vue3与Vue2的不同

#### main.js文件的不同

在vue2中的main.js，是通过**Vue构造函数**实现挂载元素

```js
//导入 Vue构造函数
import Vue from "vue";
new Vue(options).$mount("#app");
```

在vue3中的main.js，**vue3已经没有了Vue构造函数**，使用的是 **vue3具名导出**的 **createApp 函数**（也就是说vue3中，是export 函数名 导出的）

```js
//导出 createApp函数
import { createApp } from "vue";
import App from "App.vue";
createApp(App).mount("#app");	
```

#### 插件使用方式的不同

vue2中，使用插件：通过构造函数调用use()

```js
import Vue from "vue";
Vue.use("插件");
```

vue3中，使用插件：通过实例调用use()

```js
import { createApp } from "vue";
import App from "App.vue";
const app = createApp(App);
app.mount("#app");
app.use("插件");
```

#### this指向不同

<img src="https://github.com/DuYi-Edu/DuYi-Vue3/blob/main/1.%20%E5%85%A5%E9%97%A8/02.%20vue3%E7%9A%84%E9%87%8D%E5%A4%A7%E5%8F%98%E5%8C%96/%E8%AF%BE%E4%BB%B6/vue3%E7%9A%84%E7%BB%84%E4%BB%B6%E5%AE%9E%E4%BE%8B%E4%BB%A3%E7%90%86.jpg?raw=true" alt="vue3的组件实例代理.jpg" style="zoom:50%;" />

**vue2**的this指向：**Vue实例对象**

**vue3**的this指向：**组件代理对象（但是对于操作数据等功能，使用this效果依然相同)**

#### compositionAPI（重点）和optionAPI

这些API其实就是我们使用vue时自带的API，如：data、mothods、computed等等

vue2使用的是optionAPI                                            vue3使用的是compositionAPI

<img src="C:\Users\Ronin\AppData\Roaming\Typora\typora-user-images\image-20210122144120494.png" alt="image-20210122144120494" style="zoom:50%;" />                                                        <img src="C:\Users\Ronin\AppData\Roaming\Typora\typora-user-images\image-20210122144341728.png" alt="image-20210122144341728" style="zoom:50%;" />

颜色表示功能模块代码区域

可以看出，**compositionAPI**是**高内聚的**，**把相同功能模块的代码都聚在一起**，使用compositionAPI维护和编写起来比较高效

## vue3的compositionAPI（初了解）

#### setup函数

实际上可以这么理解：vue3使用setup，把所用模块需要使用的代码都进行了集中处理，最后返回模块需要用到的数据（通过调用函数传参的方式）

并且可以简单理解为：vue3中用 setup函数代替了 vue2中的data，并且setup更加强大

使用：

**setup函数接收两个参数，参数1：一个对象，里面是所有的属性值，参数2：上下文，里面有以前this的函数，如 ctx.emit（手动触发函数） **

```js
//导入ref函数，目的是实现数据响应式
import {ref} from "vue";
export default = {
	setup(props,ctx){
        let count = ref(0);  //把变量的默认值传进ref，他就会返回一个具有响应式的数据，相当于count = 0
        const say = ()=>{
            	console.log(count.value);
            	count.value++;
        }
		return {
            count,
            say
        }
	}
}
```

**特点：**

- setup函数会自动执行
- 并且会在**所有生命周期钩子函数执行之前**调用
- **this为undefined**（也就是指：不能用this）
- 通常return一个对象，**对象中的属性**会被放到**实例**上
- setup中定义的数据，如果**没有**使用 **refAPI**，则**不具有响应式**（也即是数据改变了页面依然没反应）

#### ref函数

**使用：** let 变量 = ref( 数据的默认值 )；

ref的作用就是将数据编程响应式数据，

**实际的过程**：其实它是**返回了一个对象**，对象中有属性value，是以**访问器**的形式存在着

数据执行任何操作的时候，他都会提交给代理进行处理

**注意：**实例上这些数据的**值类型依然是原本的数据类型**，如  ： typeof count   依然返回  Number

**主要记住的一点：**

- 在setup中，count是一个对象
- 在实例代理中，count是一个 count.value（也就是指：如果是在**setup中**要**操作count值**，则使用 **count.value**）
- 如果变量使用ref定义的，最好使用 **变量名Ref** 命名方式

<img src="https://github.com/DuYi-Edu/DuYi-Vue3/blob/main/1.%20%E5%85%A5%E9%97%A8/02.%20vue3%E7%9A%84%E9%87%8D%E5%A4%A7%E5%8F%98%E5%8C%96/%E8%AF%BE%E4%BB%B6/vue3%E5%AF%B9ref%E7%9A%84%E7%89%B9%E6%AE%8A%E5%A4%84%E7%90%86.jpg?raw=true" alt="vue3对ref的特殊处理.jpg" style="zoom:50%;" />

#### watchEffect

翻译：监听副作用

**主要作用：**用于监听响应式数据的改变而出发函数

**特点：**如果watchEffect接收的函数中**用到了响应式数据**，那么**该函数就会监听这些响应式数据**，数据一旦发生改变就会被调用。

**使用方式：**（使用场景：个个函数中都可以）

```js
import {ref,watchEffect} from "vue";
export default = {
    setup(){
        let count = ref(0);
        watchEffect(()=>{
            //该函数内的响应式数据发生变化，则就会调用该函数
            console.log(count);
        })
    }
}
```

#### 生命周期函数

在vue3中，钩子函数已经变成普通的函数导出了

所以可以通过导入的方式获取使用：

```js
import { onMounted , onUnmounted }from "vue";
//onMounted：组件挂载完毕的钩子函数
onMounted(()=>{
    ...
})

//onUnmounted：组件销毁后的钩子函数
onUnmounted(()=>{
    ...
})
```

#### computed

计算属性函数

**使用**：

```js
import {computed} from "vue";
//语法1：传一个对象，里面写get、set
computed({
    get(){...},
    set(data){...}
})
//语法2：传一个函数，该函数就相当于vue2中的computed使用，也就是如果里面用到的响应式数据，它会自动监听这些数据
computed(()=>{
    return ...
})
```

## 关于网页地址hash的一些操作

#### 监听事件

onhashchange事件

```js
window.addEventListener("hashchange",function(){
	...
})
```

#### 获取hash值

```js
location.hash
//一般获取后用正则截取需要的字符
http://127.0.0.1/#/123
const hash = location.hash.replace(/#\/?/,"");  //123
```

# 就业-----------------------------------------

## vite原理

在现实工作中，因为vite 还是属于提案阶段，可能还有很多问题，所以很多公司仍然在使用 vue-cli 构建

> vite: https://github.com/vitejs/vite
>
> **面试题**：谈谈你对vite的理解，最好对比webpack说明？

**webpack 原理图**

[![image-20200929144416064](https://camo.githubusercontent.com/e6776f4a501258194bed935abad6de6dced00cacd613b82427927b6de7daf7bf/687474703a2f2f6d6472732e7975616e6a696e2e746563682f696d672f32303230303932393134343431362e706e67)](https://camo.githubusercontent.com/e6776f4a501258194bed935abad6de6dced00cacd613b82427927b6de7daf7bf/687474703a2f2f6d6472732e7975616e6a696e2e746563682f696d672f32303230303932393134343431362e706e67)

webpack是：

- 入口文件中读取依赖

- 然后针对依赖进行加载

- 生成chunk

- 打包生成压缩后的文件

- 再放到服务器。

  **特点：**先压缩打包，再上传到服务器（热更新：如果更改了某部分内容，那么跟这部分内容相关的都需要重新加载，速度慢）

**vite 原理图**

![image-20210126134548040](C:\Users\Ronin\AppData\Roaming\Typora\typora-user-images\image-20210126134548040.png)

vite是：

- 一开始先开发一个服务器

- 然后根据页面加载文件（如：index.js）（注意：**vite页面文件引入js的方式是 ES6模块化方式** ，也就是 type = “module”）

- 然后读取index.js的依赖

- 服务器根据依赖找到文件，将文件解析后返回给页面

  **特点：**vite并不是先打包再上传服务器，而是一开始先打开服务器，之后服务器根据页面的内容和依赖进行请求文件

> 面试题答案：
> 		首先，**vite的优势都体现在开发环境中**。
>
> webpack会**先打包，然后启动开发服务器，请求服务器时直接给予打包结果**。 
>
> 而**vite**是**直接启动开发服务器，请求哪个模块再对该模块进行实时编译**。 由于现代浏览器本身就支持ES Module，会自动向依赖的Module发出请求。
> 		vite充分利用这一点，将**开发环境下的模块文件**，就作为**浏览器要执行的文件**，而不是像webpack那样进行打包合并。 
> 		由于vite在**启动的时候不需要打包**，也就**意味着不需要分析模块的依赖、不需要编译**，因此启动速度非常快。
> 		**当浏览器请求某个模块时，再根据需要对模块内容进行编译**。
>
> 这种**按需动态编译**的方式，极大的缩减了编译时间，项目越复杂、模块越多，vite的优势越明显。 在HMR方面，当改动了一个模块后，仅需让浏览器重新请求该模块即可，不像webpack那样需要把该模块的相关依赖模块全部编译一次，效率更高。 
> 		**当需要打包到生产环境时，vite使用传统的rollup进行打包**，因此，**vite的主要优势在开发阶段**。
> 		
>
> 		另外，由于vite利用的是ES Module，因此在**代码中不可以使用CommonJS（因为这些代码直接发送到浏览器，浏览器压根不认识CommonJS的代码）**

## 效率提升

> 客户端渲染效率比vue2提升了1.3~2倍
>
> SSR渲染（服务器渲染）效率比vue2提升了2~3倍

> **面试题**：vue3的效率提升主要表现在哪些方面？
>
> 下面五个方面

#### 静态提升

下面的静态节点会被提升

- **元素节点**
- **没有绑定动态内容**

```js
// vue2 的静态节点
render(){
  createVNode("h1", null, "Hello World")
  // ...
}

// vue3 的静态节点
const hoisted = createVNode("h1", null, "Hello World") //直接在外面创建好，那么他就只会创建一次
function render(){
  // 直接使用 hoisted 即可
}
```

静态属性会被提升

```js
<div class="user">
  {{user.name}}
</div>
//静态属性提升
const hoisted = { class: "user" }

function render(){
  createVNode("div", hoisted, user.name)
  // ...
}
```

#### 预字符串化

```js
<div class="menu-bar-container">
  <div class="logo">
    <h1>logo</h1>
  </div>
  <ul class="nav">
    <li><a href="">menu</a></li>
    <li><a href="">menu</a></li>
    <li><a href="">menu</a></li>
    <li><a href="">menu</a></li>
    <li><a href="">menu</a></li>
  </ul>
  <div class="user">
    <span>{{ user.name }}</span>
  </div>
</div>
```

当编译器遇到**大量连续的静态内容**（静态内容：指的是没有使用动态数据的标签，写死的），会直接**将其编译为一个普通字符串节点**

```js
const _hoisted_2 = _createStaticVNode("<div class=\"logo\"><h1>logo</h1></div><ul class=\"nav\"><li><a href=\"\">menu</a></li><li><a href=\"\">menu</a></li><li><a href=\"\">menu</a></li><li><a href=\"\">menu</a></li><li><a href=\"\">menu</a></li></ul>")
```



vue2中，它不管是不是静态节点，都会渲染成虚拟节点树

<img src="https://camo.githubusercontent.com/667c0610653739cf3101765e896d97098e4605ca2131f3dca4ca843d0a49d8db/687474703a2f2f6d6472732e7975616e6a696e2e746563682f696d672f32303230303932393137303230352e706e67" alt="image-20200929170205828" style="zoom: 50%;" />

vue3中，它**会监测到哪些是静态节点，哪些是动态节点**，然后**根据不同节点进行渲染（静态节点生成字符串，动态节点生成虚拟节点树）**

<img src="https://camo.githubusercontent.com/752ef7a8a95d59000fa02196e81b66c12c3c6ad2b0e8edd0d455a80c30312e7e/687474703a2f2f6d6472732e7975616e6a696e2e746563682f696d672f32303230303932393137303330342e706e67" alt="image-20200929170304873" style="zoom: 67%;" />

#### 缓存事件处理函数

对处理事件函数进行缓存（因为很多时候，事件处理函数都是写完就不会改变的）

```js
<button @click="count++">plus</button>
// vue2
render(ctx){
  return createVNode("button", {
    onClick: function($event){
      ctx.count++;
    }
  })
}

// vue3
render(ctx, _cache){
  return createVNode("button", {
    onClick: cache[0] || (cache[0] = ($event) => (ctx.count++))//它会通过cache进行对事件函数缓存
  })
}
```

#### Block Tree

**vue2**在对比新旧树的时候，并不知道哪些节点是静态的，哪些是动态的，因此只能一层一层比较，这就浪费了大部分时间在比对静态节点上

**vue3中**，BlockTree会**对节点进行标记，标记哪些是动态节点，哪些是静态节点**，那么到时候就知道哪些是静态节点（不需要对比）

```js
<form>
  <div>
    <label>账号：</label>
    <input v-model="user.loginId" />
  </div>
  <div>
    <label>密码：</label>
    <input v-model="user.loginPwd" />
  </div>
</form>
```

**vue2的对比新旧树：**

<img src="https://camo.githubusercontent.com/17074722dac04e3a98dd9cf19745fa1f43dbe44d677680435f39db41b9559aa6/687474703a2f2f6d6472732e7975616e6a696e2e746563682f696d672f32303230303932393137323030322e706e67" alt="image-20200929172002761" style="zoom: 33%;" />

**vue3的BlockTree处理：**

<img src="C:\Users\Ronin\AppData\Roaming\Typora\typora-user-images\image-20210126143720114.png" alt="image-20210126143720114" style="zoom: 67%;" />

#### PatchFlag

**vue2**在对比每一个节点时，并不知道这个节点哪些相关信息会发生变化，因此只能将所有信息依次比对

**vue3中**，PatchFlage**能够辨别出哪些信息是动态的内容，哪些是静态的内容**，并且做上标记，供对比的时候处理。

```js
<div class="user" data-id="1" title="user name">
  {{user.name}}
</div>
```

![image-20200929172805674](https://camo.githubusercontent.com/51ad1779d1ed2c96b479cee42b5ed611c58a93d32016fff9890300fdb588338d/687474703a2f2f6d6472732e7975616e6a696e2e746563682f696d672f32303230303932393137323830352e706e67)

## API和数据响应式的变化

> **面试题1**：为什么vue3中去掉了vue构造函数？
>
> **面试题2（常考）**：谈谈你对vue3数据响应式的理解？

#### 去掉了Vue构造函数

在过去，如果遇到一个页面有多个`vue`应用时，往往会遇到一些问题

```js
<!-- vue2 -->
<div id="app1"></div>
<div id="app2"></div>
<script>
  Vue.use(...); // 此代码会影响所有的vue应用
  Vue.mixin(...); // 此代码会影响所有的vue应用
  Vue.component(...); // 此代码会影响所有的vue应用
                
	new Vue({
    // 配置
  }).$mount("#app1")
  
  new Vue({
    // 配置
  }).$mount("#app2")
</script>
```

在`vue3`中，去掉了`Vue`构造函数，转而使用`createApp`创建`vue`应用

```js
<!-- vue3 -->
<div id="app1"></div>
<div id="app2"></div>
<script>  
	createApp(根组件).use(...).mixin(...).component(...).mount("#app1")
  createApp(根组件).mount("#app2")
</script>
```

> 更多vue应用的api：https://v3.vuejs.org/api/application-api.html

#### 组件实例中的API

在`vue3`中，**组件实例是一个`Proxy`**，它仅提供了下列成员，功能和`vue2`一样

属性：https://v3.vuejs.org/api/instance-properties.html

方法：https://v3.vuejs.org/api/instance-methods.html

#### 对比数据响应式

vue2和vue3**均在相同的生命周期（beforeCreate之后，created之前）完成数据响应式**，但做法不一样

vue2中，它会将所有数据都遍历一遍，并且如果后面的数据需要新增加一些属性，还得需要$set等方法的辅助（显得比较麻烦）

vue3中，它把这些操作都交给了代理（proxy），并且proxy是动态返回的，也就是你需要用到哪些数据，他只会返回给你对应的数据，其他数据并不做遍历。

<img src="https://camo.githubusercontent.com/7b18bafe3f8567211e9d86584d439d51793ef8987d3964ff4791d5d39de5e728/687474703a2f2f6d6472732e7975616e6a696e2e746563682f696d672f32303230313031343135353433332e706e67" alt="image-20201014155433311" style="zoom: 50%;" />

## **面试题参考答案**

**面试题1**：为什么vue3中去掉了vue构造函数？

```
vue2的全局构造函数带来了诸多问题：
1. 调用构造函数的静态方法会对所有vue应用生效，不利于隔离不同应用
2. vue2的构造函数集成了太多功能，不利于tree shaking，vue3把这些功能使用普通函数导出，能够充分利用tree shaking优化打包体积
3. vue2没有把组件实例和vue应用两个概念区分开，在vue2中，通过new Vue创建的对象，既是一个vue应用，同时又是一个特殊的vue组件。vue3中，把两个概念区别开来，通过createApp创建的对象，是一个vue应用，它内部提供的方法是针对整个应用的，而不再是一个特殊的组件。
```

**面试题2（常考）**：谈谈你对vue3数据响应式的理解

```
vue3不再使用Object.defineProperty的方式定义完成数据响应式，而是使用Proxy。
除了Proxy本身效率比Object.defineProperty更高之外，由于不必递归遍历所有属性，而是直接得到一个Proxy。所以在vue3中，对数据的访问是动态的，当访问某个属性的时候，再动态的获取和设置，这就极大的提升了在组件初始阶段的效率。
同时，由于Proxy可以监控到成员的新增和删除，因此，在vue3中新增成员、删除成员、索引访问等均可以触发重新渲染，而这些在vue2中是难以做到的。
```

## 模板中的变化

#### v-model

组件之间的数据传递

**`vue2`** 比较让人诟病的一点就是提供了**两种双向绑定**：**`v-model`和`.sync`**，

在**`vue3`中**，**去掉了`.sync`修饰符**，**只需要使用`v-model`进行双向绑定即可**。

为了让`v-model`更好的针对多个属性进行双向绑定，**`vue3`作出了以下修改：**

- 当**对自定义组件使用`v-model`指令**并且**没有传参数时**，**绑定的默认属性名**由原来的**`value`变为`modelValue`**，
  																	 					   	 **默认事件名**由原来的**`input`变为`update:modelValue`**

  ```html
  <!-- App.vue组件中 -->
  
  <!-- vue2 -->
  <ChildComponent :value="pageTitle" @input="pageTitle = $event" />
  <!-- 简写为 -->
  <ChildComponent v-model="pageTitle" />
  
  <!-- vue3 -->
  <ChildComponent
    :modelValue="pageTitle"
    @update:modelValue="pageTitle = $event"
  />
  <!-- 简写为 -->
  <ChildComponent v-model="pageTitle" /> <!-- 这里把pageTitle的值传递给了modelValue -->
  <script>
  	export default = {
          data(){
              return{
                  pageTitle:false
              }
          }
      }
  </script>
  ```

  ```html
  <!-- ChildComponent组件内的接收 -->
  <template>
      <div :class = "{cheaked:modelValue}">
          123
      </div>
  	<button @click = handleCheaked>
          点击更改modelValue
      </button>
  </template>
  <script>
  export default = {
  	props:{
  		modelValue:Boolean, //接收App.vue组件传递过来的值(pageTitle的值)，此时cheaked类名还不能加上去
  	},
      setup(props,ctx){
          const handleCheaked = ()=>{
         //触发 ChildComponent 的 update:modelValue 事件，并传参，修改了pageTitle从而修改App.vue的modelValue的值
         //App.vue的modelValue改变，组件中的modelValue也改变，div的cheaked类名就加上去了
              ctx.emit("update:modelValue",!props.modelValue)
  		}
      }
  }
  </script>
  ```

  

- **去掉了`.sync`修饰符，它原本的功能由`v-model`的参数替代**
  如下： v-model : title = “pageTitle”
  那么组件props中接收时，就不是默认的 modelValue了，而是参数 title

  ```html
  <!-- vue2 -->
  <ChildComponent :title="pageTitle" @update:title="pageTitle = $event" />
  <!-- 简写为 -->
  <ChildComponent :title.sync="pageTitle" />
  
  <!-- vue3 -->
  <ChildComponent :title="pageTitle" @update:title="pageTitle = $event" />
  <!-- 简写为 -->
  <ChildComponent v-model:title="pageTitle" />
  ```

- **`model`配置被移除**

- **允许自定义`v-model`修饰符**

  **主要作用就是**：监听组件是否使用了自定义修饰符，通过这个自定义修饰符就能**对数据做出想要的处理**

  vue2 无此功能

  如下图，表示当使用这种修饰符的方式时，他**就会自动向组件传递一个属性** ：

  **默认情况**下为 ： modelModifiers : { cap : true}

  **如果传递了参数**则是： 参数Modifiers : { cap : true }

  组件内需要通过 props接收，拿到后，里面的值就是修饰符，如下：值就为  {cap : true}

  有这个值后，我们就可以在 **ctx.emit()调用事件函数前，对数据做一些处理**

  ![image-20201008163021918](https://camo.githubusercontent.com/216697fc3cd13be5fede2b53e45f0fd1fba8ce9b0add09ed09a3324f53bc0ef6/687474703a2f2f6d6472732e7975616e6a696e2e746563682f696d672f32303230313030383136333032322e706e67)

#### v-if v-for

**v-if 的优先级 现在高于 v-for**

所以，以下代码会报错：

```html
<div v-for = "item in arr" v-if = "item.show">
    <!-- 因为vue3中，v-if的优先级比 v-for的高，就导致它无法读取到item.show ，所以会报错-->
	<span>{{ item.msg }}</span>
</div>
```



#### key

- 当使用`<template>`进行`v-for`循环时（也就是有多个根节点时），需要把`key`值放到`<template>`中，而不是它的子元素中
  如：

  ```html
  <!-- vue2 -->
  <template v-for = "item in arr">
  	<div key=1></div>
      <span key=2></span>
  </template>
  <!-- vue3 -->
  <template v-for = "item in arr" :key="item.id">
  	<div></div>
      <span></span>
  </template>
  ```

  

- 当使用`v-if v-else-if v-else`分支的时候，**不再需要指定`key`值**（vue2时为了防止被复用，所以需要加key值来辨别），因为`vue3`会自动给予每个分支一个唯一的`key`

  即便要手工给予`key`值，也必须给予每个分支唯一的`key`，**不能因为要重用分支而给予相同的 key**

#### Fragment

`vue3`现在**允许组件出现多个根节点**

## 组件的变化

#### 路由

与vue2不同的是，安装的版本是 next 版本

**使用前需要安装插件** ： npm i vue-router@next

**使用**

在vue3中，去除的东西：

- 去除了 Router 构造函数，使用的是里面导出的函数 **createRouter**
- 去除了 model （设置模式 hash / history）、baseURL（设置初始重定向位置）

```js
//通常做法是，创建router.js文件,里面导出一个路由供main.js使用
import { createRouter,createWebHistory } from "vue-router";
import Home from "./views/Home.vue";
export default createRouter({
    history:createWebHistory("/home"), //model:history,baseURL:"/home"
    routes:[
        {path:"/home",component:Home},
        {...}
    ]
})
        
```

```js
//main.js
import { createApp } from 'vue'
import App from './App.vue'
import './index.css'
import router from "./router.js";
//实例.use(插件)： 使用插件
createApp(App).use(router).mount('#app')
```

#### 异步组件

通过 vue 中的  **defineAsyncComponent** 函数创建异步组件

**使用：**

- 传递一个**函数**，函数返回一个promise，一般通过 **import( )**返回异步组件promise对象
- 传递一个**对象**，对象中的 **loader** 属性放的值就是像传递函数时的那个**函数**
                           对象中的**loadingComponent属性**可以**配置promise对象处于pendding时显示的组件**（指：组件还在加载中）
                           对象中的**errorComponent属性**可以**配置promise对象出错时（rejected状态）显示的组件**

```js
<template>
    <async-com /> //使用异步组件
</template
<script>
import { defineAsyncComponent,h } from "vue";
import Loading from "./components/Loading.vue";
import Error from "./components/Error.vue"; //组件内没有放内容，而是设置了插槽 slot，用于接收render传递的内容

//传递函数
const asyncCom = defineAsyncComponent(()=>import("./asyncCom"));

//传递对象
const asyncCom = defineAsyncComponent({
    loader:()=>import("./asyncCom"),
    loadingComponent: Loading, //当promise状态处于 pendding时，自动显示Loading组件
    errorComponent:{           //当promise状态处于 rejected时，显示Error组件，这里使用了render方式，也可以直接放组件对象
    	render(){
    		return h(Error,"出错了"); //把 "出错了" 插到Error组件的插槽(solt)中
		}
	}
})

export default = {
    components:{
        asyncCom,
    }
}
</script>
```

**可以封装一个生成异步组件函数**

```js
import { defineAsyncComponent,h } from "vue";
import Loading from "./components/Loading.vue";
import Error from "./components/Error.vue"; //Error组件内没有放内容，而是设置了插槽 slot，用于接收render传递的内容

//传入位置(path)返回一个异步组件
function getAsyncComponent(path){
    return defineAsyncComponent({
    loader:()=>import(path),
    loadingComponent: Loading, //当promise状态处于 pendding时，自动显示Loading组件
    errorComponent:{           //当promise状态处于 rejected时，显示Error组件，这里使用了render方式，也可以直接放组件对象
    	render(){
    		return h(Error,"出错了"); //把 "出错了" 插到Error组件的插槽(solt)中
		}
	}
})
}
```



#### render函数的改动

与vue2不同的是，render函数的需要使用的渲染函数 h， 不再是从函数中接收

而是**从vue 中导出**

```js
import { h } from "vue";
```

#### Teleport组件

vue3中新增的组件，能够直接使用

**作用**：将组件内的元素插到 某个标签里的最后面

**使用**：

```html
<Teleport to = "body"> <!-- to的值填 css选择器，表示插入到哪个标签内 -->
	<div v-if = "show">
        <button @click = "show = !show">
            关闭
        </button>
    </div>
</Teleport>
```



## 进度条插件

**安装**：npm i nprogress

**使用**：

```js
//index.js
import "nprogress/nprogress.css";//获取样式
import NProgress from "nprogress";
NProgress.configure({
    trickSpeed:50,//配置进度条的速度
    showSpinner:false//禁用显示加载指针（也就是那个加载的圆圈）
})

NProgress.start();//进度条启动
NProgress.done();//直接让进度条完成
```

在**异步组件中的使用**：

```js
const AsycnPage = defineAsyncComponent({ //创建一个异步页面组件（其实跟异步组件一样）
	loader:async ()=>{
		NProgress.start(); //开始进度条
		const comp = await import("./Home");
		NProgress.done(); //结束进度条
		return comp;
	}
})
```

## **reactivity API（最复杂，笔试最难）**

#### **获取响应式数据**

| API            | 传入                 | 返回                             | 备注                                                         |
| -------------- | -------------------- | -------------------------------- | ------------------------------------------------------------ |
| **`reactive`** | 普通对象             | `对象代理`  （获取时直接点获取） | 深度代理对象中的所有成员                                     |
| **`readonly`** | 普通对象  或对象代理 | `对象代理`                       | **只能读取**代理对象中的成员，**不可修改**                   |
| **`ref`**      | `any`                | `{ value: ... }`                 | 对value的访问是响应式的 <br />**如果给value的值是一个对象， 则会通过`reactive`函数进行代理 <br />如果已经是对象代理，则直接使用代理** |
| **`computed`** | `function`           | `{ value: ... }`                 | 当读取value值时，会**根据情况**决定是否要运行函数                                           （**查看是否有缓存 / 数据改变后再次读取value值时**） |

返回值决定了获取方式

**注意：**

**readonly 和 reactive 同时使用时**

```js
//使用 readonly ，里面参数放的是一个代理对象时
const state = reactive({a:1});
const readonlyState = readonly(state);
console.log(readonly === state); //false

//					    代理		 代理
//因为实际过程是： readonly --> state ---> {a : 1} ，也即是套了两层代理
```

**ref 和 reactive 同时使用时**

```js
const state = reactive({a:1});
const stateRef = ref(state);
console.log(stateRef === state); //true

//因为stateRef认为state本身就是响应式数据的，所以stateRef 直接就等于state
```

应用：

- 如果想要让一个对象变为**响应式数据**，可以使用**`reactive`或`ref`**
- 如果想要让一个对象的**所有属性只读**，使用**`readonly `** 
- 如果想要让一个**非对象数据变为响应式数据**，使用**`ref`** 
- 如果想要根据**已知的响应式数据得到一个新的响应式数据**，使用**`computed`** 

**笔试题1**：下面的代码输出结果是什么？

```js
import { reactive, readonly, ref, computed } from "vue";

const state = reactive({
  firstName: "Xu Ming",
  lastName: "Deng",
});
const fullName = computed(() => {
  console.log("changed");
  return `${state.lastName}, ${state.firstName}`;
});
console.log("state ready"); //state ready
console.log("fullname is", fullName.value); //changed fullname is Deng,Xu Ming
console.log("fullname is", fullName.value); //fullname is Deng,Xu Ming
const imState = readonly(state);
console.log(imState === state); //false

const stateRef = ref(state);
console.log(stateRef.value === state); //true

state.firstName = "Cheng";  //此时并不会运行computed，需要读到 fullName.value才会运行
state.lastName = "Ji"; 

console.log(imState.firstName, imState.lastName); //Cheng  Ji
console.log("fullname is", fullName.value); //changed fullname is Ji,Cheng
console.log("fullname is", fullName.value); //fullname is Ji,Cheng

const imState2 = readonly(stateRef); 
console.log(imState2.value === stateRef.value); //false
```

**笔试题2**：按照下面的要求完成函数

```js
import {reactive,readonly} from "vue";
function useUser(){
  // 在这里补全函数
//    const userOrigin = reactive({});
//    const user = readonly(userOrigin);
//    const setUserName = name =>{
//        userOrigin.name = name;
//    }
//    const setUserAge = age =>{
//        userOrigin.age = age;
//    }
  return {
    user, // 这是一个只读的用户对象，响应式数据，默认为一个空对象
    setUserName, // 这是一个函数，传入用户姓名，用于修改用户的名称
    setUserAge, // 这是一个函数，传入用户年龄，用户修改用户的年龄
  }
}
```

**笔试题3**：按照下面的要求完成函数

```js
import {ref,readonly} from "vue";
function useDebounce(obj, duration){
  // 在这里补全函数
    let timer = null;
    const valueOrigin = ref(obj);
    const value = readonly(obj);
    const setValue = obj =>{
        clearTimeout(timer);
        timer = setTimeout(()=>{
            valueOrigin.value = {...valueOrigin.value , ...obj};
        },duration)
    }
  return {
    value, // 这里是一个只读对象，响应式数据，默认值为参数值
    setValue // 这里是一个函数，传入一个新的对象，需要把新对象中的属性混合到原始对象中，混合操作需要在duration的时间中防抖
  }
}
```

#### **监听数据变化**

**watchEffect**

相对watch的特点就是：

- 首先会立即执行一次
- 会自动监听函数中用到的响应式数据（监听不到没用到的数据）

```js
const stop = watchEffect(() => {
  // 该函数会立即执行，然后追中函数中用到的响应式数据，响应式数据变化后会再次执行
})

// 通过调用stop函数，会停止监听
stop(); // 停止监听
```

**watch**

使用： watch( 需要监听的数据 ， 回调);

- watch的**第一个**参数：**需要通过函数返回数据的方式进行传递**
- watch的**第一个**参数：**如果是 ref 的响应式数据，则可以直接传递数据**
- watch的**第二个**参数：**是一个回调函数，监听的数据改变时触发，有两个参数，分变表示  新的值 ， 旧的值**

```js
// 等效于vue2的$watch

// 监听单个数据的变化
import { watch,reactive,ref } from "vue";
const state = reactive({ count: 0 })
watch(() => state.count, (newValue, oldValue) => {
  // ...
}, options)

//如果是ref数据，则可以直接传递数据
const countRef = ref(0);
watch(countRef, (newValue, oldValue) => {
  // ...
}, options)

// 监听多个数据的变化，使用数组方式
//那么回调得到的参数也会对应的接收两个数组
watch([() => state.count, countRef], ([new1, new2], [old1, old2]) => {
  // ...
});
```

**注意：无论是`watchEffect`还是`watch`，当依赖项变化时，回调函数的运行都是异步的（微队列）**

**应用**：除非遇到下面的场景，否则均建议选择`watchEffect`

- 不希望回调函数一开始就执行
- 数据改变时，需要参考旧值
- 需要监控一些回调函数中不会用到的数据

笔试题: 下面的代码输出结果是什么？

```js
import { reactive, watchEffect, watch } from "vue";
const state = reactive({
  count: 0,
});
watchEffect(() => {
  console.log("watchEffect", state.count);
});
watch(
  () => state.count,
  (count, oldCount) => {
    console.log("watch", count, oldCount);
  }
);
console.log("start");
setTimeout(() => {
  console.log("time out");
  state.count++;
  state.count++;
});
//连续两次改变值的时候，由于执行速度很快，它会认为是改变了一次，最开始的一次到最后一次  0 -> 2
state.count++;
state.count++;

console.log("end");
//watchEffect 0
//start
//end
//watchEffect 2
//watch 2 0
//time out
//watchEffect 4
//watch 4 2
```

#### 判断

| API          | 含义                                                         |
| ------------ | ------------------------------------------------------------ |
| `isProxy`    | 判断某个数据是否是由`reactive`或`readonly`                   |
| `isReactive` | 判断某个数据是否是通过`reactive`创建的 详细:https://v3.vuejs.org/api/basic-reactivity.html#isreactive |
| `isReadonly` | 判断某个数据是否是通过`readonly`创建的                       |
| `isRef`      | 判断某个数据是否是一个`ref`对象                              |

使用：

```js
import { isProxy,reactive } from "vue";
const state = reactive({a:1});
console.log(isProxy(state));//true
```



#### 转换

**unref**

等同于：`isRef(val) ? val.value : val`

应用：

```js
function useNewTodo(todos){
  todos = unref(todos);
  // ...
}
```

**toRef**

得到一个响应式对象某个属性的ref格式

```js
const state = reactive({
  foo: 1,
  bar: 2
})

const fooRef = toRef(state, 'foo'); // fooRef: {value: ...}

fooRef.value++
console.log(state.foo) // 2

state.foo++
console.log(fooRef.value) // 3
```

**toRefs**

把一个响应式对象的所有属性转换为ref格式，然后包装到一个`plain-object`中返回

```js
const state = reactive({
  foo: 1,
  bar: 2
})

const stateAsRefs = toRefs(state)
/*
stateAsRefs: not a proxy
{
  foo: { value: ... },
  bar: { value: ... }
}
*/
```

应用：（主要目的就是：方便展开使用）

```js
setup(){
  const state1 = reactive({a:1, b:2});
  const state2 = reactive({c:3, d:4});
  return {
    ...state1, // lost reactivity
    ...state2 // lost reactivity
  }
}

setup(){
  const state1 = reactive({a:1, b:2});
  const state2 = reactive({c:3, d:4});
  return {
    ...toRefs(state1), // reactivity
    ...toRefs(state2) // reactivity
  }
}
// composition function
function usePos(){
  const pos = reactive({x:0, y:0});
  return pos;
}

setup(){
  const {x, y} = usePos(); // lost reactivity
  const {x, y} = toRefs(usePos()); // reactivity
}
```

#### 降低心智负担

**所有的`composition function`均以`ref`的结果返回**，以保证`setup`函数的返回结果中不包含`reactive`或`readonly`直接产生的数据

```js
function usePos(){
  const pos = reactive({ x:0, y:0 });
  return toRefs(pos); //  {x: refObj, y: refObj}
}
function useBooks(){
  const books = ref([]);
  return {
    books // books is refObj
  }
}
function useLoginUser(){
  const user = readonly({
    isLogin: false,
    loginId: null
  });
  return toRefs(user); // { isLogin: refObj, loginId: refObj }  all ref is readonly
}

setup(){
  // 在setup函数中，尽量保证解构、展开出来的所有响应式数据均是ref
  return {
    ...usePos(),
    ...useBooks(),
    ...useLoginUser()
  }
}
```

## Composition API

> 面试题：composition api相比于option api有哪些优势？

不同于reactivity api，composition api提供的函数很多是与组件深度绑定的，不能脱离组件而存在。

#### setup

```js
// component
export default {
  setup(props, context){
    // 该函数在组件属性被赋值后立即执行，早于所有生命周期钩子函数
    // props 是一个对象，包含了所有的组件属性值
    // context 是一个对象，提供了组件所需的上下文信息
  }
}
```

context对象的成员

| 成员  | 类型 | 说明                    |
| ----- | ---- | ----------------------- |
| attrs | 对象 | 同`vue2`的`this.$attrs` |
| slots | 对象 | 同`vue2`的`this.$slots` |
| emit  | 方法 | 同`vue2`的`this.$emit`  |

#### 生命周期函数

| vue2 option api | vue3 option api       | vue 3 composition api           |
| --------------- | --------------------- | ------------------------------- |
| beforeCreate    | beforeCreate          | 不再需要，代码可直接置于setup中 |
| created         | created               | 不再需要，代码可直接置于setup中 |
| beforeMount     | beforeMount           | onBeforeMount                   |
| mounted         | mounted               | onMounted                       |
| beforeUpdate    | beforeUpdate          | onBeforeUpdate                  |
| updated         | updated               | onUpdated                       |
| beforeDestroy   | **改** beforeUnmount  | onBeforeUnmount                 |
| destroyed       | **改**unmounted       | onUnmounted                     |
| errorCaptured   | errorCaptured         | onErrorCaptured                 |
| -               | **新**renderTracked   | onRenderTracked                 |
| -               | **新**renderTriggered | onRenderTriggered               |

新增钩子函数说明：

| 钩子函数                  | 参数（接收的事件源） | 执行时机                           |
| ------------------------- | -------------------- | ---------------------------------- |
| renderTracked（适合调试） | DebuggerEvent        | **渲染vdom收集到的每一次依赖时**   |
| renderTriggered           | DebuggerEvent        | **某个依赖变化导致组件重新渲染时** |

DebuggerEvent上的属性:

- **target**: 跟踪或触发渲染的对象
- **key**: 跟踪或触发渲染的属性
- **type**: 跟踪或触发渲染的方式

#### 面试题参考答案

面试题：composition api相比于option api有哪些优势？

> 从两个方面回答：
>
> 1. 为了更好的**逻辑复用**和**代码组织**
> 2. 更好的**类型推导**

```js
有了composition api，配合reactivity api，可以在组件内部进行更加细粒度的控制，使得组件中不同的功能高度聚合，提升了代码的可维护性。对于不同组件的相同功能，也能够更好的复用。
相比于option api，composition api中没有了指向奇怪的this，所有的api变得更加函数式，这有利于和类型推断系统比如TS深度配合。
```

## 共享数据

#### vuex方案

安装`vuex@4.x`

两个重要变动：

- **去掉了构造函数`Vuex`，而使用`createStore`创建仓库**
- **为了配合`composition api`，新增`useStore`函数获得仓库对象**

#### global state

**由于`vue3`的响应式系统本身可以脱离组件而存在**，因此可以充分利用这一点，轻松制造多个全局响应式数据

**就是通过封装一个js文件，文件导出需要用到的数据或函数即可，如果数据不想被外部修改，那么就可以用 readonly 包装**

```js
// store/useLoginUser 提供当前登录用户的共享数据
// 以下代码仅供参考
import { reactive, readonly } from "vue";
import * as userServ from "../api/user"; // 导入api模块

// 创建默认的全局单例响应式数据，仅供该模块内部使用
const state = reactive({ user: null, loading: false });

// 对外暴露的数据是只读的，不能直接修改
// 也可以进一步使用toRefs进行封装，从而避免解构或展开后响应式丢失
export const loginUserStore = readonly(state);

//对外暴露函数
// 登录
export async function login(loginId, loginPwd) {
  state.loading = true;
  const user = await userServ.login(loginId, loginPwd);
  state.loginUser = user;
  state.loading = false;
}
// 退出
export async function loginOut() {
  state.loading = true;
  await userServ.loginOut();
  state.loading = false;
  state.loginUser = null;
}
// 恢复登录状态
export async function whoAmI() {
  state.loading = true;
  const user = await userServ.whoAmI();
  state.loading = false;
  state.loginUser = user;
}
```

#### Provide&Inject

在`vue2`中，提供了`provide`和`inject`配置，可以让开发者在高层组件中注入数据，然后在后代组件中使用

[![image-20201026173949874](https://camo.githubusercontent.com/e40f3619ad36455b409c74068ee29801584e7dab0f96f40322f3f4b8030d0d52/687474703a2f2f6d6472732e7975616e6a696e2e746563682f696d672f32303230313032363137333934392e706e67)](https://camo.githubusercontent.com/e40f3619ad36455b409c74068ee29801584e7dab0f96f40322f3f4b8030d0d52/687474703a2f2f6d6472732e7975616e6a696e2e746563682f696d672f32303230313032363137333934392e706e67)

除了兼容`vue2`的配置式注入，**`vue3`在`composition api`中添加了`provide`和`inject`方法，可以在`setup`函数中注入和使用数据**

[![image-20201026174039594](https://camo.githubusercontent.com/aa092a872c541d03998533f57a3246e492e9c96664c018d98c4ea485a7b7f81d/687474703a2f2f6d6472732e7975616e6a696e2e746563682f696d672f32303230313032363137343033392e706e67)](https://camo.githubusercontent.com/aa092a872c541d03998533f57a3246e492e9c96664c018d98c4ea485a7b7f81d/687474703a2f2f6d6472732e7975616e6a696e2e746563682f696d672f32303230313032363137343033392e706e67)

考虑到有些数据需要在整个vue应用中使用，`vue3`还在**应用实例中**加入了`provide`方法，用于提供整个应用的共享数据

```js
creaetApp(App)
  .provide("foo", ref(1))
  .provide("bar", ref(2))
	.mount("#app");
```

[![image-20201026174611477](https://camo.githubusercontent.com/03e730cea3e471d094f80a7c4a0242a15a2476220ea96aa8dca9cb8196cb1c8a/687474703a2f2f6d6472732e7975616e6a696e2e746563682f696d672f32303230313032363137343631312e706e67)](https://camo.githubusercontent.com/03e730cea3e471d094f80a7c4a0242a15a2476220ea96aa8dca9cb8196cb1c8a/687474703a2f2f6d6472732e7975616e6a696e2e746563682f696d672f32303230313032363137343631312e706e67)

因此，我们可以利用这一点，在整个vue应用中提供共享数据

```js
// store/useLoginUser 提供当前登录用户的共享数据
// 以下代码仅供参考
import { readonly, reactive, inject } from "vue";
const key = Symbol(); // Provide的key

// 在传入的vue应用实例中提供数据
export function provideStore(app) {
  // 创建默认的响应式数据
  const state = reactive({ user: null, loading: false });
  // 登录
  async function login(loginId, loginPwd) {
    state.loading = true;
    const user = await userServ.login(loginId, loginPwd);
    state.loginUser = user;
    state.loading = false;
  }
  // 退出
  async function loginOut() {
    state.loading = true;
    await userServ.loginOut();
    state.loading = false;
    state.loginUser = null;
  }
  // 恢复登录状态
  async function whoAmI() {
    state.loading = true;
    const user = await userServ.whoAmI();
    state.loading = false;
    state.loginUser = user;
  }
  // 提供全局数据
  app.provide(key, {
    state: readonly(state), // 对外只读
    login,
    loginOut,
    whoAmI,
  });
}

export function useStore(defaultValue = null) {
  return inject(key, defaultValue);
}

// store/index
// 应用所有store
import { provideStore as provideLoginUserStore } from "./useLoginUser";
// 继续导入其他共享数据模块...
// import { provideStore as provideNewsStore } from "./useNews"

// 提供统一的数据注入接口
export default function provideStore(app) {
  provideLoginUserStore(app);
  // 继续注入其他共享数据
  // provideNewsStore(app);
}

// main.js
import { createApp } from "vue";
import provideStore from "./store";
const app = createApp(App);
provideStore(app);
app.mount("#app");
```

#### 对比

|              | vuex（适合大型项目） | global state（适合中小型项目） | Provide&Inject（适合中小型项目） |
| ------------ | -------------------- | ------------------------------ | -------------------------------- |
| 组件数据共享 | ✅                    | ✅                              | ✅                                |
| 可否脱离组件 | ✅                    | ✅                              | ❌                                |
| 调试工具     | ✅                    | ❌                              | ✅                                |
| 状态树       | ✅                    | 自行决定                       | 自行决定                         |
| 量级         | 重                   | 轻                             | 轻                               |