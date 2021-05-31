今天在使用`TypeOrm`查询数据库发现会把`bigint`转为`string`, 查看官方文档发现

~~~sh
Note about bigint type: bigint column type, used in SQL databases, doesn't fit into the regular number type and maps property to a string instead.
~~~

这就引出了`JavaScript`如何处理`bigint`这个话题?