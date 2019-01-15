---
title: leet-code-535-encode-and-decode-tinyurl
cover: /img/home-bg.jpg
date: 2019-01-09 19:25:31
categories: ['misc']
tags: ['leetcode']
---
### Puzzle
[https://leetcode.com/problems/encode-and-decode-tinyurl/](https://leetcode.com/problems/encode-and-decode-tinyurl/)
題目本身就是要實作短網址, 我覺得這題真的是殺手級的題目, 很多地方可以討論, 且在實作的過程中還不能使用任何的 Digest 技術, Base64, MD5, SHA, 通通都不能用

Example:
Inpute: https://leetcode.com/problems/design-tinyurl
tinyurl: http://tinyurl.com/4e9iAk 

### Thought
先看一下 url 結構吧
```
https:// host / path1 / path2 ? key1=value2&key2=value2 #anchor
```

然後 url 允許的字符
```
* a ~ z (26)
* A ~ Z (26)
* 0 ~ 9 (10)
* !*'();:@&=+$,/?#[] (保留字, 會被 urlencode 成 % 開頭 + Hex, e.g. %20 表示空白)
```

* 解法大概是需要 `Map` 的結構做 key, value 的查詢, key 存 hash, value 存 original url
* 重點就是 key 的設計, 要盡量減少碰撞, 如果是用 `Redis`, `MySQL` 之類來設計的話, 可以想一下如何在大量的資料下做 select 
* 還有試了 Digest, Base64, MD5, SHA 都不能用, XD
* 粒度最大的是 host, 如果是資料庫的話, 可以考慮把這個欄位拿來做 Partition
* path 跟 query 粒度比較細, 只要想好這邊的 digest 就可以有效減少碰撞, 但我一點概念都沒有喔 

### Solution
#### * 直接用 java HASH code, (其實我試了很多種方法)
```java
class EncodeAndDecodeTinyUrl {

    private static final String baseUrl = "http://tinyurl.com/";
    private static final Map<String, String> HASH_MAP = new HashMap<>();

    /**
     * Encodes a URL to a shortened URL.
     *
     * @param longUrl
     * @return
     */
    public String encode(String longUrl) {
        HASH_MAP.put(String.format("%h", longUrl), longUrl);
        return baseUrl + String.format("%h", longUrl);
    }

    /**
     * Decodes a shortened URL to its original URL.
     *
     * @param shortUrl
     * @return
     */
    public String decode(String shortUrl) {
        return HASH_MAP.get(shortUrl.replace(baseUrl, ""));
    }
```
---
#### * Base62 + Redis 解法
[https://hackernoon.com/url-shortening-service-in-java-spring-boot-and-redis-d2a0f8848a1d?fbclid=IwAR3kklW3xtlHS4FcJ6MZDkvtoZu-ord2KXKN-TUQWdPAJ4L0vED7sJweegU](https://hackernoon.com/url-shortening-service-in-java-spring-boot-and-redis-d2a0f8848a1d?fbclid=IwAR3kklW3xtlHS4FcJ6MZDkvtoZu-ord2KXKN-TUQWdPAJ4L0vED7sJweegU)

這篇文章裡面的解法, 是透過 Redis 做類似 Map 的事情,
1. 先用 [id, original] 的方式建立字典
2. 再拿 `id / 62` 取得 **商數** , **餘數** 去算出對應的 **Base62** 的 tinyUrl
3. 一樣透過上面的算法, 反推 id, 再從 Redis 取得 original url

缺點就是會遇到 id 數量上限的瓶頸, 我自己稍微想了一下, 我大概會把 host 資訊存到 main table, 雖然還是有機會遇到 id 上限的問題, 但會比較低一點, 然後 path 做 digest 存 secnod table, 後續的 query + anchor 做第二次 digest 存 third table. 這樣效能會比較差一點, 但碰撞機會應該會比較低 (還沒深入研究碰撞計算 XD)

大概會是這樣的做法
```
output: http://tinyurl.com/host(id)/path(digest)/query+anchor(digest)
Example: http://tinyurl.com/1/abcde/abcde
```