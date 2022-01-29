---
title: line-bot-sdk-5-mins
cover: /img/home-bg.jpg
date: 2020-02-17 22:11:31
categories: ['java', 'spring-boot']
tags: ['line-bot-sdk', 'spring-boot']
---

## [LINE BOT JAVA SDK](https://github.com/line/line-bot-sdk-java)
寫點騙人氣的文章, [LINE BOT JAVA SDK](https://github.com/line/line-bot-sdk-java) 是我 Follow 很久的專案, 把 Source code 看過一次會學到很多東西, Servlet dispatch, API Client (OKHttp3 + Retrofit2), token 封裝等等, 使用 SDK 來開發的話, 大約 5mins 就可以把簡單的 LINE Server Side 的流程架設完畢。

## Steps
* Intellij 是我目前用最順手的工具, 先用 Intellij + [Spring boot initializr](https://start.spring.io/) 快速建立一個 gradle 專案吧  
* Dependencies: 選用 `Spring Web` 跟 `Spring Data JPA`, 資料庫我是用 Postgres (`PostgreSQL Driver`)
* 手動加入 `line-bot-spring-boot` dependency
  ```
  implementation "com.linecorp.bot:line-bot-spring-boot:$rootProject.lineVersion"
  ```

* 不習慣用 Spring boot initializr 可以考慮在 build.gradle 貼上對應的 dependencies
  ```groovy
  ext {
    lineVersion = '3.3.1'
  }

  dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'

    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'org.postgresql:postgresql'

    annotationProcessor 'org.springframework.boot:spring-boot-configuration-processor'
    annotationProcessor 'org.projectlombok:lombok'

    implementation "com.linecorp.bot:line-bot-spring-boot:$rootProject.lineVersion"

    testImplementation('org.springframework.boot:spring-boot-starter-test') {
      exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
    }
  } 
  ```

* 去申請 LINE 開發需要的相關流程 (因為已經申請好很久了, 步驟有點忘了, 有錯再糾正我吧)
  * Official Account: [https://www.linebiz.com/tw/](https://www.linebiz.com/tw/)
  * 建立 Porvider 與 Channel: [http://at-blog.line.me/tw/archives/provider_channel_setting.html](http://at-blog.line.me/tw/archives/provider_channel_setting.html)
  
* 記得到 LINE Develope Setting 後台打開對應的設定
![line-bot-response-setting](/img/line-bot-sdk-5-mins/line-bot-response-setting.png)

* 申請 long-live channel token
進到 [https://developers.line.biz/console/channel/{channel_id}/messaging-api](#) 底下有個設定, 把它打開就有了

### Spring-boot application.properties 設定
若使用 **Spring boot initializr** 初始化 Spring boot 專案, 預設是會建立一個空白的 application.properties, 真是懷念 LARAVEL 完整的設定結構啊, 因為是空白的 application.properties 所以要勤快一點去爬文件
> https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html

---
我自己是習慣 `yml` 的格式, 所以我通常都會改為 `application.yml`, 記得要把 lint-bot 對應的 properties 設定上去, 不然會打不了 LINE Message APIs
```yml
debug: true

# 把變數抽到上面比較好替換, 如果你的 yml 後來變大怪獸的話這招很有用 XD
database:
  host: localhost
  post: 5432
  name: botdemo
  user: postgres
  password: 

# spring boot
spring:
  application.name: line-bot
  datasource:
    name: postgres
    url: jdbc:postgresql://${database.host}:${database.post}/${database.name}
    username: ${database.user}
    password: ${database.password}

  profiles.active: dev

# line config
# https://github.com/line/line-bot-sdk-java/tree/master/line-bot-spring-boot
line.bot:
  handler.enabled: true
  handler.path: /callback
  channel-token: 'your-channel-token'
  channel-secret: 'your-channel-secret'
  channelTokenSupplyMode: fixed
  connectTimeout: 60000
  readTimeout: 60000
  writeTimeout: 60000
```

## Demo code
沒什麼好說的, 就是 copy + past, 這樣你就能輕鬆解決一張 Scrum 的 ticket XD;

因為 LINE BOT SDK 幫你在 Servlet 做好訊息類型的 Dispatch, 所以 `Event` 跟 `MessageEvent` 的事件都幫你安排在不同的 handler methods, 所以程式碼會比自己寫 switch 來的乾淨一點, 也省下不少力氣去 Serialize 跟 deserialize 對新手來說, 或是參加 hackathon 都是省時省力的...  
```java
@LineMessageHandler
public class Handler {

  @EventMapping
  public TextMessage handleTextMessageEvent(MessageEvent<TextMessageContent> event) {
    // 這邊做的就是簡單的 echo
    System.out.println("event: " + event);
    return new TextMessage(event.getMessage().getText());
  }

  @EventMapping
  public void handleDefaultMessageEvent(Event event) {
    // 就是加入聊天室, 離開聊天室, 還有一些有的沒的事件
    System.out.println("event: " + event);
  }
}
```

## Testing 
隨便從 LINE 發送一個訊息, 在 log 就能觀察到了
```
2020-02-17 23:10:35.456 DEBUG 62976 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : POST "/callback", parameters={}
2020-02-17 23:10:35.459 DEBUG 62976 --- [nio-8080-exec-1] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped to com.linecorp.bot.spring.boot.support.LineMessageHandlerSupport#callback(List)
event: MessageEvent(replyToken=******, source=UserSource(userId=******), message=TextMessageContent(id=******, text=Hello ), timestamp=2020-02-17T15:10:34.337Z)
2020-02-17 23:10:35.625  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : --> POST https://api.line.me/v2/bot/message/reply
2020-02-17 23:10:35.625  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : Content-Type: application/json; charset=UTF-8
2020-02-17 23:10:35.625  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : Content-Length: 119
2020-02-17 23:10:35.625  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : Authorization: Bearer jwt token
2020-02-17 23:10:35.625  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : User-Agent: line-botsdk-java/3.3.1
2020-02-17 23:10:35.625  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : 
2020-02-17 23:10:35.625  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : {"replyToken":"******","messages":[{"type":"text","text":"Hello "}],"notificationDisabled":false}
2020-02-17 23:10:35.625  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : --> END POST (119-byte body)
2020-02-17 23:10:35.631 DEBUG 62976 --- [nio-8080-exec-1] m.m.a.RequestResponseBodyMethodProcessor : Using 'application/json', given [*/*] and supported [application/json, application/*+json, application/json, application/*+json]
2020-02-17 23:10:35.631 DEBUG 62976 --- [nio-8080-exec-1] m.m.a.RequestResponseBodyMethodProcessor : Nothing to write: null body
2020-02-17 23:10:35.634 DEBUG 62976 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed 200 OK
2020-02-17 23:10:35.927  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : <-- 200 OK https://api.line.me/v2/bot/message/reply (301ms)
2020-02-17 23:10:35.927  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : Server: nginx
2020-02-17 23:10:35.927  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : Content-Type: application/json
2020-02-17 23:10:35.927  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : x-line-request-id: 49908719-170f-42fc-a1c6-e00579385e21
2020-02-17 23:10:35.927  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : x-content-type-options: nosniff
2020-02-17 23:10:35.927  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : x-xss-protection: 1; mode=block
2020-02-17 23:10:35.927  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : x-frame-options: DENY
2020-02-17 23:10:35.927  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : Vary: Accept-Encoding
2020-02-17 23:10:35.927  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : Expires: Mon, 17 Feb 2020 15:10:35 GMT
2020-02-17 23:10:35.927  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : Cache-Control: max-age=0, no-cache, no-store
2020-02-17 23:10:35.927  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : Pragma: no-cache
2020-02-17 23:10:35.927  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : Date: Mon, 17 Feb 2020 15:10:35 GMT
2020-02-17 23:10:35.927  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : Connection: keep-alive
2020-02-17 23:10:35.928  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : 
2020-02-17 23:10:35.928  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : {}
2020-02-17 23:10:35.928  INFO 62976 --- [api.line.me/...] com.linecorp.bot.client.wire             : <-- END HTTP (2-byte body)
```

## Murmur
以行銷的角度來說 bot 真的不是什麼太厲害的技術或是工具, bot 只是額外幫你觸及了另一群使用者, 如果沒有好的策略或是好的 UX 引導, 就是無魂有體的稻草人, 還有協助你做 bot 的工具真的一堆, 在做商業開發的前期我會推薦就先朝以下幾個平台工具去市場測試水溫看看 DAU, MAU, 先講就不傷身體, 再投入成本整合 prodfeed, payflow, etc.

* Chatfuel - https://chatfuel.com/, 老牌也很穩定 APIs 也更新很快, 但只支援 Facebook
* Bottender - https://bottender.js.org/, 在開源社群很活躍的 APIs, 支援多種 Message 平台 (LINE, Facebook, SLACK, Telegram, etc.)
* Super8 - https://no8.io/
* Chatcompose - https://www.chatcompose.com/
* GoSky - https://www.goskyai.com/
* Chatbot - https://www.chatbot.com/
* Intercom - https://www.intercom.com/
* Full2house - https://www.full2house.com/
* Manychat - https://manychat.com/
  

我覺得另一個體驗很棒的訂房 bot - https://www.snaptravel.com/, 這才是我理想的產品啊 ...


