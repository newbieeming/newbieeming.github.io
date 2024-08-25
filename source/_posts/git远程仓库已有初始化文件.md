---
title: git远程仓库已有初始化文件
date: 2024-01-30 20:59
categories:
  - Git
---

### git远程仓库已有README等初始化文件

> 本地项目未commit

```shell
git init
git remote add origin xxx(项目地址)
git pull origin master
```

> 本地项目已commit

```shell
git remote add origin xxx(项目地址)
git pull origin master --allow-unrelated-histories
```