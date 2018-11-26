---
title: 單元測試藝術 Ch1 Ch2
cover: /img/home-bg.jpg
date: 2018-05-25 21:04:36
categories: reading
tags: unit_test
---
### Ch1 單元測試基礎
* 單元測試的範圍, 可以小到一個 function, 大到多個 Class
* 呼叫一個 `沒有回傳值` 的第三方 API, 也可以被歸類於單元測試
* 單元測試執行時, 要隔離其他沒有相關的程式
* 理想的單元測試, 容易自動化, 容易被實現, 非臨時性
* 整合測試, 測試更完整, 但涵蓋許多單元的運作, 難以快速釐清問題
* 沒有測試的程式碼, 在某些程度上也算是 Legacy code
* Test-Driven Development (TDD), 測試先行的開發模式, 在開發之前, 就先釐清程式會產生的結果
* TDD 的好處在於協助開發者釐清複雜的邏輯處理, 降低程式碼的複雜度
* TDD 並不能保證設計出一個 perfect 的系統

### Ch2 單元測試
* 主要介紹 C# 的測試框架 (NUnit)
* 測試包含三種行為 Arrange, Act, Assert
* 撰寫測試時, `可讀性` 是最重要的考量
* 善用 setup 與 tearDown, 省去一些重複的操作, 並可以確保每次的測試, 不會影響下一次的測試
* 驗證預期的例外