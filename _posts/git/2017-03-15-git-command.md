
---
layout: post
title: "git基本命令"
description: ""
category: git
tags: []

---

#### 常用命令

```
    git init        //初始化一个git本地仓库。
    git status      /查看仓库的状态（是否有新改动的文件）
    git diff xxx    //查看xxx文件具体的改动。
    git clone       //远程仓库地址   从目标仓库克隆一份代码到本地仓库。
    git add filename  //在本地仓库新增一个文件 （git add . 增加所有文件）。
    git commint -m "message"  //提交文件到本地仓库。
```

---

#### 版本回退

``` 
    git reset --hard HEAD^      //退回到上一个版本
    git reset --hard HEAD^^     //退回到前2个版本
    git reset --hard HEAD~100   //退回到前100个版本
    git reset --hard commit_id  //如何还能拿到未来的一个commit id,就能重新回到这个版本上。
    git checkout -- filename    //撤销最近一次修改（其实是用版本库里的版本替换工作区的版本）
    git rm filename         //删除一个文件

```
---
    
#### 添加远程仓库

```
    git remote add origin git@server-name:path/repo-name.git    //新建一个远程仓库
    git push -u origin master   //第一次推送到远程的master仓库
    git push origin master      //推送最新的修改到远程仓库master
```

---
    
#### 从远程仓库克隆到本地

```
    git clone git@github.com:path/xxx.git       //从远程仓库地址克隆到本地  
    git pull git@github.com:path/xxx.git        //从远程git仓库拉取代码。
    git push git@github.com:path/xxx.git        //将本地仓库代码提交到远程git仓库。
```

---

#### 分支管理

```
    git checkout -b branch-name :创建并切换到当前分支。
    git checkout branch-name : 切换分支。
    git merge branch-name : 合并指定分支到当前分支。
    git merge --no-ff -m 'xxx' dev   :禁用fast forword 模式合并分支，这样能看到合并的历史记录。
    git branch branch-name : 创建本地分支。
    git branch -d branch-name : 删除分支。
    git branch -a 查看远程分支。
    git branch   查看本地分支。
```

---


#### 查看提交日志

```
    git log                          //查看提交的历史记录
    git log --pretty=oneline         //历史记录行内显示
    git log --graph --pretty=oneline //查看分支合并图
```

---

#### 暂存代码 

```
    git stach       //把当前工作现场“储藏”起来。
    git stach list  //查看隐藏的内容。
    git stash pop   //恢复stach区的内容并删除stach区。
    git stach apply //恢复stach区的内容不删除stach区。
    git stach apply stash@{0}   //恢复某个版本的内容。
```
---