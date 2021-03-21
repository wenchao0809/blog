本文主要`webpack`模块化的实现原理。关于`前端模块化前世今生`可以读下面这篇文章。

 [前端模块化的前世今生](https://mp.weixin.qq.com/s/88q3P-rRDxVSD7HJxYAigg)
 
在我们应用程序中，形如 index.html 文件、一些 bundle 和各种资源，都必须以某种方式加载和链接到应用程序，一旦被加载到浏览器中。在经过打包、压缩、为延迟加载而拆分为细小的 chunk 这些 webpack 优化 之后，你精心安排的 `/src` 目录的文件结构都已经不再存在。所以 webpack 如何管理所有所需模块之间的交互呢？

webpack有简单的解释用runtime 和 manifest，管理所有模块的交互，下面通过实例看一下`runtime`和`mainifest` 都包含内容。

	
实例包含以下模块

* a.js

~~~js
export default function a() {
    console.log('a module')
}
~~~

* b.js

~~~js
export default function b() {
    console.log('b module')
}
~~~

* index.js

~~~js
import a from './module/a'
import b from './module/b'

a()
b()
~~~

### webpack 配置文件

~~~js
module.exports = {
    mode: 'development',
    entry: path.resolve('./src/index.js'),
    output: {
		path: path.resolve('./dist'),
		filename: '[name].[hash].js',
		// important avoid request source 404
		publicPath: '/'
    },
    optimization: {
        runtimeChunk: 'single'
    },
    resolve: {
		extensions: ['.ts', '.tsx', '.vue', '.json', '.js' ]
    },
    plugins: [new HtmlWebpackPlugin()]
}
~~~

注意这里我们把 `mode`设为`development` 是为了使输出的文件内容更易读。如果设为`production`则webpack会压缩文件。

### package.json
~~~json
{
  "name": "webpackmodule",
  "version": "1.0.0",
  "description": "learn webpack module",
  "main": "index.js",
  "scripts": {
    "build": "webpack",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "dwc",
  "license": "ISC",
  "devDependencies": {
    "html-webpack-plugin": "^3.2.0",
    "webpack": "^4.41.2",
    "webpack-cli": "^3.3.10"
  }
}

~~~

执行`npm run build`会输出以下文件

* index.html

~~~js
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Webpack App</title>
  </head>
  <body>
  <script type="text/javascript" src="/runtime.5b73b2a8180b31600e0c.js"></script><script type="text/javascript" src="/main.5b73b2a8180b31600e0c.js"></script></body>
</html>
~~~ 
这是`HtmlWebpackPlugin`自动生成的，可以看到首先加载的是`runtime.js`其次是`main.js`

* runtime.js

~~~js
 // 定义了一个立即执行函数
 (function(modules) { // webpackBootstrap
 	// install a JSONP callback for chunk loading
 	function webpackJsonpCallback(data) {
 	// 接收一个数组，前三个元素分别是当
 	// data[0]当前加载的chunkId数组，
 	// data[1] 当前chunk所依赖的模块定义是个路径名和模块定义的对象
 	// data[2] 当前模块加载所依赖的前置模块类似[[0, 'runtime']]
 	// 结构是数组中包含数组， 其中第一个元素是模块id剩下元素是加载第一个模块所需的前置模块
 		var chunkIds = data[0];
 		var moreModules = data[1];
 		var executeModules = data[2];

 		// add "moreModules" to the modules object,
 		// then flag all "chunkIds" as loaded and fire callback
 		var moduleId, chunkId, i = 0, resolves = [];
 		for(;i < chunkIds.length; i++) {
 			chunkId = chunkIds[i];
 			if(Object.prototype.hasOwnProperty.call(installedChunks, chunkId) && installedChunks[chunkId]) {
				 // 检查异步加载的模块
 				resolves.push(installedChunks[chunkId][0]);
 			}
 			installedChunks[chunkId] = 0;
 		}
 		for(moduleId in moreModules) {
 			if(Object.prototype.hasOwnProperty.call(moreModules, moduleId)) {
 				modules[moduleId] = moreModules[moduleId];
 			}
 		}
 		if(parentJsonpFunction) parentJsonpFunction(data);

 		while(resolves.length) {
 			resolves.shift()();
 		}

 		// add entry modules from loaded chunk to deferred list
 		deferredModules.push.apply(deferredModules, executeModules || []);

 		// run deferred modules when all chunks ready
 		return checkDeferredModules();
 	};
 	function checkDeferredModules() {
 		var result;
 		for(var i = 0; i < deferredModules.length; i++) {
 			var deferredModule = deferredModules[i];
 			var fulfilled = true;
 			for(var j = 1; j < deferredModule.length; j++) {
 				var depId = deferredModule[j];
 				if(installedChunks[depId] !== 0) fulfilled = false;
 			}
 			if(fulfilled) {
 				deferredModules.splice(i--, 1);
 				result = __webpack_require__(__webpack_require__.s = deferredModule[0]);
 			}
 		}

 		return result;
 	}

 	// The module cache
 	var installedModules = {};

    // 保存chunk的状态
    // 0已加载 undefined 未加载 null 预加载
 	var installedChunks = {
 		"runtime": 0
 	};

 	var deferredModules = [];

 	// The require function
 	function __webpack_require__(moduleId) {

 		// Check if module is in cache
 		if(installedModules[moduleId]) {
 			return installedModules[moduleId].exports;
 		}
 		// Create a new module (and put it into the cache)
 		var module = installedModules[moduleId] = {
 			i: moduleId,
 			l: false,
 			exports: {}
 		};

 		// Execute the module function
 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);

 		// Flag the module as loaded
 		module.l = true;

 		// Return the exports of the module
 		return module.exports;
 	}


 	// 暴露modules
 	__webpack_require__.m = modules;

 	// 暴露模块缓存
 	__webpack_require__.c = installedModules;

 	// 在exports上定义getter函数
 	__webpack_require__.d = function(exports, name, getter) {
 		if(!__webpack_require__.o(exports, name)) {
 			Object.defineProperty(exports, name, { enumerable: true, get: getter });
 		}
 	};

 	// 在 exports定义__esModule属性
 	__webpack_require__.r = function(exports) {
 		if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
 			Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
 		}
 		Object.defineProperty(exports, '__esModule', { value: true });
 	};

 	// create a fake namespace object
 	// mode & 1: value is a module id, require it
 	// mode & 2: merge all properties of value into the ns
 	// mode & 4: return value when already ns object
 	// mode & 8|1: behave like require
 	__webpack_require__.t = function(value, mode) {
 		if(mode & 1) value = __webpack_require__(value);
 		if(mode & 8) return value;
 		if((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
 		var ns = Object.create(null);
 		__webpack_require__.r(ns);
 		Object.defineProperty(ns, 'default', { enumerable: true, value: value });
 		if(mode & 2 && typeof value != 'string') for(var key in value) __webpack_require__.d(ns, key, function(key) { return value[key]; }.bind(null, key));
 		return ns;
 	};

 	// getDefaultExport function for compatibility with non-harmony modules
 	__webpack_require__.n = function(module) {
 		var getter = module && module.__esModule ?
 			function getDefault() { return module['default']; } :
 			function getModuleExports() { return module; };
 		__webpack_require__.d(getter, 'a', getter);
 		return getter;
 	};

 	// Object.prototype.hasOwnProperty.call
 	__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };

 	// __webpack_public_path__
 	__webpack_require__.p = "/";
     // window["webpackJsonp"]初始为数组
 	var jsonpArray = window["webpackJsonp"] = window["webpackJsonp"] || [];
     var oldJsonpFunction = jsonpArray.push.bind(jsonpArray);
     // 这里重写push方法为webpackJsonpCallBack
 	jsonpArray.push = webpackJsonpCallback;
 	jsonpArray = jsonpArray.slice();
 	// 加载下之前加载过的模块
 	for(var i = 0; i < jsonpArray.length; i++) webpackJsonpCallback(jsonpArray[i]);
 	var parentJsonpFunction = oldJsonpFunction;

 	// run deferred modules from other chunks
 	checkDeferredModules();
 })
/
 ([]);
~~~
可以看到 `runtime.js`定义了一些全局变量

* `window["webpackJsonp"]  = []`
* `webpack["webpackJsonp"].push = webpackJsonpCallback`
* `__webpack_require__`等一系列方法。

**`runtime.js`是`webpack`的引导模块**

`runtime.js`加载完成后会加载`main.js`

* main.js


~~~js
(window["webpackJsonp"] = window["webpackJsonp"] || []).push([["main"],{

/***/ "./src/index.js":
/*!**********************!*\
  !*** ./src/index.js ***!
  \**********************/
/*! no exports provided */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony import */ var _module_a__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./module/a */ \"./src/module/a.js\");\n/* harmony import */ var _module_b__WEBPACK_IMPORTED_MODULE_1__ = __webpack_require__(/*! ./module/b */ \"./src/module/b.js\");\n\n\n\nObject(_module_a__WEBPACK_IMPORTED_MODULE_0__[\"default\"])()\nObject(_module_b__WEBPACK_IMPORTED_MODULE_1__[\"default\"])()\n\n//# sourceURL=webpack:///./src/index.js?");

/***/ }),

/***/ "./src/module/a.js":
/*!*************************!*\
  !*** ./src/module/a.js ***!
  \*************************/
/*! exports provided: default */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, \"default\", function() { return a; });\nfunction a() {\n    console.log('a module')\n}\n\n//# sourceURL=webpack:///./src/module/a.js?");

/***/ }),

/***/ "./src/module/b.js":
/*!*************************!*\
  !*** ./src/module/b.js ***!
  \*************************/
/*! exports provided: default, test1 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, \"default\", function() { return b; });\n/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, \"test1\", function() { return test1; });\nfunction b() {\n    console.log('b module')\n}\n\nconst test1  = 10\n\n//# sourceURL=webpack:///./src/module/b.js?");

/***/ })

},[["./src/index.js","runtime"]]]);
~~~

在`main.js`中会调用`window["webpackJsonp"]`的`push`方法， 上面有说道`runtime.js`内有重写`push`方法为`webpackJsonpCallback `

#### `webpackJsonpCallback`方法

~~~js
function webpackJsonpCallback(data) {
        // 加载main.js时为[main]
         var chunkIds = data[0];
         // 所依赖的模块
         var moreModules = data[1];
         // 加载main.js为 [["./src/index.js","runtime"]]
 		var executeModules = data[2];

 		// add "moreModules" to the modules object,
 		// then flag all "chunkIds" as loaded and fire callback
 		var moduleId, chunkId, i = 0, resolves = [];
 		for(;i < chunkIds.length; i++) {
 			chunkId = chunkIds[i];
 			if(Object.prototype.hasOwnProperty.call(installedChunks, chunkId) && installedChunks[chunkId]) {
 				resolves.push(installedChunks[chunkId][0]);
 			}
 			installedChunks[chunkId] = 0;
 		}
 		for(moduleId in moreModules) {
             // 遍历保存将模块保存在modules
 			if(Object.prototype.hasOwnProperty.call(moreModules, moduleId)) {
 				modules[moduleId] = moreModules[moduleId];
 			}
 		}
 		if(parentJsonpFunction) parentJsonpFunction(data);

 		while(resolves.length) {
 			resolves.shift()();
 		}

 		// add entry modules from loaded chunk to deferred list
 		deferredModules.push.apply(deferredModules, executeModules || []);

 		// run deferred modules when all chunks ready
 		return checkDeferredModules();
 	};
 	
function checkDeferredModules() {
 		var result;
 		for(var i = 0; i < deferredModules.length; i++) {
 			var deferredModule = deferredModules[i];
             var fulfilled = true;
             // 注意这里j从开始
 			for(var j = 1; j < deferredModule.length; j++) {
                // depId为'runtime'
                // 为什么从1开始， 我认为第一个元素为当前所需加载模块， 之后的元素都是其所依赖的模块
                // 这段代码就是在检查所依赖的模块是否加载完毕
 				var depId = deferredModule[j];
 				if(installedChunks[depId] !== 0) fulfilled = false;
 			}
 			if(fulfilled) {
                 // 初次加载index.js会走到这里，因为其所依赖的runtime已加载完毕
                 // 删除数组第一个元素
                 deferredModules.splice(i--, 1);
                 // 加载index.js
 				result = __webpack_require__(__webpack_require__.s = deferredModule[0]);
 			}
 		}

 		return result;
 	}
~~~

在`webpackJsonpCallback `中最终会执行`checkDeferredModules`, 后者会检查当前其所依赖模块是否加载完毕， 如果已经加载完毕则加载当前模块。这里依然是调用`__webpack_require__`加载 `index.js`， 最终会执行在`main.js`传入的包装函数。

对于每个独立的`chunk`在加载完毕后都会调`webpackJsonpCallback`, 这个函数接收一个数组包含三个元素，分别是

* `chunkids` 当前加载的模块
* `modules` 模块定义，包括当前模块和当前模块所引入的模块。
* `executeModules` 数组中包含数组其中被包含数组的第一个元素是等待加载的模块 之后的元素是第一个元素加载所依赖的模块，就是说执加载第一个模块之前要把所有依赖模块都加载完毕`checkDeferredModules` 会在每次调用`webpackJsonpCallback `之后检查依赖模块是否加载完毕。

# module index.js

~~~js
 (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony import */ var _module_a__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./module/a */ \"./src/module/a.js\");\n/* harmony import */ var _module_b__WEBPACK_IMPORTED_MODULE_1__ = __webpack_require__(/*! ./module/b */ \"./src/module/b.js\");\n\n\n\nObject(_module_a__WEBPACK_IMPORTED_MODULE_0__[\"default\"])()\nObject(_module_b__WEBPACK_IMPORTED_MODULE_1__[\"default\"])()\n\n//# sourceURL=webpack:///./src/index.js?");

/***/ })
~~~

仔细看`eval`内的代码写成普通函数如下

~~~js
function loadindex(module, exports, __webpack_exports__, __webpack_require__){
	__webpack_require__.r(__webpack_exports__);
	var _module_a__WEBPACK_IMPORTED_MODULE_0__ = 	__webpack_require__(/*! ./module/a */ \"./src/module/a.js\");
	var _module_b__WEBPACK_IMPORTED_MODULE_1__ = 	__webpack_require__(/*! ./module/b */ \"./src/module/b.js\");
	Object(_module_a__WEBPACK_IMPORTED_MODULE_0__[\"default\"])()
	Object(_module_b__WEBPACK_IMPORTED_MODULE_1__[\"default\"])()
}
~~~
首先调用`__webpack_require__.r`定义`__esModule`属性，然后调用`__webpack_require__`引入`a`和`b`这和我们的代码一致。

`a.js`和`b.js`我们看其中一个就行

~~~js
 (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, \"default\", function() { return a; });\nfunction a() {\n    console.log('a module')\n}\n\n//# sourceURL=webpack:///./src/module/a.js?");

/***/ }),

 	// 定义模块属性
 	__webpack_require__.d = function(exports, name, getter) {
 		if(!__webpack_require__.o(exports, name)) {
 			Object.defineProperty(exports, name, { enumerable: true, get: getter });
 		}
 	};
~~~

同上先定义`__esModule`属性，然后调用`__webpack_require__.d`在`exports`上定义了一个`default`的`getter`属性用于返回a函数。

~~~js
__webpack_require__.d(__webpack_exports__, \"default\", function() { return a; });
function a() {  console.log('a module')
~~~

经过上面的分析，我们可以看到`webpack`是如何实现`es6`module的加载。

# 代码拆分

上面的例子非常简单，两个模块在入口都会被加载， 利用`webpack`更常见的场景是`代码拆分`，看一下`webpack`是如何实现代码拆分的。

修改`index.js`的代码如下

~~~js
import a from './module/a'
import('./module/b')
    .then(b => b.default())
    
a()
~~~

重新执行`npm run build`,此时相比之前会多输出一个`chunk`对应`b.js`.

~~~text
Built at: 2019-12-16 5:02:41 PM
                          Asset       Size   Chunks                         Chunk Names
      0.462f7d03c3544a394c4a.js  727 bytes        0  [emitted] [immutable]  
                     index.html  281 bytes           [emitted]              
   main.462f7d03c3544a394c4a.js   1.26 KiB     main  [emitted] [immutable]  main
runtime.462f7d03c3544a394c4a.js   8.96 KiB  runtime  [emitted] [immutable]  runtime
~~~

`index.html`的代码是不变的，变得只有`index.js`

~~~js
(window["webpackJsonp"] = window["webpackJsonp"] || []).push([["main"],{

/***/ "./src/index.js":
/*!**********************!*\
  !*** ./src/index.js ***!
  \**********************/
/*! no exports provided */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony import */ var _module_a__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./module/a */ \"./src/module/a.js\");\n\n__webpack_require__.e(/*! import() */ 0).then(__webpack_require__.bind(null, /*! ./module/b */ \"./src/module/b.js\"))\n    .then(b => b.default())\n    \nObject(_module_a__WEBPACK_IMPORTED_MODULE_0__[\"default\"])()\n\n//# sourceURL=webpack:///./src/index.js?");

/***/ }),

/***/ "./src/module/a.js":
/*!*************************!*\
  !*** ./src/module/a.js ***!
  \*************************/
/*! exports provided: default */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, \"default\", function() { return a; });\nfunction a() {\n    console.log('a module')\n}\n\n//# sourceURL=webpack:///./src/module/a.js?");

/***/ })

},[["./src/index.js","runtime"]]]);
~~~

我们再次写成普通的函数

~~~js
function loadIndex() {
	__webpack_require__.r(__webpack_exports__);
	var _module_a__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./module/a */ \"./src/module/a.js\");
	// 这句代码在快加载完毕后传入then的函数是， _webpack_require__.bind(null, /*! ./module/b */ \"./src/module/b.js\"， 引入模块
	// 之后的代码才开始访问模块内容
	_webpack_require__.e(/*! import() */ 0).then(__webpack_require__.bind(null, /*! ./module/b */ \"./src/module/b.js\"))\n    .then(b => b.default())
	Object(_module_a__WEBPACK_IMPORTED_MODULE_0__[\"default\"])()
}
__webpack_require__.e = function requireEnsure(chunkId) {
	var promises = [];


	// JSONP chunk loading for javascript

	var installedChunkData = installedChunks[chunkId];
     if(installedChunkData !== 0) { // 0 means "already installed".
        // 正在加载
		if(installedChunkData) {
			promises.push(installedChunkData[2]);
		} else {
			// setup Promise in chunk cache
			var promise = new Promise(function(resolve, reject) {
				installedChunkData = installedChunks[chunkId] = [resolve, reject];
			});
			promises.push(installedChunkData[2] = promise);
			// 此时installedChunks[chunkId]; = installedChunkData = [resolve, reject, promise]

             // start chunk loading
             // 创建一个script标签
			var script = document.createElement('script');
			var onScriptComplete;

			script.charset = 'utf-8';
			script.timeout = 120;
			if (__webpack_require__.nc) {
				script.setAttribute("nonce", __webpack_require__.nc);
             }
             // 拼接文件名
			script.src = jsonpScriptSrc(chunkId);

			// create error before stack unwound to get useful stacktrace later
			var error = new Error();
			onScriptComplete = function (event) {
				// avoid mem leaks in IE.
				script.onerror = script.onload = null;
				clearTimeout(timeout);
				var chunk = installedChunks[chunkId];
				if(chunk !== 0) {
					if(chunk) {
						var errorType = event && (event.type === 'load' ? 'missing' : event.type);
						var realSrc = event && event.target && event.target.src;
						error.message = 'Loading chunk ' + chunkId + ' failed.\n(' + errorType + ': ' + realSrc + ')';
						error.name = 'ChunkLoadError';
						error.type = errorType;
						error.request = realSrc;
						chunk[1](error);
					}
					installedChunks[chunkId] = undefined;
				}
			};
			var timeout = setTimeout(function(){
                 // 加载超时 120s
				onScriptComplete({ type: 'timeout', target: script });
			}, 120000);
             script.onerror = script.onload = onScriptComplete;
             // 插入标签
			document.head.appendChild(script);
		}
	}
	return Promise.all(promises);
};
~~~

# 这个是拆分快， 也就是b.js打包出来的代码
~~~js
(window["webpackJsonp"] = window["webpackJsonp"] || []).push([[0],{

/***/ "./src/module/b.js":
/*!*************************!*\
  !*** ./src/module/b.js ***!
  \*************************/
/*! exports provided: default, test1 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, \"default\", function() { return b; });\n/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, \"test1\", function() { return test1; });\nfunction b() {\n    console.log('b module')\n}\n\nconst test1  = 10\n\n//# sourceURL=webpack:///./src/module/b.js?");

/***/ })

}]);
~~~

在`webpackJsonpCallback`有一些代码我们上面没有说,在`script`标签加载完成拆分的代码后会执行这个回调， 这里传入的`chunkId`是`0`

~~~js
function webpackJsonpCallback(data) {
 		var chunkIds = data[0];
 		var moreModules = data[1];
 		var executeModules = data[2];

 		// add "moreModules" to the modules object,
 		// then flag all "chunkIds" as loaded and fire callback
 		var moduleId, chunkId, i = 0, resolves = [];
 		for(;i < chunkIds.length; i++) {
 			chunkId = chunkIds[i];
 			if(Object.prototype.hasOwnProperty.call(installedChunks, chunkId) && installedChunks[chunkId]) {
// installedChunks[chunkId][0]就是异步加载的resolve函数
// 结合	__webpack_require__.e 看主要场景 代码拆分
 				resolves.push(installedChunks[chunkId][0]);
 			}
 			installedChunks[chunkId] = 0;
 		}
 		for(moduleId in moreModules) {
 			if(Object.prototype.hasOwnProperty.call(moreModules, moduleId)) {
 				modules[moduleId] = moreModules[moduleId];
 			}
 		}
 		if(parentJsonpFunction) parentJsonpFunction(data);

 		while(resolves.length) {
             // resolve已经加载完成的异步模块
            	  // 这里会resolve之前加载js的promise
 			resolves.shift()();
 		}

 		// add entry modules from loaded chunk to deferred list
 		deferredModules.push.apply(deferredModules, executeModules || []);

 		// run deferred modules when all chunks ready
 		return checkDeferredModules();
 	};
~~~

通过上面的分析我们可以得出`webpack`拆分代码的加载流程

调用**_webpack_require__.e** > 创建**script标签加载** > 执行**webpackJsonpCallback() resolve Promise** > **执行Promise回调调用_webpack_require引入模块** > 执行回调调用引入代码

### 总结

上面只是`webpack`对`ES Module`的实现，对于其他模块加载的实现，大致是一致的。






