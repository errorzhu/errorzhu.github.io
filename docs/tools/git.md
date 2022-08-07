# git 常用命令



## 一图胜千言

![](images\wiki\git.png)

## stash

```
将当前改动保存至stash 队列
git stash
git stash list
git stash pop
git stash clear
```

## fetch 和 pull的区别

```
fetch 获取远程仓库,并不合入本地仓库
pull = fetch + merge/fetch + rebase
```

## cherry pick

```
挑选某分支部分提交至当前分支
git cherry-pick commit_id
```

## rebase vs merge

```
merge产生了merge commit 而且有交叉
rebase 重新追加提交，保持线性
```

