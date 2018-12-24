---
title: Java split empty strings
cover: /img/home-bg.jpg
date: 2018-12-24 19:05:23
categories: ['java']
tags: ['java']
---
### Sample
在 code review 的時候發現一個有趣的狀況, 預期 `.split(",")` 處理完結果應該是 5, 然而實際上卻是 3
```java
String str = "a,b,c,,";
String[] ary = str.split(",");

// result 3
System.out.println(ary.length);
```

### 改用 split(regex, limit)
* 如果 limit > 0, 最終處理的 array 長度不會大於 limit, regex express 匹配的次數最多為 n - 1 次,
* 如果 limit < 0, regex express 會盡可能的處理匹配, 包含對 空字串 匹配的問題,
* 如果 limit = 0, regex express 會儘可能地處理匹配, 但會放棄處理 空字串 匹配的問題。

```java
@Test
public void assert_split_1() {
    String example = "boo:and:foo";
    String[] act = example.split(":", 2);
    assertEquals("boo", act[0]);
    assertEquals("and:foo", act[1]);
}

@Test
public void assert_split_2() {
    String example = "boo:and:foo";
    String[] act = example.split(":", 5);
    assertEquals("boo", act[0]);
    assertEquals("and", act[1]);
    assertEquals("foo", act[2]);
}

@Test
public void assert_split_3() {
    String example = "boo:and:foo";
    String[] act = example.split(":", -2);
    assertEquals("boo", act[0]);
    assertEquals("and", act[1]);
    assertEquals("foo", act[2]);
}

@Test
public void assert_split_4() {
    String example = "boo:and:foo";
    String[] act = example.split(":", -1);
    assertEquals("boo", act[0]);
    assertEquals("and", act[1]);
    assertEquals("foo", act[2]);
}

/**
 * 等同於 .split(regex)
 */
@Test
public void assert_split_5() {
    String example = "boo:and:foo";
    String[] act = example.split("o", 0);
    assertEquals("b", act[0]);
    assertEquals("", act[1]);
    assertEquals(":and:f", act[2]);
}

```

### Conclusion
在使用 `.split(regex)` 盡量改為 `.split(regex, limit)` 去避免空字串切分的問題。
```java
String str = "a,b,c,,";
String[] ary = str.split(",", -1);

// result 5
System.out.println(ary.length);
```