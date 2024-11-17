以下是常用的 Git 命令整理：

### 基本配置
- `git config --global user.name "Your Name"`：配置全局用户名
- `git config --global user.email "your.email@example.com"`：配置全局邮箱

### 仓库管理
- `git init`：初始化新的 Git 仓库
- `git clone <repo>`：克隆远程仓库

### 文件操作
- `git add <file>`：将指定文件添加到暂存区
- `git add .`：将所有修改的文件添加到暂存区
- `git commit -m "message"`：提交暂存区的更改并附加提交信息
- `git commit -am "message"`：添加并提交所有已跟踪文件的更改

### 分支管理
- `git branch`：查看所有本地分支
- `git branch <branch-name>`：创建新分支
- `git checkout <branch-name>`：切换到指定分支
- `git checkout -b <branch-name>`：创建并切换到新分支
- `git merge <branch-name>`：将指定分支合并到当前分支
- `git branch -d <branch-name>`：删除本地分支

### 状态与历史
- `git status`：查看文件当前状态
- `git log`：查看提交历史

### 撤销更改
- `git reset <file>`：将文件从暂存区移除
- `git checkout -- <file>`：撤销工作区的更改
- `git reset --hard HEAD`：恢复到上一次提交的状态

### 远程操作
- `git pull`：从远程仓库拉取并合并代码
- `git push`：推送本地分支到远程仓库

以上是常用 Git 命令的汇总，帮助快速上手和管理代码。