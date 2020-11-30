### 1. nginx查看配置文件位置
~~~bash
nginx -t
~~~
### 2. `nginx`转发带`cookie`

~~~nginx
location ^~ /api/ {
  proxy_pass https://business-test.haina.com/api/;
  proxy_set_header Cookie $http_cookie;
  proxy_cookie_domain business-test.haina.com localhost;
  proxy_set_header Host business-test.haina.com;
}
~~~


ie11好像会失效

### 3. `nginx`转发注意事项

需要注意的是，在以上的配置中， 我们制定了`URI`为`/api/`，`nginx`规定
如果代理服务器地址中是带有URI的，此URI会替换掉 `location` 所匹配的URI部分。  
而如果代理服务器地址中是不带有URI的，则会用完整的请求URL来转发到代理服务器

官方文档描述
`If the URI is specified along with the address, it replaces the part of the request URI that matches the location parameter.
If the address is specified without a URI, or it is not possible to determine the part of URI to be replaced, the full request URI is passed (possibly, modified).`

### 4. `nginx` `alias`配置实例


~~~nginx
 server {
   listen        8080;
    server_name  localhost;
    location / {
       root   /home/admin/kindling/front/dist;
       index  index.html;
       try_files $uri $uri/ /index.html;
    }
    location /preview {
        alias /home/admin/kindling/front/preview/dist/;
        index  index.html;
        try_files $uri $uri/ /preview/index.html;
    }
    location ^~ /api/basic/ {
        proxy_pass http://localhost:3000;
        client_max_body_size    100m;
    }
}
~~~

`alias`必须指定`URI`


### 5. `location`路径匹配

location 路径正则匹配： 

  
| 符号 | 说明 |
| --- | --- |
| `~` | 正则匹配，区分大小写 |
| `~*` | 正则匹配，不区分大小写 |
| `^~` | 普通字符匹配，如果该选项匹配，则，只匹配改选项，不再向下匹配其他选项 |
| `=` | 普通字符匹配，精确匹配 |
| `@` | 定义一个命名的 location，用于内部定向，例如 error\_page，try\_files |

  

2.2 匹配优先级：


路径匹配，优先级：（跟 location 的书写顺序关系不大）   

1.  **精确匹配**：
    
    `=`前缀的指令严格匹配这个查询。
    
    如果找到，停止搜索。
    
2.  **普通字符匹配**：
    
    所有剩下的常规字符串，最长的匹配。
    
    如果这个匹配使用`^〜`前缀，搜索停止。
    
3.  **正则匹配**：
    
    正则表达式，在配置文件中定义的顺序，匹配到一个结果，搜索停止；
    
4.  **默认匹配**：
    
    如果第3条规则产生匹配的话，结果被使用。
    
    否则，如同从第2条规则被使用。