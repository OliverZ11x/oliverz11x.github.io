```python
git命令
git status查看文件状态
 
git add <file>添加到暂存区
 
git commit -m '说明'提交到本地
 
git push提交到远程仓库
 
git commit --amend重新提交前一个提交
 
git reset <--soft | --mixed | --hard> <file>撤销暂存区的文件
    hard重置stage区和工作目录
    soft保留工作目录，并把重置 HEAD 所带来的新的差异放进暂存区
    mixed保留工作目录，并清空暂存区(默认)
 
git checkout <file>撤销未暂存的修改文件
 
git checkout <branch> <file>将某个文件回退到指定提交
 
git log日志（hash码）
 
git remote -v远程仓库地址
 
git remote add <name> <gitUrl>添加远程仓库
 
git fetch <name>拉取远程仓库
 
git push <remote name> <branch name>推送到某远程仓库的某一个分支
 
git remote show <remote name>查看某个远程仓库
 
git remote rename <old remote name> <new remote name>修改仓库名
 
git tag -a <name> -m '说明'创建附注标签（比如版本号）
 
git tag <name>创建轻量标签（比如版本号）
 
git tag -a v1.2 <版本hash值>创建历史提交的标签
 
git push <remote name> <推送的标签>推送标签到远程仓库
 
git push <remote name> --tags推送所有表签到远程仓库
 
git tag -d <name>删除标签
 
git push origin :refs/tags/<name>删除远程仓库标签
 
git branch <name>创建分支
 
git checkout<name> 切换分支
 
git checkout -b <name>创建并切换到分支
 
git branch -d <name>删除分支
 
git push <remote name> --delete <name>删除远程分支
 
git branch --merged合并到当前分支的分支
 
git branch --no-merged没有合并到当前分支的分支
 
git rebase <name>变基操作(当前分支合并到name分支和merge的当前分支不同)
 
git merge <name>将name分支合并到当前分支

```