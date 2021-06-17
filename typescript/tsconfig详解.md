## compilerOptions 编译选项

### target 编译目标

设置编译后代码运行的目标环境， 默认`ES3`

* `ES3 (default)`
* `ES5`
* `ES6/ES2015 (synonymous)`
* `ES7/ES2016`
* `ES2017`
* `ES2018`
* `ES2019`
* `ES2020`
* `ESNext`


`target` 的配置将会改变哪些 `JS` 特性会被降级，而哪些会被完整保留 例如，如果 `target` 是 `ES5` 或更低版本，箭头函数 `() => this` 会被转换为等价的 函数 表达式。

改变 `target` 也会改变 `lib` 选项的默认值。 你可以根据需要混搭 `target` 和 `lib` 的配置，你也可以为了方便只设置 `target`。

特殊的 `ESNext` 值代表你的 `TypeScript` 所支持的最高版本。这个配置应当被谨慎使用，因为它在不同的 `TypeScript` 版本之间的含义不同，并且会导致升级更难预测

### module 

编译目标代码类型有下列值

* `CommonJS`
* `UMD`
* `AMD`
* `System`
* `ESNext`
* `None`

### outFile

如果有指定，打包后的文件将被合并到指定的文件中。 只能用于 `module` 为`System`和`AMD`中

### outDir

如果被指定，`.js` （以及 `.d.ts, .js.map` 等）将会被生成到这个目录下。 原始源文件的目录将会被保留， 如果没有指定，`.js` 将被生成至于生成它们的 `.ts` 文件相同的目录中。

~~~js
example
├── index.js
└── index.ts
~~~

以下配置构建会生成一下文件

~~~json
{
  "compilerOptions": {
    "outDir": "dist"
  }
}
~~~

tsc

~~~js
example
├── dist
│   └── index.js
├── index.ts
└── tsconfig.json
~~~

`outDir`是根据`tsconfig.json`的目录计算的， `rootDir`可以配置生成的目录结构
### rootDir

默认: 所有输入的非声明文件中的最长公共路径。若 `composite` 被指定，则是包含 `tsconfig.json` 文件的目录

如下目录结构

~~~js
MyProj
├── tsconfig.json
├── core
│   ├── a.ts
│   ├── b.ts
│   ├── sub
│   │   ├── c.ts
├── types.d.ts
~~~

默认情况下则是`/MyProj/core`, 因为非声明文件的最长公共路径是`/Myproj/core`, 如果当前`outDir`配置是`dist`则会生成下面目录

~~~
MyProj
├── dist
│   ├── a.ts
│   ├── b.ts
│   ├── sub
│   │   ├── c
~~~

另外 `rootDir`必须包含`include`的所有文件否则会报错




## include

指定编译包含的路径模式基于`tsconfig.json`文件位置， 做相对路径计算，支持`glob`模式, 默认值如果指定了`files`为`[]`否则为`**/*`

* `*`匹配0个活n个字符
* `?`匹配任意一个字符
* `**/`匹配任意深度的嵌套目录

如果模式未指定扩展文件名， 则默认支持一下扩展名`.ts`、`.tsx`、`.d.ts`如果`allowJs`设为`true`则还支持`.js`、`.jsx`

如下示例 则会匹配以下文件

~~~js
{
  "include": ["src/**/*", "tests/**/*"]
}
~~~

~~~js
.
├── scripts                ⨯
│   ├── lint.ts            ⨯
│   ├── update_deps.ts     ⨯
│   └── utils.ts           ⨯
├── src                    ✓
│   ├── client             ✓
│   │    ├── index.ts      ✓
│   │    └── utils.ts      ✓
│   ├── server             ✓
│   │    └── index.ts      ✓
├── tests                  ✓
│   ├── app.test.ts        ✓
│   ├── utils.ts           ✓
│   └── tests.d.ts         ✓
├── package.json
├── tsconfig.json
└── yarn.lock
~~~

## Reference

这个顶级配置选项允许将的大的项目拆分成小而简单的项目， 配置如下

~~~js
{
  "references": [
    { "path": "../core" }
  ]
}
~~~

如果指定`references`会发生一下默认操作

* 从`reference`引入模块将被替换加载其输出声明文件
* 如果`reference`指定生成了`outFile`则其输出的声明文件将对当前项目可见。
* `Build mode`将自动构建`referenced`项目。

### composite


