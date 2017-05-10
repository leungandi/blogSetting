---
title: 搭建hexo+gitHub博客
date: 2017-05-09 14:10:14
tags:
categories: 编程
---

## 准备环境
### 安装node.js
点击进入[node.js官网](https://nodejs.org/en/download/)  
![image](F:\blog教程素材\nodejs\nodejs官网下载.jpg)  
下载完成后，双击打开  
![image](F:\blog教程素材\nodejs\nodejs安装.jpg)  
一路next,安装完成。  

 **安装完成，让我们来检测一下node.js是否安装成功吧**   
![image](F:\blog教程素材\nodejs\nodejs_version.jpg)  
出现上图所示,恭喜你!安装成功了....  

---

### 安装git 
git使用一般有两种方式，一种是图形化界面（GUI），另一种是通过命令行，我们这里要使用的是后者，[点击这里](https://git-scm.com/downloads)进入git的下载网站下载git的安装包  
![image](F:\blog教程素材\git\index.jpg)

--- 

### 安装Hexo

Hexo是搭建博客的核心,[点击进入hexo首页](https://hexo.io/)

![image](F:\blog教程素材\hexo\index.jpg)  
- **首先创建博客本地的路径**  
![image](/images/Hexo_blog/hexo/1_newdir.jpg)  
比如：我这里使用的是e:\my_blog文件夹  
- **下载安装hexo**  
```
$ npm install -g hexo-cli

```
**安装完成，让我们来检测一下hexo是否安装成功吧**   
输入：
```
$ hexo -version

```
![image](/images/Hexo_blog/hexo/2_version.jpg)  
出现上图所示,恭喜你!安装成功了....  

---

## 配置博客

### hexo初始化  
```
//我们在刚开始建好的博客文件夹下执行(我这里使用的是e:\my_blog)
$ hexo init

```

等等init完成后,继续执行以下指令  
```
//node.js的命令，根据博客既定的dependencies配置安装所有的依赖包
$ npm install

```

初始化完成后,目录如下：  
![image](/images/Hexo_blog/hexo/4_dir.jpg) 

### hexo本地发布  

到这里我们已经开始运行博客了,是不是已经有点迫不及待了,让我们先看以下运行效果  

```
//本地发布
$ hexo s

```
![image](E:\myBlog\source\image\Hexo_blog\hexo\5_server.jpg)  
使用浏览器打开[localhost:4000](http://localhost:4000)，可以看到如下的博客首页界面  
![image](/images/Hexo_blog/hexo/6_index.jpg)

对于博客的配置，我们需要用到_config.yml文件，下面是该文件的默认参数信息：
```
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/
# 网站配置
# Site
title: Hexo # 网站标题
subtitle: # 网站副标题
description: # 网站描述
author: John Doe # 您的名字
language: # 网站使用的语音
timezone: # 网站时区。Hexo 默认使用您电脑的时区。时区列表。比如说：America/New_York, Japan, 和 UTC 。
# 网址配置
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://yoursite.com # 网址
root: / # 网站根目录
permalink: :year/:month/:day/:title/ # 文章的永久链接格式
permalink_defaults: # 永久链接中各部分的默认值
# 目录配置
# Directory
source_dir: source # 资源文件夹，这个文件夹用来存放内容,我们写的文章就存放在这里
public_dir: public # 公共文件夹，这个文件夹用于存放生成的站点文件。
tag_dir: tags # 标签文件夹
archive_dir: archives # 归档文件夹
category_dir: categories # 分类文件夹
code_dir: downloads/code # Include code 文件夹
i18n_dir: :lang # 国际化（i18n）文件夹
skip_render:
# 文章配置
# Writing
new_post_name: :title.md # 新文章的文件名称
default_layout: post # 预设布局
titlecase: false # 把标题转换为 title case
external_link: true # 在新标签中打开链接
filename_case: 0 # 把文件名称转换为 (1) 小写或 (2) 大写
render_drafts: false # 显示草稿
post_asset_folder: false # 启动 Asset 文件夹
relative_link: false # 把链接改为与根目录的相对位址
future: true # 显示未来的文章
highlight: # 代码块的设置
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:

# 分类 & 标签
# Category & Tag
default_category: uncategorized # 默认分类
category_map: # 分类别名	
tag_map: # 标签别名
# 日期 & 时间格式
# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: landscape #主题配置
# 部署设置
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type:


```

## 博客发布
我们可以把博客发布到github，这样别人就可以看到我们写的博客了，下面我们就一起来发布吧!  
**重要**:*首先你要有个gitHub账号,如果没有,请[点这里](https://github.com/)注册，具体的注册过程就不在这里描述。*  
- 配置仓库  
![image](/images/Hexo_blog/git/1_index.jpg)
登录账号后，在Github页面的右上方选择New repository进行仓库的创建。
![image](/images/Hexo_blog/git/2_create.jpg)  
在仓库名字输入框中输入：
```
xxx.github.io//xxx表示你的昵称

```
然后点击==Create repository==来完成创建  

### 配置_config.yml  
我们在博客目录中找到_config.yml配置文件，然后找到Deployment的配置
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git //type类型为git
  repo: https://github.com/leungandi/xxx.github.io.git //这里填写你刚刚创建的仓库地址
  branch: master //这里填写master分支

```

### 发布运行  
到此为止,我们可以使用hexo指令来上传博客到gitHub

```
$ hexo -g //生成静态文件

$ hexo -d //部署完整(就是发布到我们gitHub仓库)
    
```

等待上传完成,我们就可以使用gitHub的域名来访问我们的博客了!!!


