[package.json官方文档](https://docs.npmjs.com/files/package.json.html#dependencies)

# dependencies(依赖关系)

`依赖关系`是一个包含`包名`和`版本范围`的映射简单对象,版本是一个字符串由一个或多个描述符组成，`依赖关系`也可以用`压缩包`和`git URL`指定。

请不要再`依赖关系`中放`测试工具`或者`编译器`，这些更应该放入下文的`devDependencies`。

`npm`使用的是语义化版本管理 

[语义化版本管理](https://semver.org/lang/zh-CN/)

**版本描述可以有以下形式**

`主要版本号.次要版本号.修订斑斑好-预发布版本号`类似下面这个例子
`1.2.4-beta.0`

* `Version` 必须完全匹配指定的版本
*  `>Version` 必须大于至此那个的版本
*  `>=Version` 大于等于指定版本
*  `<Version` 小于指定版本
*  `<=Version` 小于等于当前版本
*  `~version` 如果指定了次要版本则允许修订版本变动， 否则允许次要版本变动，看下下面的实例。

~~~js
~1.2 := >=1.2.0 <1.(2+1).0 := >=1.2.0 <1.3.0 (Same as 1.2.x)
~1 := >=1.0.0 <(1+1).0.0 := >=1.0.0 <2.0.0 (Same as 1.x)
~~~

*  `^Version` 允许修改从左到右的非0数字

~~~js
^1.2.3 := >=1.2.3 <2.0.0
^0.2.3 := >=0.2.3 <0.3.0
^0.0.3 := >=0.0.3 <0.0.4
~~~

*  `1.2.x ` 1.2.1 1.23 not 1.3.0
*  `version1 - version2` 相当于 >=version1 <=version2
*  range1 || range2 满足范围1和范围2

# devDependencies

开发环境依赖这也是我们最常用的，我们通常会把项目本地运行或构建的依赖库放入这个对象，但是思考下下面两个问题

1. 如果把依赖都放入`devDependencies`对项目有什么影响？
2. 如果把依赖都放入`dependencies ` 对项目有什么影响？

上面两个问题的答案都是没什么影响

对于项目本身而言， 无论是将依赖放入`dependencies`还是`devDependencies `依赖包都会被正确的下载进`node_modules`中，所以对项目的本地运行以及构建没什么影响。

但是如果你要将你的项目发布到`npm`给开发者使用那就要区分二者了， 放在`dependencies`中的依赖会在开发者安装包时同时安装， 而放入`devDependencies`中的依赖则不会安装到开发者电脑上，所以如果将所有依赖都放入`dependencies `就会导致开发者安装不必要的包。

所以`dependencies `和`devDependencies真正的区别在于这里。

但是在项目中还是建议大家区分下依赖, 毕竟将所有依赖放入到一个地方可读性会变差。

# peerDependencies

这个通常用于插件编写，如果你的插件依赖于特定版本的某个特定的`api`就可以用`peerDependencies`指定，如果当前版本和指定版本不符就会发出警告。

> 注意，npm 1 与 npm 2 会自动安装同等依赖，npm 3 不再自动安装，会产生警告！手动在package.json文件中添加依赖项可以解决。

# bundledDependencies / bundleDependencies

打包依赖，bundledDependencies是一个包含依赖包名的数组对象，在发布时会将这个对象中的包打包到最终的发布包里。


执行打包命令npm 将包含bundledDependencies。 但是值得注意的是，这两个包必须先在devDependencies或dependencies声明过，否则打包会报错。

# optionalDependencies

可选依赖

如果有一些依赖包即使安装失败，项目仍然能够运行或者希望npm继续运行，就可以使用optionalDependencies。另外optionalDependencies会覆盖dependencies中的同名依赖包，所以不要在两个地方都写。

# 只安装dependencies

在生产环境可以只安装`dependencies`

~~~sh
npm i --production
~~~
