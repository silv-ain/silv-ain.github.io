---
title: "AngularJS"
tags:
  - Knowledge
  - Web
  - AngularJS
date: "2024-10-11"
thumbnail: "/assets/Knowledge/Web/20241011_AngularJS/thumbnail.png"
---

# Intro

---

AngularJS LTS는 2022년을 마지막으로 공식적인 지원이 종료되었다.
그러나 여전히 AngularJS로 개발된 페이지가 존재하기 때문에 해당 프레임워크에 대하여 간단히 소개하려 한다.

# AngularJS

---

AngularJS는 클라이언트 사이드의 `MVC 구조`를 제공하는 Javascript 기반의 프론트엔드 웹 애플리케이션 프레임워크이다.

SPA(Sigle Page Application)는 <u>하나의 웹 페이지가 실행될 때 페이지 주소가 바뀌지 않으면서 새로운 View를 동적으로 로드</u>하는 것으로, 단일 페이지 내에서 사이트의 전체 컨텐츠를 로드한다. 즉, 메인 페이지가 로드되면 링크를 클릭해도 전체 페이지가 다시 로드되지 않고 페이지 자체의 섹션만 업데이트되는 구조이다.


### MVC 구조

MVC는 Model, View, Controller의 약자로, 애플리케이션을 세 가지 역할로 구분한 디자인 패턴의 한 종류이다.
![MVC Architecture](/assets/Knowledge/Web/20241011_AngularJS/MVC Architecture in AngluarJS.png){: style="border: 1px solid; padding:3%; width:94%;"}

- Model : <u>데이터를 정의하는 부분</u>으로 내부적으로 JSON 구조로 설정된다. 데이터 셋팅을 HTML 태그를 이용해서 값을 참조하게 되고, 참조하기 위해 HTML 확장 속성인 `ng-model`을 사용한다.
- View : <u>사용자가 보는 환경</u>으로 HTML/CSS 등을 의미하며, View 단에서 Model과 Controller를 정의한다.
- Controller : 실질적으로 <u>데이터에 대한 처리 및 업데이트</u>를 수행하는 역할을 수행하며, Model에서 데이터를 가져와 요청을 수행하고 처리된 데이터를 다시 View에 갱신해준다. `$scope`를 이용하여 View에서 등록된 Model의 데이터를 Controller에서 사용 및 업데이트하는 **양방향 데이터 바인딩**을 수행한다. Controller를 선언하기 위해 HTML 확장 속성인 `ng-controller`를 사용한다.

사용자가 브라우저에 URL을 입력하면 해당 URL에 따라 서버에서 Controller를 호출한다.
Controller는 요청에 맞는 Model과 View를 사용하여 사용자 요청에 대한 응답을 생성한 후 해당 응답을 사용자에게 전송한다.

### MVC Code

AngularJS의 Code 구조는 트리 구조로 계층적으로 구조화되어 있다.

가장 상위의 속성은 `ng-app`으로, 보통 <html>태그에 설정한다. 이는 JS 부분에서 module()을 이용하여 참조할 수 있다.
두 번째로 Controller 구현 시 보통 div 단위로 이루어진다. 하나의 기능 구조를 Controller로 묶어 그 영역의 기능을 수행하는 Controller 코드를 만들 수 있고, 기능 영역에 따라 여러 개를 만들 수 있다.
마지막으로 Controller에서 Binding을 처리할 수 있게 도와주는 인터페이스 `$scope`를 이용하여 실시간으로 View에 데이터를 업데이트한다.

$rootScope는 ng-app 태그가 포함된 HTML 문서 최상단에 위치하는 $scope로, 하나의 HTML 에서 여러개의 ng-controller 를 정의할 경우 계층적으로 $scope 를 갖게 되어 자연스럽게 계층 구조를 가지게 되는 것이다.
이에 의해 하위 $scope는 정의되지 않은 속성(메서드)을 찾기 위해 상위 계층으로 올라가며 자동으로 탐색한다.

```html
<html ng-app="myApp">
  <div ng-controller="myCtrl"></div>
</html>
```

위의 코드와 같이 myApp 모듈의 myCtrl 컨트롤러를 이용하여 실시간으로 View에 데이터를 Binding한다.

```js
/* app.js */
angular.module('myApp', ['ui.router']);

/* controllers > myCtrl.js */
myApp.controller('myCtrl', function($scope, myService) {
    $scope.testval = 'TEST VALUE';
    $scope.testFunc = function() {
        myService.getTest($scope);
    }
}

/* services > myService.js */
myApp.factory('myService', function($q, $http) {
    return {
        getTest: function($scope) {
            $http.get('/getTest.json').success(function(response){
                if (response.flag == 1) {
                    $scope.testval = response.result;
                }
            })
        }
    }
})
```

### Route

Routing 기능은 SPA(Single Page Application)를 지향하는 프레임워크에서 사용하는 중요한 기능이다.
다수의 UI를 가진 AngularJS의 경우 각 UI를 서로 다른 HTML 파일로 만들고, <u>URL 주소에 따라 특정 화면으로 이동시켜주는 기능</u>이다.

SPA 환경에서는 단일 페이지에서 HTML Element 교체로 이루어져야 하는데, 이를 route 모듈과 ng-view directive의 조합으로 가능하게 한다. `href 방식(#)`으로 이동할 페이지의 링크를 설정한 후 버튼을 누르면, 라우터 기능을 이용하여 <u>페이지의 리로드없이</u> 해당 페이지를 보여준다.

### Scope

AngularJS에서의 Scope는 Controller나 Directive의 유효 범위 내 저장공간이라고 생각하는 것이 이해하기 쉽다.
View는 Scope를 통해 Controller 내부에 정의된 Model(Data)이나 핸들러 함수에 접근할 수 있다. 즉 해당 Controller의 Scope에서 Model과 핸들러 함수를 찾게 된다.

```html
<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.9/angular.min.js"></script>
<div ng-app="myApp" class="ng-scope" style="display: flex; gap: 20px;">
  <div ng-controller="birthPlace" class="ng-scope">
    <p>Where are you from?</p>
    Country: <input type="text" ng-model="country" /><br />
    City: <input type="text" ng-model="city" /><br />
    <input type="button" value="Click" ng-click="born()" /><br />
    <p ng-bind="bornPlace"></p>
  </div>
  <div ng-controller="personalIdentification" class="ng-scope">
    <p>Please enter your personal information.</p>
    Name: <input type="text" ng-model="name" /><br />
    Age: <input type="text" ng-model="age" /><br />
    E-mail: <input type="text" ng-model="email" /><br />
    <input type="button" value="Click" ng-click="enter()" /><br />
    <p ng-bind="personalData" style="white-space: pre-line;"></p>
  </div>
</div>
```

```js
var myApp = angular.module("myApp", []);
myApp.controller("birthPlace", function ($scope) {
  $scope.born = function () {
    $scope.bornPlace = "I am from " + $scope.country + ", " + $scope.city;
    $scope.$apply();
  };
});
myApp.controller("personalIdentification", function ($scope) {
  $scope.enter = function () {
    $scope.personalData = "I'm " + $scope.name + ", " + $scope.age + " years old.\n " + "If you need to contact me, you can reach me at " + $scope.email;
    $scope.$apply();
  };
});
```

위의 예시 코드를 실행하게 되면, 사용자가 값을 입력할 때마다 scope에 coutnry, city 변수에 저장되며 버튼을 클릭한 경우 사용자가 입력한 값이 bornPlace 변수에 저장된 후 출력되게 된다.
![UI](/assets/Knowledge/Web/20241011_AngularJS/example.png){: style="border: 1px solid; padding:3%; width:94%; "}

<br>
$scope에 선언된 함수는 기존 Javascript 방식처럼 **개발자도구 > 콘솔**에서 바로 실행이 불가하며, 함수가 선언된 controller 객체를 가져와서 실행해야 한다. 예를 들면 아래와 같다.

```js
angular.element(document.getElementsByClassName("ng-scope")[1]).scope().born();
angular.element(document.querySelectorAll('[ng-controller="birthPlace"]')[0]).scope().born();
angular.element(document.querySelector("body > div > div:nth-child(2)")).scope().enter();
```

### Directives

Directives는 HTML을 확장하는 새로운 속성으로, HTML 컴파일러($compile)에게 지정된 동작을 DOM 요소에 연결하거나 DOM 요소와 그 하위 요소를 변환할 수 있게 돕는다.
애플리케이션에 기능을 제공하는 내장 지시문과 사용자가 직접 Directives를 정의하는 지시문이 있다.
<br>

**내장 Directives**
내장 Directives 종류는 굉장히 다양하지만 그 중 일부만 설명하니, 더 궁금하면 [AngularJS Directives][AngularJS Directives]를 참고하자.
- ng-app : AngularJS 애플리케이션의 루트 요소 정의
- ng-bind : 애플리케이션 데이터를 HTML 요소에 연결(\{\{ 변수 \}\}와 동일)
- ng-model : HTML 컨트롤(input, select, textarea)의 값을 애플리케이션 데이터에 바인딩
- ng-repeat : HTML 요소 반복 <br>

<br>

사용자가 직접 Directives를 정의할 때는 `.directive` 함수를 이용한다.
지시어의 이름을 지정할 때에는 낙타 표기법을 사용하고, 호출 시에는 `-`으로 구분하여 호출한다. 
Directives를 생성하면 기본적으로 요소, 속성, 클래스, 주석으로만 제한되지만, `restrict` 속성을 통해 일부 메서드에 의해서만 호출되도록 제한할 수 있다.

```html
<body ng-app="myApp">
<h1-test></h1-test>
<script>
  var app = angular.module("myApp", []);
  app.directive("h1TestDirective", function() {
    return {
      restrict : "A",
      template : "<h1>Custom Directive h1TestDirective Created!</h1>"
    };
  });
</script>
```

**restrict 값**
기본적으로 값은 `EA`이며, 이는 요소와 속성 모두 Directive를 호출할 수 있음을 의미한다.
클래스(C) 사용 시에는 restrict 속성에 `C` 값을 추가해야 한다. 또한 주석(M) 사용 시에는 restrict 속성에 `M` 값 추가 및 replace 속성에 `true` 값을 추가해야 한다.
```html
- 요소(E) <h1-test-directive></h1-test-directive>
- 속성(A) <div h1-test-directive></div>
- 클래스(C) --> <div class="h1-test-directive"></div>
- 주석(M) <!-- directive: h1-test-directive -->
```


[AngularJS Directives]:https://www.javatpoint.com/angularjs-directives