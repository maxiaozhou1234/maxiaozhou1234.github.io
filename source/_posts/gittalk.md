---
title: 【其他】gittalk 配置问题
date: 2020-02-11 14:58:05
categories: default
tags: [gittalk,hexo]
---

使用 gittalk 为 Hexo 添加评论功能，遇到问题及解决方法
### 1.申请及配置
#### 1.1 注册 gittalk
可通过 [Register a new OAuth application](https://github.com/settings/applications/new) 进行注册。如果已经注册过，可以在 github 首页点击头像下拉，“Settings -- Developer settings -- OAuth Apps” 查看你的app，选择你注册的 app 进行再次编辑。
#### 1.2 配置填写
* Application name： 应用名称，随意
* Homepage URL： 网站URL，对应自己博客地址
* Application description ：描述，随意
* Authorization callback URL：# 网站URL，博客地址就好

* 点击注册，页面会出现其中Client ID和Client Secret在后面的配置中需要用到

如我的 gittalk 填写如下：
```
 Application name： CommentApp
 Homepage URL： https://maxiaozhou1234.github.io # 网站URL，对应自己博客地址
 Application description ：repo # 描述，随意
 Authorization callback URL：https://maxiaozhou1234.github.io # 网站URL，博客地址就好，如果有独立域名，可填写你的域名用于跳转
```

#### 1.3 在主题的 _config.yml 进行配置
```
#Cmments
comment:
  gitalk:
    enable: true ## 开启gitalk
    owner: ## GitHub的用户名
    repo: ## 此评论存放的GitHub仓库
    client_id: ## 复制刚才生成的clientID，例如. 75752dafe7907a897619
    client_secret: ## 复制刚才生成的clientSecret，例如. ec2fb9054972c891289640354993b662f4cccc50
    admin: ## Github的用户名
	id: location.pathname
    language: zh-CN ## Language
    pagerDirection: last # Comment sorting direction, available values are last and first.
```
主题的配置，可以参考[【Hexo博客折腾】BlueLake博客主题的详细配置](https://chaooo.github.io/article/20161229.html)

### 2. 搭建过程遇到的问题
#### 2.1 评论区显示 Error: Not Found

遇到 Error: Not Found，这个问题是主题 _config.yml 中 gittalk 配置中 repo 填写错误，修改为你的博客主页即可，如我的博客配置如下：
> repo： maxiaozhou1234.github.io

#### 2.2 博客评论登录跳转到首页问题

申请配置是填写的`Homepage URL`,`Authorization callback URL` 不正确导致，第一个填博客首页，第二个是授权回调页面，因为我没有使用独立的域名，所以两个都填博客首页，如下
> Homepage URL：https://maxiaozhou1234.github.io
> Authorization callback URL：https://maxiaozhou1234.github.io

如果你是有自己独立的域名，将 `Authorization callback URL` 填写为你的域名，前提是你已完成了域名的绑定，还有注意 https 和 http 区别，需完全一致。

参考文章：[解决配置gitalk插件后初始化登录时跳转回首页](https://blog.csdn.net/w47_csdn/article/details/88858343)
