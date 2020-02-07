---
title: gradle-101
cover: /img/home-bg.jpg
date: 2020-02-06 14:39:57
categories: ['dev_ops']
tags: ['gradle']
---
大部分的 Java 專案通常都會配置 [maven](https://maven.apache.org/) 或是 [gradle](https://gradle.org/) 來做專案管理, 這兩個工具的角色就類似 PHP 的 [composer](https://getcomposer.org/) 或是 nodejs 的 [npm](https://www.npmjs.com/), 可以用來協助建置專案, 套件依賴管理, 建立 task 模組做 deploy 或 testing。

選用 maven 或是 gradle 端看團隊成員的狀況, 如果有熟悉 Kotlin 或是 Groovy 選用 gradle 會比較好一點, 但 2 者都不會太難入門 (我自己覺得啦 XD), maven 雖然比較老派, 但一些套件比較穩定。

---
## Install
```bash
# install
brew install gradle

# check version
gradle -v
```

---
## Initial gradle project
```bash
# 建立 project 目錄
mkdir project-name

# 進入 project 目錄, 並且初始化 gradle 專案
cd project-name && gradle init

# list project 底下的檔案與目錄 (之後在細說)
ls 
build.gradle  gradle  gradlew  gradlew.bat  settings.gradle
```

* `build.gradle` 專案的主要維運腳本。
* `gradle` 包裝 gradle 程式的主體, 裡面有 [gradle-wrapper.jar](https://docs.gradle.org/4.10.3/userguide/gradle_wrapper.html), 所以 gradle 運行時需要 JVM 的環境。
* `gradlew` 是一個以直接在 UNIX 環境執行的執行檔, 會比較建議執行 gradle 相關操作時使用 `gradlew`, 不用在 target 或是 CI Runner 再安裝 gradle CLI。
* `gradlew.bat` 是一個可以直接在 WINDOWS 環境執行的執行檔。
* `settings.gradle` 是用來做此專案的 gradle 運行時的 configuration, 比如說: **[Multi-Porjects](https://docs.gradle.org/current/userguide/multi_project_builds.html)** 的架構, 或是添加 `org.gradle.daemon=true` 在背景運行之類的設定。

---
## Basic Task
### setting.gradle
```groovy
// 會在 Initialization phase 被執行
println 'This is executed during the initialization phase.'

rootProject.name = 'gradle-multi-module'
```

### build.gradle
```groovy
plugins {
    // 需要 java plugin
    id 'java'
}

task hello {
    doFirst {
        println 'Executed doFirst block!'
    }
    doLast {
        println 'Executed doLast block!'
    }

    // 這行會在 Configure project 階段執行
    println 'Hello World!'
}
```

### Execute task
```bash
./gradlew hello

# 也可以攜帶 -q 參數, 代表 quiet 模式, 會減少一些不必要的 stdout 輸出(比如說: project configure)
# ./gradlew -q hello

> Configure project :
Hello World!

> Task :hello
Executed doFirst block!
Executed doLast block!
```

---
## Build phases
參考 [build lifecycle](https://docs.gradle.org/current/userguide/build_lifecycle.html)
### Initialization
依據 `settings.gradle` 決定是 single / multi-projects 再去建立 Gradle 的 project instance.
> Gradle supports single and multi-project builds. During the initialization phase, Gradle determines which projects are going to take part in the build, and creates a Project instance for each of these projects.

### Configuration
官方說明很簡略, 就只是說在此階段會讀取整個 `build.gradle` 再去決定要 executed 的部分。
> During this phase the project objects are configured. The build scripts of all projects which are part of the build are executed.

### Execution
gradle 決定 task 的 subset(**doFirst**, **doFirst**), 經過 configuration 階段後再去執行。
> Gradle determines the subset of the tasks, created and configured during the configuration phase, to be executed. The subset is determined by the task name arguments passed to the gradle command and the current directory. Gradle then executes each of the selected tasks.

所以在上面的執行範例中, 看到 `Hello World` 會在 `Configure project` 階段就被執行, 所以在寫 task 的時候 **要注意這個 task 要在哪個 phases 被執行**。

---
## Task with groovy DSL
### build.gradle
```groovy
task count {
    doLast {
        4.times {
            println "$it"
        }
    }
}
```

### Execute task
```bash
./gradlew count   
This is executed during the initialization phase.

> Configure project :
Hello World!

> Task :count
0
1
2
3
```

---
## Task dependencies
### build.gradle
```groovy
// demo task
task hello {
    doFirst {
        println 'Executed doFirst block!'
    }
    doLast {
        println 'Executed doLast block!'
    }

    // 這行會在 Configure project 階段執行
    println 'Hello World!'
}

task greeting {
    dependsOn 'hello'
    doLast {
        println 'Morning~'
    }
}
```

### Execute task
```bash
./gradlew greeting
This is executed during the initialization phase.

> Configure project :
Hello World!

> Task :hello
Executed doFirst block!
Executed doLast block!

> Task :greeting
Morning~

```

---
## Dynamic tasks
### build.gradle
```groovy
4.times { counter ->
    task "task$counter" {
        doLast {
            println "I'm task number $counter"
        }
    }
}
```

### Execute task
```bash
./gradlew task1   
This is executed during the initialization phase.

> Configure project :
Hello World!

> Task :task1
I'm task number 1

```

## Zip task
### build.gradle
```groovy
task zip(type: Zip, group: 'Archive', description: 'Archives source in a zip file') {
    archiveName 'demo.zip'
    destinationDirectory = file("$buildDir/dist")
    from 'src'  // from 這邊的設定還不是很懂
}
```

### Execute Task
```bash
This is executed during the initialization phase.

> Configure project :
Hello World!
Zip files from src to /Users/jerry80409/eton/gradle-multi-module/build/dist
```