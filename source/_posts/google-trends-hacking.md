---
title: google-trends-hacking
cover: /img/home-bg.jpg
date: 2018-11-26 23:02:02
categories: hacking
tags: misc
---
### 給我資料
最近試著在蒐集一些看起來有點用途的資料, Google Trends 看起來是一個 “滿有價值” 的資料平台, 是 Google 蒐集的全球趨勢資料, 包含最近流行的關鍵字, 關鍵字與關鍵字的比較, etc., 但 Google 並沒有提供相關的 API 介面, 所以要取得資料的做法我第一個想到的是 Crawler, 於是順手找了一下大神的開源專案, 左看右看上看下看, 都覺得太複雜不順手XD。

### Hack Fun
剛好讀到一篇很有趣的文章 [hacking the google trends api](http://techslides.com/hacking-the-google-trends-api), 文章內容提到了幾個有趣的 resource

這是 2014 年的 Resource
* [Google Trends? Visualization](https://trends.google.com/trends/hottrends/visualize?pn=p1)
* [Google Trends? Top-Charts](https://trends.google.com/trends/topcharts)
* [Hot Trends Rss Feed](https://trends.google.com/trends/hottrends/atom/feed?pn=p1)

當然還有今年（2018）最重要的世足趨勢
* [World-Cup-2018](https://googletrends.github.io/world-cup-2018/?lang=en&view=GB-ENG)
* [Knowledge Graph](https://www.google.com/intl/es419/insidesearch/features/search/knowledge.html)

### Chrome DevTools
2014 年那篇 hacking 有點過期了, 所以我稍微調整了一下做法,
以 Google-Trends 每日搜尋趨勢 為例, 先取得 uri。

* 打開 Chrome DevTools
* 點選 Network Tab
* 勾選 Preserve log
* 重整頁面
* 觀察這個頁面的 Requests 行為
* 找到一個像這樣的請求, 複製他的 curl 語法

![hacking-with-chrome-devtools](/img/google-trends-hacking/google-trends-hacking-with-chrome.png)

```bash
curl 'https://trends.google.com/trends/api/dailytrends?hl=zh-TW&tz=-480&geo=TW&ns=15' 
-H 'dnt: 1' 
-H 'accept-encoding: gzip, deflate, br' 
-H 'accept-language: zh-TW,zh;q=0.9,en-US;q=0.8,en;q=0.7' 
-H 'user-agent: Mozilla/5.0 ...' 
-H 'accept: application/json, text/plain, */*' 
-H 'referer: https://trends.google.com/trends/trendingsearches/daily?geo=TW' 
-H 'authority: trends.google.com' 
-H 'cookie: ...' 
-H 'x-client-data: ...' 
--compressed
```

### 對 query 的猜測
hl: Location 或是 Language
tz: timezone 
ns: ??
可以得到, 像這樣的資料
![hacking-result-json](/img/google-trends-hacking/google-trends-hacking-json.png)

### That’s Rock
簡單地用 Retrofit2 + Okhttp3 做個接口(沒更新了)
https://github.com/jerry80409/open-data-demo

### Reference
沒實際使用過這些 Reference, 但可從一些文件去推斷 google trends 的 uri

* [npm-google-trends-api](https://www.npmjs.com/package/google-trends-api)
* [pytrends](https://github.com/GeneralMills/pytrends)
* [fcu.edu.tw](http://myweb.fcu.edu.tw/~mhsung/Research/InformationSystem/JSON/JSON_24.htm)