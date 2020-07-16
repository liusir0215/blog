---
title: docker入门
top: false
cover: false
toc: true
mathjax: true
tags:
  - docker
  - centos7
categories:
  - docker
date: 2020-07-13 21:58:18
password:
summary:
---
# 前言

## 背景
服务在部署的时候对环境都有一定的依赖，环境不一样可能导致服务的运行需要对环境进行适配，额外增加了不少工作量。以nodejs为例，nodejs因为最近几年飞速发展导致存在不同版本的nodejs共存的情况。虽然有nvm、nvs等非常优秀的node版本管理器，但是还是可能会出现版本的问题。
docker的出现完美的解决了这些问题，docker可以实现一次创建或配置可以在任意平台正常运行，而且在从一个平台迁移到另外一个平台的时候不用担心需要安装nodejs环境。[docker](https://wiki.jikexueyuan.com/project/docker-technology-and-combat/why.html)查看docker的优势。

# 安装docker
centos7可以使用`yum`的方式来安装docker，首先安装devicemapper存储类型

```shell
sudo yum update
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

# 添加yum源
sudo yum-package-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

更新yum源并安装docker

```shell
sudo yum update
sudo yum install docker-ce
```

启动docker

```shell
# 设置docker自启动
sudo systemctl enable docker

# 启动docker
sudo systemctl start docker
```

# 服务docker化
## 创建node应用
我将最近正在开发的koa2应用拿过来进行docker化，如果你没有需要docker化的服务可以初始化一个[express](https://expressjs.com/zh-cn/starter/hello-world.html)应用

## 创建Dockerfile
在应用的根目录下新建`Dockerfile`文件

```shell
touch Dockerfile
```

使用编辑器打开`Dockerfile`文件，首先确认我们需要从哪个镜像进行构建。当前nodejs stable版本是12.x。

```shell
FROM node:12
```

创建一个文件夹存放应用程序代码，这将是你的应用程序工作目录：

```shell
# Create app directory
WORKDIR /usr/src/app
```

此镜像中 Node.js 和 NPM 都已经安装，所以下一件事对于我们而言是使用 npm 安装你的应用程序的所有依赖。请注意，如果你的 npm 的版本是 4 或者更早的版本，package-lock.json 文件将不会自动生成。

```shell
# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm ci --only=production
```

请注意，我们只是拷贝了 package.json 文件而非整个工作目录。这允许我们利用缓存 Docker 层的优势。bitJudo 对此有一个很好的解释，请[见此](http://bitjudo.com/blog/2014/03/13/building-efficient-dockerfiles-node-dot-js/)。 进一步说，对于生产环境而言，注释中提及的`npm ci`命令协助提供了一个更快、可靠、可再生的构建环境。欲知详情，可以参考[此处](https://blog.npmjs.org/post/171556855892/introducing-npm-ci-for-faster-more-reliable)。

因为应用使用到了`typescript`和`pm2`，所以全局需要安装`pm2 typescript eslint`

```shell
# 安装全局三方包并且ts编译成js
RUN npm install pm2 typescript eslint -g
```

在 Docker 镜像中使用`COPY`命令绑定你的应用程序：

```shell
# Bundle app source
COPY . .
```

应用程序绑定的端口为`3000`，所以你可以使用`EXPOSE`命令使它与`docker`的镜像做映射：

```shell
EXPOSE 3000
```

最后但同样重要的事是，使用定义运行时的`CMD`定义命令来运行应用程序。这里我们使用最简单的`npm start`命令，它将运行`node app.js`启动你的服务器。因为服务是通过`pm2`应用`ecosystem.config.js`来开启的：

```shell
# express服务
# CMD [ "node", "server.js" ]
CMD [ "pm2-runtime", "start", "ecosystem.config.js" ]
```

最后我的`Dockerfile`文件看上去是这个样子的

```shell
FROM node:12
# FROM alpine AS builder
# RUN apk add --no-cache --update nodejs nodejs-npm

# Create app directory
WORKDIR /usr/src/app

# USER nodejs
# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

# 安装全局三方包并且ts编译成js
RUN npm install pm2 typescript eslint -g

# If you are building your code for production
# ENV NODE_ENV production
RUN npm install
# RUN npm ci --production

# Bundle app source
COPY . .
RUN npm run deploy

EXPOSE 3000
CMD [ "pm2-runtime", "start", "ecosystem.config.js" ]
```

## 创建.dockerignore
在`Dockerfile`的同一个文件夹中创建一个`.dockerignore`文件，带有以下内容：

```shell
node_modules
npm-debug.log
```

这将避免你的本地模块以及调试日志被拷贝进入到你的 Docker 镜像中，以至于把你镜像原有安装的模块给覆盖了。

## 构建镜像
进入到`Dockerfile`所在的那个目录中，运行以下命令构建`Docker`镜像。开关符`-t`让你标记你的镜像，以至于让你以后很容易地用`docker images`找到它。

```shell
docker build -t <your username>/node-web-app .
```

`Docker`现在将给出你的镜像列表：

```shell
$ docker images

# Example
REPOSITORY                      TAG        ID              CREATED
node                            12         1934b0b038d1    5 days ago
<your username>/node-web-app    latest     d64d3505b0d2    1 minute ago
```

## 运行镜像
使用`-d`模式运行镜像将以分离模式运行`Docker`容器，使得容器在后台自助运行。开关符`-p`在容器中把一个公共端口导向到私有的端口，请用以下命令运行你之前构建的镜像：

```shell
docker run -p server_port:3000 -d <your username>/node-web-app
```

把你应用程序的输出打印出来

```shell
# Get container ID
$ docker ps

# Print app output
$ docker logs <container id>

# Example
Running on http://localhost:3000
```

如果你需要进入容器中，请运行 exec 命令：

```shell
# Enter the container
$ docker exec -it <container id> /bin/bash
```

比如想要通过pm2查看当前机器的运行状态可以通过以下命令查看

```shell
docker exec -it <container id> pm2 monit
```

![pm2 monit](http://img95.699pic.com/element/40102/6907.png_300.png!/clip/0x300a0a0)

## 测试
为测试你的应用程序，给出与 Docker 映射过的端口号：

```shell
$ docker ps

# Example
ID            IMAGE                                COMMAND    ...   PORTS
ecce33b30ebf  <your username>/node-web-app:latest  npm start  ...   49160->3000
```
在上面的例子中，在容器中`Docker`把端口号`3000`映射到你机器上的`49160`。

现在你可以使用`curl`（如果需要的话请通过`sudo yum install curl`安装）调用你的程序了：

```shell
$ curl -i localhost:49160

HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 12
ETag: W/"c-M6tWOb/Y57lesdjQuHeB1P/qTV0"
Date: Mon, 13 Nov 2017 20:53:59 GMT
Connection: keep-alive

Hello World!
```

# References
[1] [把一个 Node.js web 应用程序给 Docker 化](https://nodejs.org/zh-cn/docs/guides/nodejs-docker-webapp/)
