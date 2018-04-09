---
title: 使用Hexo+Github搭建个人博客
tags: Blog
abbrlink: 48639
date: 2018-04-09 11:47:12
---
本篇不在介绍Hexo和Github以及Github pages相关的东西，直接介绍我搭建的每一个步骤，如果想要了解Hexo和Github这块的东西，可以自行去百度或者google了解。

# 搭建步骤 #
1. **安装node.js**  
	直接去官网下载即可。https://nodejs.org/en/,	  
	node.js在我们的博客搭建过程主要提供包管理服务，  
	Node.js的包管理器npm，是全球最大的开源库生态系统。

2. **安装Git**  
	直接官网下载即可。https://git-scm.com/download/win  
	Git是一款免费、开源的分布式版本控制系统，用于敏捷高效地  
	处理任何或小或大的项目，版本控制。

3. **进入Node.js的根目录，执行如下命令安装Hexo**  
`npm install hexo -g`

4. **在磁盘上创建一个目录用来存放Blog的所有文件**  
	例如 `E:\blog\MrByte`

5. **在步骤4中创建的目录中执行如下命令初始化Blog**  
`hexo init`
6. **安装依赖包**  
`npm install`
7. **测试-创建一个篇新的博客**  
`hexo new "My New Post"`
8. **启动服务并测试本地发布**  
执行命令:  
`hexo server`  
在浏览器输入敲回车:	`http://localhost:4000`
9. **搭桥到github**  
	
	**A、**创建一个repository，其中[repository name]的格式必须如下:  
		例如: yourname.github.io  
		其中yourname部分必须和github账号名一致，否则无效。  
	**B、**打开git bash命令行窗口，配置github账号信息,执行如下两条命令  
`git config --global user.name "yourName"`  
`git config --global user.email "yourEmail"`  
把yourName改为github账号，把yourEmail改为注册Github用的邮箱即可。   
	**C、**在git bash命令窗口中执行如下命令创建SSH  
`ssh-keygen -t rsa -C "youremail@example.com"`  
然后在生成的id_rsa.pub中复制其全部内容,双引号中的邮箱改为注册Github用的邮箱即可。  
	**D、**然后登入github，选择[Settings]-[SSH and GPG keys]-[New SSH key]  
		然后粘贴上述复制的SSH public key即可。  
	**E、**然后在git bash命令行中执行如下命令行进行验证  
`ssh -T git@github.com`  
	**F、**在blog项目根目录中找到[_config.yml]文件，使用文本编辑器打开,  
每个字段的冒号后面都有一个空格。 

	> deploy:  
	> type: git  
	> repository: https://ByteMr:******@github.com/ByteMr/ByteMr.github.io.git  
	> branch: master  
	
	上述repositoy字段中的地址中，冒号后后面出现*号，这里应该替换成你真实的github账号密码，ByteMr则为真实的github账号

10. **上传到github**  
	**A、**执行如下命令，这样才能将你写好的文章部署到github服务器上并让别人浏览到  
	`npm install hexo-deployer-git --save`  
	**B、**发布  
	>  hexo clean  
	>  hexo generate  
	>  hexo deplo  
	
11. **测试是否发布成功**  
	浏览器输入：	http://yourgithubname.github.io



