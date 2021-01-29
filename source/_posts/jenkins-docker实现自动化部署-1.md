title: jenkins+docker实现自动化部署
author: 米勒
date: 2021-01-27 14:20:30
tags:
---
> 本教程实现自动从`Github`下载`Vue`项目进行自动部署。
>
> `Docker`负责集成项目需要的环境，不需要预先下载各种环境在电脑上
>
> `Jenkins`负责执行脚本，比如下载`Github`的项目，执行安装命令和运行命令等

<!--more-->

## 相关应用
https://docs.docker.com/

https://www.jenkins.io/zh/

## 流程
1. 可以连接网络的`Linux`环境，我这里使用 [vultr](https://my.vultr.com/) 开一个linux服务器。
2. 下载安装`Docker`和`Git`，`Jenkins`我们也将运行在`Docker`里，不会`Docker`的同学可以 [Docker入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html) 本地环境试一下。
3. 使用`Docker`手动部署项目，这里将配置`Dockerfile`文件引入`Node`和`Nginx`对项目进行打包运行。单纯使用`Docker`可以部署项目并且运行后，再去使用`Jenkins`帮我们自动执行这些命令。
4. 安装部署`Jenkins`，我们将用`Docker`安装`Jenkins`，所以到这一步，你已经会使用`Docker`并且运行镜像了。
5. 配置`Jenkins`进行单次一键部署，它将执行：从`Github`克隆项目 > 执行上面`Docker`的一系列命令并且运行镜像。
6. 配置`Jenkins`进行定时自动部署，配置时间点，自动克隆项目并且打包部署
7. 配置`Jenkins`监听`Github`指定项目变化进行自动部署
# Docker篇
## 安装Docker

> 这里有两种安装方式
>
> 1. `yum install docker`
> 2. [`yum install docker-ce`](https://www.cnblogs.com/zyh-smile/p/12350371.html)
>
> 第二种方式解决`docker.service`不存在问题，可以解决镜像源更换问题提高下载速度，可以[点击](https://www.cnblogs.com/zyh-smile/p/12350371.html)查看安装教程。
>
> 建议使用第二种安装
 
 ## 使用Docker运行项目，会使用Docker可以跳过
 
 准备一个项目，我这里使用`Vue`项目，使用多阶段构建`Node`+`Nginx`进行部署
 
 首先创建一个`Vue`项目，这个就不赘述了，在项目根目录创建配置文件
 
 
 ***.dockerignore***
```
node_modules
npm-debug.log
.git
```
配置打包镜像将过滤的文件，跟 `.gitignore` 同理

***nginx/default.conf***
```
server {
    listen       90;
    server_name  localhost;

    #charset koi8-r;
    access_log  /var/log/nginx/host.access.log  main;
    error_log  /var/log/nginx/error.log  error;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```
创建 `nginx` 文件夹，下放 `default.conf` ，也可以直接放根目录，最后在`Dockerfile`将使用这个文件代替nginx默认配置 ，这里配置 `nginx` 反向代理的配置，比如端口，主入口等。

***Dockerfile***
```
# 引入node
FROM node:latest as js_builder

# 打包项目生成dict
COPY . /build
WORKDIR /build
RUN npm config set registry https://registry.npm.taobao.org/
RUN npm install
RUN npm run build

# 引入nginx
FROM nginx:latest

# 将dist文件夹 复制到 /usr/share/nginx/html 下
# 分段构建 --from=上面的FROM镜像内的 build/dist 包含WORKDIR
COPY --from=js_builder build/dist/ /usr/share/nginx/html/
COPY nginx/default.conf /etc/nginx/conf.d/default.conf
```
`Docker` 构建镜像时会根据这个配置执行，这里先引入 `node` 版本`lastest(最近的)` 取一个别名为 `js_builder` 或者其他，因为先引入了 `node` 会先进这个 `node` 的镜像中，类似虚拟机，在这个环境中执行项目打包指令，最后生成 `dist` 打包完文件夹，在 [Docker入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html) 中有讲到。

下面引入 `nginx` ，`--form=js_builder` 指向上面的 `node` 镜像，复制 `build/dist/` 文件夹，如果上面 `COPY . .` 那这边复制 `dist/` 就行了，复制到 `usr/share/nginx/html` 目录下，是 `nginx` 的环境，如果不知道目录在哪里，运行用 `-it` 映射当前 `shell` ,在 [Docker入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html) 中有讲到。
再将 `nginx` 配置复制替换掉。

配置完项目，上交至 `Github` ，这里下载一个 `Git` 

`yum install git`
![upload successful](/img/auto/pasted-6.png)

安装完后，我们创建一个文件夹存放项目 `mkdir test` , `cd test` , `git clone ...` , `cd ...`
![upload successful](/img/auto/pasted-7.png)

创建完后，使用 `Docker` 构建出 `Image` 镜像

`docker build -t test_image .`
![upload successful](/img/auto/pasted-8.png)
可以看到正在一步步执行 `Dockerfile` 文件内容，最后一个 . 指的是 `Dockerfile` 所在目录为当前

打包完后，查看镜像 `docker image ls`
![upload successful](/img/auto/pasted-10.png)
看到有一些 none 是因为重复构建了，可以 `docker image rm ID` 删掉

运行镜像 `docker container run -p 10000:90 --name test_container test_image`，可以在 `-p` 前加一个 `-d` 就可以后台运行了
![upload successful](/img/auto/pasted-12.png)
这边10000端口映射镜像内的90端口，为什么是90呢，上面 `default.conf` 配置的

使用 `Docker` 跑起了项目，已经成功一半了，接下来就使用 `Jenkins` 帮你做这些事情😊

# Jenkins篇
## 配置部署Jenkins

> 🆗到这里，你的 `Docker` 已经入门了，接下来安装 `Jenkins`

从云端pull下 `Jenkins` 镜像 `docker pull jenkins/jenkins` ， 查看一下 `docker image ls`
![upload successful](/img/auto/pasted-14.png)
这就是 `Jenkins` 镜像了，运行它，并且将文件映射到本地，这个很重要，部署的项目和默认登录密码都在其中，而且克隆下的项目也在其中，我们需要进入这个项目运行`Docker`。接下来，可能会遇到一些坑。

> 先创建一个目录存放映射的 `Jenkins`，再运行 `Jenkins`并且映射至创建目录
使用 `-v [存放目录]:[被映射目录]`
```
# cd ~
# mkdir jenkins
# docker run -d -p 9000:8080 -v /root/jenkins/:/var/jenkins_home --name jenkins_container jenkins/jenkins
```
`docker run -d -p 9000:8080 -v /root/jenkins/:/var/jenkins_home --name jenkins_container jenkins/jenkins`
![upload successful](/img/auto/pasted-15.png)
运行项目，`-d`后台运行，最后会给出一个容器`ID`

`docker container ls` 查看 发现并没有容器
![upload successful](/img/auto/pasted-16.png)
这个时候可能是报错了，文件权限问题，取消`-d`再试一下

`docker run -p 9000:8080 -v /root/jenkins/:/var/jenkins_home --name jenkins_container jenkins/jenkins`
![upload successful](/img/auto/pasted-18.png)
报错误，文件权限问题，这里请查看官方[#177issue](https://github.com/jenkinsci/docker/issues/177)

如果还是无法解决，重装`Docker`，点击[这个教程](https://www.cnblogs.com/zyh-smile/p/12350371.html)

运行成功后，`docker ps` 可以看到容器已经运行
![upload successful](/img/auto/pasted-22.png)

进入 `9000` 端口，等待 `Jenkins` 开启
![upload successful](/img/auto/pasted-24.png)

获取密码 `vi /root/jenkins/secrets/initialAdminPassword`
![upload successful](/img/auto/pasted-25.png)

点击安装推荐的插件，然后等待完成
![upload successful](/img/auto/pasted-26.png)

创建一个账号,一路默认进入Jenkins
![upload successful](/img/auto/pasted-27.png)

## 使用Jenkins一键部署
***注意如果插件没安装完整，例如没有Git等，去这里安装插件***
![upload successful](/img/auto/pasted-30.png)

安装`ssh`
![upload successful](/img/auto/pasted-33.png)

配置`ssh`
![upload successful](/img/auto/pasted-35.png)
拉到最下面
windows开启SSH [点击这里](https://blog.csdn.net/ujsDui/article/details/84105303)
![upload successful](/img/auto/pasted-38.png)

创建项目
![upload successful](/img/auto/pasted-28.png)
![upload successful](/img/auto/pasted-29.png)

你的项目地址
![upload successful](/img/auto/pasted-31.png)

你的源码，注意你的分支，默认是`master`，目前`Github`已经默认为`main`
![upload successful](/img/auto/pasted-32.png)

配置构建环境，这里需要通过`ssh`发送指令。
![upload successful](/img/auto/pasted-37.png)

配置完后，手动构建试试
![upload successful](/img/auto/pasted-40.png)
在控制台中就可以看到运行状态
![upload successful](/img/auto/pasted-41.png)
最后出现success证明成功
![upload successful](/img/auto/pasted-47.png)

## 定时部署（轮询SCM）
![upload successful](/img/auto/pasted-42.png)
![upload successful](/img/auto/pasted-45.png)
参考 https://juejin.cn/post/6844903982066843661

## 监听Git变化部署
参考 https://juejin.cn/post/6844903986017878029

注意：创建 `Webhooks` 的规则，`Git` 改变代码时会向这个地址发送请求，确保 `Jenkins` 地址暴露在公网上
![upload successful](/img/auto/pasted-46.png)