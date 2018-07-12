---
title: 部署hexo博客到github
date: 2018-07-11 15:55:19
tags:
- hexo
- github
- 部署博客
category:
- 个人博客
- 
---

# 部署hexo博客到github

#### 准备工具
&nbsp;&nbsp;&nbsp;&nbsp;[Node.js](https://nodejs.org/en/)
&nbsp;&nbsp;&nbsp;&nbsp;[Git](https://git-scm.com/downloads)
&nbsp;&nbsp;&nbsp;&nbsp;[GitHub](https://github.com/)

#### Hexo安装
在新目录下输入命令：
**npm install -g hexo-cli**

#### 创建博客文件夹
博客目录下输入命令：
**hexo init**

#### 安装依赖包
**npm install**

#### 修改配置文件
修改*_config.yml*文件
	url: http://github.com/
	root: /newblog
	permalink: :year/:month/:day/:title/
	permalink_defaults:
root为文件目录的跟目录

	deploy: # 部署相关配置
	  type: git # 使用 Git 提交
	  repository: https://github.com/xxx/xxx.github.io.git # 博客仓库地址
*repository后地址为github的项目地址*

#### 生成文件
**hexo g 或者 hexo generate**

#### 启动服务
**hexo s 或 hexo server**
浏览器输入 http://localhost:4000/(root的目录)查看

#### 发布
**hexo clean && hexo g && hexo d**

#### 注意
如果发布时候报错： ERROR Deployer not found: Git
输入命令：**npm install hexo-deployer-git --save**，重新发布即可









