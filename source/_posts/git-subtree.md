---
title: 通过git subtree来创建自己的子仓库
date: 2019-06-25 08:16:31
tags:
- git
- subtree
- composer
- laravel
---

## 简述
查看[`laravel/framework`](https://github.com/laravel/framework)，就会发现其实`laravel`主包是由多个子包构建。
每个子包又都可以独立使用，主包中相关子包的`commit`必须一致，同时还需要同步更新方便。如何做到这一点的呢？

## Subtree

### subtree 是什么
`git subtree` 可以实现一个仓库作为其他仓库的子仓库。或者说，可以实现一个主仓库包含多个子仓库。

### subtree 的使用

查看`git subtree --help`，发现命令如下：

```bash
git subtree add   -P <prefix> <commit>
git subtree add   -P <prefix> <repository> <ref>
git subtree pull  -P <prefix> <repository> <ref>
git subtree push  -P <prefix> <repository> <ref>
git subtree merge -P <prefix> <commit>
git subtree split -P <prefix> [OPTIONS] [<commit>]
```

## 测试

使用以下简写来代表仓库名称：

- `A`：主仓库
- `B`：子仓库

1. 首先新建一个A仓库，进入A的主目录，在A仓库中执行命令（假定所有子仓库目录都存在于`A/src`中）
```bash
git subtree add -P src/B https://github.com/B.git master
```
此时subtree则在`A`仓库中绑定了`B`仓库

2. 进入`A/src/B`
```bash
touch test
```

3. 在`A`根目录提交`commit`
```bash
git add src/B/test
git commit -m "subtree test"
git push origin master
```

那是如何同步相关`commit`到子仓库呢？`subtree push`即可
```bash
git subtree push -P src/B https://github.com/B.git master
```

即此，已经完成`A`,`B`仓库相关`commit`同步

## 优化
如果，`A/src`中有很多子仓库，每次更新完成后，手动更新N个子仓库非常让人头疼，索性搞个Bash跑一遍

```bash
#!/usr/bin/env bash
set -euo pipefail

TARGET_DIRECTORY="$(pwd)/src"

function git.branch {
  br=`git branch | grep "*"`
  echo ${br/* /}
}
BRANCH=$(git.branch)

# each所有目录
for file in `ls ${TARGET_DIRECTORY}`
do
    if [[ -d "${TARGET_DIRECTORY}/${file}" ]]; then
        echo -e "==================Current:${file}==================\n"
        echo `git subtree push -P src/${file} git@github.com:${file}.git ${BRANCH}`
        echo -e "\n"
    fi
done
```

## Composer replace
可以看到在`laravel/composer.json`中有`replace`属性
```
"replace": {
    "illuminate/auth": "self.version",
    "illuminate/broadcasting": "self.version",
    "illuminate/bus": "self.version"
    ......
}
```
那么如果是`PHP`情况下我们也可以，使用`replace`来在主包中加载子包依赖，从而提升统一版本更新的便捷性。

## 相关资料

[Composer Replace使用](https://docs.phpcomposer.com/04-schema.html#replace)
[Git subtree使用](https://git-scm.com/docs/git-pull#Documentation/git-pull.txt-subtreeltpathgt)
