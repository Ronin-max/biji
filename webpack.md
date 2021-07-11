# webpack核心原理---------------------

## 浏览器端的模块化

**问题：**

- **效率问题**：精细的模块划分带来了更多的JS文件，更多的JS文件带来了更多的请求，降低了页面访问效率
- **兼容性问题**：浏览器目前仅支持ES6的模块化标准，并且还存在兼容性问题
- **工具问题**：浏览器不支持npm下载的第三方包

这些仅仅是前端工程化的一个缩影（也就是说问题远远不止这些）

当开发一个具有规模的程序，你将遇到非常多的非业务问题，这些问题包括：执行效率、兼容性、代码的可维护性可扩展性、团队协作、测试等等等等，我们将这些问题称之为工程问题。工程问题与业务无关，但它深刻的影响到开发进度，如果没有一个好的工具解决这些问题，将使得开发进度变得极其缓慢，同时也让开发者陷入技术的泥潭。

#### 根本原因

思考：上面提到的问题，为什么在node端没有那么明显，反而到了浏览器端变得如此严重呢？

答：在node端，运行的JS文件在本地，因此可以本地读取文件，它的效率比浏览器远程传输文件高的多

**根本原因**：在浏览器端，开发时态（devtime）和运行时态（runtime）的侧重点不一样

**开发时态，devtime：**（希望能解决的点）

1. 模块划分越细越好
2. 支持多种模块化标准
3. 支持npm或其他包管理器下载的模块
4. 能够解决其他工程化的问题

**运行时态，runtime：**（希望能解决的点）

1. 文件越少越好
2. 文件体积越小越好
3. 代码内容越乱越好（为了防止别人在网页查看了代码，被人修改，越乱越安全）
4. 所有浏览器都要兼容
5. 能够解决其他运行时的问题，主要是执行效率问题

这种差异在小项目中表现的并不明显，可是一旦项目形成规模，就越来越明显，如果不解决这些问题，前端项目形成规模只能是空谈

#### 解决办法

既然开发时态和运行时态面临的局面有巨大的差异，因此，我们需要有一个工具，这个工具能够让开发者专心的在开发时态写代码，然后利用这个工具将开发时态编写的代码转换为运行时态需要的东西。

这样的工具，叫做**构建工具**

<img src="C:\Users\Ronin\AppData\Roaming\Typora\typora-user-images\image-20201217213841226.png" alt="image-20201217213841226" style="zoom:67%;" />

这样一来，开发者就可以专注于开发时态的代码结构，而不用担心运行时态遇到的问题了。

#### 常见的构建工具

- **webpack**
- grunt
- gulp
- browserify
- fis
- 其他

## webpack简介

webpack是基于模块化的打包（构建）工具，它把一切视为模块

它通过一个开发时态的入口模块为起点，分析出所有的依赖关系，然后经过一系列的过程（压缩、合并），最终生成运行时态的文件。

webpack的特点：

- **为前端工程化而生**：webpack致力于解决前端工程化，特别是浏览器端工程化中遇到的问题，让开发者集中注意力编写业务代码，而把工程化过程中的问题全部交给webpack来处理
- **简单易用**：支持零配置，可以不用写任何一行额外的代码就使用webpack
- **强大的生态**：webpack是非常灵活、可以扩展的，webpack本身的功能并不多，但它提供了一些可以扩展其功能的机制，使得一些第三方库可以融于到webpack中
- **基于nodejs**：**由于webpack在构建的过程中需要读取文件，因此它是运行在node环境中的**
  （并不是指开发时要在node中进行，是指使用webpack打包时，需要在node环境中运行）
- **基于模块化**：webpack在构建过程中要分析依赖关系，方式是通过模块化导入语句进行分析的，它**支持各种模块化标准**，包括但不限于**CommonJS**、**ES6 Module**

## webpack的安装

webpack通过npm安装，它提供了两个包：

- **webpack**：核心包，包含了webpack构建过程中要用到的所有api
- **webpack-cli**：提供一个简单的cli命令，它调用了webpack核心包的api，来完成构建过程

安装方式：

- 全局安装：可以全局使用webpack命令，但是无法为不同项目对应不同的webpack版本

  ```
  npm i -D webpack webpack-cli -g
  ```

  

- **本地安装**：**推荐**，每个项目都使用自己的webpack版本进行构建

  ```js
  npm i -D webpack webpack-cli
  // -D 安装到生产环境的依赖，是因为当webpack构建完工程后，后面的工程运行的时候已经不需要webpack了
  ```

  

## 使用

```js
//全局安装
webpack

//本地安装
npx webpack
```

默认情况下，**webpack会以`./src/index.js`**作为**入口文件**分析依赖关系，**打包到`./dist/main.js`文件中**

通过**--mode选项**可以**控制webpack的打包结果的运行环境**（也就是说：控制打包完之后的文件在生产环境运行还是开发环境运行）

```js
npx webpack --mode=development//表示打包完的文件在开发环境运行    production表示生产环境
```

所以我们可以在package.json文件的scripts脚本中设置快捷命令

**如果在后面加上   --watch，表示监控，所以加了这个参数之后，只要文件变化，就会自动执行该命令，方便开发使用**

```js
//设置脚本中npx可以省略
"build" : "webpack --mode=production --watch",
"dev" : "webpack --mode=development"
```

**如果开发使用了ES6模块化，打包出来的文件中，因为webpack已经通过其他方式模拟了模块化，所以后面页面引入main.js不需要设置 type=“module”了**

## 模块化兼容性

由于webpack同时支持CommonJS和ES6 module，因此需要理解它们互操作时webpack是如何处理的

#### 同模块化标准

如果导出和导入使用的是同一种模块化标准，打包后的效果和之前学习的模块化没有任何差异

#### **不同模块化标准**

不同的模块化标准，webpack按照如下的方式处理

**ES6导出 ------>  CommonJS导入**

CommonJS的**require**导入，相当于ES6的 **import * from “./a”;**

**需要注意的是：CommonJS导入的对象的属性，才是ES6导出的值**

如： ES6: export default = 3;        CommonJS : const obj = require(“./a”);            // **obj = { default: 3 }**

![image-20201217230113431](C:\Users\Ronin\AppData\Roaming\Typora\typora-user-images\image-20201217230113431.png)

**CommonJS导出 ------>ES6导入**

ES6的**导入**有两种方式是一样效果：**import * from “./a.js”**、 **import c from “./a.js”**（**也就是全部导出或默认导出**），

相当于CommonJS的 **require** 导入（得到的是一个**module.exports导出的对象**）；

还有一种就是：**import { a , b } from “./a.js”**（可以简单理解为对象解构，得到a = 1,b = 2）

![image-20201217230807722](C:\Users\Ronin\AppData\Roaming\Typora\typora-user-images\image-20201217230807722.png)

**所以有了webpack后，不同的模块化也可以相互使用**

## 最佳实践

代码编写最忌讳的是精神分裂，选择一个合适的模块化标准，然后贯彻整个开发阶段。

**（绝大部分的第三方库，使用的是CommonJS的方式导出）**

## webpack打包模块化原理

**模拟使用CommonJS模块化打包的原理**

**注意：只有在开发环境才是这样的原理( 使用eval )，生产环境不是**

```js
//合并两个模块
//  ./src/a.js
//  ./src/index.js
//每一个路径都是唯一的，并且可以理解为每一个路径对应着一个函数(模块)
(function (modules) {
    var moduleExports = {}; //用于缓存模块的导出结果

    //require函数（也就是__webpack_require）相当于是运行一个模块，得到模块导出结果(module.exports的结果)
    function __webpack_require(moduleId) { //moduleId就是模块的路径
        if (moduleExports[moduleId]) {
            //检查是否有缓存
            return moduleExports[moduleId];
        }

        var func = modules[moduleId]; //得到该模块对应的函数（也就是传进来对象的值）
        var module = {//CommonJS的module
            exports: {}
        }
        func(module, module.exports, __webpack_require); //运行模块
        var result = module.exports; //得到模块导出的结果
        moduleExports[moduleId] = result; //缓存起来
        return result;
    }

    //执行入口模块
    return __webpack_require("./src/index.js"); //require函数相当于是运行一个模块，得到模块导出结果
})({ //该对象保存了所有的模块，以及模块对应的代码
    "./src/a.js": function (module, exports) {//以地址作为对象的属性，函数作为值进行传递到立即执行函数中
        //console.log("module a");
        //module.exports = "a";
        //eval可以把字符串当作代码执行，相当于上面代码执行
        //并且使用eval的原因是：方便在浏览器调试，点击报错信息进去调试时，不会显示其他无关代码，并且可以设置报错信息
        //    //# sourceURL=webpack:///./src/a.js"  这段代码就是设置浏览器报错提示信息，不设置时是显示vmxxxx:js第几行
        eval("console.log(\"module a\")\nmodule.exports = \"a\";\n //# sourceURL=webpack:///./src/a.js")
    },
    "./src/index.js": function (module, exports, __webpack_require) {
        //webpack为了避免require冲突，所以使用__webpack_require命名导入
        //console.log(index module);
        //var a = require("./src/a.js");
        //console.log(a);
        eval("console.log(\"index module\")\nvar a = __webpack_require(\"./src/a.js\")\na.abc();\nconsole.log(a)\n //# sourceURL=webpack:///./src/index.js")
      
    }
});
```

## **配置文件**

webpack提供的cli支持很多的参数，**例如`--mode`**，但更多的时候，我们会使用更加灵活的配置文件来控制webpack的行为

**默认情况下**，webpack会**读取`webpack.config.js`文件**作为**配置文件**，**webpack.config.js文件一般创建在根目录中**

但也可以**通过CLI参数`--config`**来**指定某个配置文件**

配置文件中通过CommonJS模块**导出一个对象**，**对象中的各种属性对应不同的webpack配置**

**注意：配置文件中的代码，必须是有效的node代码**
**原因：**

首先，我们开发阶段写的代码最后是不参与运行的，所以可以随便使用模块化。但是因为这个配置文件（webpack.config.js）**是需要运行的**，而且需要在**打包过程中运行**，所以**只能在node环境中运行**，而ES6模块化是不能在node环境执行的，并且配置文件在打包完成后是完全不参与运行的，是对工程无关的，只是在打包过程执行。
	**至于webpack是兼容ES6模块化和CommonJS模块化是指**：支持在开发时态兼容两种模块化，并且写的那些导入依赖或导出，都是为了让webpack直到该文件依赖了哪些文件，并且通过其他方式模拟出相同的功能（原理是使用了立即执行函数）。

当命令行参数与配置文件中的配置出现冲突时，**以命令行参数为准**。

**基本配置：**
**之前这些东西是需要手动命令设置的，或者为了方便，设置了脚本快捷命令**
有了这个配置文件，就相当于多了一种配置方式

1. **mode**：编译模式，字符串，取值为**development**或**production**，指定编译结果代码运行的环境，会影响webpack对编译结果代码格式的处理

2. **entry**：入口，字符串（后续会详细讲解），指定入口文件

3. **output**：出口，对象（后续会详细讲解），指定编译结果文件
   配置方式就是通过CommonJS导出，如：

   ```js
   module.exports = {
   	mode:"development",//打包生成开发环境的代码
   	entry:"./src/index.js",//指定入口文件
   	output:{
           filename:"main.js"//打包结果文件
       }
   }
   ```

   

**面试可能常问：为什么webpack需要使用CommonJS，为什么要在node环境进行**

上面有描述

对照下图，按照理解说明即可：打包过程需要执行一些配置文件，而执行这些文件的环境只有在node环境才可以直接执行文件。浏览器只能执行html等文件。
		并且开发过程如何使用模块化，对于最后执行文件并没有影响，因为在打包过程中，已经将相关模块化的依赖通过其他方式模拟了一样的功能了，所以可以理解为最后打包的结果其实并没有使用模块化。

<img src="C:\Users\Ronin\AppData\Roaming\Typora\typora-user-images\image-20201218225021176.png" alt="image-20201218225021176" style="zoom: 67%;" />

## devtool 配置

[toc]

#### source map 源码地图

> 本小节的知识与 webpack 无关

前端发展到现阶段，**很多时候都不会直接运行源代码**，**可能需要对源代码进行合并、压缩、转换等操作**，**真正运行的是转换后的代码**

这就**给调试带来了困难**，因为当运行发生错误的时候，我们更加希望能看到源代码中的错误，而不是转换后代码的错误

> jquery压缩后的代码：https://code.jquery.com/jquery-3.4.1.min.js

为了解决这一问题，chrome浏览器率先支持了source map，其他浏览器纷纷效仿，**目前**，几乎**所有新版浏览器都支持了source map**

source map实际上是一个**配置**，配置中不仅**记录了所有源码内容**，**还记录了和转换后的代码的对应关系**

下面是浏览器处理source map的原理

![image-20201218230210753](C:\Users\Ronin\AppData\Roaming\Typora\typora-user-images\image-20201218230210753.png)

![image-20201218230424634](C:\Users\Ronin\AppData\Roaming\Typora\typora-user-images\image-20201218230424634.png)

**最佳实践**：

1. source map 应在开发环境中使用，作为一种调试手段
2. source map 不应该在生产环境中使用，source map的文件一般较大，不仅会导致额外的网络传输，还容易暴露原始代码。即便要在生产环境中使用source map，用于调试真实的代码运行问题，也要做出一些处理规避网络传输和代码暴露的问题。

## webpack中的source map

上面webpack打包原理中说到eval的使用，其实eval也可以理解为一个简易版的source map

使用 webpack 编译后的代码难以调试，可以通过在配置文件使用 devtool 配置来**优化调试体验**（其实就是为了在浏览器调试更方便）

如：

```js
module.exports = {
	devtool:"none"//表示直接就是打包后的代码，没有做任何配置（也就是没有源码地图）
}
```

具体的配置见文档：https://www.webpackjs.com/configuration/devtool/

## **webpack 编译过程**

webpack 的作用是将源代码编译（构建、打包）成最终代码

**整个过程大致分为三个步骤**：

1. 初始化
2. 编译
3. 输出

#### 初始化

此阶段，webpack会将**CLI参数**、**配置文件**、**默认配置**进行融合，形成一个最终的配置对象。

对配置的处理过程是依托一个第三方库`yargs`完成的

此阶段相对比较简单，主要是为接下来的编译阶段做必要的准备

目前，可以简单的理解为，初始化阶段主要用于产生一个最终的配置

#### 编译

1.**创建chunk**

chunk是webpack在内部构建过程中的一个概念，译为`块`，它表示通过某个入口找到的所有依赖的统称。

**可以理解为：通过入口文件找到所有的依赖关系，而这所有依赖组成的东西就成为chunk（块）**

根据入口模块（**默认为`./src/index.js`**）创建一个chunk

<img src="C:\Users\Ronin\AppData\Roaming\Typora\typora-user-images\image-20201219134826928.png" alt="image-20201219134826928" style="zoom: 50%;" />

**每个chunk都有至少两个属性**：

- **name**：默认为main
- **id**：唯一编号，开发环境和name相同，生产环境是一个数字，从0开始

2.**构建所有依赖模块**

<img src="https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/1.%20webpack%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/1-8.%20%E7%BC%96%E8%AF%91%E8%BF%87%E7%A8%8B/assets/2020-01-09-12-32-38.png" alt="img" style="zoom:67%;" />

> AST在线测试工具：https://astexplorer.net/

**保存到dependencies中后：查找到导入模块的地址（如：require( “./a.js” )），将它替换成 _webpack_require(“./ src / a.js”)，也就是前面webpack打包原理的属性名，转换完之后的代码再记录到 chunk中的模块记录**

**上图说根据dependencies的内容递归加载模块**：是指如果入口模块有不止一个依赖，他会先从第一个依赖开始查找、分析、保存，当第一个依赖加载好之后，又会回到入口模块再从第二个依赖开始查找，如果第二个模块在dependencies有记录，则直接拿加载好的。

**模拟过程**：

```js
//查看入口文件 ./src/index.js ，显示未加载

//加载模块index
require("./a");
console.log("index");
module.exports = "index";

//分析依赖
["./src/a.js"] //分析完后保存到依赖项中的数组里面

//然后替换依赖函数
_webpack_require("./src/a.js");
console.log("index");
module.exports = "index";

//然后将转换后的代码保存起来     格式是：模块id:"./src/a.js"  转换后的代码（字符串形式）：...

//然后又去找  index依赖的 a模块，然后又是按照上面步骤进行，直到所有依赖都加载过一次后，如果遇到是已加载的依赖，就不需要加载了
```

**编译过程结束后，开始打包生成 “./dist / main.js”文件**

```js
//文件名： ./dist/main.js
//文件内容：根据一个固定的模板，把对应内容放到里面
(function(modules){

})({
	"./src/index.js":function(module,exports,_webpack_require){//属性名：模块id
		//根据编译时保存的  转换后的代码，放到这里面
		eval(index模块转换后的代码);
	},
    "./src/a.js":function(module,exports,_webpack_require){
        eval(a模块转换后的代码);
    }
})

```

简图

<img src="https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/1.%20webpack%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/1-8.%20%E7%BC%96%E8%AF%91%E8%BF%87%E7%A8%8B/assets/2020-01-09-12-35-05.png" alt="img" style="zoom: 80%;" />

3.**产生chunk assets**（资源）

在第二步完成后，chunk中会产生一个模块列表，列表中包含了**模块id**和**模块转换后的代码**

接下来，webpack会根据配置为chunk生成一个**资源列表**，**即`chunk assets`**，资源列表可以理解为是生成到最终文件的文件名和文件内容

<img src="https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/1.%20webpack%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/1-8.%20%E7%BC%96%E8%AF%91%E8%BF%87%E7%A8%8B/assets/2020-01-09-12-39-16.png" alt="img" style="zoom:67%;" />

> **chunk hash**是**根据所有chunk assets的内容生成的一个hash字符串** 
>
> **hash：**一种算法，具体有很多分类，特点是将一个任意长度的字符串转换为一个固定长度的字符串，而且可以保证原始内容不变，产生的hash字符串就不变
>
> **hash的作用：它可以反映文件内容有没有变化，因为文件内容一旦改变，哈希值就会变化，如果文件内容不改变，则哈希值是不会变化的。**

简图

![img](https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/1.%20webpack%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/1-8.%20%E7%BC%96%E8%AF%91%E8%BF%87%E7%A8%8B/assets/2020-01-09-12-43-52.png)

4.**合并chunk assets**

将多个chunk的assets合并到一起，并产生一个总的hash

<img src="https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/1.%20webpack%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/1-8.%20%E7%BC%96%E8%AF%91%E8%BF%87%E7%A8%8B/assets/2020-01-09-12-47-43.png" alt="img" style="zoom:80%;" />

所以其实**编译过程最终**是生成了一个  **总的资源列表，列表中存储着模块id 和 转换后的代码，以及hash字符串**

#### 输出

此步骤非常简单，webpack将**利用node中的fs模块**（文件处理模块），**根据编译产生的总的assets，生成相应的文件**。

<img src="https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/1.%20webpack%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/1-8.%20%E7%BC%96%E8%AF%91%E8%BF%87%E7%A8%8B/assets/2020-01-09-12-54-34.png" alt="img" style="zoom:80%;" />

#### 总过程

<img src="https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/1.%20webpack%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/1-9.%20%E5%85%A5%E5%8F%A3%E5%92%8C%E5%87%BA%E5%8F%A3/assets/2020-01-09-15-51-07.png" alt="img" style="zoom:80%;" />

**涉及术语**

1. module：模块，分割的代码单元，webpack中的模块可以是任何内容的文件，不仅限于JS
2. chunk：webpack内部构建模块的块，一个chunk中包含多个模块，这些模块是从入口模块通过依赖分析得来的
3. bundle：chunk构建好模块后会生成chunk的资源清单，清单中的每一项就是一个bundle，可以认为bundle就是最终生成的文件
4. hash：最终的资源清单所有内容联合生成的hash值
5. chunkhash：chunk生成的资源清单内容联合生成的hash值
6. chunkname：chunk的名称，如果没有配置则使用main
7. id：通常指chunk的唯一编号，如果在开发环境下构建，和chunkname相同；如果是生产环境下构建，则使用一个从0开始的数字进行编号

## 入口和出口

**JS文件里面 ./  的两种表示**

1. 模块化代码中，**比如require("./")**，表示**当前js文件所在的目录**（相对路径）

2. 在路径处理中，**"./"**表示**node运行目录**（也就是运行当前文件的文件夹路径，绝对路径）

   **__dirname**：是node环境中global的一个属性，所有情况下，都表示**当前运行的js文件所在的目录**，它是一个**绝对路径**

> **node内置模块** - **path**: https://nodejs.org/dist/latest-v12.x/docs/api/path.html
>
> ```js
> //该对象提供了大量路径处理的函数
> var path = require("path");//导出一个对象
> var result = path.resolve("./","child","abc","123"); //会将参数与当前文件所在路径整合成一个绝对路径
> console.log(result);//E:\小练习\渡一demo\webpack\child\abc\123
> ```
>
> 

<img src="https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/1.%20webpack%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/1-9.%20%E5%85%A5%E5%8F%A3%E5%92%8C%E5%87%BA%E5%8F%A3/assets/2020-01-09-15-51-07.png" alt="img" style="zoom:80%;" />

#### **出口**

这里的出口是针对**资源列表的文件名或路径的配置**

出口通过**output进行配置**

**在webpack.config.js文件配置:**

```js
var path = require("path");//为了可以兼容其他操作系统的路径书写（因为不同操作系统，路径的写法可能不同）
module.exports = {
	mode:"development",
    //出口
	output:{
         filename:"target.js"//配置chunk输出的文件名（默认是main.js）
		path:path.resolve(__dirname,"target")//必须配置一个绝对路径，表示资源放置的文件夹，默认是dist
    //path.resolve(__dirname,"target") 表示返回 当前文件夹(webpack.config.js)所在目录下 target文件夹 的绝对路径字符串
	}
}
```

**filename的其他写法：**

- **静态写法**：直接写文件名.js
- **相对路径写法**： 如 **target / main.js**，这样他就**会在path配置的目录下**  的 **target文件夹下**，并**命名为  main.js**
- **动态配置**：**[name].js**，**表示以chunk的name为名称输出**。

#### **入口**

**入口真正配置的是chunk**

入口通过**entry进行配置**

```js
var path = require("path");
module.exports = {
	mode:"development",
    //入口
	entry:{//属性名：配置chunk的名称（如下，将index.js模块的chunk名称配置为main），属性值：入口模块的地址（也称启动模块）
		main:"./src/index.js"//默认写法  如果只配置一个chunk的时候，也相当于这样写：  entry:"./src/index.js",
         a:"./src/a.js" //将a.js模块的chunk名称配置为 a
        
    //这样配置了两个chunk 并且 输出是以动态写法配置名称，那么最终输出也会是两个js文件，并且分别以 配置的chunk命名
	}
    output:{
    	path:path.resolve(__dirname,"target"),
         filename:"[name]-[hash : 4].js" //加hash是为了解决浏览器缓存问题（中间的 - 是随便写的，加什么都行）
	}
}
```

**注意：**

- **如果入口配置了两个chunk，但是出口文件名 没有使用动态配置，则会导致报错：文件冲突**
- 一个chunk还可以有两个启动模块，多个模块用数组装起来，如：  **a : [ “ ./src / a.js” , “ ./src / index.js”]**，这样配置，则**执行chunk输出的文件时**，他会执行 **a.js模块的代码**  和  **index.js模块的代码**

#### **配置出口filename的规则**：

- **name**：chunk在入口配置的name（**也就是entry的属性名**）

- **hash**: **总的资源hash**（也就是说，这是最终合并后的hash值，所以一旦有一个chunk内容改变了，就会导致其他的chunk生成的名字也会改变，因为用的是同一个hash），通常用于**解决浏览器缓存问题**（**因为hash会因为文件内容的改变而改变**）

  **hash的使用**：**[ hash : 取几位数 ]**

  **缓存问题**：也就是浏览器会对当前请求的文件（包括js文件）进行缓存，如果js文件内容发生了改变，但是浏览器看到文件名并  没有改变，导致浏览器去读取了 之前缓存的文件，进而导致页面内容没有发生改变。

- **chunkhash**: **使用每一个chunk的hash**（使用chunkhash，就不会导致一个chunk内容改变，全部chunk输出的文件名都跟着改变）

- **id**: 使用chunkid，不推荐（因为在**开发模式**的id 是**名字**，**生成模式**的id 是**数字**）

## 入口和出口的最佳实践 {ignore}

具体情况具体分析

下面是一些经典场景

#### 一个页面一个JS

<img src="https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/1.%20webpack%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/1-10.%20%E5%85%A5%E5%8F%A3%E5%92%8C%E5%87%BA%E5%8F%A3%E7%9A%84%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5/assets/2020-01-10-12-00-28.png" alt="img" style="zoom:80%;" />

**源码结构**（也就是src目录）

```
|—— src
    |—— pageA   页面A的代码目录
        |—— index.js 页面A的启动模块
        |—— ...
    |—— pageB   页面B的代码目录
        |—— index.js 页面B的启动模块
        |—— ...
    |—— pageC   页面C的代码目录
        |—— main1.js 页面C的启动模块1 例如：主功能
        |—— main2.js 页面C的启动模块2 例如：实现访问统计的额外功能
        |—— ...
    |—— common  公共代码目录
        |—— ...
```

**webpack配置**（也就是 webpack.config.js文件）

```
module.exports = {
    entry:{
        pageA: "./src/pageA/index.js",
        pageB: "./src/pageB/index.js",
        pageC: ["./src/pageC/main1.js", "./src/pageC/main2.js"]
    },
    output:{
        filename:"[name].[chunkhash:5].js"
    }
}
```

**这种方式适用于页面之间的功能差异巨大、公共代码较少的情况，这种情况下打包出来的最终代码不会有太多重复**

如果工共代码比较多，那么也就意味着生成的文件中会出现较多重复代码（注意：这里指的工共代码是指开发时可以公用的代码）

以后**面试**可能会被问道：

**打包出来的js文件过后，他里边存在着工共代码，而这些工共代码的重复会造成什么样的影响？**

**答**：

这样会**导致页面**的**传输量变大**，因为生成的文件中，**都有着重复代码出现**，也就**相当于工共代码被传输了很多次**，**但是并不会影响后期的维护**，因为维护代码时，是在开发时态进行维护的，而那时候的代码并没有出现重复代码，因为重复代码都在工共代码文件中。

#### 一个页面多个JS

<img src="https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/1.%20webpack%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/1-10.%20%E5%85%A5%E5%8F%A3%E5%92%8C%E5%87%BA%E5%8F%A3%E7%9A%84%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5/assets/2020-01-10-12-38-03.png" alt="img" style="zoom:80%;" />

源码结构

```
|—— src
    |—— pageA   页面A的代码目录
        |—— index.js 页面A的启动模块
        |—— ...
    |—— pageB   页面B的代码目录
        |—— index.js 页面B的启动模块
        |—— ...
    |—— statistics   用于统计访问人数功能目录
        |—— index.js 启动模块
        |—— ...
    |—— common  公共代码目录
        |—— ...
```

webpack配置

```
module.exports = {
    entry:{
        pageA: "./src/pageA/index.js",
        pageB: "./src/pageB/index.js",
        statistics: "./src/statistics/index.js"
    },
    output:{
        filename:"[name].[chunkhash:5].js"
    }
}
```

这种方式适用于页面之间有一些**独立**、**相同的**功能，**专门使用一个chunk抽离这部分JS有利于浏览器更好的缓存这部分内容。**

因为如果**使用一个chunk抽离这部分**，那么就**会独立生成一个js文件**，当浏览器**第一次读到该文件后**就会**缓存起来**，后面如果其他页面还需要用到该文件，**只需要走缓存就行了**，**减少了代码的传输量**

> 思考：**为什么不使用多启动模块的方式？**
>
> 答： 因为**如果使用多启动模块**而不是抽离的方式，如： pageA: [ "./src/pageA/index.js" , "./src/statistics/index.js" ] ，**就会因为没有将重复代码重新创建一个chunk从而生成一个js文件，以至于浏览器并没有缓存这些重复代码，导致每个页面使用都会重复加载重复的代码**

#### 单页应用

所谓单页应用，是指整个网站（或网站的某一个功能块）**只有一个页面**，页面中的**内容全部靠JS创建和控制**。 **vue和react都是实现单页应用的利器。**

![img](https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/1.%20webpack%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/1-10.%20%E5%85%A5%E5%8F%A3%E5%92%8C%E5%87%BA%E5%8F%A3%E7%9A%84%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5/assets/2020-01-10-12-44-13.png)

源码结构

```
|—— src
    |—— subFunc   子功能目录
        |—— ...
    |—— subFunc   子功能目录
        |—— ...
    |—— common  公共代码目录
        |—— ...
    |—— index.js
```

webpack配置

```
module.exports = {
    entry: "./src/index.js",
    output:{
        filename:"index.[hash:5].js"
    }
}
```

## loader

> webpack做的事情，仅仅是分析出各种模块的依赖关系，然后形成资源列表，最终打包生成到指定的文件中。 **更多的功能需要借助webpack loaders和webpack plugins完成。**

**webpack loader**： loader**本质上是一个函数**，它的作用是**将某个源码字符串转换成另一个源码字符串返回**。

<img src="https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/1.%20webpack%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/1-11.%20loader/assets/2020-01-13-10-39-24.png" alt="img" style="zoom:80%;" />

loader函数的将在模块解析的过程中被调用，以得到最终的源码。

**全流程：**

<img src="https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/1.%20webpack%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/1-11.%20loader/assets/2020-01-13-09-28-52.png" alt="img" style="zoom: 67%;" />

**chunk中解析模块的流程：**

![img](https://github.com/DuYi-Edu/DuYi-Webpack/raw/master/1.%20webpack%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/1-11.%20loader/assets/2020-01-13-09-29-08.png)

#### **chunk中解析模块的更详细流程：**

<img src="https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/1.%20webpack%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/1-11.%20loader/assets/2020-01-13-09-35-44.png" alt="img" style="zoom: 80%;" />

#### **处理loaders流程：**

<img src="https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/1.%20webpack%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/1-11.%20loader/assets/2020-01-13-10-29-54.png" alt="img" style="zoom:80%;" />

#### **loader配置：**

**配置前**会在根目录创建一个文件夹（通常就叫loaders），里面放着一些规则(js文件)，通过module.exports导出一个函数，用于对匹配到的模块进行处理。如：

```js
module.exports = function(sourceCode){//接收的参数是  匹配模块里的代码
	//sourceCode : 变量 a = 1;
	return "var a = 1";//返回新的字符串，然后就会拿这个代码去合成抽象语法树
}
```

**完整配置**

```js
module.exports = {
    module: { //针对模块的配置，目前版本只有两个配置，rules、noParse
        rules: [ //模块匹配规则，可以存在多个规则
            { //每个规则是一个对象
                test: /\.js$/, //匹配的模块正则   匹配模块的路径 含有.js的字符  也可以根据对应模块名匹配: /index.js$/
                use: [ //匹配到后，使用哪些加载器（loader）
                    {  //其中一个规则
                        loader: "模块路径", //loader模块的路径(如：./loaders/test-loader)，该字符串会被放置到require中
                        options: { //向对应loader传递的额外参数，主要也是用于匹配对应代码
                            changeVar : "变量"
                            //如果额外参数不多，还可以写到 loader的路径末尾，如：./loaders/test-loader?参数名=参数值

                        }
                    }
                ]
            },
            {...}//更多规则对象
        ]
    }
}
```

**简化配置**

```
module.exports = {
    module: { //针对模块的配置，目前版本只有两个配置，rules、noParse
        rules: [ //模块匹配规则，可以存在多个规则
            { //每个规则是一个对象
                test: /\.js$/, //匹配的模块正则
                use: ["模块路径1", "模块路径2"]//loader模块的路径，该字符串会被放置到require中
            }
        ]
    }
}
```

**执行规则的顺序是从下往上执行的，也就是在数组最后面开始往前执行**

#### 作用

**所以loader实际作用就是可以改动源代码，变成我们想要的形式，而中间就是一些匹配规则，能够直到哪些需要使用规则**

**而至于开发时的代码，就可以随便写了**

## plugin

**loader的功能定位是转换代码**，而一些其他的操作难以使用loader完成，比如：

- 当webpack生成文件时，顺便多生成一个说明描述文件
- 当webpack编译启动时，控制台输出一句话表示webpack启动了
- 当xxxx时，xxxx

**这种类似的功能需要把功能嵌入到webpack的编译流程中，而这种事情的实现是依托于plugin的**

<img src="https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/1.%20webpack%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/1-14.%20plugin/assets/2020-01-15-12-45-16.png" alt="img" style="zoom:80%;" />

plugin的**本质**是一个**带有apply方法的对象**

```js
var plugin = {
    apply: function(compiler){
        //在这里注册监听事件
    }
}
```

通常，**习惯上**，我们会将该对象**写成构造函数**的模式

一般是在根目录创建一个plugin文件夹，里面放着一些js文件，文件内容就是**导出构造函数**

```js
module.exports = class MyPlugin{
    apply(compiler){
	//在这里注册监听事件
    }
}
```

然后就是在 webpack.config.js配置文件中进行配置

要将插件应用到webpack，**需要把插件对象配置到webpack的plugins数组中**，如下：

```js
module.exports = {
    plugins:[
        new MyPlugin()
    ]
}
```

apply函数会在初始化阶段，创建好Compiler对象后运行。

compiler对象是在初始化阶段构建的，整个webpack打包期间只有一个compiler对象，后续完成打包工作的是compiler对象内部创建的compilation

**apply方法**会在**创建好compiler对象后调用**，并向方法传入一个compiler对象
**根据下图也就是说：**compiler对象创建好之后，即使改变文件内容，**也只是重新走创建compilation过程**，而不会再次创建compiler对象，所以从初始化开始后，**apply方法其实只会运行一次**

<img src="https://raw.githubusercontent.com/DuYi-Edu/DuYi-Webpack/master/1.%20webpack%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD/1-14.%20plugin/assets/2020-01-15-12-49-26.png" alt="img" style="zoom:80%;" />

compiler对象提供了大量的钩子函数（hooks，可以理解为事件），plugin的开发者可以注册这些钩子函数，参与webpack编译和生成。

你可以**在apply方法中使用下面的代码注册钩子函数**:

```js
class MyPlugin{
    apply(compiler){
        compiler.hooks.事件名称.事件类型(name, function(compilation){
            //事件处理函数
        })
    }
}
```

**事件名称**

即要监听的事件名，即钩子名，所有的钩子：https://www.webpackjs.com/api/compiler-hooks

**事件类型**

这一部分使用的是 Tapable API，这个小型的库是一个专门用于钩子函数监听的库。

它提供了一些事件类型：

- tap：注册一个同步的钩子函数，函数运行完毕则表示事件处理结束
- tapAsync：注册一个基于回调的异步的钩子函数，函数通过调用一个回调表示事件处理结束
- tapPromise：注册一个基于Promise的异步的钩子函数，函数通过返回的Promise进入已决状态表示事件处理结束

**处理函数**

处理函数有一个事件参数`compilation`

## 区分环境

有些时候，我们**需要针对生产环境和开发环境分别书写webpack配置**

#### 硬核区分：

也可以**直接创建两个配置文件替代默认的webpack.config.js文件**，通过在package.js文件中，**配置脚本: webpack --config 配置文件名**  来使他执行对应的环境

**webpack.dev.js配置文件**:  使用开发环境

```js
module.exports = {
    mode: "development",
    devtool: "suorce-map" //生成源码地图文件
}
```

**webpack.pro.js配置文件**：使用生成环境

```js
module.exports = {
    mode: "production",
    devtool: "none"
}
```

**配置package.js文件的脚本命令**（为了方便）

```js
"scripts": {
        "dev": "webpack --config webpack.dev.js",
        "pro": "webpack --config webpack.pro.js"
    }
```

之后执行，只需要 **npm  run  对应命令**          即可



#### 使用同一个配置文件

为了更好的适应这种要求，**webpack允许配置不仅可以是一个对象**，还可以是一个**函数**

```js
module.exports = env => {
    return {
        //配置内容
    }
}
```

在开始构建时，**webpack如果发现配置是一个函数**，**会调用该函数**，**将函数返回的对象作为配置内容**，因此，开发者可以根据不同的环境返回不同的对象

**在调用webpack函数时**，webpack会向函数**传入一个参数env**，该**参数的值来自于webpack命令中给env指定的值**，例如

```js
npx webpack --env abc // env: "abc"    也就是传给函数的env的值  就是 "abc"

npx webpack --env.abc // env: {abc:true}
npx webpack --env.abc=1  // env： {abc:1}
npx webpack --env.abc=1 --env.bcd=2 // env: {abc:1, bcd:2}
```

这样一来，我们就可以在命令中指定环境，在代码中进行判断，根据环境返回不同的配置结果。

如：在webpack.config.js中

```js
module.exports = function(env){
	if(env === "dev"){
		return {
			mode : "development",
			devtool:"source-map"
		}
	}else if(env === "pro"){
		return {
			mode:"production",
			devtool:"none"
		}
	}
}
```

那么在package.js文件中就可以配置脚本命令

```js
"scripts":{
	"dev":"webpack --env dev",
	"pro":"webpack --env pro"
}
```

之后就可以根据想要的环境执行对应命令

如：**npm run dev**  或   **npm run pro**

## 其他细节配置 

**细枝末节的知识**

- 知道他能干什么（主要）
- 了解一下他怎么做的，如果忘了，那就忘了，问题不大

## context

```js
context: path.resolve(__dirname, "app")
```

**该配置会影响入口和loaders的解析**，**入口**和**loaders**的相对路径**会以context的配置作为基准路径**，这样，你的配置会独立于CWD（current working directory 当前执行路径）

如：**配置入口**

```js
const path = require("path");
module.exports = {
	mode:"development",
	devtool:"source-map",
	entry:{
		index:"./index.js", //通过 context配置后，就不需要写  "./src/index.js"了
		a:"./a.js"
	},
	context:path.resolve(__dirname,"src");//配置查找路径的相对地址  这里表示配置在：该文件当前目录的src文件夹
}
```



## output

#### library

```
library: "abc"
```

这样一来，打包后的结果中，会将自执行函数的执行结果暴露给abc
可以理解为：在打包后得到的最终文件中，**创建一个全局变量 abc** ，**并把入口文件导出的结果，赋值给abc**

其实jquery就是使用了这个配置，将 $ 暴露在全局中，供我们使用

```js
var abc = (function(...){...})(...)
```



#### libraryTarget

```
libraryTarget: "var"
```

该配置可以更加精细的**控制如何暴露入口包的导出结果**

其他可用的值有：

- **var**：默认值，暴露给一个普通变量
- **window**：暴露给window对象的一个属性
- **this**：暴露给this的一个属性
- **global**：暴露给global的一个属性
- **commonjs**：暴露给exports的一个属性
- 其他：https://www.webpackjs.com/configuration/output/#output-librarytarget

## target

```js
target:"web" //默认值  也即是 默认是在web环境运行
```

**设置打包结果最终要运行的环境**，常用值有

- **web**: 打包后的代码运行在web环境中
- **node**：打包后的代码运行在node环境中
- 其他：https://www.webpackjs.com/configuration/target/

## module.noParse

```js
noParse: /jquery/
```

**不解析正则表达式匹配的模块**，通常**用它来忽略那些大型的单模块库**（注意是单模块），以提高**构建性能**

如jquery，因为我们导入使用的jquery模块的main绑定的其实也是他目录下dist文件夹下的jquery.js，所以jquery也是打包生成的文件，而对于这种已经是打包生成的文件，就不需要再进行打包解析了，不然只会增长解析时间。

## resolve

resolve的相关配置主要用于控制模块解析过程

#### modules

```js
modules: ["node_modules"]  //默认值   表示模块(没有./开头的模块)的查找位置 ，如：require("jquery")，他就会去node_module找
//还可以放多个值，则表示先从第一个元素开始找，找不到再从第二个元素查找
//如
modules:["node_modules","abc"]
```

当解析模块时，如果遇到导入语句，`require("test")`，**webpack会从下面的位置寻找依赖的模块**

1. **当前**目录下的`node_modules`目录
2. **上级**目录下的`node_modules`目录
3. ...

#### extensions

```js
extensions: [".js", ".json"]  //默认值
```

当解析模块时，遇到无具体后缀的导入语句，例如`require("test")`，会依次测试它的后缀名

- **test.js**

- **test.json**

  **有可能会遇到这样的面试题：**

  ```
  场景：在使用webpack打包过程中
  require("./a");
  
  问题：为什么我没有书写后缀名，仍然可以找到a.js ？
  
  答1：因为会自动补全后缀名（正确，但是不会高分）
  答2：因为node会自动补全后缀名【错误！！！！！】（跟node没有任何关系，根本就没有执行文件）
  答3：因为webpack在解析过程中会自动补全后缀名（满分）
  ```

  

#### alias

```js
alias: {
  "@": path.resolve(__dirname, 'src'),
  "_": __dirname
}
```

**有了alias（别名）后**，导入语句中可以加入配置的键名，**例如`require("@/abc.js")`**，webpack会**将其看作是`require(src的绝对路径+"/abc.js")`**

在大型系统中，源码结构往往比较深和复杂，别名配置可以让我们更加方便的导入依赖

## externals

```js
externals: {
    jquery: "$", //把jquery的模块替换成  导出一个$   
    lodash: "_"  //把lodash的模块替换成  导出一个_
}
//因为我们在页面上已经使用了 jquery和lodash的CDN引入了，所以就会有全局变量 $ 和 _，这样也就能让代码正常使用jquery和lodash了
```

**从最终的bundle中排除掉配置的配置的源码**，例如，入口模块是

```js
//index.js    导入了两个模块，这样就会导致最终打包生成的文件中也会有jquery和lodash的源码
require("jquery")
require("lodash")
```

生成的bundle是：

```js
(function(){
    ...
})({
    "./src/index.js": function(module, exports, __webpack_require__){
        __webpack_require__("jquery")
        __webpack_require__("lodash")
    },
    "jquery": function(module, exports){
        //jquery的大量源码
    },
    "lodash": function(module, exports){
        //lodash的大量源码
    },
})
```

**但有了上面的配置后，则变成了**（减少了大量的代码）

```js
(function(){
    ...
})({
    "./src/index.js": function(module, exports, __webpack_require__){
        __webpack_require__("jquery")
        __webpack_require__("lodash")
    },
    "jquery": function(module, exports){
        module.exports = $;
    },
    "lodash": function(module, exports){
        module.exports = _;
    },
})
```

**使用场景：**

**这比较适用于一些第三方库来自于外部CDN的情况**，这样一来，**即可以在页面中使用CDN**，**又让bundle的体积变得更小**，还不影响源码的编写

## stats

stats控制的是构建过程中控制台的输出内容