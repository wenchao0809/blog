### `chmod`

#### 权限简介

*   Linux系统上对文件的权限有着严格的控制，用于如果相对某个文件执行某种操作，必须具有对应的权限方可执行成功。
*   Linux下文件的权限类型一般包括读，写，执行。对应字母为 r、w、x。
*   Linux下权限的粒度有 **拥有者 、群组 、其它组** 三种。每个文件都可以针对三个粒度，设置不同的rwx(读写执行)权限。通常情况下，一个文件只能归属于一个用户和组， 如果其它的用户想有这个文件的权限，则可以将该用户加入具备权限的群组，一个用户可以同时归属于多个组。
*   Linux上通常使用chmod命令对文件的权限进行设置和更改

~~~bash

参数说明：
 
[可选项]
  -c, --changes          like verbose but report only when a change is made (若该档案权限确实已经更改，才显示其更改动作)
  -f, --silent, --quiet  suppress most error messages  （若该档案权限无法被更改也不要显示错误讯息）
  -v, --verbose          output a diagnostic for every file processed（显示权限变更的详细资料）
       --no-preserve-root  do not treat '/' specially (the default)
       --preserve-root    fail to operate recursively on '/'
       --reference=RFILE  use RFILE's mode instead of MODE values
  -R, --recursive        change files and directories recursively （以递归的方式对目前目录下的所有档案与子目录进行相同的权限变更)
       --help		显示此帮助信息
       --version		显示版本信息
[mode] 
    权限设定字串，详细格式如下 ：
    [ugoa...][[+-=][rwxX]...][,...]，
    其中
    [ugoa...]
    u 表示该档案的拥有者，g 表示与该档案的拥有者属于同一个群体(group)者，o 表示其他以外的人，a 表示所有（包含上面三者）。
    [+-=]
    + 表示增加权限，- 表示取消权限，= 表示唯一设定权限。
    [rwxX]
    r 表示可读取，w 表示可写入，x 表示可执行，X 表示只有当该档案是个子目录或者该档案已经被设定过为可执行。
 	
[file...]
    文件列表（单个或者多个文件、文件夹）
~~~

**示例：**

*  设置所有用户可读文件 `test.js`

```bash
chmod ugo+r nginx.conf
```

* 设置文件`test.js`和`test1.js`权限为所拥有者和所属同一个群组可读写

```bash
chmod a+r,ug+w test.js test1.js
```

* 设置`test`目录下和所有子目录皆设为任何人可读写

```bash
chmod -R a+rw 
```

#### 使用数字修改权限
linux中 4 、2 和 1表示读、写、执行权限（具体原因可见下节权限详解内容），即 r=4，w=2，x=1 。

**示例：**
* 设置所有人可续科协

~~~bash
chmod 777 file  (等价于  chmod u=rwx,g=rwx,o=rwx file 或  chmod a=rwx file)

~~~

### 更改文件拥有者(chown)

* 命令格式

~~~
    chown [选项]... [所有者][:[组]] 文件...
    
     **必要参数:**

　　　　-c 显示更改的部分的信息

　　　　-f 忽略错误信息

　　　　-h 修复符号链接

　　　　-R 处理指定目录以及其子目录下的所有文件

　　　　-v 显示详细的处理信息

　　　　-deference 作用于符号链接的指向，而不是链接文件本身

　　**选择参数:**

　　　　--reference=<目录或文件> 把指定的目录/文件作为参考，把操作的文件/目录设置成参考文件/目录相同拥有者和群组

　　　　--from=<当前用户：当前群组> 只有当前用户和群组跟指定的用户和群组相同时才进行改变

　　　　--help 显示帮助信息

　　　　--version 显示版本信息
~~~

**示例：**

改变`test`文件夹的所有者和组
~~~bash

chown -R nginx:nginx test
~~~


参考文章

*  [https://blog.csdn.net/u013197629/article/details/73608613](https://blog.csdn.net/u013197629/article/details/73608613)