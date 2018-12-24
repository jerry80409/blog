---
title: Spring-data JPA 搭配 Optional 的使用
cover: /img/home-bg.jpg
date: 2018-12-24 17:25:36
categories: ['java']
tags: ['java', 'spring-data']
---
### Optional Class
這是 java8 才有的 class, 一樣是用容器的概念 (container) 來封裝一個 [value-base](https://docs.oracle.com/javase/8/docs/api/java/lang/doc-files/ValueBased.html) 的 class, 無法直接讓你取得 value 的狀態, 而是需要透過 `.get()` 的方式取得, 或是透過 `.isPresent()` 來檢查是否 non-null。在 JPA select 資料的時候超好用的。我自己覺得寫得很棒的一篇 [Tony blog](http://blog.tonycube.com/2015/10/java-java8-4-optional.html)。

### Value-base Class
`java.util.Optional` 與 `java.time.LocalDateTime` 皆屬於 values-base class, 下面的特點用 `java.time.LocalDateTime` 就很好理解。
* 具有 final 或 immutable 的特質。
* 已有 `equals()`, `hashCode()`, `toString()` 的方法, 這些狀態來自於 class instance 本身, 而不是從其他 variable 計算得來
* make no use of identity-sensitive operations such as reference equality (==) between instances, identity hash code of instances, or synchronization on an instances's intrinsic lock; (這句後面 identity hash code 跟 synchronization 的情境不是很理解)
* 應該使用 `equals()`, 而不是 `==`,
* 沒有 constructors 方法可以被建構, 而是透過 Factory 方法實例化, 
* 當 `equals()` 發生時, 兩個 instance 是可以互換的, 且不影響後續的運算。

簡單來說像這樣
```java
@Test
public void assert_equals() {
    LocalDateTime date = LocalDateTime.parse("2017-02-03T12:30:30");
    LocalDateTime date1 = LocalDateTime.parse("2017-02-03T12:30:30");
    assertTrue(date1.equals(date));
}

@Test
public void assert_hashcode_equals() {
    LocalDateTime date = LocalDateTime.parse("2017-02-03T12:30:30");
    LocalDateTime date1 = LocalDateTime.parse("2017-02-03T12:30:30");
    assertEquals(date.hashCode(), date1.hashCode());
}

@Test
public void assert_to_string() {
    LocalDateTime date = LocalDateTime.parse("2017-02-03T12:30:30");
    assertEquals("2017-02-03T12:30:30", date.toString());
}
```

### Jpa and Optional
一般來說用 Jpa 做 CRUD 都會透過 JpaRepository
```java
@Repository
public interface UserInfoRepository extends JpaRepository<UserInfo, String> {

    UserInfo findByName(String name);
}
```

這樣寫其實也沒什麼不合理, 只是在商業邏輯處理時, 要多一個 if 判斷, 判斷是否有查詢到 userInfo 的資料.
```java
public void business() {

    UserInfo user = userInfoRepository.findByName("dumdum");

    if (Objects.isNull(user)) {
        // throw exception

    } else {
        // do something
    }

}
```

改為用 Optional 試試看
```java
@Repository
public interface UserInfoRepository extends JpaRepository<UserInfo, String> {

    Optional<Userinfo> findByName(String name);
}
```

```java
public void business() {

    Optional<Userinfo> userOpt = userInfoRepository.findByName("dumdum");
    UserInfo user = userOpt.orElseThrow(() ->
            new Exception("查無 userInfo(dumdum) 資訊."));

    // do something
}
```

找不到 user 還可以替換為預設 user, 這樣就不用處理 Exception, 當然並不是所有的情境都可以預設一個 instance.
```java
public void business() {

    Optional<Userinfo> userOpt = userInfoRepository.findByName("dumdum");
    UserInfo user = userOpt.orElse(new UserInfo("Rocko")));

    // do something
}
```