# Git学习记录
## 分支管理
### 远程分支
1. github增加新仓库
```sh
git init
git remote add origin xxx.git
git push -u origin master
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