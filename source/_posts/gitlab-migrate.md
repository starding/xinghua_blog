---
title: docker-gitlab 数据迁移
og_title: docker-gitlab 数据迁移
date: 2016-01-18 09:22:59
category: 作为工程师
---


## 问题来源：docker 的 file system 神坑
我们的 gitlab 是使用 docker 部署的，方便快捷。

但是后来服务器上的某个应用出现了内存泄露，没想到重启服务器之后整个 docker 就跪了，启动容器时总是遇到一系列比较底层的 filesystem unkown 错误，而且报错细节不止一种。调查之后，发现是 docker 使用的 devicemapper 出现了 bug，而且查看 docker 官方的 github，有不少人遇到这个 bug，讨论话题在此：[docker fails to mount the block device for the container on devicemapper](https://github.com/moby/moby/issues/4036)，感觉掉到一个深不见底的神坑里了。

stackoverflow 或 github 上给出的解决办法是：关掉 docker，删除/var/lib/docker 中的所有内容，然后重启 gilab。我觉得可能是另一个神坑，没敢往里跳。于是选择慢慢研究这个问题，先在另外一台服务器上重新部署一下 gitlab，并且把数据迁移过去就行了。

## gitlab 数据迁移
原以为迁移会很简单，没想到也有不少坑等着跳。

先把原服务器上的 gitlab 的 volumes 备份了一下，直接把备份好的 volumes 文件拷贝到新服务器上，按照原来的方法挂载，启动。

### 遇到的问题
首先发现是 redis 啥啥没有权限，于是 google 了一番，把权限都改了。
然后出了一个 unfoudmethod 错误。调查是[数据库没有 migrate](https://gitlab.com/gitlab-org/gitlab-foss/-/commit/ff98c631c1004247656677568989e5ed68fc88f3) 的造成的。

还有一个隐藏的坑是 gitlab 的版本问题。类似的关键服务应用，尽量不要使用 latest 作为版本号，不然出了问题，还得去找自己使用的究竟是什么版本，如果找不到…准备拼人品吧。

### 最终的解决办法
感觉这么一个一个坑的跳太二了，后来发现原来已经有人把坑都填好了：[Migrate a Gitlab Docker container from version 8.0.4 to 8.2.0](https://cmanios.wordpress.com/2015/12/04/migrate-a-gitlab-docker-container-from-version-8-0-4-to-8-2-0/)

## 附录：下面是原文（已翻译）
前几天，我需要将 docker 部署的 gitlab 实例从 8.04 版迁移到 8.2.0 版本。我完全是按照 Gitlab Docker 镜像文档中的步骤进行的，但是出了一堆问题，简直日了狗。不过谢天谢地，在经过数小时的搜索挣扎之后，我终于搞定了。

在本教程中，假定 Gitlab 的 volumes 存储在 /home/bob/docker-data/gitlab 目录中 (当然，你需要根据你的实际目录结构来适当改写本教程)。然后下面是我成功迁移 gitlab 时遵循的所有步骤：

### 1.停止 Gitlab 容器
```bash
docker stop gitlab
```

### 2.备份所有 Docker Volumes (所有的 gitlab 文件)
```sh
backupDate=$(date +"%Y%m%d%H%M%S") \
  && cd /home/bob/docker-data/ \
  && sudo tar zvcf gitlab-data-${backupDate}.tar.gz gitlab/
```

### 3.备份 gitlab 镜像（可选）
```sh
docker save -o /home/bob/gitlab-ce-image.tar gitlab/gitlab-ce:latest
```
### 4.删除 gitlab 容器 ( 译注：如果你是把数据迁到其他服务器的 docker gitlab 实例下的话，本步骤可选)
```sh
docker rm gitlab
```

### 5.拉取最新的 gitlab 镜像
```sh
sudo docker pull gitlab/gitlab-ce:latest
```

### 6.拉取完毕后，创建并运行容器
```sh
docker run -d \
  --hostname 192.168.1.1 \
  --publish 8443:443 --publish 8082:80 --publish 2224:22 \
  --name gitlab \
  --restart always \
  --volume /etc/localtime:/etc/localtime \
  --volume /home/bob/docker-data/gitlab/config:/etc/gitlab \
  --volume /home/bob/docker-data/gitlab/logs:/var/log/gitlab \
  --volume /home/bob/docker-data/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```

### 7.容器启动后，观察运行日志：
```sh
docker logs -f --tail 10 gitlab
```
日志内容

```log
[2015-11-26T15:12:26+02:00] INFO: Retrying execution of execute[create gitlab database user], 19 attempt(s) left
[2015-11-26T15:12:28+02:00] INFO: Retrying execution of execute[create gitlab database user], 18 attempt(s) left
... (some lines omitted) ...
[2015-11-26T15:13:09+02:00] INFO: Retrying execution of execute[create gitlab database user], 0 attempt(s) left
 
 
Error executing action `run` on resource 'execute[create gitlab database user]'
 
 
Mixlib::ShellOut::ShellCommandFailed
 
Expected process to exit with [0], but received '2'
---- Begin output of /opt/gitlab/embedded/bin/psql --port 5432 -h /var/opt/gitlab/postgresql -d template1 -c "CREATE USER gitlab" ----
STDOUT: 
STDERR: psql: could not connect to server: No such file or directory
Is the server running locally and accepting
connections on Unix domain socket "/var/opt/gitlab/postgresql/.s.PGSQL.5432"?
---- End output of /opt/gitlab/embedded/bin/psql --port 5432 -h /var/opt/gitlab/postgresql -d template1 -c "CREATE USER gitlab" ----
Ran /opt/gitlab/embedded/bin/psql --port 5432 -h /var/opt/gitlab/postgresql -d template1 -c "CREATE USER gitlab" returned 2
 
Resource Declaration:
 
# In /opt/gitlab/embedded/cookbooks/cache/cookbooks/gitlab/recipes/postgresql.rb
 
153:   execute "create #{sql_user} database user" do
154:     command "#{bin_dir}/psql --port #{pg_port} -h #{postgresql_socket_dir} -d template1 -c \"CREATE USER #{sql_user}\""
155:     user postgresql_user
156:     # Added retries to give the service time to start on slower systems
157:     retries 20
158:     not_if { !pg_helper.is_running? || pg_helper.user_exists?(sql_user) }
159:   end
160: 
 
Compiled Resource:
 
# Declared in /opt/gitlab/embedded/cookbooks/cache/cookbooks/gitlab/recipes/postgresql.rb:153:in `block in from_file'
 
execute("create gitlab database user") do
action [:run]
retries 20
retry_delay 2
default_guard_interpreter :execute
command "/opt/gitlab/embedded/bin/psql --port 5432 -h /var/opt/gitlab/postgresql -d template1 -c \"CREATE USER gitlab\""
backup 5
returns 0
user "gitlab-psql"
declared_type :execute
cookbook_name "gitlab"
recipe_name "postgresql"
not_if { #code block }
end
```

有两个已知的权限问题，在 [official documentation](https://docs.gitlab.com/omnibus/docker/) 和 [#9611](https://github.com/gitlabhq/gitlabhq/issues/9611) 中描述。为了解决这两个问题，执行：
```sh
docker exec -it gitlab update-permissions
docker exec gitlab chown -R gitlab-redis /var/opt/gitlab/redis
```

然后重启 gitlab 容器：
```sh
docker restart gitlab
```

### 8.然后我们需要检查数据库迁移是否成功以及避免#3255 问题。
登录到 gitlab 容器中的 shell：

```sh
docker exec -t -i gitlab /bin/bash
```

检查数据库迁移状况：
```bash
sudo gitlab-rake db:migrate:status
```

如果所有状态都显示为 up 状态，是没问题的。如果发现 down 状态，类似下面这样：
```log
up     20150920161119  Add line code to sent notification
down    20150924125150  Add project id to ci commit
down    20150924125436  Migrate project id for ci commits
```
就必须重新手动执行数据库 migration 命令：


```sh
sudo gitlab-rake db:migrate
```
当执行结束时，重新检查数据库 migrate 状态，确保所有状态都为 up：

```sh
sudo gitlab-rake db:migrate:status
```

### 9.继续在容器内部的 bash shell 上，重新配置 gitlab:
```sh
sudo gitlab-ctl reconfigure
```

然后检查是不是所有内容都正常运行：
```sh
sudo gitlab-rake gitlab:check
```

### 10.如果一切正常，执行下面的命令：

```sh
sudo gitlab-rake gitlab:env:info RAILS_ENV=production
```

会返回类似下面这样的结果：

```log
System information
System:   Ubuntu 14.04
Current User: git
Using RVM:  no
Ruby Version: 2.1.7p400
Gem Version:  2.2.5
Bundler Version:1.10.6
Rake Version: 10.4.2
Sidekiq Version:3.3.0
 
GitLab information
Version:  8.2.0
Revision: d6bcf44
Directory:  /opt/gitlab/embedded/service/gitlab-rails
DB Adapter: postgresql
URL:    http://192.168.1.1:8082
HTTP Clone URL: http://192.168.1.1:8082/some-group/some-project.git
SSH Clone URL:  git@192.168.1.1:some-group/some-project.git
Using LDAP: yes
Using Omniauth: no
 
GitLab Shell
Version:  2.6.7
Repositories: /var/opt/gitlab/git-data/repositories
Hooks:    /opt/gitlab/embedded/service/gitlab-shell/hooks/
Git:    /opt/gitlab/embedded/bin/git
```

### 11.最后，清除 Redis 的缓存
不然可能会遇到这个问题： [#3619](https://gitlab.com/gitlab-org/gitlab-foss/-/issues/3619) 或这个问题 [#3609](https://gitlab.com/gitlab-org/gitlab-foss/-/issues/3609):
```sh
sudo gitlab-rake cache:clear
```

### 12.访问测试
完成上述步骤之后，访问 http://localhost:8082 应该能正常登陆