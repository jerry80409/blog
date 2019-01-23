---
title: Inversion Of Control 學習筆記
cover: /img/home-bg.jpg
date: 2019-01-11 13:42:30
categories: ['misc']
tags: ['inversion-of-control']
---
### Summary 
只是我的閱讀筆記而已, 網路上資料滿多的, 個人比較推薦的是 **Martin Fowler** 這篇, 這篇記錄是我個人的理解, 再加上一些 Spring 的開發經驗描述, 有些地方我覺得還好就沒特別紀錄, 有些則是按照我自己的經驗解釋, 我還很阿菜, 如果有錯誤的地方也歡迎指正與討論 XD

原文: [https://martinfowler.com/articles/injection.html](https://martinfowler.com/articles/injection.html)

### Inversion Of Control
Inversion Of Control (反轉控制), 在使用這個名詞之前, 我會先想一下他是用來解決怎樣的問題? **解耦**? 這是一個非常抽象的解釋, 如果只是單純的解耦, 其實會寫 **interface** 都能辦到, 我自己覺得更好的解釋方式應該是這樣 (單純只是我自己的理解):
```
讓 [A Class] 能夠自然的依據使用情境自動使用 [N Class]
```

在文章中, 用了 **MovieLister** 跟 **MovieFinder** 解釋了許多。 
最一開始直覺的寫法是直接依賴 `MovieLister -> MovieFinder`, 但如果 MovieFinder 需要從 SQL、XML、WebService 讀入清單, 那透過 interface 就能解決, 但最終還是必須在 MovieLister 決定哪一種 implement(SQL, XML, WebService)? 為了解決這種不自然的依賴, 所以作者先從 **Dependency Injection** 角度來解釋。

### Dependency Injection
Dependency Injection (依賴注入), 這個名詞的主要觀念, 就是把 new instance 的主控權移交給外部, 細說部實作就自己參照 Martin Fowler, 簡單記錄一下而已。

#### 1. Constructor Injection
簡單來說, 建構 MovieLister 的時候, 就選好要用哪一種 MovieFinder
```java
class MovieLister {
    // 由建構子注入 MovieFinder
    public MovieLister(MovieFinder finder) {
        this.finder = finder;
    }
}
```

#### 2. Setter Injection
做法其實跟 Constructor Injection 差不多, 優點是在測試的時候更容易替換 MovieFinder 的實作。
```java
class MovieLister {
    
    private MovieFinder finder;

    // 由 setter method 注入 MovieFinder
    public void setFinder(MovieFinder finder) {
        this.finder = finder;
    }
}
```

#### 3. Interface Injection
算是 Setter Injection 更進階的用法, 透過 interface 規範 instance 一定要實作 setter, 因為是 interface 的關係, 所以相對於 Setter Injection 更有彈性, 更統一, 舉例來說你在 InjectFinder 不但可以 `void injectFinder()` 還可以 `void injectEditor()` 之類的, 在測試的時候也比較容易製作 stub.
```java
/**
 * 在 interface 定義 injectFinder
 */
public interface InjectFinder {
    void injectFinder(MovieFinder finder);
}
```

```java
/**
 * 透過 implement interface 來注入 MovieFinder
 */
class MovieLister implements InjectFinder {
    public void injectFinder(MovieFinder finder) {
        this.finder = finder;
    }
}
```

### Service Locator
Service Locator (服務定位), 這個做法有種我認知的容器 (第三者) 的味道, 這種做法不同於 DI (外部注入), 而是透過 ServiceLocator 來封裝, 所以 MovieLister 是依賴 ServiceLocator 來取得 MovieFinder 的實體, 透過 Singleton design pattern 可以快速地實現。
```java
class ServiceLocator {
    
    // singleton pattern
    private static ServiceLocator soleInstance;

    private MovieFinder movieFinder;

    // 2. Constructor, 容器初始化 (在容器裡註冊 MovieFinder)
    public ServiceLocator(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // 4. 取得 MovieFinder instance
    public static MovieFinder movieFinder() {
        return soleInstance.movieFinder;
    }

    // 1. 載入 ServiceLocator
    public static void load(ServiceLocator locator) {
        soleInstance = locator;
    }
}
```

```java
class MovieFinder {

    private static final List<Movie> MOVIES = Stream.of(
        new Movie("The Godfather", "Francis Ford Coppola"),
        new Movie("The Shawshank Redemption", "Frank Darabont"),
        new Movie("Inception", "Christopher Nolan"),
        new Movie("Fight Club", "David Fincher")
    ).collect(Collectors.toList());

    List<Movie> findByDirector(final String director) {
        return MOVIES.stream()
            .filter(movie -> director.equals(movie.getDirector()))
            .collect(Collectors.toList());
    }
}

class MovieLister {

    // 3. 透過 singleton pattern, 直接從 ServiceLocator 取得 MovieFinder instance
    private MovieFinder finder = ServiceLocator.movieFinder();

    Movie[] moviesDirectedBy(final String director) {
        return finder.findByDirector(director).toArray(new Movie[0]);
    }
}
```

請注意上面的第 3 個步驟 `ServiceLocator.movieFinder()`, 如果沒有在某個地方做 `configure()`, 就會發生 NPE(NullPointerException), 算是 ServiceLocator 的缺點吧, 但基本上大部分的 Framework 都會幫你處理這個 configure 的初始化, 而且如果不是 LazyLoading 的話, 在 configure 階段就能知道初始化成功或失敗。
```java
class Tester {
    
    private void configure() {
        ServiceLocator.load(
            new ServiceLocator(new MovieFinder()));
    }

    public void testSimple() {
        configure();
        MovieLister lister = new MovieLister();
        Movie[] movies = lister.moviesDirectedBy("Francis Ford Coppola");

        assertEquals(1, movies.length);
        assertEquals("The Godfather", movies[0].getTitle());

    }
}
```

#### 1. Segregated Interface for the Locator
作者針對 Service Locator 做了更進一步的設計, 前面的 ServiceLocator 的缺點就是無法單獨使用 MovieFinder, 所以透過 [role interface](https://martinfowler.com/bliki/RoleInterface.html) 來把 MovieFinde 從 ServiceLocator 抽離, 作者的 Sample Code 讓我想了很久, 怎麼想都會是 NPE, 或是 StackOverflow XD

```java
interface MovieFinderLocator {
    MovieFinder movieFinder();
}
```

```java
class ServiceLocator implements MovieFinderLocator {

    private static ServiceLocator soleInstance;
    private MovieFinder movieFinder;

    // 不是很能體會這樣寫法的優勢, 而且容易造成 StackOverflow
    // MovieFinderLocator locator = ServiceLocator.locator();
    // MovieFinder finder = locator.movieFinder();
    
    // 2. Constructor, 容器初始化 (在容器裡註冊 MovieFinder)
    public ServiceLocator(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // 4. 取得 MovieFinder instance
    @Override
    public MovieFinder movieFinder() {
        return soleInstance.movieFinder;
    }

    // 1. 載入 ServiceLocator
    public static void load(ServiceLocator locator) {
        soleInstance = locator;
    }

    // 3. 對外提供取得 ServiceLocator instance
    public static ServiceLocator locator() {
        return soleInstance;
    }
}
```

```java
class MovieLister {

    // 單獨使用 movieFinder, 不太確定是否為作者所提的 Segregated Interface for the Locator ?
    private MovieFinder finder = ServiceLocator.locator().movieFinder();

    Movie[] moviesDirectedBy(final String director) {
        return finder.findByDirector(director).toArray(new Movie[0]);
    }

    public MovieFinder getFinder() {
        return this.finder;
    }
}
```

```java
@Test
public void test2() {
    // configure
    MovieFinder finder = new MovieFinder();
    ServiceLocator.load(new ServiceLocator(finder));

    MovieLister lister = new MovieLister();
    Movie[] movies = lister.moviesDirectedBy("Francis Ford Coppola");

    // 測試 lister 的 finder 是 ServiceLocator 所提供的 finder
    assertEquals(lister.getFinder().hashCode(), finder.hashCode());

    assertEquals(1, movies.length);
    assertEquals("The Godfather", movies[0].getTitle());
}
```

#### 2. A Dynamic Service Locator
多了 HashMap 來記錄 Servicies, 然後再藉由 `loadService()`, `getService()`, 來達到 Dynamic Service Locator, 滿好理解的就沒實作了。

#### 3. Using both a locator and injection with Avalon
這個做法看起來滿理想, 也滿好理解的, 透過 ServiceManager 來做 ServiceLocator, 再透過 Role Interface 來達到隔離其他不相關的 Servicies, 

### 作者總結
1. DI, ServiceLoactor 會增加閱讀與理解的難度, 不管怎麼解耦合, 最終也只是將依賴對象轉換到另一個單位, 最終取決於這樣的情境是否能夠對開發, 測試上有所優化。
2. DI 對開發者比較容易掌握到物件相依狀況, 容易調整擴展; ServiceLocator 相對於 DI 更難理解與閱讀, 在一些比較常用的工具(HttpClient, SQLConnection, Logger), 我自己習慣以 ServiceLocator 的方式去開發, ServiceLocator 會比較適合, 有點像 Spring 的 **Configuration Bean**; 如果只是單純的提供給某些業務使用, 那 DI 的做法會比較合適, 容易調整擴展。
3. 代碼配置, 文件配置, 寫過 Spring xml 應該都痛過, 代碼配置缺點就是翻 API 理解名詞, 翻到快升天。

