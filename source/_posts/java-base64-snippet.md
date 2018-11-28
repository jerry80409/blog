---
title: Java8 base64
cover: /img/home-bg.jpg
date: 2018-11-28 13:45:49
categories: ['java8']
tags: ['base64']
---
### Base64 
java8 支援 `Base64.Decoder`, `Base64.Encoder`, [API docs](https://docs.oracle.com/javase/8/docs/api/java/util/Base64.html)
```java
final Base64.Decoder decoder = Base64.getDecoder();
final Base64.Encoder encoder = Base64.getEncoder();
final String text = "字串文字";
final byte[] textByte = text.getBytes("UTF-8");

//編碼
final String encodedText = encoder.encodeToString(textByte);

//解碼
System.out.println(new String(decoder.decode(encodedText), "UTF-8"));
```

### 基本原理
text -> ascii -> 取 6bits -> mapping 64 code -> 空白捕 '='
[https://zh.wikipedia.org/wiki/Base64](base64 wiki : https://zh.wikipedia.org/wiki/Base64)
