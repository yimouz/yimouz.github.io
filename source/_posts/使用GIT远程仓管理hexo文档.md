---
title: 使用GIT远程仓管理hexo文档
date: 2020-05-22 08:58:27
tags: hexo
categories: 杂记
author: 是啊蓁阿
summary: 以hexo为例,使用git从配置到切换分支策略跟拉取的简单演示
  
---
 

### Hexo简单的命令

1.  终端执行`hexo new 新文章`命令，此时本地`source/_post`目录下会新建`新文章.md`文件
2.  打开 `新文章.md`编写完成后，终端执行`hexo g`编译
3.  终端执行`hexo s`本地预览
4.  终端执行`hexo d`推送到远程仓**master**分支

### 场景备份的建议

- 个人pc比较多,有时候不是专门用一台机器来做一件事,公司家庭暂住地等等等等,多台pc的使用场景
- 硬盘损坏,
- 电脑丢失
- 如果你还算比较珍你的部落格,不管是什么情况,都建议进行备份,鸡蛋不在一个篮子里,有备自然无患  
有些人选择新建一个远程仓来进行拉取上传,有些人选择每次用u盘热拷数据,而我选择在原来远程仓基础上新建一个分支进行源码上传,原因嘛,自然是怎么方便怎么来.



#### 新建git分支
终端执行`git branch`  
查看分支情况,如果没有意外你只有一个**master**分支  
终端执行`git checkout -b dev`  
创建并切换到**dev**分支,其中**dev**是你的自定义分支名  
再次执行`git branch`  
你可以看到列出的分支包含**dev/master**  
当前分支带有*号

#### 发布时选择的分支

github仓库初始化生成一个master分支，这没什么特殊含义,只是init的时候就叫这个而已.  
不过这对hexo来讲,生成的静态页面也是默认在master分支,如非魔改框架,请务必留着.  
我这里选择**dev**分支作为我的源码托管分支,  
你可以在克隆项目的时候指定分支`git clone -b dev xxurl`如果你正好也感兴趣,我会在后面描述  

#### 本地与远程仓库关联

终端执行<code class="shell  language-shell">git remote add origin git@github.com:yimouz/yimouz.github.io.git</code>  
>这里的origin只是一个易标识,你可以修改,比如hexo,后面这一串是仓库ssh地址,请不要错误关联到我的仓库,错误关联但没有配对的密钥自然没有用.除了ssh还可以选择http方式,具体看你个人选择.

#### 推送到远程仓库

推送到dev分支。
本地有许多文件,保留文章跟一些你觉得重要的，其他无关的文件并不希望推送，  
使用`.gitignore`文件吧
把这些不想要的推送的保存到你ignore,比如如下一些:   
.DS_Store  
Thumbs.db  
db.json  
*.log  
node_modules/  
public/  
.deploy*/  
添加完就可以试一下推送了  
终端执行：
 ```
git add .
git commit -m '你的提交信息'
git push hexo dev
``` 

#### 删除dev分支的某些文件 

dev继承自master会带上master已经存在的分支内容  
类似public  
如果你跟我一样看着他们常常难受  
不要犹豫,本地环境.   
按下你的delete fireout it  
删除完,终端执行 
当然不删也可以
 ```
git add .
git commit -m '同样填写你的注释信息'
git push hexo dev
```
你会发现.ignore里携带的都没有带上去了.  
这就是ignore

### 多pc

回到这个问题,新电脑了怎么办,硬盘坏了怎么办  
1. 下载git 
2. 安装node npm 
3. 克隆你的分支`git clone -b dev 你的url`   
4. 初始化init
5. 密钥
6. 关联到你的远程仓

 > 当然不要忘记
 ```
npm install hexo   
npm install hexo-deployer-git -save
```
> 是不是又回到了熟悉的环境
### 好了知道你有多pc使用习惯了,git就非常适合这种多端协作的场景,你要知道他是干嘛的
在公司划水,写写博客 一顿操作在push之后进入贤者模式  
晚上或者周末在家,心潮澎拜,idea遍及  
马不停蹄打开git
```
/git pull <远程仓ssh> <远程分支名>:<本地分支名>   

git pull git@github.com:yimouz/yimouz.github.io.git dev:dev
/拉取并合并到你的本地环境  
```
 > 至此就结束了!