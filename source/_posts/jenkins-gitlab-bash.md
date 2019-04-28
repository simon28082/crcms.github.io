---
title: Gitlab+Jenkins+Bash 搭建多流水线自动部署
date: 2019-04-26 07:21:14
tags:
---

![jenkins pipeline](/images/jenkins-gitlab-bash/1.png "jenkins pipeline")


## 简介
因为涉及到的子项目非常多，为了方便统一更新部署测试等相关运行代码，所以`Jenkinsfile`在本项目中只是起到一个阶段挂载的作用，大部分部署代码使用的是`Bash`。
此方法并不适合大规模集群部署。

## Jenkins安装

### 安装启动
```
docker-compose up -d jenkins
```
> 本环境中使用的是laradock中Jenkins进行安装，如果非`Docker`请参考官方文档进行安装

### 增加Jenkins插件
在`系统管理`->`插件管理`中安装`Blue Ocean`和`GitLab`插件

### 在项目根目录新建Jenkinsfile

关于Jenkinsfile相关文档，请参考[官网](https://jenkins.io/zh/doc/book/pipeline/)




### 增加单独Build的bash项目，于用执行具体步骤流程

因公司内部使用，So ......


### 创建流水线

打开 `Open Blue Ocean` 选择一个创建流水线项目，当然需要加上gitlab的ssk-key，否则无法进行拉取

### Gitlab配置webhook
- `Jenkins` -> `系统配置` -> `Gitlab` 进行配置测试连接Gitlab
- `Jenkins项目` -> `选择分支` -> `view configure` -> `构建触发器` 找到gitlab webhook选项
- `gitlab管理` -> `设置` -> `集成` 增加webhook url

## 注意

### 关于自动部署
使用的是`scp`+`ssh`+`ssh-key`+`hook.sh`来实现的自动远程部署以及部署后需要执行的命令
- 服务器增加远程部署帐号（如果需要`sudo`，则要开启`sudo`免密，在`sudoers.d`中增加`account ALL=(ALL:ALL) NOPASSWD: ALL`，具体权限以项目需求为准）
- 使用`scp`上传
- `ssh` 登录，并执行常规操作
- `hook.sh` 进行部署的后期操作

> 此方法实现简单，但安全性并不是特别高。
    所以必须要保证ssh-key不可丢失，以及最好限制ssh的登录IP

> 关于部署失败进行回滚操作，暂未实现，不过可以提供一个思路参考：
备份部署前的代码，失败则解压上一次的代码进行覆盖回滚，执行`hook.sh`即可

### 关于Bash执行及异常停止
默认Bash并不会异常停止，以及需要打印变量输出，以及管道错误处理，需要在Bash头上加上如下代码：
```
set -euxo pipefail
```
> 具体请参考：[Bash 脚本 set 命令教程](http://www.ruanyifeng.com/blog/2017/11/bash-set.html?utm_source=tool.lu)

### Jenkins Dockfile 执行时加载路径问题
关于使用`Dockfile`作为`agent`环境，因为多子项目共用，并未放在单独的项目下，在公用`build`其调用路径相当于以当前项目作为根目录，层层往上，如下：

建议使用-v挂载一些常用缓存目录，如`composer/.cache`等

```
agent {
        dockerfile {
            filename "../../build/php/Dockerfile"
            dir "../../build/php"
            additionalBuildArgs ''
            args '-v /tmp/composer:/root/.composer'
        }
}
```

### Gitlab webhook 访问本地网络
`管理区域` -> `设置` -> `Outbound requests` -> `允许钩子和服务访问本地网络`

## 模板

### 本地Jenkinsfile一个简单模板
```
pipeline {
    agent {
        dockerfile {
            filename "../../build/php/Dockerfile"
            dir "../../build/php"
            additionalBuildArgs ''
            args '-v /tmp/composer:/root/.composer'
        }
    }

    environment {
        EX_JENKINS_HOME = "${JENKINS_HOME}"
        EX_CURRENT_APP = "project-name"
        EX_CURRENT_ENV = "${BRANCH_NAME}"
        DO_RELEASE = 'yes'
    }

    stages {
        stage('Prepare') {
            steps {
                echo 'Prepare'
            }
        }

        stage('Build') {
            steps {
                sh "${JENKINS_HOME}/build/scripts/build.sh php"
            }
        }

        stage('Test') {
            steps {
                sh "${JENKINS_HOME}/build/scripts/test.sh php"
            }
        }

        stage('Release?') {
            when {
                branch 'product'
            }
            steps {
                script {
                    env.DO_RELEASE = input "Should we continue?"
                }
            }
        }

        stage('Deploy') {
            when {
                environment name: 'DO_RELEASE', value: 'yes'
            }
            steps {
                sh "${JENKINS_HOME}/build/scripts/deploy.sh php"
            }
        }
    }
}
```

### PHP Dockerfile agent
```
FROM ubuntu:latest

MAINTAINER simon

USER root

ARG ARG_TIMEZONE=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/${ARG_TIMEZONE} /etc/localtime && echo ${ARG_TIMEZONE} > /etc/timezone

RUN apt-get update -yqq \
    && apt-get install -y sudo \
    && apt-get install -y php7.2-dev \
    && apt-get install -y php7.2-mbstring \
    && apt-get install -y php7.2-curl \
    && apt-get install -y php7.2-mysql \
    && apt-get install -y php7.2-zip \
    && apt-get install -y php7.2-gd \
    && apt-get install -y php7.2-bcmath \
    && apt-get install -y composer


#### php pecl #############################
RUN pecl channel-update pecl.php.net

ARG ARG_PHP_MODS_AVAILABLE_PATH=/etc/php/7.2/mods-available
ARG ARG_PHP_CONF_PATH=/etc/php/7.2/cli/conf.d

# mongodb
RUN pecl install mongodb \
    && echo "extension=mongodb.so" > ${ARG_PHP_MODS_AVAILABLE_PATH}/mongodb.ini \
    && ln -s ${ARG_PHP_MODS_AVAILABLE_PATH}/mongodb.ini ${ARG_PHP_CONF_PATH}/20-mongodb.ini

# redis
RUN pecl install igbinary \
    && echo "extension=igbinary.so" > ${ARG_PHP_MODS_AVAILABLE_PATH}/igbinary.ini \
    && ln -s ${ARG_PHP_MODS_AVAILABLE_PATH}/igbinary.ini ${ARG_PHP_CONF_PATH}/20-igbinary.ini \
    && pecl install redis \
    && echo "extension=redis.so" > ${ARG_PHP_MODS_AVAILABLE_PATH}/redis.ini \
    && ln -s ${ARG_PHP_MODS_AVAILABLE_PATH}/redis.ini ${ARG_PHP_CONF_PATH}/20-redis.ini

# swoole
RUN pecl install swoole \
    && echo "extension=swoole.so" > ${ARG_PHP_MODS_AVAILABLE_PATH}/swoole.ini \
    && ln -s ${ARG_PHP_MODS_AVAILABLE_PATH}/swoole.ini ${ARG_PHP_CONF_PATH}/20-swoole.ini

#### clear #############################
# clear Ubuntu image will run automatically apt-get clean,so is not need apt-get clean
RUN apt-get -y autoremove

WORKDIR "/data"

CMD ["/bin/bash"]
```