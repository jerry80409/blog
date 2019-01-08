---
title: leet-code-929-unique-email-addresses
cover: /img/home-bg.jpg
date: 2019-01-09 01:27:24
categories: ['misc']
tags: ['leetcode']
---
### Puzzle
[https://leetcode.com/problems/unique-email-addresses/](https://leetcode.com/problems/unique-email-addresses/)
輸入一組 emails array, 請依據 host 跟 domain 做 unique 處理,

1. host 名稱一樣, domain 名稱不一樣, 視為不同的 email
2. host 若有包含 ".", 則可以視為 ""
3. host 若有包含 "+", 則忽略後續的字串

在解這題的時候, 我的想法其實很簡單, 就只是單純的資料粒度判斷,
`domain` 的粒度最粗, 優先分類處理, `host` 其次

### Test
```java
@Test
public void test_1() {
    String[] emails = new String[]{
        "test.email+alex@leetcode.com",
        "test.e.mail+bob.cathy@leetcode.com",
        "testemail+david@lee.tcode.com"
    };

    int act = uniqueEmail.numUniqueEmails(emails);
    assertEquals(2, act);
}
```

### Solution
很直覺的用 lambda 就解出來了, 但效能很差, 約 200ms XD

```java
int numUniqueEmails(String[] emails) {
    return (int) Arrays.stream(emails)
        .collect(Collectors.groupingBy(e -> e.split("@")[1]))
        .entrySet()
        .stream()
        .map(groupMails -> groupMails.getValue().stream()
            .map(local -> local.replace(".", ""))
            .map(local -> local.split("\\+")[0])
            .distinct()
            .count())
        .count();
    }
```

要好一點的, 還是用 foreach 吧, 約 63ms XD
```java
int numUniqueEmails(String[] emails) {
    Set<String> result = new HashSet<>();
    for (String email: emails) {
        String[] split = email.split("@");
        String host = split[0];
        String domain = split[1];
        String newHost = host.replace(".", "").split("\\+")[0];
        result.add(domain + "@" + newHost);
    }
    return result.size();
}
```