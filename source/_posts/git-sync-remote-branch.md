---
title: git-sync-remote-branch
cover: /img/git-sync-remote-branch/simon-launay-233889-unsplash.jpg
date: 2018-11-28 14:37:25
categories: ['git']
tags: ['git']
---
### 同步 local branch
在團隊開發過程中, 多多少少都會有些意外, 還好這個不是災難性的意外, (笑
這個問題是發生在 `git pull origin master`, `pull master` 其實會做兩件事 `git fetch` 跟 `git merge`, 因為本地還存在 a branch 所以會檢查 remote 的 a branch 做 merge, 但 remote 已經 PR 結束了, 被 merge 到 master 裡面, 按照開發流程 merge 完成的 branch 會被刪除, 所以 remote 的 a branch 沒了, 因此噴了一個警告。
```
error: cannot lock ref 'refs/remotes/origin/feature/lambda/dynamoDb': 'refs/remotes/origin/feature/lambda' exists; cannot create 'refs/remotes/origin/feature/lambda/dynamoDb'
```

SourceTree 給我的提示是這樣, 所以就查了一下 `prune` 的用途
```
error: some local refs could not be updated; try running
 'git remote prune origin' to remove any old, conflicting branches
```

### Git remote prune
先看一下文件
```
prune
           Deletes stale references associated with <name>. By default, stale
           remote-tracking branches under <name> are deleted, but depending on
           global configuration and the configuration of the remote we might
           even prune local tags that haven't been pushed there. Equivalent to
           git fetch --prune <name>, except that no new references will be
           fetched.

           See the PRUNING section of git-fetch(1) for what it'll prune
           depending on various configuration.

           With --dry-run option, report what branches will be pruned, but do
           not actually prune them.
```

所以我比較建議的做法是, 先跑一次 `--dry-run` 看看哪些會刪除, 再執行 `prune origin`, 這樣就能保持 local 跟 remote 的 branch 同步。
```
git remote prune --dry-run origin    # 確定一下有沒有可能會刪錯
git remote prune origin
```