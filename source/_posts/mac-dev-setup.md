---
title: mac 開發環境設定
cover: /img/home-bg.jpg
date: 2018-12-07 12:34:20
categories: ['misc']
tags: ['mac_os']
---
### Install Homebrew
```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### Iterm2
[https://sourabhbajaj.com/mac-setup/iTerm/](https://sourabhbajaj.com/mac-setup/iTerm/)
```bash
brew cask install iterm2
```

### Oh-my-zsh (option)
其實我不太擅長寫 `z_shell`, 單純只是覺得好看而已
```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
echo "source ~/.bash_profile" >> .zshrc
source ~/.zshrc
```

移除 oh-my-zsh
```bash
uninstall_oh_my_zsh
```

### Bash-it (option)
如果還是習慣寫 `bash_shell`, 我會推薦 [bash-it](https://github.com/Bash-it/bash-it) 這個套件。
這邊有多種選擇 [https://github.com/k4m4/terminals-are-sexy](https://github.com/k4m4/terminals-are-sexy), 從 zsh, fish, bash, putty, etc. 應有盡有。

### Fonts
[https://github.com/Homebrew/homebrew-cask-fonts/tree/master/Casks](https://github.com/Homebrew/homebrew-cask-fonts/tree/master/Casks)
```bash
brew tap caskroom/fonts
brew search nerd
brew cask install font-sourcecodepro-nerd-font
brew cask install font-sourcecodepro-nerd-font-mono
```

### Java
```bash
brew install java8 maven gradle      
brew cask install intellij-idea
```

### Python3
```bash
brew install python
source ~/.zshrc
python3 --version            # Python 3.7.1
pip3 --version               # pip 18.1 from /usr/local/lib/python3.7/site-packages/pip (python 3.7)
pip3 install pipenv          # (option) 自己習慣用 pipenv
pip3 install pytest          # (option) 自己習慣用 pytest
```

### Node
安裝 `nvm`, 用來切換 node.js 版本
```bash
brew install nvm
echo "export NVM_DIR="$HOME/.nvm" >> ~/.zshrc
echo ". "/usr/local/opt/nvm/nvm.sh" >> ~/.zshrc
nvm install stable
```

### Mysql
```bash
brew install mysql
brew services start mysql            # start mysql service
brew cask install tableplus          # mysql gui, support mysql_8.0
brew cask install sequel-pro         # mysql gui
```

### Docker CE
```bash
brew cask install docker
docker --version && docker-compose --version && docker-machine --version
# Docker version 18.09.0, build 4d60db4
# docker-compose version 1.23.2, build 1110ad01
# docker-machine version 0.16.0, build 702c267f
```

### Git
```bash
brew cask install sourcetree
```

### Common
```bash
brew cask install boom-3d            # 音效增強
brew cask install google-chrome      # 自己常用的瀏覽器
brew cask install telegram           # 通訊軟體
brew cask install evernote           # 筆記工具
brew cask install visual-studio-code # 什麼都能寫的萬用 IDE

brew install tree                    # show directory tree
brew install jq                      # 操作 json 工具
```

### REST client
[https://www.getpostman.com/](https://www.getpostman.com/)
```bash
brew cask install postman
```

### Google cloud platform tools
安裝 `gsutil`
```bash
curl https://sdk.cloud.google.com | bash
```

安裝 `gcloud`
```bash
curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-208.0.1-darwin-x86_64.tar.gz
tar -zxvf google-cloud-sdk-208.0.1-darwin-x86_64.tar.gz .
./google-cloud-sdk/install.sh
source ~/.zshrc
gcloud init
```

### Hexo (option)
寫 blog 的套件, 需要 node.js 環境
```bash
npm install hexo-cli -g
```
