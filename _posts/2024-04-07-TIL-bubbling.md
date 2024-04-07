---
title: "JavaScript의 이벤트 버블링"
excerpt: " "
categories:
  - Programming Language
tags:
  - JavaScript
toc: true
toc_sticky: true
# last_modified_at : 2022-04-12T00:30:00
---

## JavaScript의 이벤트 처리 흐름

HTML 문서 내부의 각 element들은 한 태그 안에 다른 태그가 위치하는 식으로 계층적 트리 구조를 이룬다. 이 때문에 특정 element에서 발생한 이벤트가 다른 element로 전파되는 **이벤트 전파 (Event Propagation)**이 발생하게 된다.

```javascript
<div onclick="alert('div clicked.')">
  DIV
  <p onclick="alert('p clicked.')">
    P
    <span onclick="alert('span clicked.')">SPAN</span>
  </p>  
</div>
```

위와 같은 HTML이 있다고 하자. 이 때 가장 안쪽에 있는 'SPAN' span 태그를 클릭하면 어떤 일이 일어날까?

span 태그를 클릭했으니 span 태그에 바인딩된 onclick 핸들러가 실행되면서 브라우저 대화창으로 'span clicked.'라는 문구만 출력될 것으로 예상할 수 있지만, 실제로 저 HTML을 브라우저에서 열어 보면 span을 클릭했는데도 3개의 alert 창이 모두 표시되는 것을 확인할 수 있다. 이렇게 이벤트가 연쇄적인 흐름을 형성하며 발생하는 것을 이벤트 전파라고 부르며, 이벤트 전파는 전파 방향에 따라 **이벤트 버블링**과 **이벤트 캡쳐링**으로 구분한다.

## 이벤트 버블링 (Event Bubbling)

이벤트 버블링은 **자식 엘리먼트로부터 부모 엘리먼트로** 향하는 이벤트 전파를 말한다. 이벤트가 가장 깊은 엘리먼트에서 최상위 부모 엘리먼트로 이동하는 것이 물속에서 떠오르는 거품과 닮았다고 해 버블링이라는 이름이 붙었다. 이벤트 버블링이 발생할 때 어떤 엘리먼트에서 이벤트가 발생하면 그 이벤트는 이벤트가 발생한 엘리먼트의 부모 엘리먼트로 연쇄적으로 전달되며 이는 이벤트가 최상위 요소에 도달할 때까지 반복된다. 위의 예시에서는 span 엘리먼트에서 마우스 클릭 이벤트가 발생했고, 이 이벤트가 상위의 p, div 엘리먼트까지 버블링되며 해당 엘리먼트에 바인딩된 onclick 이벤트 핸들러를 함께 실행했다고 말할 수 있다.

## 이벤트 버블링 제한

이벤트 버블링은 거의 모든 HTML 이벤트에서 기본적으로 발생한다. 그러나 몇몇 특수한 상황에서 이벤트의 버블링을 막아야 하는 상황이 있는데, 이 때는 `event.stopPropagation()`을 통해 이벤트가 더 이상 전파되지 않도록 막을 수 있다. 이렇게 이벤트 전파를 막을 수 있지만, 이벤트 버블링 자체가 후술할 이벤트 위임(Event delegation)을 통해 이벤트 핸들링을 더욱 쉽게 만들어주고 이벤트 전파를 막아야 하는 상황은 대부분 stopPropagation이 아닌 다른 방법으로도 해결을 할 수 있으므로 stopPropagation의 사용은 지양하는 편이 좋다.

## 이벤트 위임 (Event Delegation)

이벤트 버블링은 다양한 상황에서 이벤트 핸들링을 보다 쉽게 해 준다. 예를 들어, `<ul>` 태그 안에 수많은 `<li>` 태그가 있다고 하자.

```html
<ul>
  <li>Event 1</li>
  <li>Event 2</li>
  <li>Event 3</li>
  <li>Event 4</li>
  ...
  <li>Event 99</li>
  <li>Event 100</li>
</ul>
```

각각의 <li> 태그를 클릭하면 그 태그의 색깔을 바꾸는 JavaScript 코드를 넣으려고 한다. 만약 이벤트 버블링이 없다면 onclick 이벤트 핸들러를 100개의 <li> 태그에 따로따로 지정해 주어야 한다. 하지만, 모든 <li> 태그에서 발생한 onclick 이벤트는 <ul> 태그로 버블링되므로 <ul> 태그에 onclick 이벤트 핸들러를 한 번만 정의해주면 하위 100개 <li> 태그에서 발생한 이벤트를 모두 감지하고 적절한 핸들러 함수를 실행해 줄 수 있다. 이러한 이벤트 핸들링 기법을 이벤트 위임이라고 한다.

```javascript
document.querySelector("ul").addEventListener('click', (event) => event.target.style.color = 'red')
```

## `event.target`과 `event.currentTarget`

이벤트가 발생한 엘리먼트는 `event` 객체 내의 `target` 속성으로 확인할 수 있다. `event` 객체에는 `target` 속성과 `currentTarget` 속성이 있는데, 이름만 보면 차이가 없어 보이는 이 두 속성이 모두 존재하는 이유가 바로 이벤트 버블링과 이벤트 위임이다.

`event.currentTarget`은 **이벤트 핸들러가 추가된 엘리먼트**를, `event.target`은 **이벤트가 발생한 엘리먼트**를 말한다. 이벤트 버블링과 이벤트 위임에 의하면 어떤 엘리먼트에서 발생한 이벤트로 인해 이벤트 핸들러가 동작할 때, 그 핸들러가 꼭 실제로 이벤트가 발생한 엘리먼트에 바인딩된 것이 아닐 수 있다. 하위 엘리먼트에서 발생한 이벤트가 버블링되며 상위 엘리먼트의 이벤트 핸들러를 동작시켰을 수 있는 것이다. 이 과정에서의 구분을 위해 `event.target`과 `event.currentTarget`이 별도로 존재한다. 

## 이벤트 캡쳐링

이벤트 캡쳐링 (Event capturing)은 이벤트 버블링과는 반대로 **부모로부터 자식 엘리먼트로** 향하는 이벤트 전파를 말한다. 대부분의 이벤트들은 최상위 엘리먼트에서 이벤트가 발생한 target 엘리먼트까지 전달되는 캡쳐링 단계와, target 엘리먼트에서 최상위 엘리먼트로 전달되는 버블링 단계를 거친다. 그러나 우리가 일반적으로 사용하는 여러 이벤트 핸들러들은 기본적으로 버블링 단계에서만 실행되도록 설정되어 있기 때문에 캡쳐링 단계는 눈으로 확인하기 어렵고, 실제 코드상에서도 거의 쓰이지 않는다. 이런 단계가 내부적으로 존재한다 정도로 알고 넘어가면 될 것 같다.