---
title: switch-statements-with-lambda
cover: /img/home-bg.jpg
date: 2022-12-19 13:52:47
categories: ['java']
tags: ['design-pattern']
---
## Enums
通常 _SWITCH_ 會被用於需要大量使用 _if-else_ 的情境, 在某些時候加入 enums 的設計, 可以讓 switch 更容易被理解與管理.
```java
enum Player {
  // 戰士
  WARRIOR,
  
  // 弓箭手
  ARCHER,
  
  // 法師
  WIZARD,
}
```

```java
String swordAttack() {
  return "劍術攻擊";
}

String shootAttack() {
  return "射箭攻擊";
}

String fireAttack() {
  return "火焰攻擊";
}

// 攻擊
String attack(Player player) {
  switch(player) {
    // 戰士攻擊
    case WARRIOR:
      return this.swordAttack();

    // 弓箭手攻擊
    case ARCHER:
      return this.shootAttack();

    // 法師攻擊
    case WIZARD:
      return this.fireAttack();

    // 其他職業拋出異常  
    default:
      throw new IllegalArgumentException("Invalid player: " + player);
  }
}
```

## Supplier
`Supplier` 是 java8+ 的 lambda 滿好用的特性之一, 我通常會用它來封裝一些無參數傳入的 methods, 封裝到 Supplier 的 method, 並不會馬上被執行, 在某些層面上也算是一種 lazy loading 的應用, _SWITCH_ 就可以簡單被改寫成:
```java
String attack(Player player) {
  switch(player) {
    // 戰士攻擊
    case WARRIOR:
      Supplier<String> warriorSupplier = () -> this.swordAttack();
      return warriorSupplier.get();

    // 弓箭手攻擊
    case ARCHER:
      Supplier<String> archerSupplier = () -> this.shootAttack();
      return archerSupplier.get();
      
    // 法師攻擊
    case WIZARD:
      Supplier<String> wizardSupplier = () -> this.fireAttack();
      return wizardSupplier.get();
      
    // 其他職業拋出異常    
    default:
      throw new IllegalArgumentException("Invalid player: " + player);
  }
}
```

## Map
上述的寫法, 其實與原本的 _SWITCH_ 寫法沒差多少, 要應用 java8+ lambda 的 Stream 特性, 可能需要把這些條件判斷搜集起來, 藉由 `MAP` 的 KEY-VALUE 的特性, 讓 Stream 可以串接起來, 並透過 KEY 去篩選 Enums.
```java
String attack(Player player) {
  Map<Player, Supplier<String>> attackMap = new HashMap();
  attackMap.put(Player.WARRIOR, () -> this.swordAttack());
  attackMap.put(Player.ARCHER, () -> this.shootAttack());
  attackMap.put(Player.WIZARD, () -> this.fireAttack());

  // 確保線程安全, 推薦使用 Immutable 類型
  final Map<Player, Supplier<Void>> map = Collections.unmodifiableMap(attackMap);

  // 藉由 Stream 方式串接整個流程
  map.entrySet().stream()
    .filter(entry -> player.equals(entry.getKey()))
    .map(Entry::getValue)
    .map(Supplier::get)
    .findFirst()
    .orElseThrow(() -> new IllegalArgumentException("Invalid type: " + type));  
}
```

## Optimize
然而都已經是 _MAP_ 的資料結構了, 其實可以讓時間複雜度降低為 `O(1)`, 可以捨棄 `Stream` 優化改寫為:
```java
String attack(Player player) {
  Map<Player, Supplier<String>> attackMap = new HashMap();
  attackMap.put(Player.WARRIOR, () -> this.swordAttack());
  attackMap.put(Player.ARCHER, () -> this.shootAttack());
  attackMap.put(Player.WIZARD, () -> this.fireAttack());

  // 確保線程安全, 推薦使用 Immutable 類型
  final Map<Player, Supplier<String>> map = Collections.unmodifiableMap(attackMap);

  return Optional.ofNullable(map.get(player))
    .orElseThrow(() -> new IllegalArgumentException("Invalid type: " + player));
}
```
