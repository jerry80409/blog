---
title: spring-batch-fault-tolerant
cover: /img/spring-batch-fault-tolerant/johann-siemens-591-unsplash.jpg
date: 2018-11-27 15:37:44
categories: ['spring-boot']
tags: ['spring-boot', 'spring-batch']
---
### Overview
**spring-batch** 其實是很深的東西, 其中的容錯性 `fault-tolerant` 是個很棒的設計, 在批次處理中如果因為某些 exception 或是 http timeout exception 發生時, 只要指定好 exception handler, spring-batch 會自動重新運行該失敗的 step, 不需要整個 job 重新來過, 可以節省很多時間跟運算成本。

### Docs
* Spring Batch - [Reference Documentation](https://docs.spring.io/spring-batch/4.0.x/reference/html/index.html)
* [Step Restart](https://docs.spring.io/spring-batch/4.1.x/reference/html/step.html#stepRestart)
* [Spring Batch参考文档中文版](https://www.bookstack.cn/read/SpringBatchReferenceCN/README.md)

### Configure Job
簡單示範一下, 大小寫轉換的 job。

1. 這個 job 只會有一個 step
2. step 的 reader 操作, 用來讀取字串到 queue 裡面
3. step 的 processor 操作, 用來將字串轉為 upper case
4. step 的 writer 操作, 將處理完的字串 println 輸出

### ItemReader
```java
@Slf4j
public class SimpleReader implements ItemReader<String> {

    private ConcurrentLinkedQueue<String> queue;

    /**
     * for spring bean init
     */
    public void init() {
        log.info("init simple reader...");
        queue = new ConcurrentLinkedQueue<>();
        queue.addAll(Arrays.asList(
            "message 1", "message 2", "message 3", "message 4", "message 5",
            "message 6", "message 7", "message 8", "message 9"
        ));
    }

    @Override
    public String read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
        // 一次輸出一個 message, 直到 queue 為空
        return queue.isEmpty() ? null : queue.poll();
    }
}
```

### ItemProcessor
```java
@Slf4j
public class SimpleProcessor implements ItemProcessor<String, String> {
    
    @Override
    public String process(String msg) throws Exception {
        if ("message 5".equals(msg)) {
            // 簡單示範 message 5, 處理時必定會拋出自定義的 RetryException 
            throw new RetryException("message 5 was FAILED");
        }
        if ("message 8".equals(msg)) {
            // 簡單示範 message 8, 處理時必定會拋出自定義的 RetryException
            throw new RetryException("message 8 was FAILED");
        }
        log.info("processing : {}", msg);
        return msg.toUpperCase();
    }
}
```

### RetryException
自定義的 RetryException, 當失敗的資料拋到這裡時, 可以做一些簡單的 log 或 notification
```java
@Slf4j
public class RetryException extends Exception {

    public RetryException(String message) {
        super(message);
        log.warn("Catch Retry Exception: {}", message); 
    }
}
```

### ItemWriter
```java
@Slf4j
public class SimpleWriter implements ItemWriter<String> {
    
    @Override
    public void write(List<? extends String> msgList) throws Exception {
        msgList.forEach(System.out::println);
    }
}
```

### Job
當使用了 `.faultTolerant()` 必定需要 `.retry(RetryException.class)` 或 `.skip(RetryException.class)` 的配合。
```java
@Configuration
@EnableBatchProcessing
public class FooJob {

    @Autowired
    public JobBuilderFactory jobBuilderFactory;

    @Autowired
    public StepBuilderFactory stepBuilderFactory;


    @Bean(initMethod = "init")
    @StepScope
    public SimpleReader simpleReader() {
        return new SimpleReader();
    }

    @Bean
    @StepScope
    public SimpleProcessor simpleProcessor() {
        return new SimpleProcessor();
    }

    @Bean
    @StepScope
    public SimpleWriter simpleWriter() {
        return new SimpleWriter();
    }

    @Bean
    public Job simpleJob() {
        return jobBuilderFactory.get("SIMPLE_JOB")
            .incrementer(new RunIdIncrementer())
            .flow(simpleStep())     // 在操作設計上, 個人比較喜歡用 flow, 在設計上更方便組合 step
            .end()
            .build();
    }

    @Bean
    public Step simpleStep() {
        return stepBuilderFactory.get("SIMPLE_STEP")
            .<String, String>chunk(1)
            .reader(simpleReader())
            .processor(simpleProcessor())
            .writer(simpleWriter())
            .faultTolerant()                    // 開啟容錯設定
            .retry(RetryException.class)        // 當這個 step 某個階段拋出 RetryException
            .backOffPolicy(backOfFiveSec())     // wait 5 sec
            .retryLimit(3)                      // 最多 retry 3 次
            .skip(RetryException.class)         // 當這個 step 某個階段拋出 RetryException 做 skip 處理
            .skipLimit(1)                       // 最多 skip 1 次
            .build();
    }

    private FixedBackOffPolicy backOfFiveSec() {
        FixedBackOffPolicy fiveSecBackoff = new FixedBackOffPolicy();
        fiveSecBackoff.setBackOffPeriod(5_000);
        fiveSecBackoff.setSleeper(new ThreadWaitSleeper());
        return fiveSecBackoff;
    }
}
```

### Scheduling Launch Job
```java
@Slf4j
@Component
@AllArgsConstructor
public class LaunchJob implements InitializingBean {

    private JobLauncher jobLauncher;
    private Job job;

    @Override
    public void afterPropertiesSet() throws Exception {
        log.info("launching foo-job...");
        final JobParameters jobParameters = new JobParametersBuilder()
            .addLong("execution", System.currentTimeMillis())
            .toJobParameters();
        jobLauncher.run(job, jobParameters);
    }
}
```

### Result
1. 當 message 5 發生錯誤時, 自動 Retry 3 次
2. step 收到第 1 次 RetryExcpetion, 做 Skip 處理, 繼續往下執行
3. 當 message 8 發生錯誤時, 一樣 Retry 3 次
4. step 收到第 2 次 RetryExcpetion, 已達到 skip 上限, 視為 step 失敗
5. job 失敗

```
c.example.batchdemo.config.LaunchJob   : launching foo-job...
o.s.b.a.batch.JpaBatchConfigurer         : JPA does not support custom isolation levels, so locks may not be taken when launching Jobs
o.s.b.c.r.s.JobRepositoryFactoryBean     : No database type set, using meta data indicating: MYSQL
o.s.b.c.l.support.SimpleJobLauncher      : No TaskExecutor has been set, defaulting to synchronous executor.
o.s.b.c.l.support.SimpleJobLauncher      : Job: [FlowJob: [name=SIMPLE_JOB]] launched with the following parameters: [{execution=1533892561146}]
o.s.batch.core.job.SimpleStepHandler     : Executing step: [SIMPLE_STEP]
c.example.batchdemo.job.SimpleReader   : init simple reader...
c.e.batchdemo.job.SimpleProcessor      : processing : message 1
MESSAGE 1
c.e.batchdemo.job.SimpleProcessor      : processing : message 2
MESSAGE 2
c.e.batchdemo.job.SimpleProcessor      : processing : message 3
MESSAGE 3
c.e.batchdemo.job.SimpleProcessor      : processing : message 4
MESSAGE 4
c.e.batchdemo.job.RetryException       : Catch Retry Exception: message 5 was FAILED
c.e.batchdemo.job.RetryException       : Catch Retry Exception: message 5 was FAILED
c.e.batchdemo.job.RetryException       : Catch Retry Exception: message 5 was FAILED
c.e.batchdemo.job.SimpleProcessor      : processing : message 6
MESSAGE 6
c.e.batchdemo.job.SimpleProcessor      : processing : message 7
MESSAGE 7
c.e.batchdemo.job.RetryException       : Catch Retry Exception: message 8 was FAILED
c.e.batchdemo.job.RetryException       : Catch Retry Exception: message 8 was FAILED
c.e.batchdemo.job.RetryException       : Catch Retry Exception: message 8 was FAILED
o.s.batch.core.step.AbstractStep         : Encountered an error executing step SIMPLE_STEP in job SIMPLE_JOB

org.springframework.batch.core.step.skip.SkipLimitExceededException: Skip limit of '1' exceeded
```

