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

### moduel

编译目标代码类型有下列值

* `CommonJS`
* `UMD`
* `AMD`
* `System`
* `ESNext`
* `None`