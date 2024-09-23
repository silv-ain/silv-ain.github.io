---
title: "Converter"
tags:
    - Programming
    - Web
    - Converter
date: "2024-09-23"
thumbnail: "/assets/Programming/Web/20240923_Converter/thumbnail.png"
use_math: true
---

# Intro
---
로컬에서 HTML Entity, Base64 등을 치환하는 간단한 Converter 사용을 위해 Javascript 코드를 구현하였다. 

# Frame
---
먼저 Convert를 수행하기 위한 문자를 입력하는 등 상호작용하기 위해 HTML 코드로 간단한 UI를 생성하였다. 각각의 입력란에 `Encode`/`Decode`를 수행할 문자열을 입력한 후 버튼을 클릭하여 작동시키는 간단한 Converter다.
![UI](/assets/Programming/Web/20240923_Converter/UI.png){: style="border: 1px solid;"}


### HTML Code
꾸밈없이 간단한 뼈대를 만들었기 때문에 굳이 올리지 않아도 될 것 같지만 일단 참고해보던지!

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Converter</title>
</head>
<body>
    <script src="./convert.js"></script>
    <div id="html" style="float:left; margin:10px">
        <h1>HTML Entity Converter</h1>
        <textarea id="inputHtml" rows="10" cols="50"></textarea><br>
        <button onclick="encodeToEntities()">Encode</button>
        <button onclick="decodeFromEntities()">Decode</button>
    </div>
    <div id="url" style="float:left; margin:10px">
        <h1>URL Converter</h1>
        <textarea id="inputUrl" rows="10" cols="50"></textarea><br>
        <button onclick="encodeToUrl()">Encode</button>
        <button onclick="decodeFromUrl()">Decode</button>
    </div>
    <div id="base64" style="float:left; margin:10px">
        <h1>Base64 Converter</h1>
        <textarea id="inputBase64" rows="10" cols="50"></textarea><br>
        <button onclick="encodeToBase64()">Encode</button>
        <button onclick="decodeFromBase64()">Decode</button>
    </div>
    <div id="unicode" style="float:left; margin:10px">
        <h1>Unicode Converter</h1>
        <textarea id="inputUnicode" rows="10" cols="50"></textarea><br>
        <button onclick="encodeToUnicode()">Encode</button>
        <button onclick="decodeFromUnicode()">Decode</button>
    </div>
    <div id="UTF-8" style="float:left; margin:10px">
        <h1>UTF-8 Converter</h1>
        <textarea id="inputUtf8" rows="10" cols="50"></textarea><br>
        <button onclick="encodeToUtf8()">Encode</button>
        <button onclick="decodeFromUtf8()">Decode</button>
    </div>
    <div id="ascii" style="float:left; margin:10px">
        <h1>Asciicode Converter</h1>
        <textarea id="inputAsciicode" rows="10" cols="50"></textarea><br>
        <button onclick="encodeToAsciiCode()">Encode</button>
        <button onclick="decodeFromAsciiCode()">Decode</button>
    </div>
</body>
</html>
```

# Converter
---
이제 HTML 요소를 이용하여 Javascript로 다양한 문자들에 대하여 Convert를 수행하면 된다. 현재까진 생각없이 코딩해서 효율성이 없어보이지만.. 차차 고쳐가는 걸로!

```js
function encodeToEntities() {
    var textHtml = document.getElementById('inputHtml');
    var convertedTextHtml = ''
    for (var i=0; i < textHtml.value.length; i++) {
        convertedTextHtml += '&#' + textHtml.value.charCodeAt(i) + ';';
    }
    textHtml.value = convertedTextHtml;
}
function decodeFromEntities() {
    var textHtml = document.getElementById('inputHtml');

    var convertedTextHtml = textHtml.value.replace(/&#(\d+);/g, function(match, dec) {
        return String.fromCharCode(dec);
    });
    textHtml.value = convertedTextHtml;
}

function encodeToUrl() {
    var textUrl = document.getElementById('inputUrl');
    var convertedTextUrl = ''
    for (var i=0; i < textUrl.value.length; i++) {
        var char = textUrl.value.charAt(i);
        if(/[\W_]/.test(char)){
            convertedTextUrl += '%' + textUrl.value.charCodeAt(i).toString(16).toUpperCase();
        } else {
            convertedTextUrl += char;
        }
    }
    textUrl.value = convertedTextUrl;
}
function decodeFromUrl() {
    var textUrl = document.getElementById('inputUrl');
    textUrl.value = decodeURI(textUrl.value);
}

function encodeToBase64() {
    var textBase64 = document.getElementById('inputBase64');
    textBase64.value = btoa(textBase64.value);
    
}
function decodeFromBase64() {
    var textBase64 = document.getElementById('inputBase64');
    try {
        if (textBase64.value != "Invalid Base64 String") {
            textBase64.value = atob(textBase64.value);
        }
    } catch (e) {
        textBase64.value = "Invalid Base64 String";
    }
}

function encodeToUnicode() {
    var textUnicode = document.getElementById('inputUnicode');
    
    var convertedTextUnicode = textUnicode.value.replace(/./g, function(char) {
        return '\\u' + char.charCodeAt(0).toString(16).toUpperCase().padStart(4, '0');
    });
    textUnicode.value = convertedTextUnicode;
}
function decodeFromUnicode() {
    var textUnicode = document.getElementById('inputUnicode');
    var convertedTextUnicode = textUnicode.value.replace(/\\u([0-9A-Fa-f]{4})/g, function(match, group1) {
        return String.fromCharCode(parseInt(group1, 16));
    });
    textUnicode.value = convertedTextUnicode;
}

function encodeToUtf8() {
    var encoder = new TextEncoder();
    var textUtf8 = document.getElementById('inputUtf8');
    var convertedTextUtf8 = encoder.encode(textUtf8.value);

    var hexArray = Array.from(convertedTextUtf8, byte => '0x' + byte.toString(16).padStart(2, '0'));
    textUtf8.value = hexArray;
}
function decodeFromUtf8() {
    var decoder = new TextDecoder('utf-8');
    var textUtf8 = document.getElementById('inputUtf8');
    var hexArray = textUtf8.value.split(',').map(hex => parseInt(hex.trim(), 16));
    console.log(hexArray);

    var utf8Array = new Uint8Array(hexArray);
    var convertedTextUtf8 = decoder.decode(utf8Array);
    textUtf8.value = convertedTextUtf8;
}

function encodeToAsciiCode() {
    var textAsciicode = document.getElementById('inputAsciicode');
    var convertedTextAsciicode = '';

    for (var i = 0; i < textAsciicode.value.length; i++) {
        var charCode = textAsciicode.value.charCodeAt(i);
        if (charCode > 127) {
            textAsciicode.value = 'Invalid Asciicode';
            return;
        }
        convertedTextAsciicode += '\\x' + charCode.toString(16).toUpperCase().padStart(2, '0');
    }

    textAsciicode.value = convertedTextAsciicode;
}
function decodeFromAsciiCode() {
    var textAsciicode = document.getElementById('inputAsciicode');

     if (textAsciicode.value === 'Invalid Asciicode') {
        textAsciicode.value = 'Invalid Asciicode';
        return;
    }

    var convertedTextAsciicode = textAsciicode.value.replace(/\\x([0-9A-Fa-f]{2})/g, function(match, group1) {
        return String.fromCharCode(parseInt(group1, 16));
    });

    textAsciicode.value = convertedTextAsciicode;
}
```
