---
title: 将主页源文件托管到Github
date: 2017-12-18 15:19:37
tags:
---
把博客的源文件放在Github上托管，以方便在不同的地方进行修改，这个我还没有找其它的方法，先用传统的托管工程的想法放上去再说，后面如果发现有更先进的，再做改进。

## 创建库
在github上创建一个库，用于存放当前工程文件，如 `myblog`

## 关联库
Hexo在对目录进行初始化只，已经创建好一个 `.gitignore` 文件，此时只需要在此目录上用 `git`进行操作即可，将本地目录与远程库进行关联，命令如下
```bash
cd blog
git init
git remote add origin https://github.com/zhouf/myblog.git
```
关联后，可以通过
```
git remote -v
```
进行查看

## 提交文件
将本地文件提交到库中，执行如下操作
```bash
git add .
git commit -m "first commit to myblog"
git purh -u orogin master
```
等待操作完成即可。

### 版本过低处理
如果上传到github后，有提示uglify-js版本<2.6.0，可以尝试通过以下方式处理

删除本地 `package-lock.json`后重新提交
```bash
cd blog
del package-lock.json
git add .
git commit -m "delete package-lock.json"
git push origin master
```
