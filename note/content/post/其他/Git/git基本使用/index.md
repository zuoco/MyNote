---
title: "Git基本使用"
description:
date: 2022-06-03
math: 
license: 
hidden: false
comments: true
draft: false
categories:
  - Git
---

- [1. 基本概念](#1-基本概念)
- [2. Git的配置](#2-git的配置)
- [3. Git仓库](#3-git仓库)
- [4. 查询文件变更](#4-查询文件变更)
- [5. 远程仓库](#5-远程仓库)
- [6. git分支](#6-git分支)
- [7. git冲突](#7-git冲突)
- [8. 删除分支](#8-删除分支)
- [9. 标签](#9-标签)
- [10. 撤销修改](#10-撤销修改)
- [11. Cherry-pick](#11-cherry-pick)



# 1. 基本概念
**仓库（Repository）**：一个Git仓库是一个包含所有版本控制文件和历史记录的目录。    
- 本地仓库（Local Repository）：在你的本地计算机上的仓库。   
- 远程仓库（Remote Repository）：托管在服务器上的仓库，通常是GitHub、GitLab等。   

**工作区（Workspace）**: 你在本地计算机上的项目目录，包含了所有的文件和目录。     
**暂存区（Stage）**: 用于暂存即将提交的文件，添加到暂存区的文件会被git跟踪。     
**主分支**：新建一个仓库会分配一个默认主分支，早期为“master”，现在为“main”。         
**head**: 指向当前工作的分支的最新提交，每次提交都会更新head，保持head指向最新的提交。        

# 2. Git的配置
1. 全局配置：  
```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

2. 仓库配置：   
```bash
# 在仓库目录下执行以下命令：
# 保存在当前仓库下.git/config文件中，仓库配置会覆盖全局配置。
git config --local user.name "Your Name"
git config --local user.email "your_email@example.com"
```

3. 查看配置：      
```bash
git config --list  
```


# 3. Git仓库
**1. 创建本地仓库**   
```bash
git init
```

**2. 查看仓库状态**  
```bash
git status
```

**3. 添加文件到暂存区**  
```bash
git add <file>
git add .   # 添加所有文件
```

**4. 提交内容到本地仓库**   
```bash
git commit -m "提交信息"
```

git会为每一个提交创建一条版本历史记录，包含：        
1. commit id: 40位字符串，表示一个唯一的提交ID，用于标识提交，在命令中使用前6位即可。    
2. commit message: 提交信息，用于描述本次提交的内容。    
3. 快照： 完整版本文件，以对象树的形式存储在.git/objects目录下。

commit还有其他用法：
```bash
# 使用一次新的提交，替代上一次提交，会修改commit id
git commit --amend -m "修改提交信息"
# 提交指定文件
git commit filename
```

**5.查看提交日志**
```bash
git log
```
会列出所有提交的信息，包括commit id、commit message、作者、提交时间等。   

# 4. 查询文件变更
```bash
git diff
```
可以查看工作区和暂存区之间的差异，删除的行会以删除符号（---）标记，新增的行会以新增符号（+++）标记。   
Changes to be committed: 已经git add的文件，但是还没有git commit的文件。      
Changes not staged for commit: 还没有git add的文件。    
Unmodified files: 没有被git跟踪的文件。    

# 5. 远程仓库
**1. 关联到仓库**   
```bash
# 先创建仓库
git init
# 关联远程仓库
git remote add origin <repository_url>
```
**2. 推送本地仓库到远程仓库** 
```bash
git add .
git commit -m "xxxxxx"


# origin就是使用“git remote add origin <仓库地址>”所添加的远程仓库。
# 第一次推送时，需要使用-u设置上游分支，表示将当本地分支与远程分支关联起来。
git push -u origin main
# 后续推送
git push


# 推送指定分支到远程仓库的指定分支。
git push origin <本地分支名>:<远程分支名> 
# 将本地所有分支推送到远程仓库。
git push --all origin   
```

**3. 克隆远程仓库**   
```bash
git clone <repository_url> <path>

# 递归克隆子模块
git clone --recurse-submodules https://github.com/FreeCAD/FreeCAD.git freecad-source
```

**4. 查看关联的远程仓库**   
```bash
git remote -v
```

**5. 同步仓库**   
```bash
git pull <远程仓库名> <远程仓库分支名>   # 获取远程仓库的最新提交，并与本地仓库合并。
# git pull origin main
# git pull 默认从当前关联的远程仓库和分支拉取更新。
```

# 6. git分支
**1.**`查看分支`     
```bash
git branch     # 查看本地分支
git branch -a  # 查看所有分支，包括本地和远程分支
```

**2.** `创建分支`     
```bash
git branch <branch_name> 
```

**3.** `切换到指定分支`    
```bash
git checkout <branch_name>
git switch <branch_name>
```

**4.** `分支合并`
```bash
git merge <branch_name> # 将指定分支合并到当前分支
```
例如：   
当前有两个分支：main和dev，现在需要将dev分支合并到main分支上，执行以下命令：     
```bash
git switch main   # 切换到main分支
git merge dev     # 将dev分支合并到main分支上
```

# 7. git冲突
**合并时发生冲突：** 
```bash   
CONFLICT (content): Merge conflict in XXXXXX    
Automatic merge failed; fix conflicts and then commit the result.    
```

**发生冲突时:**   
1. 使用git status命令查看冲突文件:    
```bash
both modified:   xxxxxx # 两个分支都修改了xxxxxx文件
```
2. 打开xxxxxx文件：
```
<<<<<<< HEAD
这里是原本的代码；
=======
这里是dev分支修改的代码；
>>>>>>> dev
```
阅读代码逻辑，手动修改代码，并保存文件，然后重新add以及commit就可以了，不需要重新merge，已经merge过了。

# 8. 删除分支
合并分支后，删除分支：
```bash
git branch -d <branch_name>            # 删除本地分支
git push origin --delete <branch_name> # 删除远程分支

git branch -D <branch_name>  # 强制删除分支
```

# 9. 标签
用来标注项目中的特定版本，例如正式发布版本，如v1.0.0。  
**1. 查看标签**   
```bash
git tag # 查看所有标签
```
```bash
git show <tag_name> # 查看指定标签详细信息
```


**2. 切换到指定标签**
```bash
git checkout v1.0
```
**3. 创建标签**   


&emsp;&emsp;轻量标签：      
```bash
git tag <tag_name> # 指向当前最新的提交
```

&emsp;&emsp;附注标签：   
```bash
git tag -a <tag_name> -m "tag_message" # 指向当前最新的提交
```
&emsp;&emsp;给过去的提交打标签：   
```bash
git tag -a <标签名> <commit_id> -m "tag_message" # commit_id使用前6位即可
```

**4. 推送标签**    
&emsp;&emsp;推送单个标签：   
```bash
git push origin <tag_name>
```
&emsp;&emsp;推送所有标签：
```bash
git push origin --tags
```

**5. 删除标签**    
&emsp;&emsp;删除本地标签：  
```bash
git  tag -d <tag_name>
```
&emsp;&emsp;删除本地标签后，元远程标签还在，这时需要使用以下命令删除远程标签：
```bash
git push origin --delete <tag_name>
```

# 10. 撤销修改
&emsp;&emsp;还没有git add，想恢复到最后一次提交:  
```bash
git reset checkout -- xxxxxx.cpp
``` 
&emsp;&emsp;已经git add，想恢复到最后一次提交:  
```bash
git reset HEAD xxxxxx.cpp  # 将文件从暂存区移除，但文件的修改仍然保存在文件中
git checkout -- xxxxxx.cpp # 将文件恢复到最后一次提交的状态
```
&emsp;&emsp;已经git commit，回退到本地提交：   
```bash
# 也可以使用commit id替代HEAD~1。
git reset --soft HEAD~1        # 回退提交到当前分支的上一次提交，但是代码依然保留在暂存区，以及工作区。  # HEAD~1表示当前最新提交的上一个提交。
git reset --mixed HEAD~1       # 回退提交，删除暂存区对应代码，但工作区依然保留。                    # 默认模式
git reset --hard HEAD~1        # 回退提交，删除暂存区对应代码，删除工作区对应代码。
```

&emsp;&emsp;已经git commit，会退远程提交（创建一次新的commit，覆盖指定的一次commit）：   
```bash
git revert commit_id
git commit -m "Revert commit_id"  # 再次提交
git push origin master
```

# 11. Cherry-pick
将某个分支上的单个或多个提交（commit）“复制”到当前分支,合并提交，而不是合并整个分支。
```bash
# 切换到目标分支
git checkout <目标分支名>

# 应用指定提交（<commit-hash> 是源分支提交的哈希值）
git cherry-pick <commit-hash>
```

```bash
# 依次应用多个提交
git cherry-pick <commit1> <commit2> <commit3>

# 或使用区间语法（左开右闭）
git cherry-pick <start-commit>^..<end-commit>
```