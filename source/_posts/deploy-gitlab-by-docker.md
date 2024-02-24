---
title: docker 部署 gitlab
date: 2016-01-07 08:27:22
tags:
---

## 拉取镜像及启动容器
参考：[gitlab documentation](https://docs.gitlab.com/ee/install/docker.html)

### 首先使用 dao 加速器拉取 daocker 镜像：

```shell
dao pull gitlab/gitlab-ce
```

### 创建 volumes
简单来说 volumes，就是可以映射到容器中去的容器外部存储空间。你可以将一些比价通用的配置文件，数据，或者是代码等都使用 volumes 的形式存储在容器所在的宿主机器上。这样不仅可以永久保留数据，保证数据的安全性。同时还可以方便的修改 volumes 中的内容，然后重新映射到容器中，这对于需要经常动态修改文件的容器非常有用。
在本次部署 gitlab 的时候，创建三个 volumes，分别是/mnt/volumes/gitlab 下的 config，logs，data 目录。
启动一个 gitlab 容器

```shell
sudo docker run --detach \
    --hostname git.xiaohuruwei.com \
    --publish 8443:443 --publish 8080:80 --publish 2222:22 \
    --name gitlab \
    --restart always \
    --volume /mnt/volumes/gitlab/config:/etc/gitlab \
    --volume /mnt/volumes/gitlab/logs:/var/log/gitlab \
    --volume /mnt/volumes/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest
```

命令参数解释：

–hostname
指定容器中绑定的域名，会在创建镜像仓库的时候使用到，这里绑定 git.xiaohuruwei.com

–publish
端口映射，冒号前面是宿主机端口，后面是容器 expose 出的端口

–volume
volume 映射，冒号前面是宿主机的一个文件路径，后面是容器中的文件路径

## 配置 nginx，支持 https
参考：[gitlab set nginx](https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/nginx.md#enable-https)

nginx 配置文件
```nginx
server {
    listen 80;
    server_name git.xiaohuruwei.com;
    access_log /var/log/nginx/gitlab.xiaohuruwei.access.log;
    error_log /var/log/nginx/gitlab.xiaohuruwei.error.log;
    rewrite ^ https://git.xiaohuruwei.com;
}
```

https proxy
```nginx
server {
    listen       443 ssl;
    server_name  git.xiaohuruwei.com;
    access_log /var/log/nginx/https-gitlab.xiaohuruwei.access.log;
    error_log /var/log/nginx/https-gitlab.xiaohuruwei.error.log;
    # ssl 证书配置，这里使用的是自己生成的证书，在访问时会提示证书问题，选择相信即可。
    # 如果想要公认的证书，需要在网络上的一些授权中心生成
    ssl on;
    ssl_certificate /etc/nginx/ssl/getbase.crt;
    ssl_certificate_key /etc/nginx/ssl/getbase_nopass.key;
    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass https://localhost:8443;
    }
}
```

## 开放 gitlab 的 https 支持
仅仅由 nginx 反向代理 https 是不行的，因为还需要打开 gitlab 的 https 支持。

修改配置文件，在/mnt/volumes/gitlab/config/ 目录下的 gitlab.rb 中添加：

```ruby
# note the 'https' below
external_url "https://gitlab.example.com"
```

新建 ssl 目录，同时在该目录下添加 ssl 证书文件，命名要与上述域名中保持一致

git.xiaohuruwei.com.crt
git.xiaohuruwei.com.key

重新启动容器

```shell
docker restart gitlab
```

## 访问 gitlab 测试
打开 web 界面，默认登录名为 root，密码为 5iveL!fe（已经改为厘米脚印的默认密码），新建一个 project 仓库:test
因为 ssl 证书是自己生成的，并不具有全网通用性，因此只能先选择相信证书。在本地设置环境变量：

```shell
export GIT_SSL_NO_VERIFY=1
```

然后克隆仓库：git clone https://git.xiaohuruwei.com/root/test.git

## 开启邮件服务
默认的邮件服务不太好使，所以这里配置自己的邮件服务。参考官方 gitlab stmp 文档。

使用 163 邮箱，按照官方文档配置后，会发现发送邮件没有起作用。

```ruby
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.163.com"
gitlab_rails['smtp_port'] = 25
gitlab_rails['smtp_user_name'] = "xiaohuruwei@163.com"
gitlab_rails['smtp_password'] = "xxxx"
gitlab_rails['smtp_domain'] = "163.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = false
gitlab_rails['smtp_openssl_verify_mode'] = 'none'
```

查看 log 时，由于 log 比较杂乱，没有发现问题，后来在 ruby 社区发现有人解决过同样的问题：
[GitLab 配置通过 smtp.163.com 发送邮件](https://ruby-china.org/topics/20450)
以及网易的官方解释：
[网易服务器 smtp 机器要求身份验证帐号和发信帐号必须一致，如果用户在发送邮件时，身份验证帐号和发件人帐号是不同的，因此拒绝发送。](https://www.mail163.cn/fault/analysis/1109.html)

所以又添加了两行配置后测试可以正常使用了：

```ruby
gitlab_rails['gitlab_email_from'] = "xiaohuruwei@163.com"
user['git_user_email'] = "xiaohuruwei@163.com"
```
## ssh 方式访问
因为是使用 docker 部署的，通过 ssh 方式 (比如 git clone git@git.xiaohuruwei.com) 访问会有两层认证：

一层是 freelancer 服务器的认证
另一层是 gitlab 的认证。
后者需要使用 ssh-key
前者可能需要 ssh 本身的反向代理 (现在使用的 nginx 不支持除 http，https 以外的反向代理)，

现在发现使用端口转发的形式比较困难，但是可以改变默认的 gitlab 的 ssh 端口为非标准端口：
直接修改 gitlab 配置文件中的变量：

```ruby
gitlab_shell_ssh_port = 2222
```

然后重新启动 docker 容器，就可以在 web 界面中看到相应的 ssh 地址发生了改变：
ssh://git@git.xiaohuruwei.com:2222/root/test.git 然后就直接可以继续使用 git clone 来继续操作了

2016.01 于北京回龙观