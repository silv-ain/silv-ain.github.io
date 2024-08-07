---
title: "[Dreamhack] sql injection bypass WAF"
tags:
    - SQL
    - Injection
date: "2024-08-07"
thumbnail: "/assets/img/thumbnail/dreamhack.png"
---

# Intro
---
[Dreamhack][Dreamhack]의 WarGame 中 [sql injection bypass WAF][sql injection bypass WAF]문제와 [sql injection bypass WAF Advanced][sql injection bypass WAF Advanced]문제에 대한 **Write-up**을 작성하고자 한다.

# Analysis - sql injection bypass WAF
---
폼(uid)에 입력한 값이 SQL 쿼리문의 일부로 사용되어 그 결과가 응답값에 보여진다.
![사이트](/assets/sql_injection_bypass_WAF/analysis_website.png){:style="border:1px solid"}

아래의 코드는 해당 문제의 코드 일부를 발췌한 것으로, **SQL Injection 공격**을 통해 admin 계정의 패스워드(FLAG)를 획득하는 것이 목표이다. 파라미터(uid)값을 입력받아 keywords 배열 내 문자열에 대하여 필터링을 수행하는데, 이를 우회하는 것이 문제의 쟁점이다.
```python
keywords = ['union', 'select', 'from', 'and', 'or', 'admin', ' ', '*', '/']
def check_WAF(data):
    for keyword in keywords:
        if keyword in data:
            return True

    return False

uid = request.args.get('uid')
if uid:
    if check_WAF(uid):
        return 'your request has been blocked by WAF.'
    cur = mysql.connection.cursor()
    cur.execute(f"SELECT * FROM user WHERE uid='{uid}';")
```
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
- 대/소문자 구분을 이용한 문자열 필터링 우회 : union, select > UNION, SELECT
- 공백 우회 : 공백 > TAB


![FLAG](/assets/sql_injection_bypass_WAF/exploit_flag.png){:style="border:1px solid"}



[Dreamhack]:https://dreamhack.io/
[sql injection bypass WAF]:https://dreamhack.io/wargame/challenges/415
[sql injection bypass WAF Advanced]:https://dreamhack.io/wargame/challenges/416