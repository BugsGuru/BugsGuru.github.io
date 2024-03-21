---
title: "Git命令"
date: 2024-03-21

categories: [常用命令]
tags: [git]
keywords: [git]

description: "Git命令"
---

### 换行符 CRLF CR LF
#### merge时忽略行尾对比
```sh
git config --global merge.renormalize true
```
#### 提交混合LF、CRLF时产生警告  
```sh
# false允许 true禁止
git config --global core.safecrlf warn
```
#### 自动把\r\n换为\n
```sh
# false 不做处理
# true 自动更换crlf，win下checkout时\n自动换为\r\n，在提交时在自动换回\n
git config --global core.autocrlf input
```


### 删除分支
#### 直接删除远程lwq分支
```sh
git push origin -d lwq
```
#### 删除本地的分支（远程已经删除的）
```sh
git remote prune origin
```


### 删除 readme.md 的跟踪
#### 并保留在本地
```sh
git rm --cached readme.md
```
#### 并且删除本地文件
```sh
git rm --f readme.md
```


### Git 文件还原
#### 追踪的文件更改未提交:
```sh
git checkout .
```
#### 清除未追踪追踪的文件(夹)
```sh
git clean -dff
```
#### 还原文件至上上次提交
```sh
git reset --hard HEAD^^
```
#### 还原commit（文件不动）至上上次提交
```sh
git reset --soft HEAD^^
```