---
title: "[n00bzCTF] Passwordless"
tags:
    - CTF
    - Web
    - n00bzCTF
date: "2024-08-12"
thumbnail: "/assets/CTF/Web/240812_Passwordless/thumbnail.png"
---

# Intro
---
이 포스트에서는 n00bzCTF에서 주최한 CTF의 Web 문제에 대하여 살펴볼 예정이다.
> Tired of storing passwords? No worries! This super secure website is passwordless! Author: NoobMaster

# Analysis
---
해당 사이트는 `passwordless Username`을 통해서 로그인을 수행한다.
![site](/assets/CTF/Web/240812_Passwordless/01_analysis_1.png){: style="border: 1px solid;"}

Username 별로 고유의 UUID가 존재하고, `/UUID` 페이지에 접근 시 사용자에 해당하는 페이지를 렌더링한다.
만약 Username 자체를 `admin123`으로 입력하여 요청한 것이 아닌, 접근한 페이지에 해당하는 UUID가 `admin123의 UUID`라면 FLAG를 출력하는 구조이다.

```Python
leet=uuid.UUID('13371337-1337-1337-1337-133713371337')

@app.route('/',methods=['GET','POST'])
def main():
    global username
    if request.method == 'GET':
        return render_template('index.html')
    elif request.method == 'POST':
        username = request.values['username']
        if username == 'admin123':
            return 'Stop trying to act like you are the admin!'
        uid = uuid.uuid5(leet,username) # super secure!
        return redirect(f'/{uid}')
@app.route('/<uid>')
def user_page(uid):
    if uid != str(uuid.uuid5(leet,'admin123')):
        return f'Welcome! No flag for you :('
    else:
        return flag
```

### uuid.uuid5
Python Docs에서 `uuid.uuid5(namespace, name)`에 대한 공식 정의는 다음과 같고, 간단하게 말하자면 `namespace(UUID)`와 `name`을 기반으로 고유한 UUID를 생성하는 uuid 모듈 내 함수이다.<br>
> Generate a UUID based on the SHA-1 hash of a namespace identifier (which is a UUID) and a name (which is a bytes object or a string that will be encoded using UTF-8).

# Exploit
---
해당 페이지는 admin123에 해당하는 UUID를 알 수 있다면, 직접적으로 `/{admin123의 UUID}` 페이지에 접근하여 FLAG 확인이 가능하다.

`uuid.uuid5(namespace, name)`의 namespace는 leet 변수로 고정되어 있으므로 name을 admin123으로 설정하여 admin123 계정의 UUID를 획득하자.

```Python
from flask import Flask
import uuid

leet = uuid.UUID('13371337-1337-1337-1337-133713371337')
print(str(uuid.uuid5(leet, 'admin123'))) # 3c68e6cc-.....
```

이제 획득한 admin123 계정의 UUID 페이지로 접근하면 FLAG를 획득할 수 있다.
![flag](/assets/CTF/Web/240812_Passwordless/02_exploit_1.png){: style="border: 1px solid;"}
