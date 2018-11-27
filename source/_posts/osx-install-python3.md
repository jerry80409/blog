---
title: osx-install-python3
cover: /img/osx-install-python3/chris-ried-512801-unsplash.jpg
date: 2018-11-27 21:23:24
categories: ['python']
tags: ['python3']
---
### python2
OSX 自帶 python2, 查了一下, [官方建議](https://docs.python.org/3/using/mac.html) 是說不應該刪,

> The Apple-provided build of Python is installed in /System/Library/Frameworks/Python.framework and /usr/bin/python, respectively. You should never modify or delete these
```bash
python --version
Python 2.7.15
```

### ptyhon3
忘記自己在什麼時候安裝好的, 所以 upgrade 就好
```bash
brew upgrade python3
```

upgrade 完成後, 會說明 OSX 的預設 python2.7 的相關環境與設定,
而 python3 的相關環境與設定分別是 `python3`, `python3-config`, `pip3`, etc.
這些都安裝在 `/usr/local/opt/python/libexec/bin` ,
如果你還想要使用 ****python2.7**, 再自行安裝 `brew install python@2`
```bash
Updating Homebrew...
==> Upgrading 1 outdated package, with result:
python3 3.6.5 -> 3.7.0

...
Python has been installed as
  /usr/local/bin/python3

Unversioned symlinks `python`, `python-config`, `pip` etc. pointing to
`python3`, `python3-config`, `pip3` etc., respectively, have been installed into
  /usr/local/opt/python/libexec/bin

If you need Homebrew's Python 2.7 run
  brew install python@2

Pip, setuptools, and wheel have been installed. To update them run
  pip3 install --upgrade pip setuptools wheel
```

提示還說要升級, pip, setuptools, wheel
```bash
pip3 install --upgrade pip setuptools wheel
```

這樣一來基本的開發環境已經完成了, 就先照 官方文件 來個互動吧
```bash
$ python3
Python 3.7.0 (default, Aug 22 2018, 15:22:56) 
[Clang 9.0.0 (clang-900.0.39.2)] on darwin
Type "help", "copyright", "credits" or "license" for more information.

>>> the_world_is_flat = True
>>> if the_world_is_flat:
...     print("Be careful not to fall off!")
...
Be careful not to fall off!
```

先說說第一個雷, 就是 print 之前要 tab (4 個 space) 一下 :)