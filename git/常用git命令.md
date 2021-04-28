### 1.  更换远程仓库地址

~~~sh
git remote set-url origin xxx

# 或者
git remote rm origin
git remote add origin xxx
~~~

### 2. 添加多个远程仓库

~~~sh
 git remote add <name> <url>
 # 比如系统已经存在一个远程仓库origin, 添加第二个
 git remote add origin1 https://git.code.tencent.com/tx_devops/deploy-maven-pc.gi
 # 这样使用
 git push origin1 master:master
~~~

### 3. 切换到远程分支

~~~sh
git checkout -b dev origin/dev
~~~

### 4 删除分支

~~~sh

~~~
