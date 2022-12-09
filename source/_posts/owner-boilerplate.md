---
title: 微框架 owner-boilerplate 介紹
cover: /img/home-bg.jpg
date: 2018-11-28 13:08:09
categories: ['java']
tags: ['java', 'owner-boilerplate']
---
![owner-boilerplate](/img/owner-boilerplate/owner-aeonbits.png)
### Owner boilerplate
[Owner boilerplate](http://owner.aeonbits.org/docs/welcome/) 是我自己還滿喜歡用的一個微框架, 適合維運在 aws lambda 服務, 如果你的專案有複雜的 config properties, 如: mysql connection info, redis connection info, aws setting, etc. 有的沒的, 後端宅宅通常都要處理一大堆的服務, 就很推薦用這個, 
可以幫你把那些設定整理的很乾淨, 還有支援 config 的 [hot reload](http://owner.aeonbits.org/docs/reload/), 很棒吧!

### Setup
java8 support
```xml
<dependencies>
        <dependency>
            <groupId>org.aeonbits.owner</groupId>
            <artifactId>owner-java8</artifactId>
            <version>1.0.6</version>
        </dependency>
</dependencies>
```

### Demo
[http://owner.aeonbits.org/docs/usage/](http://owner.aeonbits.org/docs/usage/)


