---
title: clean-and-pre-process-csv
cover: /img/home-bg.jpg
date: 2019-08-02 14:04:17
categories: ['misc']
tags: ['open-csv']
---
### Open-CSV features
I recommed [open-csv](http://opencsv.sourceforge.net/#features), It's a nice and aweasome package :)
* Arbitrary numbers of values per line. （允許一行裡面有多個 elements, 避免跟 headers 對應問題）
* Ignoring commas in quoted elements. (忽略 element 裡面的逗號, 避免誤判)
* Handling quoted entries with embedded carriage returns (i.e. entries that span multiple lines). (忽略 element 裡面的換行符號)
* Configurable separator and quote characters (or use sensible defaults). （可自定義 separator 與 quote 切分 csv 的字符）

### Ugly data
雖然 open-csv 可以協助處理 element 裡面的 **逗號**, **換行** 的情境, 但一些狀態仍可能導致 open-csv 判讀錯誤失敗, 以下方的例子來說, `\"` 與 `say "don’t` 就會違反 open-csv 的 parsing rule, 導致 exception 發生, 如果是用 try catch 去處理, 就會造成資料缺漏, 違反資料正確性。
```
"id","title","member","msg"
"1","title1","eric","Hello, I'm Eric."
"2","title2","john","Hello, Eric, I have a question about your work, 
 plz contact me \"
"3","title3","mike","This is Mike, I just want to say "don’t try to be someone you are not meant to be.""
"4","title4","jerry",NA
```

### Clean data
初步的想法就是, 把資料清洗一次, 先移除掉換行符號, 再把會影響 open-csv 處理的因素做 `replace()` 處理。
```java
    @Test
    public void clean_data() throws IOException {
        Charset utf8 = Charset.defaultCharset();
        File originFile = ResourceUtils.getFile("sample.csv");
        File cleanFile = ResourceUtils.getFile("clean_sample.csv");

        try (BufferedReader reader = Files.newBufferedReader(originFile.toPath(), utf8);
             BufferedWriter writer = Files.newBufferedWriter(cleanFile.toPath(), utf8)) {
            String line;

            while ((line = reader.readLine()) != null) {

                line = line.replace(",NA,", ",\"-\",");
                line = line.replace(",NA", ",\"-\"");
                line = line.replace(" \"", " `");
                line = line.replace("\"\"", "`\"");
                line = line.replace("\\\"", "\"");

                line = line.replaceAll("\\r\\n|\\r|\\n", "");
                writer.append(line);
            }
        }
    }
```

再重新換行。
```java
    @Test
    public void clean_data_with_new_line() throws IOException {
        File originFile = ResourceUtils.getFile("classpath:data/clean_sample.csv");
        File cleanFile = ResourceUtils.getFile("new_line_sample.csv");

        try (Reader reader = new InputStreamReader(new FileInputStream(originFile));
             Writer writer = new OutputStreamWriter(new FileOutputStream(cleanFile))) {

            // in one line have three comm
            int commInLine = 3;
            int current;
            int countComm = 0;
            boolean shouldNewLine = false;

            char currentChar;
            char beforeChar = ' ';

            while ((current = reader.read())!=-1) {

                currentChar = (char) current;

                // two double quotation 
                if (shouldNewLine
                        && beforeChar=='"'
                        && currentChar=='"') {

                    writer.append("\n");
                    writer.append(currentChar);
                    shouldNewLine = false;

                } else {
                    writer.append(currentChar);
                }

                // count comm and check new line status
                if (currentChar == ',') {
                    countComm++;
                    if (countComm == commInLine) {
                        countComm = 0;
                        shouldNewLine = true;
                    }
                }

                beforeChar = (char) current;
            }
        }
    }
```

### Final data
```
"id","title","member","msg"
"1","title1","eric","Hello, I'm Eric."
"2","title2","john","Hello, Eric, I have a question about your work, plz contact me "
"3","title3","mike","This is Mike, I just want to say `don’t try to be someone you are not meant to be.`"
"4","title4","jerry","-"
```


