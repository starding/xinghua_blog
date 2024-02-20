---
title: 使用 daocloud 持续集成应用
date: 2016-01-07 07:24:25
category: 作为工程师
---

## 写在前面的话
来自维基百科的：[货殖崇拜编程](https://zh.wikipedia.org/wiki/%E8%B4%A7%E7%89%A9%E5%B4%87%E6%8B%9C%E7%BC%96%E7%A8%8B)

为因 GFW 屏蔽访问不了维基百科的同学准备的：

>货物崇拜编程（Cargo Cult Programming）是一种计算机程序设计中的反模式，其特征为不明就里地、仪式性地使用代码或程序架构。货物崇拜编程通常是程序员既没理解他要解决的 bug、也没理解表面上的解决方案的典型表现。

>这个名词有时也指不熟练的或没经验的程序员从某处拷贝代码到另一处，却不太清楚其代码是如何工作的，或者不清楚在新的地方是否需要这段代码。也可以指不正确或过份的应用设计模式、代码风格或编程方法，却对其原理不明就里。

>货物崇拜编程来源于“货物崇拜”这个词。其衍生词还有“货物崇拜软件工程”。

在阅读任何教程类的文章时，都不要有编程上的货殖崇拜，有些内容是需要你根据自己的实际情况修改一些内容的，需要你理解它，如果不理解某些内容，就需要先熟悉一下相关的知识。

## daocloud 介绍
daocloud 提供基于 docker 进行持续集成的服务，使用它可以很方便的完成项目的自动化构建以及持续发布等功能。

当然，如果要使用 daocloud 持续化集成应用，首先你需要注册一个 daocloud 账号。如果你效力于某个公司的话，还要通知该公司把你的账号拉到公司用户组里，这样才能使用该公司的资源。

## 持续集成简单的 html5 应用

### 本小节使用场景
* 拥有自己的主机
* 将主机添加到了 daocloud 上
* 只使用 daocloud 的自动构建和部署功能
* 镜像也要部署到自己的主机上。

### 什么是「简单的 html5 应用」
只有前端页面，并且使用 nginx 提供服务。


### 部署大致解决方案
本文假设一个使用场景：拥有自己的主机，并且绑定到了 daocloud 上，然后想通过 daocloud 进行持续化地部署到自己的主机上。如果想要部署一个纯前端项目，需要把 nginx 集成进去，然后使用 nginx 来提供服务。


### 构建应用前的准备
#### 应用目录结构
因为 daocloud 是基于 daocker 的，在使用 daocloud 之前，我们需要按照 docker 的规范，把自己的项目改成支持打包成镜像的应用，对于 html5 项目来说，就是添加 Dockerfile 文件和配置好所需要的 nginx 配置文件。比如我有一个应用叫 jinli，其目录结构为 (你的目录结构和这个可能也不一样)：

```shell
jinli/  # 项目根目录
├── jinli/  # 具体的应用目录
│   ├── images/ # 图片
│   ├── index.html  # 默认的检索 html
│   ├── scripts/  #  js 文件夹
│   └── styles/    # 存放 css 的文件夹
├── Dockerfile   # Dockerfile
├── jinli.conf   # 本应用的 nginx 配置文件
└── log/  # log 文件
```
在这个目录结构中，和 docker 镜像构建是就是 Dockerfile 这个文件，jinli.conf 是个将要一起打包放入 docker 镜像中的文件，其他的都是应用本身的文件。

#### 创建 Dockerfile 文件
Dockerfile，就是一个 daocker 的规则文件，就像 make 的 Makefile 一样。Dockerfile 描述了将自己的应用构建成 docker 镜像的过程。
在本例中，Dockerfile 的内容如下：

```dockerfile
# 选择 nginx 服务
FROM nginx:1.9.5
# copy 代码
COPY . /src
# 添加 nginx 配置文件
COPY jinli.conf /etc/nginx/conf.d/
# 去掉默认的 nginx 配置文件
RUN rm /etc/nginx/conf.d/default.conf
```

Dockerfile 解释：
* 首先，想象将要构建的 docker 镜像包含一个 linux 操作系统。
* `FROM nginx:1.9.5` 的意思就是，将 1.9.5 版本的 nginx 服务集成到自己的镜像中（其实操作系统就是在这一步引入的，nginx 本身也是一个镜像，它是建立在一个 linux 操作系统之上的，拉取 nginx 的时候，会连把 nginx 连带其附着的 linx 系统整个一起拉下来）。
* `COPY . /src` 的意思是将当前目录下的代码复制到镜像中（的操作系统中）的 /src 目录下
* `COPY jinli.conf /etc/nginx/conf.d/` 的意思是，将当前目录下写好的 nginx 配置文件，复制到镜像（的操作系统）中的相应目录下。
* 删掉原来默认的 nginx 配置（因为默认配置文件会占用掉 localhost，不知道有没有更好的解决办法）

#### 配置在 docker 中使用的 nginx 配置文件。
本示例中，nginx 配置文件命名为 jinli.conf。把它打包到镜像中，来给 html5 应用提供服务，这与平时直接在服务器上部署没有什么很大的差别，需要注意的就是：

* server_name 为 localhost，因为这个 server_name 是在 docker 容器内部使用的。
* location 中的 root 文件夹是 docker 容器中的地址，不是宿主机的地址

```nginx
server {
    listen 80;
    server_name localhost;
    access_log /src/log/jinli.access.log;
    error_log /src/log/jinli.error.log;
    # 这里你还可以选择根据需要配一些 gzip 等选项
    location / {
    root /src/jinli/;
    }
}
```
然后将应用提交到 github 上，比如具体为：xiaohuruwei/jinli.git

以上已经包含全部必须的材料了，下面进入 daocloud 环节。

### 使用 daocloud 进行持续构建镜像以及发布
因为前面的步骤已经构建好了一个支持 docker 的代码版本，而且不需要外联像 mysql 这样的外部服务，因此可以直接将其当做一个单应用的镜像来构建。

在这方面，daocloud 已经有了完善的文档，可以直接参考其中的创建新项目章节，以及持续集成章节。

第四节创建新项目，根据具体情况选择要绑定需要的 git 源
第五节持续集成和镜像构建，在简单的 html 部署中，不需要了解 daocloud.yml 的写法
第九节中的向自由主机集群上发布应用

在上面的过程中，核心部分如下：
* 绑定 git 源
* 选择要部署的 git 仓库，并构建成镜像
* 部署到自有主机上
* 将后续自动持续部署的设置打开
* 在「代码构建」具体项目的「设置」中，将持续集成打开，设置好镜像和持续集成的触发规则，比如：在向 docker-support 分支提交代码时，就触发自动持续集成。
* 在「应用列表」中的「发布」设置中，将自动发布打开。

### 发布完成之后需要做的工作
发布完成之后，可以在 daocloud 上看到所创建的容器的具体信息。包括映射到宿主机上的端口号等等，点击端口号后可以看到该应用的访问地址，打开可以测试是否部署成功了

为方便后面的叙述，这里假设映射出的端口号为 8888。因为是发布到自己的主机上，所以还需要自己配置域名以及 nginx 反向代理来对外提供访问服务。假设有一个域名已经绑定在了自己的主机上，这里假设为 xiaohuruwei.com.

登陆自己的主机，并配置 nginx 文件，如果是使用 apt-get 一类的包管理工具安装的，那么应该是在/etc/nginx/conf.d/目录下添加 jinli.conf:

```nginx
server {
    listen 80;
    server_name xiaohuruwei.com;
    access_log /var/log/nginx/jinli.access.log;
    error_log /var/log/nginx/jinli.error.log;
    client_max_body_size 200m;
    # 可以根据需要设置其他配置项
    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://127.0.0.1:8888;
    }
}
```
然后访问 xiaohuruwei.com 就可以看到内容了

另外，需要在使用 daocloud 的时候，将自动构建和持续发布都打开，这样下次使用时，只需要往 github 推代码时，触发了你设定的更新规则，就会持续构建镜像并发布了。

## 持续集成依赖外部服务的应用
以 django 应用为例

### 本文适用的场景
* 拥有自己的主机
* 将主机添加到了 daocloud 上
* 只使用 daocloud 的自动构建和部署功能
所依赖的外部服务已经（以 daocke 容器）存在于自己的主机上，镜像也要部署在自己的主机上。在部署的同时连接主机上已经存在的外部服务（如 mysql，redis 等）。

### 依赖外部服务的应用
在 docker 领域中，如果一个 docker 镜像自身需要其他镜像来提供服务，就说这个镜像是带外部服务的。

以一个简单的 django 应用为例，如果这个 django 应用自身无法完成任务，而需要 mysql 或者其他数据库作为数据的持久化服务，那么这个 django 应用就是依赖外部服务的。

### 构建前的准备
准备同 html5 接近，因为需要连接外部服务，而且因为 uwsig 服务默认不支持静态文件的处理，所以多了一些对原代码的改造。

#### 项目结构
首先是 django 应用的目录，比如有一个应用叫 taikang，目录结构如下：


```shell
taikang	# 项目根目录	
├── Dockerfile	# Dockerfile 文件
├── logs	
├── manage.py
├── Makefile	# makefile 文件
├── requirements.txt	# 应用依赖的 python 库
├── templates
└── taikang		# 应用目录
	├── __init__.py
	├── models.py
	├── settings.py   # django 的配置文件
	├── urls.py
	├── views.py
	└── wsgi.py
```
Dockerfile 文件编写
Dockerfile 文件的含义和 html5 部署中是完全相同的，这里为：

```python
# 使用 python2.7.10 版本作为基础服务
from python:2.7.10
将当前项目中的内容全 copy 到镜像中的 /src 目录下
COPY . /src
# 安装 django 应用依赖的库，这里设置为豆瓣的 pypi 源
RUN cd /src; pip install -r  requirements.txt -i http://pypi.douban.com/simple --trusted-host pypi.douban.com
# 镜像暴露出 8000 端口
EXPOSE 8000
# 设置工作目录为代码所在目录
WORKDIR  /src
# 设置应用启动命令
CMD ["make", "start-uwsgi"]
```

#### Makefile 文件编写
可以注意到，上述最后的启动命令中使用了 make 命令，这也是前面目录结构中 Makefile 文件的作用所在。有关 makefile 的知识，这里不再详述。
总之可以让你更方便的管理应用，这里直接贴出 makefile 文件：


```makefile
# uwsig 启动的 host 和端口 
host:=0.0.0.0
port:=8000
# debug 的时候直接使用 django 自带的服务启动
debug:
    ./manage.py runserver $(host):$(port)
# 启动 uwsgi 服务
start-uwsgi:
    uwsgi --socket $(host):$(port) \
        --chdir $(shell pwd) \
        --wsgi-file taikang/wsgi.py \
        --master \
        --process 4 \
        --pidfile $(shell pwd)/uwsgi.pid
# 停止 uwsgi 服务
stop-uwsgi:
    uwsgi --stop uwsgi.pid
# 重载 uwsgi 服务
reload-uwsgi:
    uwsgi --reload uwsgi.pid
# 收集静态文件
collectstatic:
    ./manage.py collectstatic --noinput
.PHONY: debug \
    collectstatic \
    reload-uwsgi \
    start-uwsgi \
    stop-uwsgi
```
#### 静态文件支持
在 uwsgi 提供服务时，静态文件需要单独进行处理，目前推荐使用 django 的 whitenoise 库。可以方便的提供静态文件服务，仅仅需要几行配置。

#### 通过环境变量来连接服务
在使用 docker 部署项目，当需要连接外部的服务时，一般通过 link + 环境变量的参数进行。而为了支持环境变量的形式，我们的 django 应用也要做一些相应的改变。具体就是修改 settings.py 中的配置。

修改 settings.py 文件，以 mysql 为例：

```python
# 原来的 mysql 配置文件
#DATABASES = {
#    'default': {
#        'ENGINE': 'django.db.backends.mysql',
#        'NAME': 'starding_taikang',
#        'USER': 'root',
#        'PASSWORD': 'your_password_here',
#        'HOST': 'localhost',
#        'PORT': '3306'
#    }
#}
# 通过获取环境变量的形式来获取服务
import os  # 导入 os 库
mysql_name = os.environ.get('MYSQL_INSTANCE_NAME') or "taikang"  # 获取数据库名称
mysql_user = os.environ.get('MYSQL_USERNAME') or "root"  # 获取 mysql 用户名
mysql_password = os.environ.get('MYSQL_PASSWORD') or "your_password"  # 获取密码
mysql_host = os.environ.get('MYSQL_ADDR') or "127.0.0.1"  # 获取服务地址
mysql_port = "3306"
# 直接使用上面的变量代替字符串
DATABASES = {
   'default': {
       'ENGINE': 'django.db.backends.mysql',
       'NAME': mysql_name,
       'USER': mysql_user,
       'PASSWORD': mysql_password,
       'HOST': mysql_host,
       'PORT': mysql_port
   }
}
```

### 使用 daocloud 部署构建好的 taikang 项目
步骤基本同 html5 部署，假设你完成镜像构建之后，产生的镜像地址为：

```
daocloud.io/xiaohuruwei/taking:latest
```
下一步在使用镜像部署的时候，需要连接外部服务，具体是手动编辑 yaml 文件。


```python
takang_test:
  image: daocloud.io/xiaohuruwei/taking:latest
  restart: always
  external_links:
  - mysql1:mysql	  # 前面的 mysql1 是你主机上已经存在的 mysql 的容器实例，冒号后面是别名
  ports:
  - '8000'
  environment:    # 环境变量    
  - MYSQL_ADDR=mysql
``` 

但是我发现这种方法似乎有一些 bug，如果上面的方法不成功，那么你就先直接部署，部署成功之后，再去修改 yaml 文件成为上面描述的样子。在修改 yaml 文件的时候，daocloud 会提示重新部署，选择确定即可。

至此，django 项目也部署完成了。然后同 html5 项目部署一样，把各种自动构建，自动发布的功能打开就行了。

注意：由于 django 项目是使用 uwsgi 部署的，直接访问 daocloud 给出的地址是错误的，这个时候必须配置 nginx 反向代理。

附上 uwsgi 部署的 django nginx 代理配置文件：

```nginx
server {
    listen 80;
    server_name xiaohuruwei.com;
    access_log /var/log/nginx/taikang.access.log;
    error_log /var/log/nginx/taikang.error.log;
    client_max_body_size 200m;
    gzip on;
    gzip_min_length 1k;
    gzip_buffers 16 64k;
    gzip_http_version 1.1;
    gzip_comp_level 6;
    gzip_types text/plain application/x-javascript text/javascript text/css application/xml;
    gzip_vary on;
    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:32801;
    }
}
```


2016.01 于北京回龙观