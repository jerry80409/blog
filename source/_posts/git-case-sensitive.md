---
title: 修正 git remote 資料夾大小寫問題
cover: /img/home-bg.jpg
date: 2018-11-28 14:25:56
categories: ['git']
tags: ['git']
---
### TL;DR
團隊有統一命名原則, 所以得改一下

### core.ignoreCase
> If true, this option enables various workarounds to enable Git to work better on filesystems that are not case sensitive, like FAT. For example, if a directory listing finds "makefile" when Git expects "Makefile", Git will assume it is really the same file, and continue to remember it as "Makefile".
> The default is false, except git-clone[1] or git-init[1] will probe and set core.ignoreCase true if appropriate when the repository is created.

`core.ignoreCase` 預設是 `false` (大小寫敏感), 所以如果沒有改這個 config, `push` 上去就會長出兩個名稱一樣, 但大小寫寫法不一樣的資料夾, `FolderA` 跟 `foldera`, 但我還是不建議把 `core.ignoreCase` 改為 `true`, 怕之後開發一團亂之類的。

### 手動解
```bash
git mv improper_Case improve_case2
git mv improve_case2 improve_case
git commit -m "modified folder cases"
```