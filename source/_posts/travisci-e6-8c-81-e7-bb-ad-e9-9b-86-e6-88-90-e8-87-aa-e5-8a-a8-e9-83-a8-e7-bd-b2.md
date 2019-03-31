---
title: TravisCI持续集成自动部署
tags:
  - TravisCI
  - 自动部署
url: 100.html
id: 100
categories:
  - PHP
date: 2018-06-01 07:46:01
---

注册Travis CI账号
-------------

在目录中增加.travis.yml
-----------------

具体设置见[官网文档](https://docs.travis-ci.com/ "travisCI文档")

设置和服务端的加密通信
-----------

进入当前git项目目录，运行命令

    ssh-keygen -t rsa -b 4096 -C 'crcms@crcms.cn' -f ./deploy_rsa
    

注意：使用此条命令会提示是否需要为ssh-keygen输入新密码，一定要为空（直接回车），因为travis不支持命令行输入，如果不为空则会卡住： 密匙拷贝到你的部署服务器上

    ssh-copy-id -i deploy_rsa.pub <ssh-user>@<deploy-host>
    

加入github部署
----------

打开github项目 `settings -> Deploy keys` 将 `deploy_rsa.pub`内容复制到此处即可

安装travis的命令行工具
--------------

    apt-get update
    apt-get install ruby ruby-dev
    gem install travis
    travis login
    

生成enc加密文件
---------

    travis encrypt-file deploy_rsa --add
    git add deploy_rsa.enc .travis.yml
    

在Travis中进行ssh解密操作
-----------------

`.travis.yml`文件中增加：

    addons:
      ssh_known_hosts: <deploy-host>
    
    before_deploy:
    - openssl aes-256-cbc -K $encrypted_<...>_key -iv $encrypted_<...>_iv -in deploy_rsa.enc -out /tmp/deploy_rsa -d
    - eval "$(ssh-agent -s)"
    - chmod 600 /tmp/deploy_rsa
    - ssh-add /tmp/deploy_rsa
    

注意： `$encrypted_<...>_key`需要自己替换，在 `travis encrypt-file deploy_rsa --add`命令中会自己生成一份相同名称的，替换即可 此时已可以正常编译，并且使用ssh免密通信

主要的注意点：
-------

`ssh-keygen` 生成时，不需要输入密码，直接回车即可