---
title: GitHub 博客快速搭建
date: 2018-07-02
tags: 
categories: 搭建

---



### GitHub 博客快速搭建 ###

[TOC]

**环境:Win7**

**搭建流程**

Node.js | Hexo | Next(themes) 

**基础**

1. 首先安装Node.js 注意**Path指定位置**(影响到balabala一些配置的存放的位置,默认C盘,建议改)
2. Node.js安装好后,选择一个文件夹安装Hexo(此处为**搭建Blog的位置**)
3. 将GitHub账号的博客地址添加到Hexo的配置文件中(根目录 **_config.yml**),并将部署方式改为Git(老版本是GitHub方式)
4. 根据指令部署后即可完成基础的博客搭建,此时查看博客仓库是否上传了静态资源(如果上传了则代表搭建成功)
5. 短暂的延迟后进入博客网页即可看到搭建的博客
6. 写博客的方式-->将编写的md文章放在posts下,然后 hexo g d 部署一下即可(每次方式用这个指令就可以上传文章了,文章会被自适应解析)

**参考https://www.cnblogs.com/fengxiongZz/p/7707219.html**

-----

**进阶**

Hexo中的themes是主题设定的位置,可以选择主题,教程中选择Next的基础主题(没有追求可以凑合用( • ̀ω•́ )✧)

themes中的(**_config.yml**)是主题的配置,里面可以根据提示做自己的简单配置,包括菜单的增减和修改,布局等等,如下

```
# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:
```

根据教程和注释完成自己的布局即可

**参考http://www.cnblogs.com/fengxiongZz/p/7707568.html**

------------

**进阶加强**

完成这些想玩点花的话可以去官网或者GitHub找喜欢的themes修改使用,教程百度一大堆( • ̀ω•́ )✧

**一大波官网主题:https://hexo.io/themes/**

**第三方的插件口头整合**

1. 第三方登录(Github)
2. 评论功能
3. 站内搜索
4. 站内统计
5. 交互式背景
6. 网易云播放器插件
7. 将网站的所有图片通过url委托给第三方云图片服务器(七牛云存储10G免费)
8. ...



-----

**维护技巧**

当不能保证在同一台电脑上传文章时,可以选择master分离的方式或者将博客部署在自己的远程服务器上







