---
title: Git学习笔记
description: ""
slug: 2021-09-02-Git学习笔记
date: 2021-09-08
image: git.png
categories:
  - 运维&云原生
tags:
  - Git
---
上班之后使用Git，可我根本不会！！！

## 1. Git 基础操作

### 1.1 配置用户信息

```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
```

**说明**：  
设置全局用户名和邮箱，提交时会记录这些信息。

___

### 1.2 创建或克隆仓库

-   **初始化本地仓库**
    
    ```bash
    git init
    ```
    
    **说明**：在当前文件夹下创建一个新的 git 仓库。
    
-   **克隆远程仓库**
    
    ```bash
    git clone <仓库地址>
    ```
    
    **例子**：
    
    ```bash
    git clone https://github.com/username/repo.git
    ```
    
    **说明**：会在本地新建一个 repo 文件夹，并把远程仓库的内容拉下来。
    

___

### 1.3 查看状态和历史

-   **查看当前状态**
    
    ```bash
    git status
    ```
    
    **说明**：查看哪些文件有修改、哪些文件未跟踪。
    
-   **查看提交历史**
    
    ```bash
    git log
    ```
    
    **说明**：显示提交历史，按 q 退出。
    
    **常用参数**：
    
    ```bash
    git log --oneline --graph --all
    ```
    
    简洁显示所有分支的提交历史。
    

___

### 1.4 添加和提交

-   **添加文件到暂存区**
    
    ```bash
    git add <文件名>git add .        # 添加所有更改
    ```
    
    **说明**：把文件的更改加入暂存区，准备提交。
    
-   **提交到本地仓库**
    
    ```bash
    git commit -m "提交说明"
    ```
    
    **说明**：把暂存区的内容提交到本地仓库。
    

___

### 1.5 远程操作

-   **查看远程仓库**
    
    ```bash
    git remote -v
    ```
    
-   **添加远程仓库**
    
    ```bash
    git remote add origin <仓库地址>
    ```
    
-   **推送到远程仓库**
    
    ```bash
    git push origin master
    git push origin <分支名>
    ```
    
-   **拉取远程仓库代码**
    
    ```bash
    git pull origin master
    ```
    
-   **抓取远程更新但不合并**
    
    ```bash
    git fetch origin
    ```
    

___

## 2. 分支管理

### 2.1 查看分支

```bash
git branch
```

### 2.2 创建分支

```bash
git branch <分支名>
```

**例子**：

```bash
git branch dev
```

### 2.3 切换分支

```bash
git checkout <分支名>
```

**例子**：

```bash
git checkout dev
```

### 2.4 创建并切换分支

```bash
git checkout -b <分支名>
```

**例子**：

```bash
git checkout -b feature/login
```

### 2.5 合并分支

```bash
git merge <分支名>
```

**例子**：

```bash
git merge dev
```

**说明**：将 dev 分支合并到当前分支。

### 2.6 删除分支

```bash
git branch -d <分支名>
```

**例子**：

```bash
git branch -d dev
```

___

## 3. 撤销与回退

### 3.1 撤销修改

-   **撤销工作区的修改**
    
    ```bash
    git checkout -- <文件名>
    ```
    
    **说明**：还原文件到最近一次提交的状态。
    
-   **撤销已暂存的文件**
    
    ```bash
    git reset HEAD <文件名>
    ```
    
    **说明**：把文件从暂存区移回工作区。
    

### 3.2 取消最近一次提交（保留更改）

```bash
git reset --soft HEAD^
```

### 3.3 取消最近一次提交（丢弃更改）

```bash
git reset --hard HEAD^
```

### 3.4 查看历史版本

```bash
git reflog
```

**说明**：显示所有 HEAD 的移动记录，可用于找回误删的提交。

___

## 4. 标签管理

### 4.1 创建标签

```bash
git tag <标签名>
```

**例子**：

```bash
git tag v1.0
```

### 4.2 推送标签到远程

```bash
git push origin <标签名>
git push origin --tags   # 推送所有标签
```

### 4.3 删除标签

```bash
git tag -d <标签名>
git push origin :refs/tags/<标签名>  # 删除远程标签
```

___

## 5. 忽略文件

### 5.1 创建 .gitignore 文件

在项目根目录新建 `.gitignore` 文件，写入要忽略的文件或文件夹：

```bash
*.test/.DS_Store
```

___

## 6. 常用高级命令

### 6.1 stash 暂存

-   **保存当前更改**
    
    ```bash
    git stash
    ```
    
-   **恢复暂存的更改**
    
    ```bash
    git stash pop
    ```
    
-   **查看 stash 列表**
    
    ```bash
    git stash list
    ```
    

___

## 7. 其他常用命令

-   **查看文件差异**
    
    ```bash
    git diff
    ```
    
-   **查看某次提交的内容**
    
    ```bash
    git show <commit_id>
    ```
    
-   **重命名分支**
    
    ```bash
    git branch -m <新分支名>
    ```
    
-   **删除远程分支**
    
    ```bash
    git push origin --delete <分支名>
    ```
    

___

## 8. 常见开发流程举例

### 场景：新建功能分支开发并合并到主分支

```bash
git checkout -b feature/xxx      # 新建并切换到新分支# 
...开发、修改文件...
git add .                        # 添加所有更改
git commit -m "完成xxx功能"       # 提交
git push origin feature/xxx      # 推送到远程
git checkout master              # 切换回主分支
git pull origin master           # 拉取最新主分支代码
git merge feature/xxx            # 合并功能分支
git push origin master           # 推送到远程主分支
git branch -d feature/xxx        # 删除本地功能分支
git push origin --delete feature/xxx   # 删除远程分支（可选）
```

___

## 9. 帮助和文档

```bash
git help <command>
git <command> --help
```

**例子**：

```bash
git help commit
```

### 10. 进阶操作

#### 10.1 变基（rebase）

-   **将当前分支的提交“移到”目标分支后面**
    
    ```bash
    git rebase <目标分支>
    ```
    
    **例子：**
    
    ```bash
    git checkout feature/xxx
    git rebase master
    ```
    
    **说明**：把 `feature/xxx` 分支的提交移到 `master` 分支最新的提交之后。常用于保持提交历史整洁。
    
-   **交互式变基（整理提交历史）**
    
    ```bash
    git rebase -i HEAD~3
    ```
    
    **说明**：对最近 3 次提交进行合并、修改说明等操作。
    

___

#### 10.2 cherry\-pick（挑选提交）

-   **把某个提交应用到当前分支**
    
    ```bash
    git cherry-pick <commit_id>
    ```
    
    **例子：**
    
    ```bash
    git cherry-pick abcd
    ```
    
    **说明**：将指定的提交“复制”到当前分支，常用于补丁、热修复。

___

#### 10.3 解决冲突

-   当合并、变基、cherry-pick 时，出现冲突，Git 会提示你哪些文件冲突。
    
-   用编辑器打开冲突文件，找到类似下面的内容：
    
    ```text
    <<<<<<< HEAD
    你的修改内容
    =======
    其他分支的内容
    >>>>>>> 分支名
    ```
    
-   修改为你想要的内容，保存。
    
-   标记为已解决并提交：
    
    ```bash
    git add <冲突文件>
    git commit
    
    #或者
    
    git merge --continue
    git rebase --continue
    ```


___

#### 10.4 删除远程分支/标签

-   **删除远程分支**
    
    ```bash
    git push origin --delete <分支名>
    ```
    
-   **删除远程标签**
    
    ```bash
    git push origin :refs/tags/<标签名>
    ```
    

___

### 11. 协作流程

#### 11.1 Fork + Pull Request （开源协作常用）

1.  在 GitHub 上 Fork 仓库到自己的账号
2.  克隆到本地，创建新分支开发
3.  提交并推送到自己仓库
4.  在 GitHub 上发起 Pull Request，等待项目维护者审核合并

#### 11.2 Code Review 流程

-   开发者提交代码到 feature 分支
-   通过 Pull Request 发起合并请求
-   其他开发者进行代码审查（Review），提出修改意见
-   修改后再次提交，直到通过审核
-   合并到主分支

___

### 12. 常见问题处理

#### 12.1 误删文件如何恢复

-   如果还没有提交，使用：
    
    ```bash
    git checkout -- <文件名>
    ```
    
-   如果已经提交，可以通过 `git log` 找到 commit\_id，然后用：
    
    ```bash
    git checkout <commit_id> -- <文件名>
    ```
    

___

#### 12.2 找回误删的分支或提交

-   用 `git reflog` 查找历史记录，找到分支或提交的哈希值，然后用：
    
    ```bash
    git checkout <哈希值>
    ```
    
    或者
    
    ```bash
    git branch <新分支名> <哈希值>
    ```
    

___

#### 12.3 撤销已推送的提交

-   用 `git revert` 生成一个反向提交：
    
    ```bash
    git revert <commit_id>
    ```
    
    **说明**：不会改变历史，只是生成一个新的反向提交。
    
-   如果确实需要强制回退（不推荐，除非你确定团队都知情）：
    
    ```bash
    git reset --hard <commit_id>
    git push origin HEAD --force
    ```
    

___

### 13. 子模块（Submodule）

-   **添加子模块**
    
    ```bash
    git submodule add <仓库地址> <路径>
    ```
    
-   **初始化和更新子模块**
    
    ```bash
    git submodule init
    git submodule update
    ```
    

___

### 14. .gitignore 高级用法

-   忽略某个文件夹下所有内容但保留文件夹结构：
    
    ```bash
    logs/*
    !logs/.gitkeep
    ```
    
-   忽略某类文件但保留部分：
    
    ```bash
    *.log
    !important.log
    ```
    

___

### 15. 常用别名设置

-   设置常用命令别名提高效率：
    
    ```bash
    git config --global alias.st status
    git config --global alias.co checkout
    git config --global alias.br branch
    git config --global alias.ci commit
    git config --global alias.lg "log --oneline --graph --all"
    ```
    
    以后可以直接用 `git st`、`git co`、`git ci` 等。
    

___

### 16. 其他实用技巧

-   **查看远程分支列表**
    
    ```bash
    git branch -r
    ```
    
-   **查看所有分支（本地+远程）**
    
    ```bash
    git branch -a
    ```
    
-   **清理本地已删除的远程分支引用**
    
    ```bash
    git remote prune origin
    ```
    
-   **批量删除未合并的本地分支**
    
    ```bash
    git branch | grep -v "master" | xargs git branch -D
    ```
    

### 17. 高级场景与实用案例

#### 17.1 多人协作中的常见冲突解决流程

1.  **拉取最新主分支代码，保持同步**
    
    ```bash
    git checkout master
    git pull origin master
    ```
    
2.  **切换到开发分支，合并主分支的最新内容**
    
    ```bash
    git checkout feature/xxx
    git merge master
    ```
    
3.  **如果有冲突，按提示解决冲突，编辑后**
    
    ```bash
    git add <冲突文件>
    git commit
    ```
    
4.  **再推送到远程仓库**
    
    ```bash
    git push origin feature/xxx
    ```
    

___

#### 17.2 代码回滚到历史某一版本

-   **临时回退查看历史版本，不影响历史**
    
    ```bash
    git checkout <commit_id>
    ```
    
    只是在本地切换到该版本，不会影响分支历史。
    
-   **永久回退分支到某一历史版本**
    
    ```bash
    git reset --hard <commit_id>
    git push origin HEAD --force
    ```
    
    注意：强制推送会覆盖远程分支，慎用。
    

___

#### 17.3 合并多个提交为一个（squash）

-   在合并分支时，将多个提交压缩为一个，保持历史简洁：
    
    ```bash
    git merge --squash <分支名>
    git commit -m "合并说明"
    ```
    
-   也可以用交互式 rebase：
    
    ```bash
    git rebase -i HEAD~3
    ```
    
    在弹出的编辑器中，把要合并的提交前的 `pick` 改为 `squash`。
    

___

#### 17.4 修改最近一次提交信息

```bash
git commit --amend
```

**说明**：可修改最近一次提交的说明或补充文件。

___

#### 17.5 只拉取远程某个分支（浅克隆）

```bash
git clone -b <分支名> --single-branch <仓库地址>
```

**例子**：

```bash
git clone -b dev --single-branch https://github.com/user/repo.git
```

___

### 18. 与常见工具结合

#### 18.1 与 GitHub /GitLab/码云等平台结合

-   **SSH Key 配置**：避免每次 push 输入密码
    
    1.  生成 SSH Key（如果没有）：
        
        ```bash
        ssh-keygen -t rsa -C "你的邮箱"
        ```
        
    2.  将 `~/.ssh/id_rsa.pub` 内容添加到平台的 SSH Key 设置中
-   **通过 Web 页面直接创建、合并、删除分支和 Pull Request**
    

___

#### 18.2 与 VSCode/IDEA 等编辑器集成

-   现代编辑器内置 Git 界面，可直接完成 add、commit、push、pull、分支切换、冲突解决等操作

___

#### 18.3 与 CI/CD 工具结合（持续集成）

-   配合 GitLab CI、GitHub Actions、Jenkins 等工具，Git 提交可自动触发构建、测试、部署流程
-   常见方式：push 到特定分支时自动部署生产环境、发布版本等

___

### 19. 易错点与注意事项

1.  **强制推送（git push --force）**
    -   只在本地分支历史需要重写且团队成员知情时使用，否则容易造成别人代码丢失

2.  **误删分支/提交**
    -   使用 `git reflog` 找回
    -   重要操作前建议打 tag 或备份分支

3.  **分支命名规范**
    -   建议采用如 `feature/xxx`、`bugfix/xxx`、`hotfix/xxx` 这样的命名方式，利于团队协作

4.  **.gitignore 变更后要提交**
    -   修改 `.gitignore` 后，需重新 add、commit、push，否则不会生效

5.  **不要将大文件、敏感信息（如密码、密钥）上传到仓库**
    -   可用 `.gitignore` 忽略，已上传需用 [BFG Repo-Cleaner](https://rtyley.github.io/bfg-repo-cleaner/ "BFG Repo-Cleaner") 等工具清理

___

### 20. Git 常见报错及解决办法

-   **fatal: refusing to merge unrelated histories**
    
    ```bash
    git pull origin master --allow-unrelated-histories
    ```
    
-   **error: failed to push some refs to …**
    
    -   远程有新提交，本地未同步，需先 pull 再 push
    -   或用 `git push --force`（慎用）

-   **Merge conflict in …**
    
    -   文件有冲突，需手动解决并提交

-   **Permission denied (publickey)**
    
    -   没有配置 SSH Key，或 Key 未添加到平台

___

### 21. 常用 Git 命令

|     操作     |            命令示例             |
|------------|-----------------------------|
|   初始化仓库    |          git init           |
|    克隆仓库    |          git clone          |
|    查看状态    |         git status          |
|   添加到暂存区   |           git add           |
|     提交     |     git commit -m “说明”      |
|    查看日志    |           git log           |
|    查看分支    |         git branch          |
|    新建分支    |         git branch          |
|    切换分支    |        git checkout         |
|    合并分支    |          git merge          |
|    删除分支    |        git branch -d        |
|     推送     |       git push origin       |
|     拉取     |       git pull origin       |
|     标签     |           git tag           |
|    删除标签    |         git tag -d          |
|   合并冲突解决   | 编辑文件 → git add → git commit |
| 交互式 rebase |    git rebase -i HEAD~N     |
|   恢复误删文件   |       git checkout –        |
| 查看 reflog  |         git reflog          |
|   添加子模块    |      git submodule add      |
