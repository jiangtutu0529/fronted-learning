#### Git
git是版本控制工具，用于多人协作，版本追踪

```
git init

git clone

git add
git commit 
git push

git fetch只拉取
git pull拉取并合并

git merge 合并
git rebase变基

git branch
git checkout 切换分支
git checkout -b 创建并切换分支

git status
git log

git reset 撤销暂存区操作，强推无记录
git revert 回退 有记录

git stash
git stash pop
git stash apply
git stash clear

git tag


```

##### merge和rebase的区别
merge将一个分支合并到另一个分支，保留分支历史，创建合并提交
rebase重写提交历史，保持线性历史，更简介，不适合共享分支

##### 如何找回被删的分支
//查找被删的分支的提交hash id
git reflog
//根据hash恢复
git checkout -b branch-name <cimmit-hash>

