---
title: Hexo部署步骤
date: 2023-05-26 13:53:36
tags: 技术
categories: 趣味
---

> 每次部署的步骤，可按以下三步来进行

```sh
hexo clean
hexo generate
hexo deploy
```

> 一些常用命令：

```sh
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy #将.deploy目录部署到GitHub
hexo help  # 查看帮助
hexo version  #查看Hexo的版本
```
