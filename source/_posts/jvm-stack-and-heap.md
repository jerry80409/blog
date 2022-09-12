---
title: jvm-stack-and-heap
cover: /img/home-bg.jpg
date: 2022-09-08 22:48:45
categories: ['java']
tags: ['jvm']
---
## JVM
java 程式碼在運作時, 會透過 compiler 將可讀的程式碼編譯為 `*.class` 的檔案, 而這個檔案會透過
Java Virtual Machine (JVM) 將其轉為 byte code, 讓作業系統執行運作, 這邊就暫時這樣粗略地解釋吧。

在 JVM 裏面, 會把 java 的資料類型分為 primitive 與 reference, 這樣設計是考量 JVM 在運行之前希望由 compiler 先將所有資料進行檢查。

### Primitive type
primitive 類型支持如, `byte` (8bits), `short`(16bits), `int`(32bits), `long`(64bits), char(16bits)。

### Reference type
而 reference 類型有, `class`, `array`, `interface`, 他們的默認值為 `null`。

## JVM Stack
JVM stacks 會隨著 JVM 的 thread 建立而跟著建立, 而 threads 的 stack 是 private 的, 也就是不能與其他 thread 共用, stack 用來存放的東西叫做 frames（具體來說我並沒有很了解 frames）, 但在 IntelliJ debug 模式中可以確實觀察得到。
![jvm-thread-and-frame-1](/img/jvm-stack-and-heap/jvm-thread-and-frame-1.jpg)

![jvm-thread-and-frame-2](/img/jvm-stack-and-heap/jvm-thread-and-frame-2.jpg)

Stack 在資料結構的定義是一種後進先出的資料結構(LIFO), 如果有在刷 leetcode 的話這是一個很方便用於資料反轉 (Reverse) 的資料結構 XD。從圖中可以看出每一層 stack 的 methods 
invoke (呼叫), 所以當程式拋出 exceptions 可以透過 `e.printStackTrace()` 來取得當前 thread 使用 stacks 所有過程。   
```java
try {
  // do something   
} catch (Exception e) {
  // 輸出 exception 所記錄的 stack trace
  e.printStackTrace();
}
```

### stack 的大小
在 oracle 所定義的 jvm 規範中, 允許 stack 動態的增加擴充, 但也允許讓開發者去設定 jvm 的 stack 大小, 可以使用 java -Xss1M 這樣的設定去啟動 JVM, 宣告 stack 大小為 1Mb 空間。
```java
java -Xss:1m myApp
```
![jvm-thread-and-frame-2](/img/jvm-stack-and-heap/jvm-stack-xss-setting.jpg)

* 當 stack 超過 threads 的最大 stack 限制時, 會拋出 `StackOverflowError`, 比如說無限遞迴。
* 當 jvm 要擴增 stack 空間時, 但沒有足夠的記憶體可以拿來擴展 stack 與 thread 時, 會拋出 `OutOfMemoryError`。

## JVM Heap
JVM Heap 是在 JVM 啟動時建立的, 但是屬於 Threads 共用的, 主要用來存放 class, array 類型, 由 GC 機制做回收, heap 跟 stack 一樣, 也是可以由開發者去定義大小。為了有最佳的效能, 官方頁面設定是說建議 `Xms` 與 `Xmx` 可設定相同的大小, 若沒有特別指定的話, 預設會使用系統的 25% 實體記憶體做設定, 另外 server mode 與 client 
mode 也會有所差異。 
```java
java -Xms:64m -Xmx:256m myApp
```
![jvm-heap-xms-setting](/img/jvm-stack-and-heap/jvm-heap-xms-setting.jpg)

## Java Visualizer
若想要更進一步學習 stack 與 heap 的運作的話, 在 intelliJ 的環境下可以安裝 [java-visualizer](https://fa20.datastructur.es/materials/guides/plugin.html#java-visualizer), 透過 debug 的 step over 就能確實觀察到 stack 與 heap 的運作了。

![jvm-visualizer](/img/jvm-stack-and-heap/jvm-visualizer.jpg)

## Reference
* https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-2.html#jvms-2.5.2
* https://alvinalexander.com/scala/fp-book/recursion-jvm-stacks-stack-frames
* https://docs.oracle.com/cd/E13150_01/jrockit_jvm/jrockit/jrdocs/refman/optionX.html#wp1024112
* https://fa20.datastructur.es/materials/guides/plugin.html#java-visualizer