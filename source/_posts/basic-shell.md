---
title: Shell 常用的基本指令
cover: /img/home-bg.jpg
date: 2018-11-26 22:48:31
categories: dev_ops
tags: bash_shell
---
### Docs
```bash
man bash    # 還滿詳盡的, 但讀的眼睛很酸, 問 google 比較快
```

### 預設的環境變數
```bash
env
```

### 環境變數與自訂變數
```bash
set
echo $$     # bash process 的 pid
echo $?     # 上個指令的執行狀況, 沒有錯誤會回傳 0
```

### crash by exception 
```bash
#!/usr/bin/env bash
set -e     # 當 script 發生 error 會中斷執行
set +e     # 當 script 發生 error 會繼續執行
```

### export
可以在 sub process 使用被 export 的變數
```bash
export var='foo bar'
bash         # 切換到 sub process
echo $var    # print 'foo bar’
```

### read
用來與使用者互動, 會讀取使用者的鍵盤輸入
```bash
echo 'pleas type your name?'
read name    # block 等待使用者輸入
echo $name
```

### array
集合, 陣列
```bash
a[0]=John
a[1]=18
a[2]=male

echo ${a[0]} is ${a[1]} age
# John is 18 age
```

### RANDOM
亂數,
```bash
echo $RANDOM            # 產生 Range 0 ~ 32767 隨機亂數 
echo $(((RANDOM%10)+1)) # 產生 Range 1 ~ 10 隨機亂數
```

### eval
evaluate(評估) 簡稱, 我不是很會用這個指令, 使用情境通常是把一個運算的結果, 迭代到新的變數裡面, 我翻譯得很怪, 看範例吧
```bash
days=365
year=days
echo \$$year         # $days, '\$' 做 escape character 輸出為 '$', $year 則輸出 'days'
echo eval \$$year    # 365, 等同於 echo $days
```

### alias
我超愛的指令, 用來 “簡化” 操作
```bash
alias ll='ls -alFh'     # -h 方便閱讀檔案 size
alias la='ls -Ah'
alias l='ls -CFh'
alias k='clear'
alias ..='cd ..'
alias md='mkdir -p'     # auto recursive create sub dirctory

alias lp='function _lp(){ lsof -n -iTCP:$1 | grep LISTEN; };_lp'    # 查看 port 被哪個 process 使用

alias psjar='ps aux | grep jar'    # 查看 jar process
alias rm='rm -i'        # 多一層 confirm 確認刪除, 更好的方法是改為 mv $@ ./trash
alias df='df -h'        # -h 方便閱讀 size
```

### source
不登出, 重新載入設定檔到目前 shell process, 在一些開發工具或語言為了方便在 shell 做一些事情, 通常都需要設定 $PATH, 設定完直接 source
編輯 .bashrc
```bash
vim ~/.bashrc
```

設定 Go 的開發環境, 在 ./bashrc 裡面添加
```bash
# go lang
export GOPATH="${HOME}/go"
export PATH="${GOPATH}/bin:${PATH}"
```

載入 .bashrc 到當下的 bash shell process
```bash
source ~/.bashrc
```

### 連續指令
```bash
command1; command2      # 執行完 cmd1 後接著執行 cmd2, 無論 cmd1 成功或失敗
command1 && command2    # 只要 cmd1 失敗, cmd2 就不執行
command1 || command2    # cmd1 失敗, 還是會執行 cmd2
```

### Redirect
這個要花點力氣記一下, 但用習慣就好了
```bash
0 # stdin, 標準輸入
1 # stdout, 標準輸出
2 # stderr, 錯誤輸出
```

看實際應用範例比較快
```bash
wc <file                    # 會計算 file 的行數, 字數, 字元數
ls -al 1>file               # 把 list directory file 的結果, 輸出到檔案裡
ls ./not-exist 2>file       # 由於 list not exist file, 所以噴錯 ls: cannot access ...到檔案裡 
command >/dev/null 2>&1     # 無論 cmd 執行的輸出的訊息, 或錯誤訊息, 皆拋出到 /dev/null (大家都稱呼這個路徑為 '黑洞')
```

關於 `2>&1` 的詳細說明 http://ibookmen.blogspot.com/2010/11/unix-2.html,
但還仍不是很懂, 不懂的東西, 就先記下他的用途跟結果就好 XD

`2>&1` 我常用的情境, 我在 run jar 都會引用 logback package 方便整理 log 訊息, 所以 stdout 的 log 訊息, 我都會捨棄掉
```bash
java -jar app.jar >/dev/null 2>&1
```

### pip
pip-line, 想像水管, 會依序執行指令

```bash
command | command2    # pip 把 cmd1 執行結果 pass 給 cmd2 繼續執行, e.g. ps aux | grep jar
cat /etc/group        # 查看 group

## 回傳 
# group-name : password : gid : members
# nobody:*:-2:
# nogroup:*:-1:

## 只要看 group-name
cat /etc/group | cut -d":" -f1        # 將 cat 的 stdout 傳給 cut 做 stdin; cut -d:切分符號 -f:取切分後的欄位 n
cat /etc/group | cut -d":" -f1 | sort # 同上, 並對 group-name a~z 排序
```
