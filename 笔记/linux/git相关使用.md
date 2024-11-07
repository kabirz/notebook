#git #worktree #subtree

worktree的用法
```shell
git worktree add <path> tag/HEAD/main/master
# 如果是想在当前的HEAD中使用新分支的话
git worktree add <path> [-b new_banrch] [tag/HEAD/main/master]
# 如果有子模块
cd <path>
git summodule update --init
```
参考：[git worktree 上手指南 - Sofiaღ - 博客园 (cnblogs.com)](https://www.cnblogs.com/caoshufang/p/16258975.html)
