---
title: Hexo 設定
cover: /img/home-bg.jpg
date: 2018-12-07 13:40:23
categories: ['misc']
tags: ['hexo']
---
### Hexo 
每個工程師都有一段踏入不歸路的小故事, 我的程式學習之路是從寫 blog 開始, 從改 css 到自己架設 wordpress 學了 php, 用了 [octopress](http://octopress.org/), [hexo](https://hexo.io/) 學了 git。總而言之如果你不嫌麻煩 `HEXO` 是個很棒的 blog 寫作套件, 方便讓你把 blog hosting 在 Github 上。

### Setup
[Hexo docs](https://hexo.io/zh-tw/docs/#%E5%AE%89%E8%A3%9D%E9%9C%80%E6%B1%82), 需要 node.js 環境支援。
```bash
node --version            # v11.2.0
npm --version             # 6.4.1
npm install hexo-cli -g   # -g 安裝在 global
```

建立 blog 資料夾, 我自己是習慣在自己的使用者目錄下建立, 打開 `http://localhost:4000` 就能預覽上線後的樣子
```bash
cd ~
hexo init blog
cd blog
npm install
hexo g; hexo s
# INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.
```

### Theme
[Hexo-theme-clean-blog](https://github.com/klugjo/hexo-theme-clean-blog), 會選用這個 theme 是因為寫 google bloger 也是用這個, 字體跟排版對中文支援比較不適合, 但可以自己微調。

修改 `~/blog/theme/clean-blog/source/css/mixins.styl`
```
sans-serif()
  font-family -apple-system, BlinkMacSystemFont, 'Open Sans', 'Helvetica Neue', Helvetica, Arial, sans-serif
```

其他細節我忘了 XD。

### Github plugin
這個套件用來把 blog hosting 到 Github 上面
```bash
cd ~/blog
npm install hexo-deployer-git --save
```

修改 _config.yml
```bash
vim  ~/blog/_config.yml
```

```bash
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/your-github-account/your-github-account.github.io.git
  branch: master
  message:
```

部署到 Github 上
```bash
hexo d
```

### Rss plugin (option)
如果希望其他人可以用 Rss 瀏覽你的 blog 的話可以考慮安裝一下這個 plugin。我自己有遇到 `PCDATA invalid Char value 8` [Issue](https://github.com/hexojs/hexo-generator-feed/issues/35) 還沒處理。
```bash
npm install hexo-generator-feed --save
```

修改 _config.yml
```bash
vim ~/blog/_config.yml
```

```bash
# Feed Atom
# https://github.com/hexojs/hexo-generator-feed
feed:
  type: atom
  path: atom.xml
  limit: 10
```

```bash
hexo g            # 這個指令除了產生文章頁面外, 可以在 http://localhost:4000/atom.xml 看到你的 rss 文章
```

### Disqus 設定
[Disqus](https://disqus.com/) 是留言管理套件, [Hexo-theme-clean-blog](https://github.com/klugjo/hexo-theme-clean-blog), 有支援 Disqus, 設定起來很方便, 只要申請好把相關資訊設定在 `blog/themes/clean-blog/_config.yml` 就好了。

```bash
vim ~/blog/themes/clean-blog/_config.yml
```

```bash
# Comments. Choose one by filling up the information
comments:
  # Disqus comments
  disqus_shortname: your-disqus-shortname
```

### Google Analytics 設定
[Google Analytics](https://analytics.google.com/analytics/web/) 網站分析工具, 用來分析流量, 瀏覽者行為的工具, 在寫文章時可以參考一下報表做改進, 一樣 [Hexo-theme-clean-blog](https://github.com/klugjo/hexo-theme-clean-blog), 有支援 Google Analytics, 設定一下就好。

```bash
vim ~/blog/themes/clean-blog/_config.yml
```

```bash
# Google Analytics Tracking ID
google_analytics: your-google-analytics-tracking-id
```

### 提交 sitemap 
`sitemap.xml` 是給搜尋引擎爬蟲閱讀的文件, 沒提交也不會怎樣, 只是更新到搜尋引擎上會比較慢一點而已。到 [Google search console](https://search.google.com/search-console/about), 沒意外的話設定好 web propertity 把網址填上去它就會自動去爬, 不太需要設定什麼。

```bash
hexo g            # 這個指令除了產生文章頁面外, 可以在 http://localhost:4000/sitemap.xml 看到你的 sitemap
```
Google search console 後台
![google-search-console](/img/setup-hexo/google-search-console.png)

### 看板娘 (option)
不要問, 但很厲害, [Hexo-helper-live2d](https://github.com/EYHN/hexo-helper-live2d/blob/master/README.zh-CN.md)

### Version Controller
除了 hosting 要開一個 Repository, 記得另外開一個 Repository 做版本管理, 把 blog 相關設定跟現有的文章 commit 上去, 方便之後在不同的電腦上編寫。

```bash
git clone your-blog-repo
cd your-blog
npm install
hexo new post your-post
```

### Ref
* [Hexo docs](https://hexo.io/)
* [Xuan's Blog - hexo config](https://coffee0127.github.io/blog/2016/08/09/hexo-configuration/)
* [Hexo SEO - 簡書](https://www.jianshu.com/p/c20bb9df1867)