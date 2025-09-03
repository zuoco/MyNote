---
title: "使用Git管理软件开发以及冲突解决"
description: 
date: 2024-06-03
hidden: false
comments: true
draft: false
categories:
  - Git
---


# 分支管理
`master分支`:   
用于生产环境的部署，不允许直接push分支，由release分支或者hotfix分支合并。    

`hotfix分支`:  
针对线上紧急问题进行修复的分支，以master分支为基线创建的，修复完成后合并到dev分支、master分支。    

`release分支`:   
预发布分支，UAT测试阶段使用，一般由test分支或hitfix分支合并。    

`dev分支`:   
开发分支，最新版本迭代代码，包括bug修复后的代码。    

`feature分支`:   
基于dev分支在本地创建，每个开发人员的本地分支，针对各自的功能进行开发，开发完成合并到dev分支，并删除该fearure分支，feature是每个开发人员的本地分支，不可推送到远程，只能在本地合并到dev分支，然后推送dev分支。   



# 日常工作
一般是从远程仓库将dev分支拉取到本地，然后创建feature分支，开发完成后，将feature分支合并到dev分支，并删除feature分支，然后推送dev分支到远程仓库。
但是要注意，为避免与团队其他成员的代码冲突，定期将 dev 分支的最新代码合并到你的功能分支。  

```bash
git clone <远程仓库地址>
cd <项目目录>
git checkout dev                          # 切换到 dev 分支
git pull origin dev                       # 拉取最新的 dev 分支代码
git checkout -b feature/your-module-name  # 创建并切换到功能分支

# 开始开发功能，提交代码
git add .                        # 添加所有修改
git commit -m "完成模块A的功能X"   # 提交描述清晰

# 开发中，要定期同步 dev 分支的更新，如果合并时出现冲突，需要手动解决冲突后重新提交。
git checkout feature/your-module-name  # 切换回功能分支
git pull origin dev                    # 合并 dev 分支的最新代码


# 如果发生冲突，就手动处理一下
git add <冲突文件>              # 标记冲突已解决
git commit                     # 提交合并后的代码


# 解决完了冲突后，推送代码到远程仓库
git push -u origin feature/user-authentication   # 

后面还需要提PR，代码审查，修改，合并分支feature到dev，可能是merge，也可能是cherry-pick
```