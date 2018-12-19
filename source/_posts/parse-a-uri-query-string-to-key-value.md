---
title: parse-a-uri-query-string-to-key-value
cover: /img/home-bg.jpg
date: 2018-12-13 17:41:11
categories: ['java']
tags: ['java']
---
### Purpose
解析 url 的 query string, 解析為 Map 的資料型態, 使用 apache httpcomponents。
* 做 urlDecode 處理
* 沒有任何 query String 回傳 Empty Map
* 確保只處理 key-value 結構的 query String

```java
@Test
public void assert_parse_query_map_is_work() {
    String uri = "http://www.domain.com?key1=value1&key2=value2";
    Map<String, String> act = UriUtil.parseQueryMap(uri);
    assertEquals("value1", act.get("key1"));
    assertEquals("value2", act.get("key2"));
}

@Test
public void assert_parse_not_complete_query_is_work() {
    String uri = "http://www.domain.com?wtf";
    Map<String, String> act = UriUtil.parseQueryMap(uri);
        
    // assert empty map
    assertEquals(0, act.size());
}
```


### Dependency
```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.6</version>
</dependency>
```

### Example
```java
package com.example;

import org.apache.http.client.utils.URIBuilder;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.UnsupportedEncodingException;
import java.net.URI;
import java.net.URISyntaxException;
import java.net.URLDecoder;
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.Objects;

public class UriUtil {

    static final Logger log = LoggerFactory.getLogger(UriUtil.class);

    private UriUtil() {
    }

    public static Map<String, String> parseQueryMap(final String uriStr) {

        Map<String, String> queryPairs = new LinkedHashMap<>();

        try {
            final URI uri = new URIBuilder(uriStr).build();
            final String rawQuery = uri.getRawQuery();
            log.info("CurrentUrl Query: {}", rawQuery);

            // 過濾沒有 query string
            // 還有過濾無法成對 keyValue 的 query, e.g. http://host/path?123
            if (Objects.isNull(rawQuery) || !rawQuery.contains("=")) {
                return queryPairs;
            }

            final String[] pairs = rawQuery.split("&");

            final String utf8 = "UTF-8";

            for (String pair : pairs) {

                // 統一 decode
                final String deCodePair = URLDecoder.decode(pair, utf8);

                // check deCodePair 空字串,
                if (!deCodePair.isEmpty()) {
                    final String[] keyValue = pair.split("=");

                    // check is key value
                    if (2 == keyValue.length) {
                        final String key = keyValue[0];
                        final String value = keyValue[1];

                        queryPairs.put(key, value);
                    }
                }
            }

            log.info("Parser url: {} \n parameters: {}", uri.toString(), queryPairs);
            return queryPairs;

        } catch (URISyntaxException uriBuilderUrlException) {
            uriBuilderUrlException.printStackTrace();
            log.warn("UriBuilder url ({}) has exception: {}", uriStr, uriBuilderUrlException.getMessage());

        } catch (UnsupportedEncodingException urlDecodeException) {
            urlDecodeException.printStackTrace();
            log.warn("Decode url ({}) query has exception: {}", uriStr, urlDecodeException.getMessage());
        }

        return queryPairs;
    }
}
```