# Git学习记录
## 分支管理
### 远程分支
1. github增加新仓库
```sh
git init
git remote add origin xxx.git
git push -u origin master
```
### 分支操作
1. 查看所有分支，支持正则`git show-branch 'bug/*'`
	- `-r`列举远程分支
	- `-a`显示特性分支和远程分支
2. 创建分支 `git branch [starting-commit]`
3. 检出分支 `git checkout branchName`
4. 创建并检出分支 `git checkout -b BranchName`
5. 删除分支 `git branch -d BranchName`

### 分支合并
1. 切换到目标分支，然后合并分支
```sh
git checkout master
git merge other_branch
```

## 追溯历史
1. 查找单个文件的某一行`git blame -L linenum, filename`
2. 查找log中涉及到某个关键字的commit. `git log -Sstring --pertty=oneline --abbrev-commit init/version.c`
3. git bisect是二分查找法，在good和bad的提交之间反复切换(相当于匿名分支)。
```
git bisect start
git bisect reset
git bisect good
git bisect bad
git bisect log
```

## 比较差异
1. 工作目录 vs 索引：`git diff`
2. 工作目录 vs 给定提交: `git diff commit`
3. 索引变更 vs 给定提交: `git diff --cached commit`
4. 给定提交 vs 给定提交: `git diff commit commit`

## 更改提交
1. 更改 工作目录、索引、仓库 `git reset [-soft | -mixed | -hard]`
2. 会退提交 `git revert commitId`
3. 在当前工作目录应用指定提交，一般用于验证分支 `git cherry-pick commitId`

## GitHub链接
1. 下载git
2. 创建git账户和邮箱
3. 创建ssh密钥 `ssh-keygen -t ed25519 -C "youremail@example.com"` 或者 `ssh-keygen -t rsa -b 4096 -C "youremail@expamle.com"`
4. GitHub账户添加ssh公钥