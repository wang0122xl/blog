# [blog地址](https://wang0122xl.github.io/blog)

> ## 注：blog-page为静态资源发布分支，勿动

## 环境配置

> ``git clone https://github.com/wang0122xl/blog.git``  
> ``cd blog``  
> ``yarn / npm install``  
> ``yarn server``

## 开发流程

* 新建文章分支[optional，多人开发时推荐]

* 新建草稿  
``yarn draft [文章名]``
* 编辑草稿（草稿位于source/_drafts文件夹内）
* 草稿编辑完后使用
  > ``yarn publish [文章名]``  
将草稿转为post

***
***或者，在确定当前分支只有一篇待发布文章，或多篇待发布文章计划同时发布时，可直接使用``yarn post [文章名]``新建文章***
***
> **最终生成的待发布文章都在source/_post文件夹内**

* 非master分支提pr至master
* master分支下运行yarn publish发布文章

### 参考链接

* **[hexo官方文档](https://hexo.io/zh-cn/docs/)**
* **[NexT官方文档](https://theme-next.iissnan.com/getting-started.html)**
