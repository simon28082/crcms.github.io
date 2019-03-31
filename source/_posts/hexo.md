---
title: hexo
date: 2019-03-31 07:02:57
tags:
  - hexo
  - next
categories:
  - 其他
---

放弃wordpress，集成hexo
---------------------

## 原因
经常迁移太麻烦，不如直接`github`+`hexo`

## 部署

### 安装Hexo，git deploy
```bash
npm install -g hexo-cli
```
详细教程请参考官方链接 [hexo](https://hexo.io "hexo")

### 创建github
- 创建名称为`*.github.io`仓库
- 在`settings`中设置`github pages`
- 设置`custom domain`
- 在域名管理中解析CNAME，指向你的*.github.io域名

### 发布内容
```bash
hexo d -g
```

## 注意：
- 在仓库目录中增加`CNAME`文件，内容为你的`custom domain`
- `hexo` `git`提交时也需要注意，具体请参考[git部署](https://hexo.io/zh-cn/docs/deployment#Git "hexo git 部署")
