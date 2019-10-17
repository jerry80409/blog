---
title: Java Equals and HashCode
cover: /img/home-bg.jpg
date: 2019-02-01 12:32:59
categories: ['java']
tags: ['misc']
---
### Summary
Java 所有的 class 皆繼承於 `java.lang.Object`, 其中 **equals()**, **hashcode()** 皆是 `native` 宣告的 methods(透過 JNI 介面, 用 c/c++ 去實作的東西), 這兩個東西在做 `equals()` 判斷的時候是緊密相關的, 但通常不太需要去 override 他們, 但是面試官很喜歡問這題...

```java
@Test
public void test() {
	String a = "123";
	String b = "123";
	String c = new String("123");

	// true
	System.out.println(a == b);

	// false
	System.out.println(a == c);

	// true
	System.out.println(a.equals(b));

	// true
	System.out.println(a.equals(c));
}
```
### "==" 
就是 References Compare, 比較記憶體的參照。
* a == b (true): 因為底層有 String pool, 所以會 reference 相同的記憶體區段,
* a == c (false): 因為 `new String("123")` 會 allocate 新的記憶體區段, 所以在做 refrenece 比較當然是不一樣的, 

### equals()
equals 的比較, 就不是 reference 的比較, 在 [API Docs](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#equals-java.lang.Object-) 裡面有這樣的敘述 "that equal objects must have equal hash codes.", 我理解為 `hashcode` 相等, 就是 equals() 成立。如果 override equals() 那相對的也需要調整 hashcode。
> Note that it is generally necessary to override the hashCode method whenever this method is overridden, so as to maintain the general contract for the hashCode method, which states that equal objects must have equal hash codes.

equals 特性如下: 
* reflexive(反射性): 在 non-null 的比較, x.equals(x) 永遠是 true
* symmetric(對稱性): 在 non-null 的比較, x.equals(y) 為 true, 則 y.equals(x) 也會是 true
* transitive(傳遞性): 在 non-null 的比較, x.equals(y) 為 true, 且 y.equals(z) 為 true, 則 x.equals(z) 也會是 true
* consistent(一致性): 在 non-null 的比較, x.equals(y) 為 true, 且 equals() 裡面比較的 objects 沒有被修改, 則 x.equals(y) 會永遠成立
* 所有的 non-null objects 在跟 null 做比較時, 永遠回傳 false

```java
@Test
public void test() {
	Member m1 = new Member(1, "Tom", Gender.M);
	Member m2 = new Member(1, "Tom", Gender.M);

	// 440467858
	System.out.println(m1.hashCode());

	// 440467858
	System.out.println(m2.hashCode());
		
	// true
	System.out.println(m1.equals(m2));
}
```
### hashcode()
* 在 equals() 裡面比較的 objects 沒有被修改的前提下, 在相同的 Java Application 運行期間都應該回傳相同的 integer.
* 如果 x.equals(y) 為 true; 則 x, y 必須產生相同的 hashcode.
* 如果 x.equals(y) 為 false; 則 x, y 必須產生完全不同的 hashcode, 在做 HashMap 的處理時, 不同的 hashcode 可以有效提高效能(不容易碰撞的意思吧)

### System.identityHashCode()
用途跟 hashcode() 差不多, 不管有沒有 override hashcode(), System.identityHashCode 會透過 `native` 去產生無法 override 的 hashcode, 且在 objects 為 null 的狀態下, 會回傳 0。

```java
@Test
public void test() {
    Member m1 = new Member(1, "Tom", Gender.M);
    Member m2 = null;

    // hashcode 被覆寫了, 回傳 32
    System.out.println(m1.hashCode());
    
    // 5072587
    System.out.println(System.identityHashCode(m1));

    // 0
    System.out.println(System.identityHashCode(m2));
}
```

### 如果只 override equals 會怎樣 ?
舉例來說, 假設 Member 只需要 `id` 來做 equlas 判斷, 然後不管其他欄位資料的一致性的話, 就需要去 **override equals()**
```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || super.getClass() != o.getClass()) return false;
    Member member = (Member) o;
    return id.equals(member.id);
}
```

但如果只 override equals() 而沒有去 **override hashcode()** 的話, 就會有 **矛盾** 的地方
```java
@Test
public void test() {
    Member m1 = new Member(1, "Tom", Gender.M);
    Member m2 = new Member(1, "Mary", Gender.F);

    // 在忽略其他欄位, 只考慮 id 的 equals(), 應該只會有 1 筆
    Set<Member> members = new HashSet<>();
    members.add(m1);
    members.add(m2);

    // [Member(id=1, name=Tom, gender=M), Member(id=1, name=Mary, gender=F)]
    System.out.println(members);
}
```

加入 override hashcode() 之後, Set 的重複問題就解決了
```java
@Override
public int hashCode() {
    return Objects.hash(id);
}
```

```java
@Test
public void test() {
    Member m1 = new Member(1, "Tom", Gender.M);
    Member m2 = new Member(1, "Mary", Gender.F);

    // 因為 override hashcode 所以只會有 1 筆
    Set<Member> members = new HashSet<>();
    members.add(m1);
    members.add(m2);

    // [Member(id=1, name=Tom, gender=M)]
    System.out.println(members);
}
```

### Apache Commons Lang
Apache Commons Lang 裡面擴增了許多 java.lang 層面的 API, 可以透過 Builder pattern 來處理 equals(), hashcode(), 好不好用就見仁見智了。

#### Dependency
```xml
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-lang3</artifactId>
  <version>3.8.1</version>
</dependency>
```

#### 一般寫法
```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Member member = (Member) o;
    return id.equals(member.id) &&
        name.equals(member.name) &&
        gender == member.gender;
}

@Override
public int hashCode() {
    return Objects.hash(id, name, gender);
}
```

#### Apache Commons Lang
```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Member member = (Member) o;
    return new EqualsBuilder()
        .appendSuper(super.equals(member))  // 加入 parenet 的 equals() 比較
        .append(id, member.id)
        .append(name, member.name)
        .append(gender, member.gender)
        .isEquals();
}

@Override
public int hashCode() {
    return new HashCodeBuilder(3, 41)  // 建構子, 一定要奇數
        .appendSuper(super.hashCode())
        .append(id)
        .append(name)
        .append(gender)
        .toHashCode();
}
```

### 結論
所以如果 x.equals(y) 為 true, 並不能直接證明 x, y 的 hashcode 相等; 
相對來說 x, y hashcode 相等, 也不能直接證明 x.equals(y)。

### Ref
* Java8 API Docs - [https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#equals-java.lang.Object-](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#equals-java.lang.Object-)
* Apache Commond Lang 3.1 Docs(HashCodeBuilder) - [https://commons.apache.org/proper/commons-lang/javadocs/api-3.1/org/apache/commons/lang3/builder/HashCodeBuilder.html](https://commons.apache.org/proper/commons-lang/javadocs/api-3.1/org/apache/commons/lang3/builder/HashCodeBuilder.html)
* Apache Commond Lang 3.1 Docs(EqualsBuilder) - [https://commons.apache.org/proper/commons-lang/javadocs/api-3.1/org/apache/commons/lang3/builder/EqualsBuilder.html](https://commons.apache.org/proper/commons-lang/javadocs/api-3.1/org/apache/commons/lang3/builder/EqualsBuilder.html)
* 良葛格(物件相等性) - [https://openhome.cc/Gossip/JavaEssence/ObjectEquality.html](https://openhome.cc/Gossip/JavaEssence/ObjectEquality.html)