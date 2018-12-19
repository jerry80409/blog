---
title: java8-convert-list-to-map
cover: /img/home-bg.jpg
date: 2018-12-13 15:15:41
categories: ['java']
tags: ['java']
---
### Json data
```json
[
    {"uid":"1","name":"阿花","age":38,"gender":"F"},
    {"uid":"2","name":"阿瓜","age":28,"gender":"M"}
]
```

### Entity
```java
@Data
@JsonInclude(JsonInclude.Include.NON_NULL)
@JsonPropertyOrder(value = {
    "uid",
    "name",
    "age",
    "gender"
})
public class Customer {
    // creativeId equals adId
    @JsonProperty("uid")
    private String uid;
    @JsonProperty("name")
    private String name;
    @JsonProperty("age")
    private Integer age;
    @JsonProperty("gender")
    private String gender;
}
```

### Test
```java
@Test
public void test() throws IOException {
    List<Customer> customerList =
        objectMapper.readValue(json, new TypeReference>() {});

    Map customerMap = customerList.stream()
        .collect(Collectors.toMap(
            Customer::getUid,       // key
            customer -> customer)); // value

    System.out.println(objectMapper.writeValueAsString(customerMap));
}
```

### Output
```json
{
  "1": {
    "uid": "1",
    "name": "阿花",
    "age": 38,
    "gender": "F"
  },
  "2": {
    "uid": "2",
    "name": "阿瓜",
    "age": 28,
    "gender": "M"
  }
}
```
