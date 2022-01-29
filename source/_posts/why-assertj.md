---
title: 為什麼要推薦 AssertJ
cover: /img/home-bg.jpg
date: 2022-01-21 21:53:49
categories: ['test']
tags: ['test']
---
## 為什麼要推薦 AssertJ
1. 整合了 Junit4 的 Assertions
2. 整合了 Junit5 的 Assertions
3. 整合了 TestNG 的 Assertions
4. 也整合了 Fest2.x (看官方文件是說 AssertJ 本身就是 Fest2.x 的分支)

Junit4, Junit5 的整合, 就很適合當作團隊的基礎驗證基礎. 

### 官方提供 Junit 轉 AssertJ 小工具
我沒實際使用過, 有需要可以前往 [github](https://github.com/assertj/assertj-core/tree/main/src/main/scripts) 下載

```
# windows 作業系統
# in the directory containing the test files
convert-junit-assertions-to-assertj.sh
```

```
# OSX 作業系統
# in the directory containing the test files
convert-junit-assertions-to-assertj-on-osx.sh
```

### 流暢的結構方便閱讀
流暢的 stream 風格, 搭配 IntelliJ IDE 開發起來心情真的很好.
```java
@Test
@DisplayName("數字測試")
void assertNumbers() {
  
  // actual
  int num = 1;
  
  // assertins
  Assertions.assertThat(num)
    .isPostive()
    .isNotNegative()
    isGreaterThanOrEqualTo(0)
    .isIn(1, 2, 3);
}
```

### 支援 Optional 的特性
`Optional` 是 Java8 之後常用來處理 NPE 的一個特性, 在一些 method 的處理上, 為了要搭配 Java8 的 `stream()` 處理, 許多 method 都會設計為回傳 Optional 的型態.

```java
@Test
@DisplayName("Optional測試")
void assertOptions() {
  
  // actual
  Optional<String> opt = Optional.of("hello");

  // assertins
  Assertions.assertThat(opt)
    .isPresent()
    .isNotEmpty()
    .get()
    .isEqualTo("hello");
      
}
```

### SoftAssertions (不知道要怎麼翻譯)
在某些測試情境是需要整個流程走完, 再總結一次驗證, 比較常用的情境像是集合型的資料驗證, 或是多個流程耦合再一起要驗證的情境.   

```java
// 暫時想不到好的例子, 先用 List<String>
@Test
@DisplayName("SoftAssertions測試")
void assertSoftAssertions() {
  // actual
  List<String> member = Lists.newArrayList("Frodo", "Sam", "Gimli");

  // assertins
  try (var softly = new AutoCloseableSoftAssertions()) {
    softly.assertThat(member).isInstanceOf(List.class);
    softly.assertThat(member.size()).as("會員人數").isEqualTo(2);
    softly.assertThat(member.get(1)).as("1 號會員姓名").isEqualTo("Frodo");
  }
}
```

![assertJ](/img/why-assertj/assertJ.png)

驗證結果, 不會因為 *會員人數* 的驗證失敗, 就沒驗證 *1 號會員姓名*. SoftAssertions 能夠在測試過程中, 了解整個作業失敗所有細節的測試.

### 反向測試
反向測試, 也是測試情境中重要的一環, 特別是一些 Exception 的處理, 在處理 Exception 的測試, AssertJ 可以簡單協助你驗證 Root Exception.

```java
@Test
@DisplayName("Root Exception 測試")
void testRootThrowable() {
  var throwable = Assertions.catchThrowable(() -> 
    new IllegalStateException("狀態錯誤", new IllegalArgumentException("參數錯誤")));
  
  Assertions.assertThat(throwable)
    .isInstanceOf(IllegalStateException.class)
    .hasMessageContaining("狀態錯誤")
    .hasRootCauseExactlyInstanceOf(IllegalStateException.class)
    .hasMessageContaining("參數錯誤");
}
```

### 自定義的 Assertions
AssertJ 可以針對某個 domain / pojo 物件定義 Assert, 隨便用 coupon 折扣卷來舉例.

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
class Coupon {

  /**
   * 折扣代碼
   */
  String code;

  /**
   * 生效日期
   */
  LocalDate effected;

  /**
   * 失效日期
   */
  LocalDate expired;
}
```

自定義的 Assert 需繼承 `AbstractAssert<SELF, ACTUAL>`, SELF 與 ACTUAL 分別是 Java 採用 Java 的 Generic Types 設計, SELF 代表自定義的 Assert 實體, ACTUAL 代表你要驗證的 domain / pojo 物件.

這範例有點長了, 抱歉啦
```java
public class CouponAssert extends AbstractAssert<CouponAssert, Coupon> {

  protected CouponAssert(Coupon coupon, Class<?> selfType) {
    super(coupon, selfType);
  }

  protected CouponAssert(Coupon coupon) {
    super(coupon, CouponAssert.class);
  }

  /**
   * 依據 AssertJ 的風格, 從這個點開始 AssertJ 的測試
   */
  public static CouponAssert assertThat(Coupon actual) {
    return new CouponAssert(actual);
  }

  /**
   * 折扣碼物件, 需要有代碼
   *
   * @return
   */
  public CouponAssert hasCode() {
    if (StringUtils.isBlank(actual.getCode())) {
      failWithMessage("折扣碼不可為空.", null);
    }
    return this;
  }

  /**
   * 折扣碼物件, 代碼長度必須 5 碼
   *
   * @return
   */
  public CouponAssert codeLengthShouldBeFive() {
    if (5 != actual.getCode().length()) {
      failWithMessage("折扣碼長度必須為 5 碼.", null);
    }
    return this;
  }

  /**
   * 折扣碼物件, 失效日期
   *
   * @return
   */
  public CouponAssert hasExpired() {
    if (Objects.isNull(actual.getExpired())) {
      failWithMessage("折扣碼需設定失效日期.", null);
    }
    return this;
  }
}
```

測試寫法
```java
@Test
@DisplayName("Coupon Domain 測試")
void testCouponCode() {
  // arrange
  Coupon coupon = Coupon.builder()
    .code("ABD")
    .effected(LocalDate.now())
    .expired(LocalDate.now().plusMonths(1L))
    .build();

  // assert
  CouponAssert.assertThat(coupon)
    .hasCode()
    .codeLengthShouldBeFive()
    .hasExpired();
}
```
