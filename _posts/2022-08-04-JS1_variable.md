---
title: "JavaScript 1. 변수와 상수"
excerpt: " "
categories: 
  - Programming Language
tags:
  - JavaScript
toc: true
toc_sticky: true
# last_modified_at : 2022-04-12T00:30:00
---


## 변수

다른 언어와 마찬가지로, JavaScript에서의 변수(variable)도 데이터를 저장하는 저장소의 역할을 한다. JavaScript에서의 변수 선언에는 `let` 키워드를 사용하고, 할당 연산자 `=`를 사용해 변수에 값을 할당한다.

```javascript
let variable = "Hello, world!";
```

한번 선언된 변수에 저장된 값은 덮어씌울 수 있고 (재할당), 기본 JavaScript 문법에서는 변수 선언 시 자료형을 선언하지 않으므로 어떤 타입의 변수든 할당할 수 있다.

변수의 이름을 정할 때는 알파벳 (대소문자 구별), 숫자, $, _의 기호만 사용할 수 있고 첫 글자는 숫자가 될 수 없다. 또, 예약어로 지정된 `let`, `return`, `function` 등의 키워드는 변수명으로 사용할 수 없다.

## 상수

변화하지 않는 값을 의미하며, `const` 키워드를 통해 선언한다. 상수는 한번 값이 할당되면 그 값을 절대 변경할 수 없으며 재할당을 시도하면 오류가 발생한다. 자주 사용되지만 기억하기 어려운 값을 상수로 선언하여 코드 가독성을 높이거나, 런타임에서 계산되어 최초로 할당된 뒤에 값이 변하지 않을 경우 상수를 사용할 수 있다. 전자의 경우 (하드 코딩된 경우)에는 상수명을 대문자로 짓는 것이 일반적이다.

```javascript
const COLOR_WHITE = "#FFFFFF";
const pageLoadTime = /* 페이지 로딩 시간 */;
```
