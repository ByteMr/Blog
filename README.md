# Blog
Hexo+Github pages搭建的个人技术博客
# 博客备份提交说明
  我们知道，我们每次发布新的文章或者博客都会在hexo根目录的[source\_posts]目录下，生成一个XXXX.md的文件，  
  当我们执行如下命令后，会重新生成博客，重新发布到站点。  
  `hexo clean`  
  `hexo g`  
  `hexo d` 
  所以我们每次更新了博客或者发布新的博客，只需要将更新后的和新增的博客提交即可。


# 导入到本地

  1、补充hexo核心目录node_modules  
  当我们将本仓库的文件及其目录下载到本地之后，其实是少一个[node_modules]目录的，这个是hexo的核心目录，可是为什么会少呢？  
  这个主要是因为，hexo规定在git提交的项目的时候，这个是ignore，不需要提交的，但是少了这个目录是不行的，  
  我们可以执行 `hexo init`命令来重新生成一个新的Hexo目录，然后copy过去即可，当然这是比较笨，也比较有效的方法。  
  2、当我们执行hexo d发布的时候，会提示如下错误  
  `ERROR Deployer not found: git`  
  这个时候我们需要执行如下命令来生成。  
  `npm install --save hexo-deployer-git`
  3、然后导入工作就结束了，可以去修改和发布新的博客了，最后依次执行`hexo clean`--> `hexo g` --> `hexo d`   命令发布即可。  
  4、记得发布新的博客和修改博客了，需要及时提交到远程仓库哦。
    
    
  
