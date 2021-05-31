### 为什么用DefinePlugin定义的环境变量要这样写`'"test"'`

假如我们定义了下面的运行时环境变量

~~~js
const env = {
  test: '/api/'
}
new webpack.DefinePlugin({
  'process.env': env
}),
~~~

我们在代码里这样使用

~~~js
axios.post(process.env.test) // 这样就会在运行时报错
~~~

由于`webpack.DefinePlugin`是在编译阶段动态的将`process.env.test`替换则上面的代码会被替换为

~~~js
axios.post(/api/) // 这是一个不合法的标识符
~~~

所以需要像下面这样写才可以


~~~js
const env = {
  test: '"/api/"'
}
new webpack.DefinePlugin({
  'process.env': env
}),
~~~