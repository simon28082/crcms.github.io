---
title: hexo 集成 travis ci 自动部署
date: 2019-04-07 10:14:52
tags:
    - hexo
    - travis
---

## 增加github token至travis ci
- 在`github Settings->Developer settings->Personal access tokens生成一个travis token`
![](/images/hexo-travis/generate-new-token.png, "Generate new token") 当出现如图Connection successful，则表示连接成功
- 进入`travis ci`项目`settings->Environment Variables`增加`token`值为环境变量

## 设置travis.yml
```$xslt
language: node_js
node_js: stable

install:
  - npm install

script:
  - hexo g

after_script:
  - cd ./public
  - git init
  - git config user.name "crcms"
  - git config user.email "crcms@crcms.cn"
  - git add .
  - git commit -m "Update docs"
  - git push --force --quiet "https://${GITHUB_TOKEN}@${GH_REF}" master:master

branches:
  only:
    - source
env:
 global:
   - GH_REF: github.com/crcms/crcms.github.io.git
```
