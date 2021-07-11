# css工程化概述 -------------------------

## css的问题

### 类名冲突的问题

当你写一个css类的时候，你是写全局的类呢，还是写多个层级选择后的类呢？

你会发现，怎么都不好

- **过深的层级**不利于编写、阅读、压缩、复用
- **过浅的层级**容易导致类名冲突

一旦样式多起来，这个问题就会变得越发严重，其实归根结底，就是类名冲突不好解决的问题

### 重复样式

这种问题就更普遍了，一些重复的样式值总是不断的出现在css代码中，维护起来极其困难

比如，**一个网站的颜色**一般就那么几种：

- primary
- info
- warn
- error
- success

如果有更多的颜色，都是从这些色调中自然变化得来，可以想象，这些颜色会到处充斥到诸如背景、文字、边框中，一旦要做颜色调整，是一个非常大的工程（**也就是如果很多组件都有用到这些主色调，那么一旦改动了主色调，那么就得全局改动**）

### css文件细分问题

在大型项目中，**css也需要更细的拆分，这样有利于css代码的维护**。

比如，有一个做轮播图的模块，它不仅需要依赖js功能，还需要依赖css样式，既然依赖的js功能仅关心轮播图，那css样式也应该仅关心轮播图，由此类推，不同的功能依赖不同的css样式、公共样式可以单独抽离，这样就形成了不同于过去的css文件结构：文件更多、拆分的更细

而同时，在真实的运行环境下，我们却希望文件越少越好，这种情况和JS遇到的情况是一致的

因此，对于css，也需要工程化管理

从另一个角度来说，css的工程化会遇到更多的挑战，因为css不像JS，它的语法本身经过这么多年并没有发生多少的变化（css3也仅仅是多了一些属性而已），对于css语法本身的改变也是一个工程化的课题

## 如何解决

这么多年来，官方一直没有提出方案来解决上述问题

一些第三方机构针对不同的问题，提出了自己的解决方案

### 解决类名冲突

一些第三方机构提出了一些方案来解决该问题，常见的解决方案如下：

**命名约定**

即提供一种命名的标准，来解决冲突，常见的标准有：

- BEM
- OOCSS
- AMCSS
- SMACSS
- 其他

**css in js**

这种方案非常大胆，它觉得，css语言本身几乎无可救药了，干脆直接用js对象来表示样式，然后把样式直接应用到元素的style中

这样一来，css变成了一个一个的对象，就可以完全利用到js语言的优势，你可以：

- 通过一个函数返回一个样式对象
- 把公共的样式提取到公共模块中返回
- 应用js的各种特性操作对象，比如：混合、提取、拆分
- 更多的花样

> 这种方案在手机端的React Native中大行其道

**css module**

非常有趣和好用的css模块化方案，编写简单，绝对不重名

具体的课程中详细介绍

### 解决重复样式的问题

**css in js**

这种方案虽然可以利用js语言解决重复样式值的问题，但由于太过激进，很多习惯写css的开发者编写起来并不是很适应

**预编译器**

有些第三方搞出一套css语言的进化版来解决这个问题，它支持变量、函数等高级语法，然后经过编译器将其编译成为正常的css

这种方案特别像构建工具，不过它仅针对css

常见的预编译器支持的语言有：

- **less**
- sass

### 解决css文件细分问题

这一部分，就要依靠构建工具，例如webpack来解决了

利用一些**loader**或**plugin**来打包、合并、压缩css文件

# **解决css文件细分问题--------------------**

## 利用webpack拆分

**解决css文件细分问题**

要拆分css，就必须把css当成像js那样的模块；要把css当成模块，就必须有一个构建工具（webpack），它具备合并代码的能力

而**webpack本身只能读取css文件的内容、将其当作JS代码进行分析，因此，会导致错误**

于是，就必须有一个loader，能够**将css代码转换为js代码**

#### css-loader

**使用前需要先安装   css-loader**：  npm i -D css-loader

css-loader的作用，就是**将css代码转换为js代码**

它的处理原理极其简单：**将css代码作为字符串导出**

例如：

```css
.red{
    color:"#f40";
}
```

**配置webpack.config.js**

```js
module.exports = {
	module:{
		rules:[
			{
				test:/\.css$/,
				use:"css-loader"
			}
		]
	}
}
```

经过css-loader转换后变成js代码：

```js
module.exports = `.red{
    color:"#f40";
}`
```

> 上面的js代码是经过我简化后的，**不代表真实的css-loader的转换后代码**，css-loader转换后的代码会有些复杂，同时会导出更多的信息，但核心思想不变

再例如：

```js
.red{
    color:"#f40";
    background:url("./bg.png")
}
```

**配置webpack.config.js**

```js
module.exports = {
	module:{
		rules:[
			{
				test:/\.css$/,
				use:"css-loader"
			},
            {
                test:/\.png$/,
                use:"file-loader"
            }
		]
	}
}
```

经过css-loader转换后变成js代码：

```js
var import1 = require("./bg.png");
module.exports = `.red{
    color:"#f40";
    background:url("${import1}")
}`;
```

这样一来，经过webpack的后续处理，会把依赖`./bg.png`添加到模块列表，然后再将代码转换为

```js
var import1 = __webpack_require__("./src/bg.png");
module.exports = `.red{
    color:"#f40";
    background:url("${import1}")
}`;
```

再例如**在css文件中import导入了其他css**：

```js
@import "./reset.css";
.red{
    color:"#f40";
    background:url("./bg.png")
}
```

会转换为：

```js
var import1 = require("./reset.css").default.toString();  //内部使用的是ES6默认导出，toString是它内部封装的函数，输出
var import2 = require("./bg.png");
module.exports = `${import1}
.red{
    color:"#f40";
    background:url("${import2}")
}`;
```

具体案例：**在渡一demo / webpack / 拆分css中使用过**

总结，css-loader干了什么：

1. **将css文件的内容作为字符串导出**
2. **将css中的其他依赖作为require导入，以便webpack分析依赖**

#### style-loader

**使用前，需要先安装style-loader**

**由于css-loader仅提供了将css转换为字符串导出的能力**，剩余的事情要交给其他loader或plugin来处理

**style-loader可以将css-loader转换后的代码进一步处理**，**将css-loader导出的字符串加入到页面的style元素中**

所以有了style-loader后，就不需要手动将 css-loader 生产的字符串插到style标签了

例如：

```
.red{
    color:"#f40";
}
```

经过css-loader转换后变成js代码：

```
module.exports = `.red{
    color:"#f40";
}`
```

**webpack.config.js的配置**

```js
module.exports = {
	module:{
		rules:[
			{
				test:/\.css$/,
				use:["style-loader","css-loader"]//这里style-loader在前面，是因为loader是按照从后往前执行的
			}
		]
	}
}
```

经过style-loader转换后变成：

```js
module.exports = `.red{
    color:"#f40";
}`
var style = module.exports;
var styleElem = document.createElement("style");
styleElem.innerHTML = style;
document.head.appendChild(styleElem);
module.exports = {}
```

> 以上代码均为简化后的代码，并不代表真实的代码 style-loader有能力避免同一个样式的重复导入

# **解决类名冲突-----------------------------**

## BEM（命名规范）

与webpack无关

BEM是一套针对css类样式的**命名规范**。

> 其他命名方法还有：OOCSS、AMCSS、SMACSS等等

BEM全称是：**B**lock **E**lement **M**odifier

一个完整的BEM类名：block__element_modifier，例如：`banner__dot_selected`，可以表示：轮播图中，处于选中状态的小圆点

**BEM中：要求类名尽量是在顶级域**：如 **.box{ }**，最好不要使用如： .box  .son  .child { }

因为如果嵌套太多层类名，会导致内存消耗相对较大

三个部分的具体含义为：

- **Block**：页面中的**大区域**，**表示最顶级的划分**，例如：**轮播图(`banner`)、布局(`layout`)、文章(`article`)**等等
- **element**：区域中的**组成部分**，例如：轮播图中的**横幅图片(`banner__img`)**、轮播图中的**容器（`banner__container`）**、布局中的**头部(`layout__header`)**、文章中的**标题(`article__title`)**
- **modifier**：可选。**通常表示状态**，例如：**处于展开状态的**布局左边栏（`layout__left_expand`）、**处于选中状态的**轮播图小圆点(`banner__dot_selected`)

在某些大型工程中，如果使用BEM命名法，还可能会增加一个前缀，来表示类名的用途，常见的前缀有：

- **l**: layout，表示这个样式是用于布局的
- **c**: component，表示这个样式是一个组件，即一个功能区域
- **u**: util，表示这个样式是一个通用的、工具性质的样式
- **j**: javascript，表示这个样式没有实际意义，是专门提供给js获取元素使用的

## css  in   js

也就是在js中书写css

css in js 的**核心思想是**：用一个JS对象来描述样式，而不是css样式表

例如下面的对象就是一个用于描述样式的对象：

```js
const styles = {
    backgroundColor: "#f40",
    color: "#fff",
    width: "400px",
    height: "500px",
    margin: "0 auto"
}
```

由于这种描述样式的方式**根本就不存在类名**，自然不会有类名冲突

至于如何把样式应用到界面上，不是它所关心的事情，你可以用任何技术、任何框架、任何方式将它应用到界面。

> 后续学习的vue、react都支持css in js，可以非常轻松的应用到界面

css in js的**特点**：

- **绝无冲突的可能**：由于它根本不存在类名，所以绝不可能出现类名冲突
- **更加灵活**：可以充分利用JS语言灵活的特点，用各种招式来处理样式
- **应用面更广**：只要支持js语言，就可以支持css in js，因此，在一些用JS语言开发移动端应用的时候非常好用，因为移动端应用很有可能并不支持css
- **书写不便**：书写样式，特别是公共样式的时候，处理起来不是很方便
- **在页面中增加了大量冗余内容**：在页面中处理css in js时，往往是将样式加入到元素的style属性中，会大量增加元素的内联样式，并且可能会有大量重复，不易阅读最终的页面代码

## **css module** 

> 通过命名规范来限制类名太过死板，而css in js虽然足够灵活，但是书写不便。 css module 开辟一种全新的思路来解决类名冲突的问题

#### 思路

css module 遵循以下思路解决类名冲突问题：

1. css的类名冲突往往发生在**大型项目中**
2. 大型项目往往会使用**构建工具**（webpack等）搭建工程
3. **构建工具**允许将css样式切分为更加精细的模块
4. 同JS的变量一样，每个css模块文件中难以出现冲突的类名，**冲突的类名往往发生在不同的css模块文件中**
5. 只需要保证构建工具在合并样式代码后不会出现类名冲突即可

<img src="https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/3.%20css%E5%B7%A5%E7%A8%8B%E5%8C%96/3-5.%20css%20module/assets/2020-01-31-13-54-37.png" alt="img" style="zoom:80%;" />

#### **实现原理**

在webpack中，作为处理css的css-loader，它实现了css module的思想

**要启用**css module，**需要将css-loader的配置`modules`设置为`true`。**

css-loader的实现方式如下：

**webpack.config.js配置**

```js
module.exports = {
	module: {
        rules: [{
            test: /\.css$/,
            use: ["style-loader", "css-loader?modules=true"] // 或直接不写true，如： modules
            //或者
            //use: ["style-loader", {
        	//	loader:"css-loader",
            //	options:{
            //		modules:true
        	//          }
            //    }]
        }]
    }
}
```

![img](https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/3.%20css%E5%B7%A5%E7%A8%8B%E5%8C%96/3-5.%20css%20module/assets/2020-01-31-14-00-56.png)

原理极其简单，**开启了css module后**，css-loader会将样式中的类名进行转换，**转换为一个唯一的hash值**。

由于hash值是根据**模块路径**和**类名**生成的，因此，不同的css模块，哪怕具有相同的类名，转换后的hash值也不一样。

<img src="https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/3.%20css%E5%B7%A5%E7%A8%8B%E5%8C%96/3-5.%20css%20module/assets/2020-01-31-14-04-11.png" alt="img"  />

#### 如何应用样式

css module带来了一个新的问题：源代码的类名和最终生成的类名是不一样的，而开发者只知道自己写的源代码中的类名，并不知道最终的类名是什么，那如何应用类名到元素上呢？

为了解决这个问题，css-loader会导出原类名和最终类名的对应关系，该关系是通过一个对象描述的

![img](https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/3.%20css%E5%B7%A5%E7%A8%8B%E5%8C%96/3-5.%20css%20module/assets/2020-01-31-14-08-49.png)

这样一来，我们就可以在js代码中获取到css模块导出的结果，从而应用类名了

**style-loader为了我们更加方便的应用类名，会去除掉其他信息，仅暴露对应关系**，如：

```js
{a1: "_9kACG8JMsBSOxY42NSsVP", a2: "lBEvhw_VoSRRF-t1mNfhM"}//导入得到的对象
```



### 其他操作



#### 全局类名

某些类名是**全局的**、**静态的**，不需要进行转换，**仅需要在类名位置使用一个特殊的语法**即可：

```css
:global(.main){
    ...
}
```

使用了global的类名不会进行转换，相反的，**没有使用global的类名**，**表示默认使用了local**

```css
:local(.main){
    ...
}
```

**使用了local的类名表示局部类名**，是可能会造成冲突的类名，**会被css module进行转换**

#### 如何控制最终的类名

绝大部分情况下，我们都不需要控制最终的类名，因为控制它没有任何意义

如果一定要控制最终的类名，需要配置css-loader的`localIdentName`

#### 其他注意事项

- css module往往**配合构建工具使用**
- css module**仅处理顶级类名**，尽量不要书写嵌套的类名，也没有这个必要
- css module**仅处理类名**，不处理其他选择器
- css module**还会处理id选择器**，不过任何时候都没有使用id选择器的理由
- 使用了css module后，**只要能做到让类名望文知意即可**，**不需要遵守其他任何的命名规范**

# **解决重复样式的问题**

## 预编译器   less

#### 基本原理

编写css时，受限于css语言本身，常常难以处理一些问题：

- 重复的样式值：例如常用颜色、常用尺寸
- 重复的代码段：例如绝对定位居中、清除浮动
- 重复的嵌套书写

由于官方迟迟不对css语言本身做出改进，一些第三方机构开始想办法来解决这些问题

其中一种方案，便是预编译器

预编译器的原理很简单，即使用一种更加优雅的方式来书写样式代码，通过一个编译器，将其转换为可被浏览器识别的传统css代码

![img](https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/3.%20css%E5%B7%A5%E7%A8%8B%E5%8C%96/3-6.%20%E9%A2%84%E7%BC%96%E8%AF%91%E5%99%A8less/assets/2020-02-03-11-48-45.png)

目前，最流行的预编译器有**LESS**和**SASS**，由于它们两者特别相似，因此仅学习一种即可（本课程学习LESS）

![img](https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/3.%20css%E5%B7%A5%E7%A8%8B%E5%8C%96/3-6.%20%E9%A2%84%E7%BC%96%E8%AF%91%E5%99%A8less/assets/2020-02-03-11-50-05.png)

less官网：http://lesscss.org/ 

less中文文档1（非官方）：http://lesscss.cn/ 

less中文文档2（非官方）：https://less.bootcss.com/ 

sass官网：https://sass-lang.com/ 

sass中文文档1（非官方）：https://www.sass.hk/ 

sass中文文档2（非官方）：https://sass.bootcss.com/

## LESS的安装和使用

从原理可知，要使用LESS，必须要安装LESS编译器

LESS编译器是**基于node开发**的，**可以通过npm下载安装**

```
npm i -D less
```

安装好了less之后，**它提供了一个CLI工具`lessc`**，通过该工具即可完成编译

```
lessc less代码文件 编译后的文件
```

**试一试**:

新建一个`index.less`文件，编写内容如下：

```js
// less代码
@red: #f40;

.redcolor {
    color: @red;
}
```

**运行命令**：

```
lessc index.less index.css
```

**可以看到编译之后的代码**：(会自动在当前目录生成一个 index.css文件)

```
.redcolor {
  color: #f40;
}
```

## LESS的基本使用

具体的使用见文档：https://less.bootcss.com/

- 变量
  **@变量名**

- 混合
  就是创建  **.size{ 样式 }**
  然后，如果需要使用size里面的样式，直接 **.size( )**即可 ，会在生成的css文件中产生  .size样式，可能会导致重复
  而且，如果使用  **.size( ) { 样式 }**，定义，则生成的css文件中，**不会有 .size类样式的产生**

- 嵌套

- 运算

- 函数
  **颜色加深函数：darken**，这样就能很方便的修改网址的主题颜色了

  语法：   darken(颜色变量，深度(百分比) )

  ```css
  @primary:blue;
  .size(){
  	height:100px;
  	margin:10px 0
  }
  .s1{
  	background-color:@primary;
  	.size();
  }
  .s2{
  	background-color:darken(@primary,10%);
  	.size();
  }
  .s3{
  	background-color:darken(@primary,20%);
  	.size();
  }
  ```

  

- 作用域

- 注释

- 导入

## 在webpack中使用 less

**中心思想：**

less代码  --->   css代码 ---->   js代码  ---->    放置到style元素中

![image-20201226131524963](C:\Users\Ronin\AppData\Roaming\Typora\typora-user-images\image-20201226131524963.png)

安装less-loader

```
npm i -D less-loader
```

**配置loader**（顺序必须是反得）

```js
module: {
        rules: [{
            test: /\.less$/,
            use: ["style-loader", "css-loader", "less-loader"]
        }]
    },
```

# PostCss 

> 本节课的内容和webpack无关！！！

## 什么是PostCss

学习到现在，可以看出，CSS工程化面临着诸多问题，而解决这些问题的方案多种多样。

如果把CSS单独拎出来看，光是样式本身，就有很多事情要处理。

既然有这么多事情要处理，何不把这些事情集中到一起统一处理呢？

PostCss就是基于这样的理念出现的。

PostCss类似于一个编译器，可以将样式源码编译成最终的CSS代码

![img](https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/3.%20css%E5%B7%A5%E7%A8%8B%E5%8C%96/3-8.%20PostCss/assets/2020-02-04-14-31-33.png)

看上去是不是和LESS、SASS一样呢？

但PostCss和LESS、SASS的思路不同，它其实只做一些代码分析之类的事情，将分析的结果交给插件，具体的代码转换操作是插件去完成的。

![img](https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/3.%20css%E5%B7%A5%E7%A8%8B%E5%8C%96/3-8.%20PostCss/assets/2020-02-04-14-37-51.png)

官方的一张图更能说明**postcss的处理流程**：

![img](https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/3.%20css%E5%B7%A5%E7%A8%8B%E5%8C%96/3-8.%20PostCss/assets/postcss-workflow.png)

> 这一点有点像webpack，webpack本身仅做依赖分析、抽象语法树分析，其他的操作是靠插件和加载器完成的。

官网地址：https://postcss.org/ github地址：https://github.com/postcss/postcss

## 安装

PostCss是基于node编写的，因此可以使用npm安装

```
npm i -D postcss
```

postcss库提供了对应的js api用于转换代码，如果你想使用postcss的一些高级功能，或者想开发postcss插件，就要api使用postcss，api的文档地址是：http://api.postcss.org/

不过绝大部分时候，我们都是使用者，并不希望使用代码的方式来使用PostCss

因此，**我们可以再安装一个postcss-cli**，通过**命令行**来完成编译

```
npm i -D postcss-cli
```

postcss-cli提供一个命令，它调用postcss中的api来完成编译

命令的使用方式为：

```
postcss 源码文件 -o 输出文件
```

## 配置文件

和webpack类似，**postcss有自己的配置文件**，该配置文件会影响postcss的某些编译行为。

**配置文件的默认名称**是**：`postcss.config.js`**

例如：

```
module.exports = {
    map: false, //关闭source-map
}
```

## 插件

光使用postcss是没有多少意义的，要让它真正的发挥作用，需要插件

postcss的插件市场：https://www.postcss.parts/

下面罗列一些postcss的常用插件

## **postcss-preset-env**

过去使用postcss的时候，往往会使用大量的插件，它们各自解决一些问题

这样导致的结果是安装插件、配置插件都特别的繁琐

于是出现了这么一个插件`postcss-preset-env`，它**称之为`postcss预设环境`**，大意就是**它整合了很多的常用插件到一起，并帮你完成了基本的配置**，你**只需要安装它一个插件**，**就相当于安装了很多插件了**。

安装好该插件后，在**postcss.config.js**中加入下面的配置

```js
module.exports = {
    plugins: {
        "postcss-preset-env": {} // {} 中可以填写插件的配置
    }
}
```

该插件的功能很多，下面一一介绍

#### **自动的厂商前缀**

某些新的css样式需要在旧版本浏览器中使用厂商前缀方可实现

例如

```css
::placeholder {
    color: red;
}
```

该功能在不同的旧版本浏览器中需要书写为

```css
::-webkit-input-placeholder {
    color: red;
}
::-moz-placeholder {
    color: red;
}
:-ms-input-placeholder {
    color: red;
}
::-ms-input-placeholder {
    color: red;
}
::placeholder {
    color: red;
}
```

要完成这件事情，需要使用`autoprefixer`库。

而`postcss-preset-env`内部包含了该库，自动有了该功能。

如果需要调整**兼容的浏览器范围**，可以通过下面的方式进行配置

**方式1（不推荐）：在postcss-preset-env的配置中加入browsers**

```
module.exports = {
    plugins: {
        "postcss-preset-env": {
            browsers: [
                "last 2 version",
                "> 1%"
            ]
        } 
    }
}
```

**方式2【推荐】：在根目录添加 .browserslistrc 文件**

创建文件`.browserslistrc`，填写配置内容

```js
last 2 version
> 1%
```

**方式3【推荐】：在package.json的配置中加入browserslist**

```js
"browserslist": [
    "last 2 version",
    "> 1%"
]
```

`browserslist`是一个多行的（**数组形式的**）标准字符串。

它的书写规范多而繁琐，详情见：https://github.com/browserslist/browserslist

一般情况下，**大部分网站都使用下面的格式进行书写:**

```
last 2 version
> 1% in CN
not ie <= 8
```

- **`last 2 version`:** 浏览器的兼容最近期的两个版本
- **`> 1% in CN`:** 匹配中国大于1%的人使用的浏览器， `in CN`可省略（表示全球）
- **`not ie <= 8`:** 排除掉版本号小于等于8的IE浏览器

> 默认情况下，匹配的结果求的是并集

你可以通过网站：https://browserl.ist/ 对配置结果覆盖的浏览器进行查询，查询时，多行之间使用英文逗号分割

> browserlist的数据来自于[CanIUse](http://caniuse.com/)网站，由于数据并非实时的，所以不会特别准确

#### 未来的CSS语法

CSS的某些前沿语法正在制定过程中，没有形成真正的标准，如果希望使用这部分语法，为了浏览器兼容性，需要进行编译

过去，完成该语法编译的是`cssnext`库，不过有了`postcss-preset-env`后，它自动包含了该功能。

你可以通过`postcss-preset-env`的`stage`配置，告知`postcss-preset-env`需要对哪个阶段的css语法进行兼容处理，它的默认值为2

```
"postcss-preset-env": {
    stage: 0
}
```

一共有5个阶段可配置：

- Stage 0: Aspirational - 只是一个早期草案，极其不稳定
- Stage 1: Experimental - 仍然极其不稳定，但是提议已被W3C公认
- Stage 2: Allowable - 虽然还是不稳定，但已经可以使用了
- Stage 3: Embraced - 比较稳定，可能将来会发生一些小的变化，它即将成为最终的标准
- Stage 4: Standardized - 所有主流浏览器都应该支持的W3C标准

了解了以上知识后，接下来了解一下未来的css语法，尽管某些语法仍处于非常早期的阶段，但是有该插件存在，编译后仍然可以被浏览器识别

#### 变量

未来的css语法是天然支持变量的

在`:root{}`中定义常用变量，使用`--`前缀命名变量

**目前谷歌最新版本其实已经支持这种语法了**

```css
:root{
    --lightColor: #ddd;
    --darkColor: #333;
}

a{
    color: var(--lightColor);
    background: var(--darkColor);
}
```

> 编译后，仍然可以看到原语法，因为某些新语法的存在并不会影响浏览器的渲染，尽管浏览器可能不认识 如果不希望在结果中看到新语法，可以配置`postcss-preset-env`的`preserve`为`false`

#### 自定义选择器

```
@custom-selector :--heading h1, h2, h3, h4, h5, h6;
@custom-selector :--enter :focus,:hover;

a:--enter{
    color: #f40;
}

:--heading{
    font-weight:bold;
}

:--heading.active{
    font-weight:bold;
}
```

编译后

```
a:focus,a:hover{
    color: #f40;
}

h1,h2,h3,h4,h5,h6{
    font-weight:bold;
}

h1.active,h2.active,h3.active,h4.active,h5.active,h6.active{
    font-weight:bold;
}
```

#### 嵌套

与LESS相同，只不过嵌套的选择器前必须使用符号`&`

```
.a {
    color: red;
    & .b {
        color: green;
    }

    & > .b {
        color: blue;
    }

    &:hover {
        color: #000;
    }
}
```

编译后

```
.a {
    color: red
}

.a .b {
    color: green;
}

.a>.b {
    color: blue;
}

.a:hover {
    color: #000;
}
```

#### postcss-apply

该插件可以支持在css中书写属性集

类似于LESS中的混入，可以利用CSS的新语法定义一个CSS代码片段，然后在需要的时候应用它

```
:root {
  --center: {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
  };
}

.item{
    @apply --center;
}
```

编译后

```
.item{
  position: absolute;
  left: 50%;
  top: 50%;
  -webkit-transform: translate(-50%, -50%);
  transform: translate(-50%, -50%);
}
```

> 实际上，该功能也属于cssnext，不知为何`postcss-preset-env`没有支持

#### postcss-color-function

该插件支持在源码中使用一些颜色函数

```
body {
    /* 使用颜色#aabbcc，不做任何处理，等同于直接书写 #aabbcc */
    color: color(#aabbcc);
    /* 将颜色#aabbcc透明度设置为90% */
    color: color(#aabbcc a(90%));
    /* 将颜色#aabbcc的红色部分设置为90% */
    color: color(#aabbcc red(90%));
    /* 将颜色#aabbcc调亮50%（更加趋近于白色），类似于less中的lighten函数 */
    color: color(#aabbcc tint(50%));
    /* 将颜色#aabbcc调暗50%（更加趋近于黑色），类似于less中的darken函数 */
    color: color(#aabbcc shade(50%));
}
```

编译后

```
body {
    /* 使用颜色#aabbcc，不做任何处理，等同于直接书写 #aabbcc */
    color: rgb(170, 187, 204);
    /* 将颜色#aabbcc透明度设置为90% */
    color: rgba(170, 187, 204, 0.9);
    /* 将颜色#aabbcc的红色部分设置为90% */
    color: rgb(230, 187, 204);
    /* 将颜色#aabbcc调亮50%（更加趋近于白色），类似于less中的lighten函数 */
    color: rgb(213, 221, 230);
    /* 将颜色#aabbcc调暗50%（更加趋近于黑色），类似于less中的darken函数 */
    color: rgb(85, 94, 102);
}
```

#### [扩展]postcss-import

该插件可以让你在`postcss`文件中导入其他样式代码，通过该插件可以将它们合并

> 由于后续的课程中，会将postcss加入到webpack中，而webpack本身具有依赖分析的功能，所以该插件的实际意义不大

#### **postcss在webpack的使用**

**使用原理：**

**postcss   ----->   普通的css ------>   js   ------->   style标签**

![image-20201227001447608](C:\Users\Ronin\AppData\Roaming\Typora\typora-user-images\image-20201227001447608.png)

**使用过程：**

**使用前需要先安装  postcss    和   postcss-loader**

```
npm i -D postcss   postcss-loader
```

**配置webpack.config.js**

```js
module.exports = {
    //配置loader
	module:{
		rules:[
			{
				test:/\.pcss$/,
				use:["style-loader","css-loader","post-loader"]
			}
		]
	}
}
```

**配置postcss.config.js**

```js
module.exports = {
	map:false,//关闭source-map
	//配置插件
	plugins:[
		"postcss-preset-env":{//自动补充前缀
			stage:0,//哪怕是处于草案阶段的语法，也需要转换
			preserve:false
		},
		"postcss-apply":{//可以使用属性集的一些语法，不需要可不写
            
        },
		"postcss-color-function":{//可以使用一些颜色的函数（如，颜色加深），不需要可不写
            
        }
	]
}
```

**创建一个 .browserslistrc 文件，用于设置可兼容的浏览器和浏览器版本**

该文件会被 postcss里面的 postcss-preset-env 读取

```js
last 3 version //表示允许兼容三个浏览器，那么他就会在补充兼容代码时，只补充三个浏览器的兼容代码
>1%  
not ie <= 8 //表示在 ie8及ie8以下的浏览器不兼容
```

**使用方法是：在index.js导入一次（为了让webpack解析）**

```js
import "./assets/index.pcss";
```



## stylelint

> 官网：https://stylelint.io/

在实际的开发中，**我们可能会错误的或不规范的书写一些css代码**，**stylelint插件会即时的发现错误**

由于不同的公司可能使用不同的CSS书写规范，stylelint为了保持灵活，它本身并没有提供具体的规则验证

你需要安装或自行编写规则验证方案

通常，我们会安装`stylelint-config-standard`库来提供标准的CSS规则判定

安装好后，我们需要告诉stylelint使用该库来进行规则验证

告知的方式有多种，比较常见的是**使用文件`.stylelintrc`**

```js
//.styleintrc
{
  "extends": "stylelint-config-standard"
}
```

此时，如果你的代码出现不规范的地方，编译时将会报出错误

```css
body {
    background: #f4;
}
```

![img](https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/3.%20css%E5%B7%A5%E7%A8%8B%E5%8C%96/3-8.%20PostCss/assets/2020-02-05-14-37-11.png)

发生了两处错误：

1. 缩进应该只有两个空格
2. 十六进制的颜色值不正确

如果某些规则并非你所期望的，可以在配置中进行设置

```
{
    "extends": "stylelint-config-standard",
    "rules": {
        "indentation": null
    }
}
```

设置为`null`可以禁用该规则，或者设置为4，表示一个缩进有4个空格。具体的设置需要参见stylelint文档：https://stylelint.io/

但是这种错误报告需要在编译时才会发生，如果我希望在编写代码时就自动在编辑器里报错呢？

既然想在编辑器里达到该功能，那么就要在编辑器里做文章

安装vscode的插件`stylelint`即可，它会读取你工程中的配置文件，按照配置进行实时报错

> 实际上，如果你拥有了`stylelint`插件，可以不需要在postcss中使用该插件了

# 抽离css文件

目前，css代码被css-loader转换后，交给的是style-loader进行处理。

style-loader使用的方式是用一段js代码，将样式加入到style元素中。

而实际的开发中，我们往往希望依赖的样式最终形成一个css文件

此时，就需要用到一个库**：`mini-css-extract-plugin`**

**该库提供了1个plugin和1个loader**

- **plugin**：负责生成css文件
- **loader**：负责记录要生成的css文件的内容，**同时导出开启css-module后的样式对象**（所以css-loader配置需要开启module）

使用方式：

**不需要style-loader加载器了（因为他会把css-loader生成的js代码插到页面）**

```js
const MiniCssExtractPlugin = require("mini-css-extract-plugin")
module.exports = {
    module: {
        rules: [
            {
                test: /\.css$/, use: [MiniCssExtractPlugin.loader, "css-loader?modules"]
            }
        ]
    },
    plugins: [
        new MiniCssExtractPlugin() //负责生成css文件
    ]
}
```

**配置生成的文件名**

同`output.filename`的含义一样，即根据chunk生成的样式文件名

配置生成的文件名，**例如`[name].[contenthash:5].css`**（他也是根据**entry**的设置的chunk命名）

**默认情况下，每个chunk对应一个css文件**