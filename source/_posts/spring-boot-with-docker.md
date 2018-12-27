---
title: Docker 建立 Spring boot
cover: /img/home-bg.jpg
date: 2018-12-27 15:15:59
categories: ['spring-boot']
tags: ['docker', 'spring-boot']
---
### Overview
Spring boot 官方教學有一份文件就有介紹 [Spring boot with docker](https://spring.io/guides/gs/spring-boot-docker/), 裡面用的 [Maven plugin](https://github.com/spotify/dockerfile-maven) 來維護 `Dockerfile`, `DockerHub`。我的做法比較陽春, 單純地透過 Dockerfile 來掛載 maven 打包好的 jar, 這個做法只是單純的想熟悉 Docker build 的流程而已。

### Init Spring boot
[SpringBoot initializr](https://start.spring.io/), 用了 `thymeleaf`, `web` dependencies.
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### Bootstrap Admin
為了炫砲一點, 我用了 [Bootstrap Admin theme](https://startbootstrap.com/template-overviews/sb-admin-2/), 直接 clone 下來, 把資源丟到 `/src/main/resources/static/` 底下 XD.
```bash
# 這個 theme 依賴 gulp 工具
npm install --save-dev gulp@4.0.0
npm install

# 運行專案看ㄧ下畫面
mvn spring-boot:run
```
![bootstrap-admin-2](/img/spring-boot-with-docker/bootstrap-admin-2.png)

### Dockerfile
因為 Oracle 調整了 java 的授權, 所以 Docker 官方無法提供 `Oracle-Java` image 讓你 build, 可以改用 open-jdk 系列, 或者可以考慮用這個 [https://hub.docker.com/r/cogniteev/oracle-java/](https://hub.docker.com/r/cogniteev/oracle-java/), 只是他的 OS 是 Ubuntu 比較肥一點。
```
# FROM: 指定 base image (https://hub.docker.com/r/cogniteev/oracle-java/)
FROM cogniteev/oracle-java

# 打開 container 8080 port
EXPOSE 8080

# 版本管理, 接收 docker build --build-arg 指定的參數
ARG APP_VERSION

# 複製編譯後的 jar , 到容器裡面
COPY target/${APP_VERSION} /opt/app/app.jar

# WORKDIR: 指定 docker 執行起來時候的預設目錄位置
WORKDIR /opt/app

# 指定容器啟動後執行的命令，並且不會被 docker run 提供的參數覆蓋。
ENTRYPOINT ["java","-jar","app.jar"]
```

### Building the docker image
```bash
# clean and package the jar file
mvn clean package

# build docker image
docker build \
  --build-arg APP_VERSION=spring-thymeleaf-docker-0.0.1-SNAPSHOT.jar \    # pass APP_VERSION arg for build
  --tag jerry80409/spring-thymeleaf-docker:0.0.1 .    # give a tag for this image
```

### Running image
```bash
docker run -d \                                 # background running
    --name spring-thymeleaf-docker \
    --publish 8080:8080 \                       # 指定 local 與 container port 的對應
    jerry80409/spring-thymeleaf-docker:0.0.1    # 指定運行的 image 與 image 版本
```

確認一下執行狀況, 再打開 localhost:8080 確認
```bash
docker ps

# list docker process
CONTAINER ID        IMAGE                                      COMMAND               CREATED             STATUS              PORTS                    NAMES
68542645c037        jerry80409/spring-thymeleaf-docker:0.0.1   "java -jar app.jar"   4 seconds ago       Up 3 seconds        0.0.0.0:8080->8080/tcp   spring-thymeleaf-docker

```

### Remove container and image
```bash
# kill docker process by name
docker kill spring-thymeleaf-docker

# remove container
docker rm spring-thymeleaf-docker                    

# remove images
docker rmi jerry80409/spring-thymeleaf-docker:0.0.1   
```

### Reference
* Docker CLI docs - [https://docs.docker.com/engine/reference/commandline](https://docs.docker.com/engine/reference/commandline)
* Dockerfile maven plugin - [https://github.com/spotify/dockerfile-maven](https://github.com/spotify/dockerfile-maven)
* Spring boot with docker - [https://spring.io/guides/gs/spring-boot-docker/](https://spring.io/guides/gs/spring-boot-docker/)