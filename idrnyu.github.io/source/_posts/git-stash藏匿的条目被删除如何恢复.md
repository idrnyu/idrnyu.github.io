---
title: git stash 藏匿的条目被删除如何恢复
date: 2024-05-10 16:34:00
tags:
- git stash
categories: 
- git
---

## 如果我们使用 git stash 藏匿了一个修改

![藏匿未提交的修改](/Dom/imgs/2024-05-10/藏匿未提交的修改.png)

<!-- more -->

![藏匿未提交的修改-0](/Dom/imgs/2024-05-10/藏匿未提交的修改-0.png)

## 然后不小心删除了藏匿的条目

![删除藏匿条目](/Dom/imgs/2024-05-10/删除藏匿条目.png)

## 可以通过 `git fsck --unreachable` 命令查看记录

![查找被删除的藏匿commit_id](/Dom/imgs/2024-05-10/查找被删除的藏匿commit_id.png)

## 使用 `git show --name-only <commitID>` 查看特征

![查找被删除的藏匿commit_id-并查看特征是否与目标一致](/Dom/imgs/2024-05-10/查找被删除的藏匿commit_id-并查看特征是否与目标一致.png)

## 找到后使用 `git stash apply <commitID>` 恢复删除的内容

![恢复删除的藏匿](/Dom/imgs/2024-05-10/恢复删除的藏匿.png)

## 使用 git stash 的建议

- 在藏匿时，记得添加备注信息，以便查找到相关的内容；
- 删除单个藏匿时，会返回删除的 commitID ，可以直接拿到这个ID还原即可

![藏匿的建议](/Dom/imgs/2024-05-10/藏匿的建议.png)
