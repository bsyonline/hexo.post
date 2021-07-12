---
title: Upgrade Fork Repo
date: 2015-10-22 15:45:26
tags:
 - Hadoop
category: 
 - Big Data
thumbnail: 
author: bsyonline
lede: "没有摘要"
---





fork 了 spring boot 的仓库，如果 spring boot 仓库有更新，如何将更新同步到自己 fork 的仓库？

1. 增加 upstream 

   ```
   git remote add upstream git@github.com:spring-projects/spring-boot.git
   ```

   

2. 查看 upstream

   ```
   $ git remote -v
   origin  git@github.com:bsyonline/spring-boot.git (fetch)
   origin  git@github.com:bsyonline/spring-boot.git (push)
   upstream        git@github.com:spring-projects/spring-boot.git (fetch)
   upstream        git@github.com:spring-projects/spring-boot.git (push)
   ```

   

3. 抓取原仓库的更新

   ```
   git fetch upstream
   ```

   

4. 合并远程分支

   ```
   git merge upstream/main
   ```

   

5. 同步 tag

   ```
   git fetch upstream --tags
   ```

   

6. 

   