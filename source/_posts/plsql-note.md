---
title: plsql_note
cover: /img/home-bg.jpg
date: 2019-09-05 15:38:33
categories: ['dba'] 
tags: ['oracle']
---
### Just noting
Debug 可以考慮用 `DBMS_OUTPUT.PUT_LINE('your-outputs')` 來處理。

### Hello world
基本語法結構。
```sql
DECLARE
  -- 變數宣告區塊
  m_var VARCHAR2(10);
BEGIN
  -- 主要執行運算區塊
  m_var := 'HELLO';
  DBMS_OUTPUT.PUT_LINE(m_var);
END;
/
```

輸出
```sql
HELLO
```

---

### Create Type
建立自訂義型別, 類似 Java class 的概念。
```sql
-- 建立 m_type, 裡面有一個欄位是 bigint 類型
CREATE OR REPLACE TYPE student_type AS OBJECT (
    s_name VARCHAR2(10),
    s_score INT
);

DECLARE
  -- 宣告 stu 變數, 為 student_type 型別
  stu student_type;
BEGIN
  -- 類似 java 的 new instance
  stu := student_type('Dumdum', 100);
  DBMS_OUTPUT.PUT_LINE('My name is ' || stu.s_name);
END;
```

輸出
* p.s. 無法直接 output 整個 student_type 物件, 需要指定 attritube 去輸出

```
My name is Dumdum
```

---
### Create Type as Table
建立一個 table 類型的型別, 類似 Java collection 的概念 (泛型)。
```sql
-- 建立 student_type
CREATE OR REPLACE TYPE student_type AS OBJECT (
    s_name VARCHAR2(10),
    s_score INT
);

-- 建立一個裝 student_teyp 的 class_room
CREATE OR REPLACE TYPE class_room AS TABLE OF student_type;
```

```sql
DECLARE  
  class class_room := class_room();
	
BEGIN  
  class := class_room (  
    student_type ('John', 88),  
    student_type ('Max', 90),  
    student_type ('Lin', 100));  

  class.EXTEND;  
  class(class.LAST) := student_type('Chen', 56);  

  FOR stu IN (
    SELECT * FROM TABLE(CAST(class AS CLASS_ROOM))) 
  LOOP
    DBMS_OUTPUT.put_line('Name = ' || stu.s_name || ', Score = ' || stu.s_score);
  END LOOP;  

END;  
```

輸出
```bash
Name = John,  Score = 88
Name = Max,   Score = 90
Name = Lin,   Score = 100
Name = Chen,  Score = 56
```

---

### REGEXP_SUBSTR()
用正則表示, 來切割字串。
```sql
REGEXP_SUBSTR(
  'text',         -- char     , 要處理的字串
  'reg_pattern',  -- char     , 正則表示 expression
  position,       -- integer  , 正則表示驗證起始位置, 預設為 1
  occurrence,     -- integer  , 指定正則匹配成功後, 哪一個匹配成空要觸發字串切割, 預設為 1
  modifier        -- char     , i:不區分大小寫; c:區分大小寫; 預設為 'c' 
  )
```

範例
```sql
SELECT 
  regexp_substr(
    'This is Dumdum', 
    '[[:alpha:]]+', 
    1, 
    3
  ) the_3th_word
FROM dual;
```

輸出
```
| the_3th_word |
|--------------|
| Dumdum       |
```

---

### REGEXP_SUBSTR() 搭配 LEVEL
`LEVEL` 的用途主要是用來標示資料 樹(tree) 結構, 但實際上使用起來不是很直覺。

範例
```sql
SELECT 
  LEVEL, 
  regexp_substr(
    'This is Dumdum testing null', 
    '[[:alpha:]]+', 
    1,  
    LEVEL) regexp_substr
FROM dual
-- regexp_count, 用正則表示匹配來計算次數, 下面為例, 用來計算有幾個 '空白', ＋1 後就是實際 word 數量
CONNECT BY LEVEL <= regexp_count('This is Dumdum testing null',  ' ') + 1;
```

輸出
```
| LEVEL | REGEXP_SUBSTR |
|-------|---------------|
| 1     | This          |
| 2     | is            |
| 3     | Dumdum        |
| 4     | testing       |
| 5     | null          |
```

