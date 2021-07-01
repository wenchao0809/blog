通常我们在前端部署是如果不是部署在`root`根据路径比如加了个前缀`/mdt/`,这样我们需要做三方面修改

以`Vue`项目为例

1. `vue-router` 需要设置`base`为`/mdt/`。
2. 请求的`api`接口地址需要添加`/mdt/` 具体看`nginx`如何配置的。
3. 所有的静态资源路径都需要加上`/mdt/`。

对于第三条我们可以配置`webpack`的`publicPath`, 由于我们现在的项目是用`vue-cli`生成的所以 需要修改`vue.config.js`中`publicPath`,修改后执行构建发现`js`的资源是正常的但是提取`css`中的
对图片资源的的引用路径居然是相对路径。

~~~css
background-image: url('../image.png')
~~~

直接看`@vue/cli-service`的源码发现在`/config/css.js`中有如下提取`css`的配置

~~~js
  // use relative publicPath in extracted CSS based on extract location
    const cssPublicPath = process.env.VUE_CLI_BUILD_TARGET === 'lib'
      // in lib mode, CSS is extracted to dist root.
      ? './'
      : '../'.repeat(
        extractOptions.filename
            .replace(/^\.[\/\\]/, '')
            .split(/[\/\\]/g)
            .length - 1
      )

        function applyLoaders (rule, isCssModule) {
          if (shouldExtract) {
            rule
              .use('extract-css-loader')
              .loader(require('mini-css-extract-plugin').loader)
              .options({
                hmr: !isProd,
                publicPath: cssPublicPath // 直接粗暴指定了提取css的publicPatch并且没有提供任务配置修改
              })
          }
        }
~~~

去`github`发现已经有人提`issue`并给出了临时解决方案， `5.0`已经解决了但是`5.0`默认使用`webpack5.0`改动比较大升级成本也比较大。

[issue](https://github.com/vuejs/vue-cli/issues/6394)


顺便看看 `vue-cli5.0`是如何解决的这个bug

~~~js
  const cssPublicPath = (isAbsoluteUrl(rootOptions.publicPath) || rootOptions.publicPath.startsWith('/'))
      ? rootOptions.publicPath
      : process.env.VUE_CLI_BUILD_TARGET === 'lib'
        // in lib mode, CSS is extracted to dist root.
        ? './'
        : '../'.repeat(
          extractOptions.filename
            .replace(/^\.[/\\]/, '')
            .split(/[/\\]/g)
            .length - 1
        )
~~~
### 临时解决方案


~~~js

chainWebpack: (config) => {
    // fix publicPath for assets
    config.module.rule('images').use('url-loader').tap((options) => {
      const newOptions = options;
      newOptions.publicPath = '/my/public/path/';
      return newOptions;
    });
    config.module.rule('fonts').use('url-loader').tap((options) => {
      const newOptions = options;
      newOptions.publicPath = '/my/public/path/';
      return newOptions;
    });
    config.module.rule('svg').use('file-loader').tap((options) => {
      const newOptions = options;
      newOptions.publicPath = '/my/public/path/';
      return newOptions;
    });
~~~