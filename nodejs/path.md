### path.basename

参数 

* path 必传
* ext 可选

返回路径的最后一部分，和unix`basenmae`命令相似，末尾的反斜杠`/`将会被忽略

如果路径和扩展名不是字符串，将抛出错误

~~~js
path.basename('/foo/bar/baz/asdf/quux.html');
// Returns: 'quux.html'

path.basename('/foo/bar/baz/asdf/quux.html', '.html');
// Returns: 'quux'
~~~

