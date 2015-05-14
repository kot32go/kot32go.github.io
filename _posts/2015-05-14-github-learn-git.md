---
layout: post
title: Git 自学教程
description: "git."
tags: [git]
---
#Git 自学教程


	学 Git 也有些时候了，但知识点零零散散，虽整理在印象笔记中，但过于繁杂，故整理一些常用的知识放置于博客。
	
一、

        1. 首先创建一个目录作为一个项目的代码仓库。

        2. 在该目录下 $ git init

        3. 将项目文件夹 a 复制到该目录下 $ git add a

        4.接下来是 commit $ git commit -m "my first commit."

二、

        1. 查看当前Git 的状态 $ git status (git diff)查看修改过的内容

        2. $ git log 查看存在哪些版本
        
        3. HEAD 代表当前版本
        
        4. 回退到越早的n个版本 $ git reset --hard HEAD~n
        
        5. 如果回退之后还想回到回退前的版本 $ git reflog 查看版本号，找到之前的commit id，再 $ git reset --hard id 就可以了
