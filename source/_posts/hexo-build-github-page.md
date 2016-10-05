title: Hexo搭建静态博客
date: 2015-09-19 10:49:20
category: Hexo
tags: 
	- hexo
	- githubpages
	- blog
---

很久很久之前就想搞个自己的博客，由于种种原因(主要是太懒)，每次都是半途而废。最开始试过用`Django` 来搭建，弄了个半成品，加上工作忙(就是懒)，也没弄成。直到后来看到 [马老师](http://pinkyjie.com) 的博客，才知道可以在`GitHub`上搭建静态博客。

<!--more-->

开始用的 `Jekyll` 试了几个主题都不太满意，然后发现了 [`Next Theme`]() 太赞，果断换到 `Hexo` 了，搭建也比较简单。

## 安装Hexo
依照 [官网](https://hexo.io/zh-cn/) 一步一步装就可以了

如果安装出现卡主或者一直没反应，如下

~~~
$ npm install hexo-cli -g
> fsevents@0.3.8 install /usr/local/lib/node_modules/hexo-cli/node_modules/hexo-fs/node_modules/chokidar/node_modules/fsevents
> node-gyp rebuild
~~~

这种多半是网络原因，将 `npm` 源换成淘宝镜像

~~~
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
~~~
然后执行安装

~~~
$ cnpm install hexo-cli -g
~~~
如果依然报错 使用以下命令试试

~~~
$ cnpm install hexo-cli --no-optional
~~~

## 新建 Blog
新建目录执行如下命令

~~~
$ hexo init
$ cnpm install
$ hexo generate (g)
$ hexo server (s)
~~~
本地环境就可以了，然后打开 localhost:4000 就可以看到了

## 部署到 GitHub
站点配置文件 `_config.yml` 配置部署地址

~~~
deploy:
  type: git
  repository: https://github.com/jacob110/jacob110.github.io.git
  branch: master
~~~
然后执行部署命令

~~~
$ hexo clean
$ hexo generate (g)
$ hexo deploy (d) -m "commit message"
~~~
commit message 为可选，部署成功就可以使用 username.github.io 访问了。

如果是新注册的 `GitHub` 账号，记得把邮箱验证下，否则会访问404.
## 使用 Next 主题
非常棒的 `Hexo` 主题，文档 [Next](http://theme-next.iissnan.com/) 写的非常详细，依据文档安装配置都很简单。

## 常用命令

~~~
hexo clean #清除 public 文件夹
hexo new(n) "postName" #新建文章
hexo new(n) page "pageName" #新建页面
hexo generate(g) #生成静态页面至public目录
hexo server(s) #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy(d) #将.deploy目录部署到GitHub
~~~



