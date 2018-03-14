---
title: Github-Pages结合Hexo搭建博客
date: 2016-01-12 17:20
tags:
 - Github
 - hexo
---

###  Github-Pages结合Hexo搭建博客

### 一 : 环境准备

- 安装Git
- 安装Node.js

### 二 : Hexo安装
```
npm install -g hexo-cli
常用命令：
hexo help #查看帮助
hexo init #初始化一个目录
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo generate #生成网页，可以在 public 目录查看整个网站的文件
hexo server #本地预览，'Ctrl+C'关闭
hexo deploy #部署.deploy目录
hexo clean #清除缓存，**强烈建议执行命令前先清理缓存，部署前先删除 .deploy 文件夹**
简写：
hexo n == hexo new
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy

```
### 三 ：Setup
```
$ hexo init folder
$ cd folder
$ npm install

hexo目录结构
├── .deploy #需要部署的文件
├── node_modules #Hexo插件
├── public #生成的静态网页文件
├── scaffolds #模板
├── source #博客正文和其他源文件，404、favicon、CNAME 都应该放在这里
 ├── _drafts #草稿
 └── _posts #文章
├── themes #主题
├── _config.yml #全局配置文件
└── package.json

```
### 四 ：设置_config.yml
```
# Hexo Configuration
## Docs: http://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/
# Site #站点信息
title:  #标题
subtitle:  #副标题
description:  #站点描述，给搜索引擎看的
author:  #作者
email:  #电子邮箱
language: zh-CN #语言
# URL #链接格式
url:  #网址
root: / #根目录
permalink: :year/:month/:day/:title/ #文章的链接格式
tag_dir: tags #标签目录
archive_dir: archives #存档目录
category_dir: categories #分类目录
code_dir: downloads/code
permalink_defaults:
# Directory #目录
source_dir: source #源文件目录
public_dir: public #生成的网页文件目录
# Writing #写作
new_post_name: :title.md #新文章标题
default_layout: post #默认的模板，包括 post、page、photo、draft（文章、页面、照片、草稿）
titlecase: false #标题转换成大写
external_link: true #在新选项卡中打开连接
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
highlight: #语法高亮
  enable: true #是否启用
  line_number: true #显示行号
  tab_replace:
# Category & Tag #分类和标签
default_category: uncategorized #默认分类
category_map:
tag_map:
# Archives
2: 开启分页
1: 禁用分页
0: 全部禁用
archive: 2
category: 2
tag: 2
# Server #本地服务器
port: 4000 #端口号
server_ip: localhost #IP 地址
logger: false
logger_format: dev
# Date / Time format #日期时间格式
date_format: YYYY-MM-DD #参考http://momentjs.com/docs/#/displaying/format/
time_format: H:mm:ss
# Pagination #分页
per_page: 10 #每页文章数，设置成 0 禁用分页
pagination_dir: page
# Disqus #Disqus评论，替换为多说
disqus_shortname:
# Extensions #拓展插件
theme: landscape-plus #主题
exclude_generator:
plugins: #插件，例如生成 RSS 和站点地图的
- hexo-generator-feed
- hexo-generator-sitemap
# Deployment #部署，将 lmintlcx 改成用户名
deploy:
  type: git
  repo: 刚刚github创库地址.git
  branch: master

```
**注意**

    配置文件的冒号“:”后面有一个空格

    repo: 刚刚github创库地址.git 如：https://github.com/kriry/kriry.github.io.git


### 五 ：发布到Github
```
1: 生成SSH密钥
ssh-keygen -t rsa -C “你的邮箱地址”，按3个回车，则密码为空。
在～.ssh下，得到两个文件id_rsa和id_rsa.pub。
2： 在GitHub上添加SSH密钥
打开id_rsa.pub，复制全文。 https://github.com/settings/ssh ，Add SSH key，粘贴进去。
3： 部署：
$ hexo server  //可以先本地预览看看,浏览器访问：http://localhost:4000
$ hexo generate  //重新生成静态博客的所有内容
$ npm install hexo-deployer-git --save  //安装git部署插件
$ hexo deploy  //发布到Github,访问：http://kriry.github.io
每次发表新文章部署按这样的流程：
$ hexo clean
$ hexo generate
$ hexo deploy

```
