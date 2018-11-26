---
title: 單元測試藝術 Ch3
cover: /img/home-bg.jpg
date: 2018-11-26 21:52:01
categories: reading
tags: unit_test
---
## Ch3 透過虛設常式解決依賴問題
虛設常式原文是 `Stubs`, 老實說我並不是很清楚事不是我平常溝通用的 `Mock`, 稍微翻過後面的章節, 好像後面會說到, 就先稍微紀錄一下書上的解釋,

> 要測試太空梭在宇宙航行安不安全, 不可能先飛上與宇宙, 因此 NASA 建立了太空梭的模擬系統, 模擬宇宙環境與操作面板。在開發中, 無法掌控的物件 (檔案系統, 執行緒, 記憶體, 時間, etc.) 就像無法控制的宇宙環境, 為了處理這一類的開發測試問題, 可以透過 stub, mock, fake, 避免直接依賴造成的問題。

此章節主要案例以將 log 寫到檔案系統為例, 在測試時並不直接將 log 寫到檔案系統（File Writting）, 因此 `File Writting` 這段程式碼應該要被設計成可以抽換, 或者在 `File Writting` 之上考慮加一個中介層, 我對這個 `中介層` 的理解, 比較像是一個代理人, 測試的時候只要代理人說我去寫 log 了, 這樣就達到我們所要的 `隔離`, 就是不直接寫檔案到系統上。

### 示意圖
![demo](/img/the-art-of-unit-testing-note-ch3/demo.png)

### 實作
實作大致上可以分為兩類,
A type: 使用 interface, 或 delegates （類似 Proxy Pattern）
B type: 在建構 class 時, injection 假的實作物件

這邊簡單的紀錄一下我讀完比較有感覺的地方, 就是 Proxy pattern 在測試的應用, 
![uml](/img/the-art-of-unit-testing-note-ch3/uml.png)

`LogAnalyzerUsingFactory` 用來抽象的, 裡面只會做兩件事,
`getManager()` 來取得 FakeExtenstionManager 或 FileExtensionManager 實體,
`isValidLogFileName()` 回傳實體的 isValid() 驗證

```java
public abstract class LogAnalyzerUsingFactory {

    public boolean isValidLogFileName(String fileName) throws IOException {

        return getManager().isValid(fileName);
    }

    /**
     * 讓 factory 可以替換 FileExtensionManager 或 FakeFileExtensionManager
     * 參考 TestableLogAnalyzer.java
     *
     * @return
     */
    protected abstract IExtensionManager getManager();
}
```

### 代理人
```java
public class TestableLogAnalyzer extends LogAnalyzerUsingFactory {

    public IExtensionManager manager;


    public TestableLogAnalyzer(IExtensionManager manager) {

        this.manager = manager;
    }


    @Override
    protected IExtensionManager getManager() {
        return new FakeExtensionManager();
    }
}
```

### 測試
裡面的 `FakeExtensionManager` 就是 stub 工人
```java
/**
 * 用測試工廠 (TestableLogAnalyzer) 包裝 stub (FakeExtensionManager)
 * TestableLogAnalyze 有點類似 proxy, 封裝了 IExtensionManager
 * 間接測試了 Manager 的實體
 *
 * @throws IOException
 */
@Test
public void override_test() throws IOException {

  FakeExtensionManager stub = new FakeExtensionManager();
  FakeExtensionManager.WILL_BE_VALID = true;

  TestableLogAnalyzer logAnalyzer = new TestableLogAnalyzer(stub);
  boolean result = logAnalyzer.isValidLogFileName("target_file.ext");

  assertTrue(result);
}
```