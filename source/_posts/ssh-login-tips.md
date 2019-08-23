---
title: SSH 登入小技巧
cover: /img/home-bg.jpg
date: 2019-08-19 11:04:39
categories: ['dev_ops'] 
tags: ['ssh']
---
### 設定 /etc/hosts
此文件是用來做 DNS (Domain-Name-Service), 編輯 `/etc/hosts` 可以讓機器辨識其他主機, 所以可以給 target server 一個 Domain Name, 方便 Localhost 辨識與登入。
```bash
sudo vim /etc/hosts
```
```bash
# 加入一組 
192.168.10.10   target.com
```

### 使用 ssh-key 
別再用密碼啦, ssh (Secure SHELL protocol) 本身就是希望你能透過 `非對稱金鑰` 來進行連線, 非對稱金鑰通常是由一組 `public key`, `private key` 組成, 所以先在本地生成一組 ssh key

```bash
# ssh key generated
ssh-keygen -t rsa -b 2048

Generating public/private rsa key pair.
Enter file in which to save the key (/home/username/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/username/.ssh/id_rsa.
Your public key has been saved in /home/username/.ssh/id_rsa.pub.
```
---
上傳到 target server, 這個動作主要是把 public key 上傳
```bash
ssh-copy-id user@target.com
```
---
login 就變簡潔了
```
ssh user@target.com
```