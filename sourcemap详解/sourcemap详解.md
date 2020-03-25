我们在编写`webpack`配置文件都会遇到`devtool`这个属性，它有很多取值，`cheap-eval-source-map`、`cheap-module-eval-source-map`等在之前我只是大概的知道这个属性只是配置如何生成`source-map`但是对每个值具体有什么区别以及`source-map`是如何被应用到浏览器调试的也是一知半解。所以`source-map`到底什么用？

众所周知我们编写的代码经过`webpack`处理已经不是原来的模样，而`source-map`文件就是保存`源文件`和`编译后文件`关系的映射，最简单的例子是如果我们代码里有报错浏览器可以通过`source-map`文件确定当前报错信息在对应`源文件`里是第几行，此时我们点击报错信息就会跳到对应行，而假如没有`source-map`文件是无法做到的。

### 前世今生

关于`source-map`的前世今生我们可以

1. `source-map`起初是为了辅助`Cloure Compiler`(一款js压缩优化工具，可类比于uglify-js)以便在浏览器内调试经过后者压缩优化后的代码。
2. 2010年，在第二代即 [Closure Compiler Source Map 2.0](https://docs.google.com/document/d/1xi12LrcqjqIHTtZzrzZKmQ3lbTv9mKrN076UB-j3UZQ/edit?hl=en_US) 中，sourcemap确定了统一的json格式及其余规范，已几乎具有现在的雏形。最大的差异在于mapping算法，也是sourcemap的关键所在。第二代中的mapping已决定使用base 64编码，但是算法同现在有出入，所以生成的.map相比现在要大很多。
3. ，第三代即 [Source Map Revision 3.0] (https://docs.google.com/document/d/1U1RGAehQwRypUTovF1KRlpiOFze0b-_2gc6fAH0KY0k/edit?hl=en_US&pli=1&pli=1#heading=h.3l2f9su3ov2l)出炉了，这也是我们现在使用的sourcemap版本。从文档的命名看来，此时的sourcemap已脱离Clousre Compiler，演变成了一款独立工具，也得到了浏览器的支持。这一版相较于二代最大的改变是mapping算法的压缩换代，使用VLQ编码生成base64前的mapping，大大缩小了.map文件的体积。


