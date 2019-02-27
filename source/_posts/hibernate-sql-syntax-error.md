---
title: Hibernate DDL Syntax Error
cover: /img/home-bg.jpg
date: 2019-02-27 16:40:35
categories: ['spring-boot']
tags: ['spring-boot', 'spring-data', 'jpa', 'mysql']
---
### I got an error
When I created an `order` table, just like ...

#### application.yml
```yaml
spring:
  datasource:
    username: ${database.username}
    password: ${database.password}
    url: jdbc:mysql://${database.host}:3306/${database.name}?characterEncoding=utf-8&useUnicode=true&useSSL=false&rewriteBatchedStatements=TRUE
  jpa:
    database: MYSQL
    hibernate.ddl-auto: update
    show-sql: true
    database-platform: com.example.springtransactionisolation.config.MySQL5InnoDBDialectUtf8mb4
    properties.hibernate.jdbc.batch_size: 100
    properties.hibernate.show_sql: ${spring.jpa.show-sql}
    properties.hibernate.format_sql: ${spring.jpa.show-sql}

logging.level.org.hibernate.SQL: debug
logging.level.org.hibernate.type: trace
```

#### entity
```java
@Data
@Entity
@NoArgsConstructor
@Table(name = "order")
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", nullable = false)
    private Long id;

    @Enumerated(EnumType.STRING)
    @Column(name = "order_status", nullable = false)
    private OrderStatus orderStatus;

    @Column(name = "update_date", nullable = false)
    private LocalDate updateDate;

}
```

#### Sql syntax log
```bash
2019-02-27 16:30:57.626  WARN 73285 --- [           main] o.h.t.s.i.ExceptionHandlerLoggedImpl     : GenerationTarget encountered exception accepting command : Error executing DDL "
    create table order (
       id bigint not null auto_increment,
        order_status varchar(255) not null,
        update_date date not null,
        primary key (id)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;" via JDBC Statement
```

### Soluation
Because the sql table `order` is a [reserved word in MySQL](https://dev.mysql.com/doc/refman/8.0/en/keywords.html#keywords-8-0-detailed-U) so that hibernate sql syntax executed error.

So I renamed the `order` entity to `order_form`...
```java
@Data
@Entity
@NoArgsConstructor
@Table(name = "order_form")
public class OrderForm {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", nullable = false)
    private Long id;

    @Enumerated(EnumType.STRING)
    @Column(name = "order_status", nullable = false)
    private OrderStatus orderStatus;

    @Column(name = "update_date", nullable = false)
    private LocalDate updateDate;

}
```

在命名 table 或 column 的時候真的不能太隨便啊...

### Ref
* MySQL Docs - [Reserved word in MySQL](https://dev.mysql.com/doc/refman/8.0/en/keywords.html#keywords-8-0-detailed-U)
* Same Issue - [StackOverFlow](https://stackoverflow.com/questions/50727853/you-have-an-error-in-your-sql-syntax-hibernate-and-mysql)