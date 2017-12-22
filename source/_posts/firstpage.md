---
title: 开始使用Hexo了
date: 2017-12-18 12:00:16
tags:
---

之前在github上试用了一个博客系统，用Jekyll写的，还不错，只是后面一直没有更新内容，最近又重新有了这样的想法，就来关注一下，发现Hexo，看上去还不错，就来试下。

关于Hexo的使用教程，网上有很多写得不错的，我在学习这个的时候，也参考了许多别人分享的经验，下面就用我的实现过程写下这篇博客，也当作我的第一篇博客了。

## 安装Nodejs环境
Hexo是用Nodejs来生成页面文件的，所以环境中需要安装Nodejs

> 直接到[官网](https://nodejs.org/en/)下载与自己系统配置的Nodejs，进行安装即可

完成后可以命令行里查看当前版本进行验证

``` bash
C:\>node -v
v8.9.3

C:\>
```

## 安装Git环境
作为开发人员来说，这应该是一个必备的工具了，如果已经安装好了，就直接跳过这部分吧
> Windows用户可以直接下载安装[git](https://git-scm.com/download/win)
>
> Mac或Linux用户可以通过命令行直接安装，详见[Hexo文档](https://hexo.io/zh-cn/docs/)

## 安装Hexo

Hexo的安装可以直接参考[官网](https://hexo.io/)，这才是最权威的，下面把官网上几个简单的步骤放在这里。

首先是在安装Hexo，在命令提示符下用如下命令即可安装好
```bash
npm install hexo-cli -g
```

现在就可以在本机创建一个博客了，用如下命令就可以实现在当前目录下创建一个blog目录，用于存放博客页面内容
``` bash
npm install hexo-cli -g
hexo init blog
cd blog
npm install
hexo server
```

* 第一条是安装Hexo环境
* 第二条语句是初始化blog目录
* 第三条语句是进入到blog目录
* 第四条语句是安装相关的依赖包和库
* 第五条语句是启动本地服务

启动好后就可以在 http://localhost:4000 查看默认的模板页面了，如果有看到默认页面，说明安装完成，在所看到的模板页面中，也是一个说明文档，如何开始写一篇博客的说明和步骤，看上去操作很简单，接下来就可以开始属于你的创作了。


## 开始写一篇自己的博客

在博客目录的命令提示符下面连接创建新页面的语句
``` bash
hexo new starthexo
```

会在`source\_posts\`目录下生成对应的md文件，如果标题中含有空格，可以使用双引号，在生成md文件时会以短线-的方式连接

在starthexo.md文件中写点什么吧...

写完成后就可以通过服务器查看页面了
``` bash
hexo server
```
可简写为
``` bash
hexo s
```
打开http://localhost:4000 查看到页面显示效果，调整好之后可以用如下命令生成静态页面
``` bash
hexo generate
```
可简写为
``` bash
hexo g
```

此命令会在`public`目录下生成静态页面文件，如果需要发布到github或是其它可以接收的服务器上，就轮到deploy命令出场了

> 稍等，还没有准备好，还得化个妆


## 资源文件处理

如果博客中有包含图片，下载文件等外部资源，可以配置启用静态资源目录，以方便Hexo生成相应的链接，修改`_config.yml`文件，将`# Writing`部分中的`post_asset_folder`修改为true
```
post_asset_folder: true
```
之后在使用`hexo new`创建新的文档时，会在`source\_posts\`自动创建一个同名的目录，把相应的资源图片或文件放入到该目录即可。

在写这篇文档时还没有开启，所以没有同名目录，手动创建一个就好了。把图片文件`pic1.png`放入，在页面中写入
```
![这里是图片的注释]('firstpage/pic1.png')
```
这是Markdown插入图片的语法，看上去没有什么问题，可在浏览器页面里看不到图片，如果希望Hexo生成页面时能正确处理这样描述的图片资源，需要安装一个`hexo-asset-image`插件，在博客根目录执行
```
npm install https://github.com/CodeFalling/hexo-asset-image --save
```
之后再用`hexo g`就可以正确生成图片路径了，不过Hexo官网推荐的使用标签插件而不是Markdown
```
{% asset_img pic1.png 这里是一张图片哟 %}
```

这里应该可以看到图片的
{% asset_img pic1.png 这里是一张图片哟 %}

## 修改页面主题

如果觉得默认的主题不够有特色，可以更换其它主题，比如[NexT](http://theme-next.iissnan.com/)，在站点目录下可以通过`git`命令直接下载
```bash
cd blog
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

下载好NexT主题后，可以在配置文件`_config.yml`中修改主题配置，将默认的`landscape`改为`next`
```
theme: next
```
NexT提供了三种不同的主题风格，可以在其主配置文件`themes\next\_config.yml`中进行修改
```
#scheme: Muse
#scheme: Mist
scheme: Pisces
```
* Muse 默认 Scheme，这是 NexT 最初的版本，黑白主调，大量留白
* Mist - Muse 的紧凑版本，整洁有序的单栏外观
* Pisces - 双栏 Scheme，小家碧玉似的清新
* Gemini - 我下载的版本中出现了第四种，测试后发现，和Pisces类似，只是稍宽一点，应该是为了适应宽屏的效果

可以在NexT主页上看到这三种不同的风格，也可以修改之后直接查看网页效果，NexT详细的配置项可参考[官网文档](http://theme-next.iissnan.com/getting-started.html)

可以通过修改配置文件`_config.yml`中的其它配置项，如`title`,`author`,`language`等来实现修改整个站点的相关信息


## 发布到Github
在github发布网页可以参考[GitHub Pages](https://pages.github.com/)，在github中创建一个`<youraccount>.github.io`的repository

使用Hexo提交到git需要安装[hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git)
```
npm install hexo-deployer-git --save
```
当前电脑之前已经通过git访问过github，有github登录信息，可以直接登录，在`_config.yml`中配置`deploy`信息即可，当然也可以发布到其它可以接受静态页面的服务器上，具体`deploy`配置信息详见[官网](https://hexo.io/zh-cn/docs/deployment.html)，当前配置信息如下
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/zhouf/zhouf.github.io.git
  branch: master
```

完成上述操作后，重新生成页面后就可以发布页面到github了，命令如下
```
hexo g
hexo deploy
```
等到上传完成了，就可以在 https://zhouf.github.io 查看写好的文档了，如果有自己的域名，绑定到自己的域名访问，那就更酷了。

好了，今天的工作就到此了，第一篇博客终于看到了，接下来希望可以在这个平台上好好写。
