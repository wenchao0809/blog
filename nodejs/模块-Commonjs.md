### 模块(Commonjs modules)

`Node.js` `CommonJS`每个文件都被当做一个单独的模块 比如`foo.js`

~~~js
const circle = require('./circle.js');
console.log(`The area of a circle of radius 4 is ${circle.area(4)}`);
~~~

`circle.js`

~~~js
const { PI } = Math;

exports.area = (r) => PI * r ** 2;

exports.circumference = (r) => 2 * PI * r;
~~~

`circle.js`导出了`area()` 和`circumference()` 两个方法 通过`exports`

`PI`变量是私有的外界是无法访问的， 这里不像浏览器如果我们在全局声明了一个变量则所有的脚本都可以访问到。
因为`Node.js`会把`module`用函数包装一层，这里可以类比一下浏览器内实现私有变量也是包装成一个立即执行函数, 真实代码如下

~~~js
(function(exports, require, module, __filename, __dirname) {
// Module code actually lives in here
// 这里是Module的内容
});
~~~

`module.exports`可以被直接修改为一个新的值如下`Square.js`直接导出了一个`class`

`Square.js`

~~~js
module.exports = class Square {
  constructor(width) {
    this.width = width;
  }

  area() {
    return this.width ** 2;
  }
};
~~~

可以直接这样引入

~~~js
const Square = require('./square.js');
const mySquare = new Square(2);
console.log(`The area of mySquare is ${mySquare.area()}`);
~~~

### 访问主模块(main module)

当一个文件被`Node.js`直接运行(`node foo.js`) `require.mian`属性被设置为`module`所以我们可以通过检测`require.main === module`来判断当前文件是直接运行还是被`require`。

### 附录：包管理技巧

`Node.js require()`函数的语义被设计得足够通用，以支持合理的目录结构。包管理器程序，如`dpkg`、`rpm`和`npm`，将有望从`Node.js`模块构建原生包而无需修改。

### require .mjs

`.mjs`是`ECMAScript 模块`如果直接`require`是会报错的

### `require`解析路径的高级算法

~~~js
require(X) from module at path Y
1. If X is a core module, // X是核心模块直接返回核心模块
   a. return the core module 
   b. STOP
2. If X begins with '/'
   a. set Y to be the filesystem root // X以/开头将Y设置为系统根路径
3. If X begins with './' or '/' or '../'
   a. LOAD_AS_FILE(Y + X) // X以相路径开头加载Y+X的文件
   b. LOAD_AS_DIRECTORY(Y + X) // 查找不到 加载Y+X路径
   c. THROW "not found" // 未找到
4. If X begins with '#' // 以#开头
   a. LOAD_PACKAGE_IMPORTS(X, dirname(Y))
5. LOAD_PACKAGE_SELF(X, dirname(Y)) // 
6. LOAD_NODE_MODULES(X, dirname(Y)) // 从node_modules加载
7. THROW "not found"

LOAD_AS_FILE(X)
1. If X is a file, load X as its file extension format. STOP // 如果X是文件直接加载
2. If X.js is a file, load X.js as JavaScript text. STOP // 尝试X.js
3. If X.json is a file, parse X.json to a JavaScript Object. STOP // 尝试 X.json
4. If X.node is a file, load X.node as binary addon. STOP // 尝试X.node, 二进制扩展

LOAD_INDEX(X) // 会在LOAD_AS_DIRECTORY内饰使用
1. If X/index.js is a file, load X/index.js as JavaScript text. STOP // 尝试X/index.js
2. If X/index.json is a file, parse X/index.json to a JavaScript object. STOP // 尝试 X/index.json
3. If X/index.node is a file, load X/index.node as binary addon. STOP // 尝试X/index.node

LOAD_AS_DIRECTORY(X)
1. If X/package.json is a file, // 当前目录存在package.json
   a. Parse X/package.json, and look for "main" field. // 解析package.json main字段
   b. If "main" is a falsy value, GOTO 2. // 如果main是假值 这里是不存在吧跳转到2
   c. let M = X + (json main field) // 设置M 为 X + main
   d. LOAD_AS_FILE(M) // 把M当做File加载 即 X+main
   e. LOAD_INDEX(M) // 把 M当做index加载
   f. LOAD_INDEX(X) DEPRECATED // 废弃了
   g. THROW "not found" // 未找到
2. LOAD_INDEX(X) // 加载index

LOAD_NODE_MODULES(X, START)
1. let DIRS = NODE_MODULES_PATHS(START) // 获取当前目录所有的node_modules 路径， 这个路径是一个数组可以Repl中输入module来查看格式 大概想下面这样可以看到大概是一层一层往上查找 node_modules目录
 /* paths: [
    '/Users/EstDing/develop/blog/repl/node_modules',
    '/Users/EstDing/develop/blog/node_modules',
    '/Users/EstDing/develop/node_modules',
    '/Users/EstDing/node_modules',
    '/Users/node_modules',
    '/node_modules',
    '/Users/EstDing/.node_modules',
    '/Users/EstDing/.node_libraries',
    '/usr/local/lib/node'
  ]*/
2. for each DIR in DIRS:
   a. LOAD_PACKAGE_EXPORTS(X, DIR)
   b. LOAD_AS_FILE(DIR/X)
   c. LOAD_AS_DIRECTORY(DIR/X)

NODE_MODULES_PATHS(START)
1. let PARTS = path split(START)
2. let I = count of PARTS - 1
3. let DIRS = [GLOBAL_FOLDERS] // 全局的路径
4. while I >= 0, // 就是一次在每一级目录上拼接node_modules, 如果已经是node_modules结尾就不会再次拼接
   a. if PARTS[I] = "node_modules" CONTINUE
   b. DIR = path join(PARTS[0 .. I] + "node_modules")
   c. DIRS = DIRS + DIR
   d. let I = I - 1
5. return DIRS

LOAD_PACKAGE_IMPORTS(X, DIR) // 以#开头执行
1. Find the closest package scope SCOPE to DIR. // 找到当前目录存在package.json最近的目录
2. If no scope was found, return. // 未找到直接return
3. If the SCOPE/package.json "imports" is null or undefined, return. // package.json的 imports字段为null或者undeined 直接返回
4. let MATCH = PACKAGE_IMPORTS_RESOLVE(X, pathToFileURL(SCOPE), // 这个函数没有具体解释
  ["node", "require"]) defined in the ESM resolver.
5. RESOLVE_ESM_MATCH(MATCH).

LOAD_PACKAGE_EXPORTS(X, DIR)
1. Try to interpret X as a combination of NAME and SUBPATH where the name
   may have a @scope/ prefix and the subpath begins with a slash (`/`).
2. If X does not match this pattern or DIR/NAME/package.json is not a file,
   return.
3. Parse DIR/NAME/package.json, and look for "exports" field.
4. If "exports" is null or undefined, return.
5. let MATCH = PACKAGE_EXPORTS_RESOLVE(pathToFileURL(DIR/NAME), "." + SUBPATH,
   `package.json` "exports", ["node", "require"]) defined in the ESM resolver.
6. RESOLVE_ESM_MATCH(MATCH)

LOAD_PACKAGE_SELF(X, DIR)
1. Find the closest package scope SCOPE to DIR. // 查找当前目录(从下往上)最近的存在package.json的目录
2. If no scope was found, return. // 未找到直接return
3. If the SCOPE/package.json "exports" is null or undefined, return.
4. If the SCOPE/package.json "name" is not the first segment of X, return.
5. let MATCH = PACKAGE_EXPORTS_RESOLVE(pathToFileURL(SCOPE),
   "." + X.slice("name".length), `package.json` "exports", ["node", "require"])
   defined in the ESM resolver.
6. RESOLVE_ESM_MATCH(MATCH)

RESOLVE_ESM_MATCH(MATCH)
1. let { RESOLVED, EXACT } = MATCH
2. let RESOLVED_PATH = fileURLToPath(RESOLVED)
3. If EXACT is true,
   a. If the file at RESOLVED_PATH exists, load RESOLVED_PATH as its extension
      format. STOP
4. Otherwise, if EXACT is false,
   a. LOAD_AS_FILE(RESOLVED_PATH)
   b. LOAD_AS_DIRECTORY(RESOLVED_PATH)
5. THROW "not found"
~~~

### 缓存

