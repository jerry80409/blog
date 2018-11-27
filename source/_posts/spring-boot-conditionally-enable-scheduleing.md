---
title: spring boot scheduleing 設定
cover: /img/home-bg.jpg
date: 2018-11-27 14:38:13
categories: ['spring-boot']
tags: ['spring-boot']
---
### Overview
因為在 localhost 做開發時, 不希望 Spring-boot 的 scheduling 被啟動, 所以透過 `ConditionalOnProperty annotation` 做了一個開關, 方便在 `applicaion.yml` 做維護切換, 在 `application-prod.yml` 才會被 enable。

### application.yml
```
# scheduling
scheduling.enabled: false
```

### Configuration bean
```java
@Configuration
@EnableScheduling
@ConditionalOnProperty(prefix = "scheduling", name = "enabled", havingValue = "true", matchIfMissing = true)
public class SchedulingConfiguration {
}
```

### Scheduling
```java
@Service
public class ScheduleTask {

    private Integer count = 1;

    /**
     * 
     * @throws InterruptedException
     */
    @Scheduled(cron = " */1 * * * * *")
    public void reportCurrentTimeCron() throws InterruptedException {
        final SimpleDateFormat DATE_FORMAT = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println(String.format("Executing [%s] Times, datetime：%s", count++, DATE_FORMAT.format(new Date())));
    }

}
```

```
Executing [1] Times, datetime：2018-08-06 11:17:03
Executing [2] Times, datetime：2018-08-06 11:17:04
Executing [3] Times, datetime：2018-08-06 11:17:05
```
