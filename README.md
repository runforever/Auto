<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1. 使用Ansible 做自动化运维</a>
<ul>
<li><a href="#sec-1-1">1.1. 目录设计</a></li>
<li><a href="#sec-1-2">1.2. Geting start</a></li>
<li><a href="#sec-1-3">1.3. 配置文件使用</a></li>
</ul>
</li>
</ul>
</div>
</div>


# 使用Ansible 做自动化运维

## 目录设计

    .
    ├── config
    │   ├── kernel
    │   ├── tmux
    │   └── vim
    ├── deploy
    │   └── project1
    │   └── project2
    ├── docker
    │   └── fig_config
    │       ├── common
    │       ├── mongo
    │       ├── mysql
    │       ├── nginx
    │       ├── rabbitmq
    │       └── redis
    ├── env
    ├── iptables
    ├── nginx
    └── ssh
1.  config 目录用于存放服务器的基础工具配置
2.  deploy 目录用于存放项目部署文件
3.  docker 目录用与存放docker相关的配置
4.  env 目录存放基础环境的安装配置文件
5.  iptables 目录用户存放iptables的配置文件
6.  nginx 用于存放nginx的配置
7.  ssh 用于存放ssh服务的配置

## Geting start

hosts文件用于存放管理服务器的配置，需要将执行Ansible命令的服务器的
公钥加入到管理服务器的.ssh/Authorize\_keys中。

配置好后执行

    export ANSIBLE_HOSTS=/hosts_file_direcory/hosts
    # 测试服务器连通
    ansible all -m ping
    # 如果没有问题服务器应该会返回 pong

## 配置文件使用

Ansible 配置使用是yaml格式

    # -------- basic env: init-env.yml -------------
    # install basic env tool task
    - name: install pkg
      apt: pkg={{ item  }} state=present update_cache=yes
      with_items:
        - gcc
        - g++
        - curl
        - tmux
        - python-dev
        - python-setuptools
        - python-pip
        - htop
        - zsh
        - software-properties-common
        - python-software-properties
        - libpq-dev
        - build-essential
        - libmysqlclient-dev
        - libevent-dev
        - git
        - wget
        - mc

    # install python lib task
    - name: install python pkg
      pip: name={{ item }}
      with_items:
        - fabric
        - virtualenvwrapper
        - ipython
        - ipdb
        - supervisor
        - fig

    # add user task
    - name: add deploy user
      user: name=deploy shell=/bin/zsh append=yes

    # ----- web basic env: web_env.yml -------
    ---
    - hosts: your_web_hosts
      remote_user: root

      tasks:
        - include: init-env.yml
        - name: init web dir
          file: path=/data owner=root group=root state=directory mode=0755

使用

    # 对所用服务器web_hosts执行初始化命令
    ansible-playbook web_env.yml

    # 指定服务器
    ansible-playbook web_env.yml --limit web_hosts

    # 调试(Dry Run)
    ansible-playbook web_env.yml --check
