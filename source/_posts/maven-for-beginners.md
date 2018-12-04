---
title: Maven 新手教學
cover: /img/home-bg.jpg
date: 2018-11-30 13:54:55
categories: ['java8']
tags: ['java', 'maven']
---
### Overview
> People develop abstractions by generalizing from concrete examples. Every attempt to determine the correct abstraction on paper without actually developing a running system is doomed to failure. No one is that smart. A framework is a resuable design, so you develop it by looking at the things it is supposed to be a design of. The more examples you look at, the more general your framework will be. 

Maven 擁有跟大多數的專案管理工具一樣, 具有建構專案, 整合測試, 管理套件相依, 檢查套件衝突問題, 友善但複雜的 plugins, 如果你有用過 `npm`, `composer` 那些工具的話, 大概就知道它的角色了。因為脫離了 intelliJ 許多東西都要自己來, 手動建立專案就是一個新的開始, 如果可以, 我還是推薦用 `intelliJ` 真的快很多啊, (tears。

在寫這篇的時候, 無聊稍微翻了一下 Maven 的起源來自於另一個 [Alexandria](http://jakarta.apache.org/alexandria/legacy/) 專案裡的一部分, 後來才轉入 [Turbine](http://turbine.apache.org/), HA 離題了。

### Dev environment
 - Mac
 - Java8
 - VSCode
  
### Maven in 5 mins 
[Maven in 5 mins](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html), 這是一個簡單瞭解 Maven 功能的教學, 初學者必看。
另一篇, 我推薦這個 [blog](http://puremonkey2010.blogspot.com/2017/06/maven-maven-part1.html), 基本的指令說明的很清楚。
建立專案的起手式
```bash
mvn archetype:generate \
  -DarchetypeGroupId=org.apache.maven.archetypes \  # 專案的 archetypes, 可以替換自己喜歡的
  -DgroupId=com.mycompany.app \                     # package name
  -DartifactId=my-app                               # app name
```

官方提供了幾種專案架構(J2EE, JSR-268, Webapp, etc.), 我沒有全部都用過, 細節可以 [參考這裡](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html)。

### Archetype-quickstart-jdk8
[https://github.com/ngeor/archetype-quickstart-jdk8](https://github.com/ngeor/archetype-quickstart-jdk8)
```bash
mvn archetype:generate \
    -DgroupId=com.examples \
    -DartifactId=java-examples \
    -Dpackaging=jar \
    -DarchetypeGroupId=com.github.ngeor \
    -DarchetypeArtifactId=archetype-quickstart-jdk8 \
    -DarchetypeVersion=1.2.0 \
    -DinteractiveMode=true
```

Archetypes, 我會推薦這個, 而且也比較符合團隊開發用, 支援:
* Java version set to 8
* jUnit updated to latest (4.12)
* fixed indentation and formatting (4 spaces)  
* .gitignore file
* checkstyle rules
* travis support
* checkstyle plugin
* jacoco plugin
* javadoc plugin
* coveralls plugin

架構長這樣
```bash
.
├── pom.xml
└── src
    ├── main
    │   └── java
    │       └── com
    │           └── examples
    │               └── App.java
    └── test
        └── java
            └── com
                └── examples
                    └── AppTest.java
```

### 測試與打包
```bash
cd java-examples
mvn clean package
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.examples.AppTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.05 sec

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO]
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ java-examples ---
[INFO] Building jar: /Users/jerry80409/java/java-examples/target/java-examples-1.0-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 4.049 s
[INFO] Finished at: 2018-11-30T15:23:16+08:00
[INFO] ------------------------------------------------------------------------
```

### 套件管理
`.pom` 就類似 npm 的 `package.json` 應該很好懂吧, 套件的依賴就寫在 `<dependencies>` tag 底下, **archetype-quickstart-jdk8** 自帶 `junit4`, 很棒吧。
```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### Profile 設定
Profile 標籤允許依據不同的環境來執行不同的建構與 plugins 組合, **archetype-quickstart-jdk8** 裡面就包含了兩種 profiles (`jacoco`, `travis`)。
```xml
<profiles>
    <!--
    This profile enables jacoco when unit tests are run.
    You can run it with mvn -P jacoco test.
    It also activates itself on Travis.
    -->
    <profile>
        <id>jacoco</id>
        ...
    </profile>

    <!--
    For the Travis profile:
    - we want to break the build on any checkstyle violation.
    - we want to be able to publish coverage report to coveralls.
    -->
    <profile>
        <id>travis</id>
        ...
    </profile>
</profiles>    
```

[jacoco](https://www.jacoco.org/jacoco/trunk/doc/maven.html) 是用來檢查你的測試 (Junit) 覆蓋了多少比例的程式碼, 執行完之後會在 `/target/site/jacoco/` 出一份 Report。
```bash
mvn help:describe -Dplugin=org.jacoco:jacoco-maven-plugin -Ddetail    # 看文件
mvn test -Pjacoco
```
![maven-jacoco](/img/maven-for-beginners/maven-jacoco.png)

### Help:describe plugin
任何的 plugin 都可以用 `help:describe` 來取得說明, 基本用法
```bash
mvn help:describe -Dplugin:[group-id]:[artifact-id] -Dgoal=[some-goal]
```

### Plugin 設定
Plugin 是 `build` 標籤底下的子標籤, 功能很強大也很複雜, 可以用來做很多自動化的事情節省不少時間, 在說明之前建議先閱讀
* [https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)
* [https://www.baeldung.com/core-maven-plugins](https://www.baeldung.com/core-maven-plugins)

再來是了解 `phases`, `goals` 的概念, 是 plugin 的主要核心, 一般來說, 一個 `phase` 通常會對應多個 `goals`, 如果沒有對應的 `goals` 在 build 的時候就不會被執行, 基本的用法就是
```bash
mvn --help
usage: mvn [options] [<goal(s)>] [<phase(s)>]
```

以 [Introduction-to-the-lifecycle](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html) 裡面的範例來做解釋,
```bash
mvn clean dependency:copy-dependencies package
```
* `clean`, `package` 是 **phases**
* `dependency:copy-dependencies` 是 **goal**
* 這個指令會先執行 `clean`, 包含 pre-clean, clean 這兩個 phases
* 再執行 `dependency:copy-dependencies`, 這是 [maven-dependency-plugin](http://maven.apache.org/plugins/maven-dependency-plugin/) 的 goals 之一, 用來將依賴套件 (dependencies) 下載到指定位置
* 再執行 `package` 依據 pom.xml 裡面定義的 `<packaging>` tag 打包成對應的格式, 預設是 `jar`

### Jacoco plugin
是 **archetype-quickstart-jdk8** 模板裡面的其中一個 plugin, 在 travis 的環境底下會自動執行
* 在 `validate` phase 會做 `mvn jacoco:prepare-agent`, 
* 在 `test phase` 會做 `mvn jacoco:report`, 
* 等同於底下這行指令
```bash
mvn jacoco:prepare-agent test jacoco:report
```

* `pom.xml` 註解
```xml
<profile>
    <id>jacoco</id>
    <activation>
        <property>
            <!-- 這是在 travis 上面才會有的環境變數, 會在 travis 環境下自動執行相關的 build -->
            <name>env.TRAVIS</name>
        </property>
    </activation>
    <build>
        <plugins>
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <executions>
                    <!-- 這個 execution 綁定了 validate phase, 會執行 jacoco:prepare-agent -->
                    <execution>
                        <id>prepare-agent</id>
                        <phase>validate</phase>
                        <goals>
                            <goal>prepare-agent</goal>
                        </goals>
                    </execution>
                    <!-- 這個 execution 綁定了 test phase, 會執行 jacoco:report -->
                    <!-- 會在 /target/site/jacoco/ 底下產生報告 -->
                    <execution>
                        <id>report</id>
                            <phase>test</phase>
                            <goals>
                                <goal>report</goal>
                            </goals>
                        </execution>
                    </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
```

### Checkstyle plugin
是 **archetype-quickstart-jdk8** 模板裡面的另一個 plugin, 在 travis 的環境底下會自動執行
* 使的的是 `com/github/ngeor/checkstyle.xml` 的 checkstyle
* 在 `test` phase 會做 `mvn checkstyle:check`, 如果你想要看 html 的話, 可以改成`mvn checkstyle:checkstyle`, 就會產生 `/target/site/checkstyle.html`
```xml
<execution>
    <id>checkstyle</id>
    <phase>test</phase>
    <goals>
        <goal>checkstyle</goal>
    </goals>
</execution>
```

* `pom.xml` 註解
```xml
<profile>
    <id>travis</id>
    <activation>
        <property>
            <name>env.TRAVIS</name>
        </property>
    </activation>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-checkstyle-plugin</artifactId>
                <executions>
                    <!-- 這個 execution 綁定了 test phase, 會執行 checkstyle:check -->
                    <!-- 會在產生報告 /target/checkstyle-result.xml -->
                    <execution>
                        <id>checkstyle</id>
                        <phase>test</phase>
                        <goals>
                            <goal>check</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <!-- 會將報告提交到 coveralls -->
                <!-- https://docs.coveralls.io/java -->
                <groupId>org.eluder.coveralls</groupId>
                <artifactId>coveralls-maven-plugin</artifactId>
                <version>4.3.0</version>
                <!--
                <configuration>
                    <repoToken>your-coveralls-project-repository-token</repoToken>
                </configuration>
                -->
            </plugin>
        </plugins>
    </build>
</profile>
```

### Ref
* [Alexandria](http://jakarta.apache.org/alexandria/legacy/)
* [Archetype-quickstart-jdk8](https://github.com/ngeor/archetype-quickstart-jdk8)
* [Baeldung - core-maven-plugins](https://www.baeldung.com/core-maven-plugins)
* [Describe Plugin](https://maven.apache.org/plugins/maven-help-plugin/examples/describe-configuration.html)
* [History-of-maven](https://maven.apache.org/background/history-of-maven.html)
* [Introduction-to-archetypes](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html)
* [Introduction-to-the-lifecycle](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)
* [Kentyeh Maven 教學](http://kentyeh.github.io/mavenStartup/index.html)