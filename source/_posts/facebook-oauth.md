---
title: facebook-oauth
cover: /img/home-bg.jpg
date: 2019-01-23 23:50:56
categories: ['facebook']
tags: ['facebook_apis', 'oauth']
---
### Facebook OAuth Login Flow
![facebook-oauth](/img/facebook-oauth/facebook-oauth.png)
從圖片中, 簡單的可以歸納幾個步驟，授權細節會在後面提到。
1. (前端) 產生 Login Button 給使用者。
2. (後端) 接收 `使用者` 的 Login Request, Redirect 給 Facebook。
3. Facebook 會彈出登入視窗給使用者, 徵求 `使用者` 授權。
4. 使用者點選同意授權，會回傳 redirect_uri 跟 code 給 `後端`。
5. (後端) 接收使用者同意授權的 scopes, code 做驗證, 並且跟 Facebook 做 server side 驗證, facebook 會要求一個 callback endpoint。
6. (後端) 接收 Facebook OAuth Callback, 並儲存使用者的 access token, 做相關產品的應用, long-terms access token `90 days 要重新授權一次`。

### 第一步是申請 App, Login 功能
Dashboard 頁面, 主要是管理 app 要啟用的 apis, 可以觀察 apis 錯誤率, rate limit 狀態, security 設定, callback, hook 設定, etc. 最重要的是 facebook 審核, 審查你所申請的 "APP" 有沒有符合他們的規範, 或違反 Facebook 原則或標準, 如果 APP 還在開發階段, 這部分就沒差, 如果要上線基本資料就要填好, [2018 劍橋事件](https://zh.wikipedia.org/wiki/Facebook) 後審查變得比較嚴格, 請參考 [審查指南](https://developers.facebook.com/docs/apps/review/)。

下方的 OAuth 重新導向 URL 設定大概像這樣, 很簡單的 server side 驗證端點
![fb-app-dash-board](/img/facebook-oauth/fb-dash-board-login.png)

### 產生「登入」對話方塊給使用者
這一步的重點只是要產生 url, 自己覺得由後端來產生比較好, 簡單來說就是要產生下列這串 url, 當中的 `state` 很重要, 很重要, 很重要!!!
除了放入使用者的一些狀態, 最好還是做一些對稱加密(AES), 是避免 csrf 攻擊的基本防禦方法, 記得在 callback endpoint 驗證 hash 是不是自己公司簽出來的資訊.
```
https://www.facebook.com/v2.12/dialog/oauth?
  client_id={app-id}
  &redirect_uri={"https://www.domain.com/login"}
  &state={"{st=state123abc,ds=123456789}"}
```

當使用者點擊這個 URL 會發生兩件事,
1. Facebook 會詢問使用者是否願意授權給這個 App
2. Facebook 知道使用者已經同意你的 App 做一些事情, Redirect 給你的 server

### Server 收到 Facebook Redirect
1. 驗證 `state` 是不是由 facebook redirect 過來的
2. 解密 `state` 取得使用者的基本資訊
3. 拿 Facebook Redirect 給你的 `code` 跟 app 的 `id`, `secret` 資訊, 取得使用者的 `access token` (short-term)
4. 再把請求 redirect 到登入成功頁面（index.html）

```java
/**
 * 接收 facebook user login callback
 * 此端點無法透過 Authorization 做保護, 所以 state 一定要作加密（加簽）, 驗證處理。
 *
 * @param code  facebook oauth response param
 * @param state facebook oauth response param
 * @return
 */
@RequestMapping(value = "/oauth/callback")
public ResponseEntity callbackHandler(@RequestParam(value = "code") String code,
                                      @RequestParam(value = "state") String state) throws IOException {

    log.info("Received Facebook login callback with code: [{}] and state: [{}]", code, state);

    // decode state
    final String decodeState = new String(decoder.decode(state), "UTF-8");

    // 我會在 state 放入 jwt, 做驗證 split status
    final String[] statusItems = decodeState.split(";");
    Map<String, String> statusMap = ImmutableMap.<String, String>builder()
        .put("jwt", statusItems[0])
        .put("perms", statusItems[1])
        .build();

    // 有敏感資訊, 設定為 debug 層級
    log.debug("Facebook Login user's jwt: [{}] and request facebook perms is: [{}]",
        statusMap.get("jwt"), statusMap.get("perms"));

    // TODO: 2018/3/13 verify jwt

    // request facebook get user access_tokens
    final HttpUrl baseURL = HttpUrl.get(new URL(ACCESS_TOKEN_URL));
    final HttpUrl accessTokenUrl = baseURL.newBuilder()
        .addQueryParameter("client_id", FACEBOOK_APP_ID)
        .addQueryParameter("client_secret", FACEBOOK_APP_SECRET)
        .addQueryParameter("redirect_uri", REDIRECT_URI)
        .addQueryParameter("code", code)
        .build();

    final OkHttpClient restClient = new OkHttpClient();
    Request request = new Request.Builder()
        .url(accessTokenUrl)
        .build();
    Response response = restClient.newCall(request).execute();

    // TODO: 2018/3/13 handler request error.

    AccessTokenDTO accessTokenDTO = 
    objectMapper.readValue(response.body().byteStream(), AccessTokenDTO.class);
    log.info("Request: [{}]", accessTokenUrl.toString());
    log.debug("Response: [{}]", accessTokenDTO);

    // TODO: 2018/3/13 insert or update facebook user info.
    log.info("Update Facebook User ...");

    // redirect to index page
    HttpHeaders headers = new HttpHeaders();
    headers.add("Location", INDEX_PAGE);
    return new ResponseEntity(headers, HttpStatus.FOUND);
}
```