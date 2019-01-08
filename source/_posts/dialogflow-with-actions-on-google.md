---
title: Dialogflow, Actions on google 學習筆記
cover: /img/home-bg.jpg
date: 2018-12-28 10:22:37
categories: ['dev_ops']
tags: ['gcp', 'dialogflow', 'actions_on_google']
---
### qwiklabs
[qwiklabs](https://google.qwiklabs.com/home) 是一個的 GCP 學習沙盒(sandbox), 讓你可以快樂的在裡面玩沙, 而不用搞壞公司的 GCP 環境還被扣費用, 每個 labs 教程結束後, 會頒發一個 badge 給你, 還滿有成就的, 上完一系列的 labs 還可以參加考試領證書, 有點貴就是了... AWS 也有 qwiklabs, 服務一樣就不介紹了。
從 catalog 裡面還可以篩選一些 [free labs](https://google.qwiklabs.com/catalog?keywords=&format%5B%5D=any&level%5B%5D=any&duration%5B%5D=any&price%5B%5D=free&modality%5B%5D=any&language%5B%5D=any), 我的 GCP 之路大概就是這樣起手...
寫這篇文章的主要 [課程](https://google.qwiklabs.com/focuses/2196?catalog_rank=%7B%22rank%22%3A1%2C%22num_filters%22%3A1%2C%22has_search%22%3Atrue%7D&parent=catalog&search_id=1782895)

### Dialogflow
對後端工程師來說 API 就是我們的 UI, 不用刻任何介面就可以把服務應用到 chat-bot 上去是很爽的一件事, 如果要更友善更開放就需要 NLP 的支援, 但是架設 NLP service 又是大工程, 要架設 NLP, 餵字庫, 算權重, etc., 聽同事介紹說 DialogFlow 在處理中文的 intent, session 很強大, 就順便在 qwiklabs 上用看看, 幾個概念了解一下就好,

* Action: an interaction built for Google Assistant that performs specific tasks based on user input.
* Intent: the goal of the Action (e.g. generate quotes). An intent takes user input and channels it to trigger an event.
* Agent (Dialogflow): a module that uses NLU and ML to transform user input into actionable data to be used by an Assistant application.

### Actions on google
主要是用來整合 Google 相關產品 (Dialogflow, etc.) 與 Google Assistant 與 Google Search 的平台, 用了就知道, 幾個概念了解一下就好
* Intent: A goal or task that users want to do, such as ordering coffee or finding a piece of music. In Actions on Google, this is represented as a unique identifier and the corresponding user utterances that can trigger the intent.
* Action: An interaction you build for the Assistant that supports a specific intent and has a corresponding fulfillment that processes the intent.
* Fulfillment: A service, app, feed, conversation, or other logic that handles an intent and carries out the corresponding Action.

### Steps
1. 需要有一個 GCP project-id
2. **Actions on google** import GCP project-id
3. **Actions on google** 的 project 裡面建立 actions, 會自動導向到 dialogflow 的 `Create Agent`
4. 建立好 Agent 後, 就建立 `intents`, 預設有 `Default Fallback Intent` 跟 `Welcome Intent`
5. 選擇 Fallback Intent, 裡面有 `Training phrases`, 訓練一些慣用語, Agent 會自己訓練跟學習
6. 一樣在 Fallback Intent 階段, 把對應的 `Response` 設定上去
7. Save `Intents`
8. 打開 `Activity Controls` 的權限, 讓 google 蒐集你的相關資料做訓練參考用吧, 我猜
9. **Actions on google** 做 Simulator 模擬測試

### Fallback Intents
Fallback Intent 就是當 Agent 無法辨識使用者的 intent 的時候, 有預設了一些 Response, 用來作 Fallback 策略的
![fallback intent](/img/dialogflow-with-actions-on-google/fallback-intent.png)

### Simulator
Simulator 可以模擬使用者的 input(語音, 文字, etc.), 小小的測試一下, 輸入 shakespeare 就沒反應了
![actions on google simulator](/img/dialogflow-with-actions-on-google/simulator.png)

### Final
課程只有 30min, 沒機會用的很深, 但比起自己整合 NLP, NLU, 餵詞, 算權重, 訓練資料, etc. 已經省很多工了, 有興趣的可以參考 [這篇](https://www.appcoda.com.tw/chatbot-dialogflow-ios/) 有比較多的介紹。