---
title: "Git的基本使用"
date: 2020-08-23T20:57:28+08:00
tags:
- git
Categories:
- Git
---

## Git的基本使用

#### 新项目提交至github

```bash
git init    //创建本地仓库
git add .   //提交全部
git commit -m "提交时的注释，尽量写，方便以后可以查看"

//添加仓库源
git remote add origin https://github.com//yourusername//yourrepository.git 
//这一步必须要写，不然后面会出现错误
git pull origin master
//将本地仓库推至远程github
git push -u origin master
```

> 上面git pull的时候可能出现 **refusing to merge unrelated histories**的情况，**这是因为远程仓库已经存在代码记录了，并且那部分代码没有和本地仓库进行关联，我们可以使用如下操作允许pull未关联的远程仓库旧代码：** 
>
> ```shell
> git pull origin master --allow-unrelated-histories
> ```



#### 项目改动之后再次提交

```bash
git add -A	//将修改过的部分添加至本地仓库
git commit -m "xxxxxxxxx"
git remote add origin https://github.com//yourusername//yourrepository.git 
git push -u origin master
```



#### 常用的git的命令

- 如果git add 错了文件，如何撤回

```bash
git reset .       //撤回所有
```

- 查看当前的远程仓库源（remote origin）

```bash
git remote -v
```

- 如果源添加错了，可以先删除，然后再追加新的源

```shell
git remote rm origin
git remote add origin "新的仓库源的地址"
```



#### 常见的git 提交的问题

- 出现提交之后的文件夹在github上面打不开

  > 这是因为该文件夹中存在 .git 文件，被认为是一个子模块，需要做的是在本地删除 .git文件

  ```shell
  git rm 文件夹
  git add 文件夹/
  ```

  然后再commit ，push 就可以了