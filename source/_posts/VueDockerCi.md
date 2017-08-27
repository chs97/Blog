---
title: 使用daocloud和docker实现vue的持续集成
date: 2017-7-22 21:14
tags:
  - JavaScript
  - Vue.js
  - Docker
categories: 前端
---

# 使用daocloud和docker实现vue的持续集成

---

## 一、Dockerfile

[Docker File 命令](https://yeasy.gitbooks.io/docker_practice/content/image/dockerfile/)

Dockerfile 是一个文本文件，其内包含了一条条的**指令(Instruction)**，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

而 `FROM` 就是指定**基础镜像**，因此一个 `Dockerfile` 中 `FROM` 是必备的指令，并且必须是第一条指令。

Dockerfile中的每一条指令都是一个layer，docker build的过程中会缓存layer，所以就存在可优化的地方。

## 二、持续集成

push代码到远程分支，自动测试 自动构建 自动发布。

daocloud使用安全构建： 创建项目->安全构建

安全构建有2步，第一步为build，构建一个临时镜像 第二步为将临时镜像打包，开始安全构建

## 三、3个关键文件
<!--more-->
### 一、Dockerfile

```dockerfile
FROM node:alpine
RUN apk add --no-cache tzdata && \
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" > /etc/timezone && \
    apk del tzdata
RUN apk update && \
    apk add --no-cache git
COPY package.json /tmp/package.json
RUN npm config set registry https://registry.npm.taobao.org
RUN cd /tmp && npm install && mkdir -p /opt/workdir && mv /tmp/node_modules /opt/workdir/
WORKDIR /opt/workdir
COPY . /opt/workdir
RUN npm run build
```

其中的

```dockerfile
COPY package.json /tmp/package.json
RUN npm config set registry https://registry.npm.taobao.org
RUN cd /tmp && npm install && mkdir -p /opt/workdir && mv /tmp/node_modules /opt/workdir/
```

就是利用缓存进行构建的优化，只有这样，才能利用上次缓存不重新安装npm包。否则每次都重新安装npm包消耗大量的构建时间。

这个Dockerfile很显然是build过程

构建结束后，会生成dist目录。我们发布的时候需要把dist的目录丢到网站root目录。

daocloud安全构建的第二步骤可以将第一步骤中的文件提取出来。

文件提取的目录为： /opt/workdir/dist

### 二、Dockerfile.build

```dockerfile
FROM nginx:alpine
MAINTAINER Chs97 chs97@w2fzu.com
COPY /opt/workdir/dist /usr/share/nginx/html/

COPY /favicon.ico /usr/share/nginx/html/favicon.ico

COPY /nginx.conf /etc/nginx/nginx.conf

CMD ["nginx", "-g", "daemon off;"]
```

将构建结果打包发布

### 三、nginx.conf

```nginx
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}

}
```

因为vue-route开启了history模式。需要配置不同的nginx.conf

## 四、提供持续集成服务的网站

1.jenkins

2.flow.ci

3.[Travis CI](https://travis-ci.org/)