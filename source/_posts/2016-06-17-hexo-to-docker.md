---
title: 转移hexo到docker
date: 2016-06-17 15:49:34
tags:
  - docker
  - hexo
---

因为博客转到了 `hexo`, 它用的是 `node`, 而 `node` 的依赖包很容易出错，出错后也很难修复.
所以把环境隔离下比较好，用现在最火的 `docker`.    

因为是转移，所以有些步骤就不说了, 默认你会一些 `node`, `hexo`, `docker`.    

## 资料    
- [Dockerizing a Node.js web app](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/)
- [node OFFICIAL REPOSITORY](https://hub.docker.com/_/node/)

## 步骤    
### 目录结构  

    - hexo-docker
      - Dockerfile
    - hblog
      - package.json
      - source
      - themes
      ...

### docker-machine ip
查看虚拟机 `ip` , 等下就可以用它来打开 `blog`, 一般是 `192.168.99.100`.    

    docker-machine ip

### Dockerfile

    # pull node 4
    FROM node:argon

    # make workdir
    RUN mkdir -p /usr/src/blog
    WORKDIR /usr/src/blog

    # install hexo
    RUN npm install -g hexo-cli

### build image

    docker build -t hexo_blog .

### run image    
  - -v ~/hblog:/usr/src/blog 是把本机的 `hblog` 目录挂载到 `docker` 容器里面的
   `/usr/src/blog` 目录, 修改  `hblog` 里面的东西就等于修改容器里面 `/usr/src/blog` 的东西
  - -p 4000:4000, 把容器的 `4000` 端口映射出来
  - -it, Keep STDIN open even if not attached and Allocate a pseudo-tty
  - bash, 进入容器的bash命令行


    docker run -v ~/hblog:/usr/src/blog -p 4000:4000 -it hexo_blog bash

进入之后就可以像在本机一样使用 `hexo` 了.

### 使用hexo
因为是转移, `package.json` 里面有所有的依赖包, 所以先安装它, 在开启 `hexo` 服务.

    npm install
    hexo server

在浏览器里面输入 `http://192.168.99.100:4000/`, 就可以查看博客了.

### 下次使用

    # 列出容器
    docker ps －a

    # 进入容器
    docker start -i {容器id}

    # 停止容器
    docker stop {容器id}
