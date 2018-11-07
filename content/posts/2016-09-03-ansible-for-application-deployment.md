---
title: "使用 Ansible 自动化部署服务"
date: 2016-09-03T15:33:37+08:00
categories: ["devops"]
tags: ["deployment", "ansible", "php"]
---
之前公司的项目主要使用 `PHP` 来处理基础的业务逻辑，其中使用了 `Yii2` 框架来处理核心的业务逻辑，部分业务逻辑以异步方式放入队列，由 `PHP CLI` 方式运行的服务来消费队列内容。

最开始的时候，每次有新版本需要上线，都要手动做一系列操作。

1. 停止队列消费守护程序。
2. 执行 `mysql` 数据库升级。
3. `git` 拉取 `release` 分支最新的代码。
4. 重新启动队列消费守护进程。

由于这个项目比较小，小修改会比较多，而且修改完就有线上更新的需求，所以上面这些操作的频率比较高。由于重复性高，都是手动操作效率低，查找了解了一些自动化工具后，选定的 Ansible 来提高效率。

<!--more-->

### 什么是 [Ansible][ansible]

Ansible 的官方标题：

> Ansible is Simple IT Automation

Ansible 是一个可简化工作的自动化 IT 工具。

Ansible 的官方介绍：

> Ansible is the simplest way to automate apps and IT infrastructure. Application Deployment + Configuration Management + Continuous Delivery.

`Ansible` 能以最简洁的方式使 IT 基础服务自动化。例如应用服务的部署、配置管理以及持续交付。

### 使用 Ansible

关于 Ansible 的使用介绍，可以查看[《Ansible中文权威指南》][ansible-doc].

### 实战

我这里使用 [`playbooks`][ansible-doc-playbooks] 来配置要操作的步骤。

例如 `example.yml`：

```yaml
---
- hosts: example.prod
  remote_user: root
  gather_facts: true
  vars:
    - app_build: "{{ansible_date_time.epoch}}"
    - base_dir: "/var/www/example"
    - app_dir: "{{base_dir}}/{{app_build}}"
    - git_url: "git@git.coding.net:example-user/example.git"
    - git_repo_key: "/var/www/.ssh/id_rsa"
    - nginx_conf_file: "/etc/nginx/sites-available/example"
    - storage_path: "{{base_dir}}/storage"
  tasks:
    - debug: msg="App Build {{app_build}}"
    - name: clone git repo
      git: depth=1 dest={{app_dir}} repo={{git_url}} key_file={{git_repo_key}} version=develop accept_hostkey=yes
    - name: update app env file
      template: dest="{{app_dir}}/.env" src=env
    - name: softlink uploads
      file: src="{{storage_path}}/uploads" dest="{{app_dir}}/public/uploads" state=link
    - name: softlink runtime
      file: src="{{storage_path}}/runtime" dest="{{app_dir}}/runtime" state=link
    - name: install composer plugin
      composer: working_dir={{app_dir}} command=require arguments="fxp/composer-asset-plugin:~1.1.1"
    - name: install composer vendor
      composer: working_dir={{app_dir}}
    - name: install bower libraries
      bower: path={{app_dir}}
    - name: change file mode
      file: path={{app_dir}} state=directory group=www-data owner=www-data recurse=yes
    - name: stop lastest service
      shell: sh {{base_dir}}/latest/scripts/stop-service.sh
    - name: softlink latest
      file: src="{{app_dir}}" dest="{{base_dir}}/latest" state=link group=www-data owner=www-data
    - name: upgrade database
      shell: php {{base_dir}}/latest/yii db/upgrade
    - name: start lastest service
      shell: nohup sh {{base_dir}}/latest/scripts/start-service.sh > /dev/null 2>&1
    - name: update nginx conf
      template: dest={{nginx_conf_file}} src=nginx.conf
    - name: restart nginx
      service: name=nginx state=reloaded
```

上面这个剧本很多操作都是我们这个项目会用到的：

* 首先，我们在剧本中定义了一个变量 `app_build`，即操作剧本时的时间戳。
* 每次部署新代码，我们用当时服务器时间戳为目录名（`app_dir`），从 `git` 拉取最新代码。
* 项目中用到了 [`dotenv`][php-dotenv] 来处理一些区分生产环境和测试环境的环境变量，这里要把 `app_build` 这个变量放入环境变量中，方便在 `PHP` 中做一些版本相关的处理（如以 `app_build` 参与计算的缓存键等）。
* 然后将一些公用的文件夹软链接到目标路径，如日志文件夹、上传文件等。
* 之后会使用 `composer`，`bower` 安装项目依赖库，设置文件权限。
* 停用上文说到的一些队列消费服务，将 `{{base_dir}}/latest` 路径软链接到 `app_dir`。之后 `latest` 目录即为最新一次 `app_build` 目录。
* 升级数据库后，更新 `nginx` 配置文件，reload nginx。

这样以后每次部署新代码，只需要终端输入一行命令：

```sh
ansible-playbook exmaple.yml
```

[ansible]: https://www.ansible.com/ "Ansible"
[ansible-doc]: http://www.ansible.com.cn/ "Ansible Documentation"
[ansible-doc-playbooks]: http://www.ansible.com.cn/docs/playbooks.html "Ansible Documentation"
[php-dotenv]: https://github.com/vlucas/phpdotenv "PHP dotenv"
