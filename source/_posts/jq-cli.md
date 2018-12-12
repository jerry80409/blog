---
title: jq 解析 json
cover: /img/home-bg.jpg
date: 2018-12-12 11:20:29
categories: ['misc']
tags: ['jq', 'json']
---
### jq
在 shell 環境下, 用來解析 json 字串的工具, 類似 `sed`, `awk`, `grep` 的字串處理工具。
[https://stedolan.github.io/jq/](https://stedolan.github.io/jq/)

### jq play
測試 jq 語法。
[https://jqplay.org/](https://jqplay.org/)

### 基本用法
格式化 response 方便閱讀。
```bash
curl 'https://api.github.com/repos/stedolan/jq/commits?per_page=5' | jq '.'
```

取得 element 0
```bash
curl 'https://api.github.com/repos/stedolan/jq/commits?per_page=5' | jq '.[0]'
```

取得 element 0, 並且將 `.commit.message` 的字串重新組合到 `message`, `.commit.committer.name` 重新組合到 `name`
```bash
curl 'https://api.github.com/repos/stedolan/jq/commits?per_page=5' | jq '.[0] | {message: .commit.message, name: .commit.committer.name}'
```

可以用 `[]` collection (array) 的形式, 打包 json 的處理結果
```bash
curl 'https://api.github.com/repos/stedolan/jq/commits?per_page=5' | jq '[.[] | {message: .commit.message, name: .commit.committer.name}]'
```

### 取得 docker bridge 的 ipv4
```bash
docker network inspect bridge
```

```json
[
  {
    "Name": "bridge",
    "Id": "fake-id",
    "Created": "2017-07-05T01:07:55.747497179Z",
    "Scope": "local",
    "Driver": "bridge",
    "EnableIPv6": false,
    "IPAM": {
      "Driver": "default",
      "Options": null,
      "Config": [
        {
          "Subnet": "172.17.0.0/16",
          "Gateway": "172.17.0.1"
        }
      ]
    },
    "Internal": false,
    "Attachable": false,
    "Ingress": false,
    "ConfigFrom": {
      "Network": ""
    },
    "ConfigOnly": false,
    "Containers": {
      "b59a12af7c87621b43c95a9adbcf047076b8a4b6b6ce0d6c5e6e48e21188ce6a": {
        "Name": "heuristic_rosalind",
        "EndpointID": "fake-endpoint-id",
        "MacAddress": "fake-mac-address",
        "IPv4Address": "172.17.0.2/16",
        "IPv6Address": ""
      }
    },
    "Options": {
      "com.docker.network.bridge.default_bridge": "true",
      "com.docker.network.bridge.enable_icc": "true",
      "com.docker.network.bridge.enable_ip_masquerade": "true",
      "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
      "com.docker.network.bridge.name": "docker0",
      "com.docker.network.driver.mtu": "1500"
    },
    "Labels": {}
  }
]
```

先把 `Containers` 先存起來
```bash
containers=$(docker network inspect bridge | jq '.[0].Containers' | jq 'keys[]')
echo $containers            # b59a12af7c87621b43c95a9adbcf047076b8a4b6b6ce0d6c5e6e48e21188ce6a
```

再拿來 select 一次
```bash
ipv4=$(docker network inspect bridge | jq ".[0].Containers.${containers}.IPv4Address")
echo $ipv4            # 172.17.0.2/16
```

