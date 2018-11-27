---
title: osx 安裝 homestead 與 laravel
cover: /img/install-laravel-homestead/harshil-gudka-455166-unsplash.jpg
date: 2018-11-27 14:06:26
categories: ['laravel']
tags: ['laravel', 'homestead']
---
### Vagrant
我對 vagrant 的理解就是, 它在 virtual-box, VMware 的架構上, 再做一層抽象, 方便在不同的 providers 建立 instance
```bash
brew cask install vagrant
```

### VirtualBox
用 virtual-box 長大的, 所以沒特別選用 hyper-v
```bash
brew cask install virtualbox
```

### Homestead
簡單的理解為 image, 裡面包含 ubuntu 16.04, mysql, nginx, 基本的 php 運作環境與相關套件
```bash
vagrant box add laravel/homestead       # install vagrant VM
vagrant box list                        # list vagrant VMs
```

### Initial Homestead
這個步驟, 主要用來初始化 Homestead, 對應 local 的 port, cpu, memory, etc., 下面有簡單的對 Homestead.yml 修改的範例
```bash
git clone https://github.com/laravel/homestead.git ~/Homestead
cd ~/Homestead
git checkout v7.9.0
bash init.sh
```

### Build a Laravel Project
```bash
mkdir -p ~/code/
cd  ~/code
composer create-project --prefer-dist laravel/laravel blog
```

### Edit the Homestead.yml
```bash
cd ~/Homestead
vim Homestead.yml
folders:
    - map: ~/code/blog
      to: /home/vagrant/code/blog
sites:
    - map: blog.app
      to: /home/vagrant/code/blog/public
```

### Update Host
```bash
sudo vim /etc/hosts
```

添加
```bash
192.168.10.10   blog.app
Reload Vagrant
vagrant reload --provision
```

我在這階段, 遇到了 Vagrant up times out
```
Timed out while waiting for the machine to boot. This means that
Vagrant was unable to communicate with the guest machine within
the configured ("config.vm.boot_timeout" value) time period.

If the box appears to be booting properly, you may want to increase
the timeout ("config.vm.boot_timeout") value.
```

解決方式, 參考 [stackoverflow](https://stackoverflow.com/questions/41064388/laravel-homestead-vagrant-up-times-out)
是把 virtual-box 的 網路已連接 勾選
![virtual-box-cable-connected](/img/install-laravel-homestead/virtual-box-cable-connected.png)

記得重新 reload
```bash
vagrant reload --provision
```

### Initial Project
Homestead 的開發環境, 真的對第一次寫 laravel 有點難度, 要手動設定一堆東西 XD
```bash
vagrant ssh
cd ~/code/blog
cp .env.example .env
php artisan key:generate    # 產生 app key
```

建立 database
```bash
mysql -uhomestead -p
CREATE DATABASE homestead CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
exit;
```

建立 table schema
```bash
php artisan migrate
```

### Https not working
其實 localhost 不用這麼糾結 https, 但我有強迫症, 因為 [#754 Issue](https://github.com/laravel/homestead/issues/754) Https Not working, issue 裡面給的參考是把 `Homestead.yml` 的 sites 做調整, 把原本的 blog.app 更改為 blog.localhost 或 blog.test, 但 blog.localhost 但在 mac OS 在解析 host name 會無限循環, 所以在 mac OS 上只能用 *.test, 參考這個 [PR](https://github.com/laravel/homestead/pull/697)

```bash
sites:
    - map: blog.test
      to: /home/vagrant/code/blog/public
```

記得 `/etc/hosts` 也要修改
```bash
192.168.10.10   blog.test
```

再 Reload 一次, 好雷啊 Orz
```bash
vagrant reload --provision
```

然後, Chrome 打開, 讓我哭一次
```
http://blog.test/   # 可以瀏覽
https://blog.test/  # 不能瀏覽 Tears
```

因為 [#834 Issue](https://github.com/laravel/homestead/issues/834), `Homestead certificate` 不被 Chrome 信任,
讓我擦乾眼淚, 重新來過...

### 解法1
參考 [stackoverflow](https://stackoverflow.com/questions/48969083/how-to-get-https-certificate-working-on-local-laravel-homestead-site/49612084#49612084)
```bash
cd ~/Homestead
vagrant ssh -c 'cat /etc/nginx/ssl/blog.test.crt' >blog.test.crt    # 把 ssl crt 複製到 localhost
```
直接點擊 blog.test.crt 把設定為 永遠信任

### 解法2
[homestead support https](https://laravel-china.org/articles/7359/let-your-homestead-site-support-https)

### Done
![homestead](/img/install-laravel-homestead/laravel-homestead-https-demo.png)

