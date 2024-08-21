---
title: "[n00bzCTF] Sillygoose"
tags:
    - n00bzCTF
    - Programming
date: "2024-08-13"
thumbnail: "/assets/CTF/Programming/20240813_Sillygoose/문제.png"
use_math: true
---

# Intro
---
n00bzCTF에서 주최한 CTF의 Programming 문제에 대하여 살펴볼 예정이다.
> There's no way you can guess my favorite number, you silly goose. Author: Connor Chang

# Analysis
---
아래의 코드는 문제에서 제공된 코드 중 일부를 발췌한 코드로, 무작위로 생성된 큰 정수(0~$10^{100}$) 를 맞추는 게임을 수행하는 프로그램이다.
특정 시도 횟수 또는 시간을 넘어서면 게임에 실패하기 때문에 이를 고려하여 코드를 작성해야 한다.
```Python
ans = randint(0, pow(10, 100))
inp = input()
inp = int(inp)
if inp > ans:
    print("your answer is too large you silly goose")
elif inp < ans:
    print("your answer is too small you silly goose")
else:
    print("congratulations you silly goose")
    f = open("/flag.txt", "r")
    print(f.read())
```

### Binary Search Algorithm
이진 탐색 알고리즘은 <u>오름차순으로 정렬된 배열</u>에서 특정 값을 찾는 알고리즘이다. 배열의 중간을 기준으로 데이터를 탐색하기 때문에 반드시 정렬된 데이터를 대상으로 탐색을 수행해야 한다.

이진 탐색의 시간 복잡도는 $O(logN)$으로 배열을 전수 조사하는 $O(N)$에 비하면 상대적으로 빠른 탐색 알고리즘에 속한다.
$O(logN)$의 시간으로 값을 찾을 수 있는 이유는 탐색 대상을 절반씩 줄여나가기 때문이다.
![Binary Search](/assets/CTF/Programming/20240813_Sillygoose/01_analysis_1.gif){: style="border: 1px solid;"}

탐색할 데이터 범위가 넓거나 많은 경우, <u>상대적으로 짧은 시간과 적은 횟수</u>로 목표값을 찾을 수 있어 많이 사용하는 알고리즘이다.

# Exploit
---
Netcat을 이용하여 서버와 통신을 하므로 socket 모듈을 이용하여 `Binary Search Algorithm`을 구현하였다. 서버에서 반환되는 특정 문자열을 KEY로 사용하여 이진 탐색을 진행한다.
```Python
import socket

def find_answer(host, port):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((host, port))

    low, high, found, turns = 0, 10**100, False, 0
    
    while low <= high:
        turns += 1
        mid = (low + high) // 2
        guess = str(mid)

        s.sendall((guess + '\n').encode())
        response = s.recv(4096).decode()
        print(response)

        if "congratulations" in response:
            found = True
            break
        elif "too large" in response:
            high = mid - 1
        elif "too small" in response:
            low = mid + 1
        elif "skill issue" in response:
            print("Exceeded maximum number of turns.")
            break
    if found:
        print(f"Found the answer: {mid}")
    else:
        print("Failed to find the answer.")
    s.close()

find_answer('IP', PORT)
```

서버의 응답을 출력하여 결론적으로 FLAG를 획득하였다!
![FLAG](/assets/CTF/Programming/20240813_Sillygoose/02_exploit_1.png){: style="border: 1px solid;"}