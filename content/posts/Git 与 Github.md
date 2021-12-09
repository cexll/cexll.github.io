---
title: Git 与 Github
subtitle: ""
date: 2021-06-24 13:38:27.901
lastmod: 2021-06-24 13:38:27.901
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    git,github
]
categories: [
    git
]

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: ""
featuredImagePreview: ""

toc:
  enable: true
math:
  enable: false
lightgallery: false
license: ""
---

<!--more-->



## 1.Open your Terminal application.

## 2.Ensure Git 1.9.5 or ⬆️ is installed, type:

```bash
git --version
```

## 3.Set your name in Git by replacing with your first and last name:
```bash
ssh-keygen
cat ~/.ssh/id_rsa.pub
git config --global user.name 
```

## 4.Set your email in Git by replacing with the email associated with your GitHub account.
```bash
git config --global user.email 
```


## 5.Set line-ending behavior for your operating system:

`On Windows: git config --global core.autocrlf true`
On Mac and Unix-like systems:
```bash
git config --global core.autocrlf input
```


## 6.To see your current configurations, type:
```bash
git config --list
```


## 7.Clone the repository by replacing with clone URL you copied in the previous step. The repository will be cloned into a new directory in this location.
```bash
git clone 
```


## 8.Type:
```bash
git status
```


## 9.Create a new branch. Replace with descriptive name:
```bash
git branch 
```


## 10.Check out to your new branch:
```bash
git checkout 
```


## 11.将您的文件添加到暂存区域，以便它成为下一次提交的一部分。
```bash
git add 
```

## 12.查看你的文件的当前状态。您的文件现在列在标题下Changes to be committed。这告诉我们该文件在暂存区域中。它也表明这是一个新文件。
```bash
git status
```


## 13.提交你的文件。一个提交告诉Git的收集所有的文件的临时区域，并将它们存储到版本控制，工作的一个单元。Git将打开默认的文本编辑器，您可以在其中输入提交消息。
```bash
git commit -m ""
```


## 14.查看提交的历史记录：
```bash
git log
```


## 15.将您的提交推送到远程，并设置跟踪分支。类型：
```bash
git push -u origin
```

## 16. Git记住用户名密码每次PUSH/PULL 不用输入用户名密码

```bash
# 进入项目目录 输入:
git config --global credential.helper store
```