### Dockerfile

`Dockerfile`用于配置镜像构建的一系列步骤

### node服务镜像构建的的Dockerfile

~~~sh
FROM node:lts-alpine3.10 #使用node镜像作为基础镜像
RUN npm install -g cnpm --registry=https://registry.npm.taobao.org #安装cnpm

WORKDIR /opt #指定容器工作目录
ADD . . #将当前目录copy到WorkDir第一.指我们当前运行的docker的文件件第二.指定容器的目录
RUN cnpm install #安装依赖
RUN npm run build #执行 npm run build 构建

CMD npm run start #指定容器启动时运行的命令

EXPOSE 7001#指定暴露的端口这个端口和你服务运行的端口一直
~~~

### 前端服务构建Dockerfile

~~~sh
FROM alpine:latest #指定基础镜像
RUN set -eux && sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories #修改apk源
RUN apk --no-cache add tzdata ca-certificates libc6-compat libgcc libstdc++ nginx curl #安装nginx等依赖
ENV TZ Asia/Shanghai #设置时区

ADD ./nginx/http.conf /etc/nginx/conf.d/ #添加nginx配置文件

WORKDIR /www/
RUN mkdir /run/nginx 


COPY ./dist ./dist #copy前端打包文件

ENTRYPOINT nginx -g 'daemon off;' #指定启动命令
EXPOSE 3000 #b暴露服务端口
~~~

### 常用命令

#### 打包镜像

`sudo docker build -t advice-backend:1.0.0 .`

#### 查看镜像

`sudo docker image ls`

####