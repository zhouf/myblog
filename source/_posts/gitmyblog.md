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

## 保存主题
之前下载的NexT主题是通过`git clone`方式安装的，其目录`themes\next`下包含有git信息，默认提交时不会提交此目录下的文件，这样会造成库中只有空`next`目录，没有目录下的文件，同步到其它电脑时就会找不到主题以及丢失配置文件。因此需要将此目录中的文件交给`blog`工程管理，删除如`.git`,`.github`,`.gitignore`等以.开头的隐藏文件，删除`test`目录，最后的目录文件如下
```
languages/
layout/
scripts/
source/
_config.yml
bower.json
gulpfile.coffee
LICENSE
package.json
README.cn.md
README.md
```
可能这也不是最小集合，未验证是否有多余的，可以通过如下命令查看`themes\next`目录下的文件是否有被纳入`git`管理
```
git status
```

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
