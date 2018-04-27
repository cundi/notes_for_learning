# issue

## Your branch and 'origin/master' have diverged

1），你应该创建一个分支指向你当前的HEAD ，如果有分支可以不用创建（git checkout -b local_changes)

2） 现在我们可以追踪到分支上的变化，我们切回到origin/master ，git checkout origin/master

3）之后git reset origin/master 删除origin/master 分支上的一次提交

4）OK，完工。git status查看下。

## Changes not staged for commit