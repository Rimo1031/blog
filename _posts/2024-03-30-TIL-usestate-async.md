---
title: "useState의 비동기적 동작"
excerpt: " "
categories:
  - Programming Language
tags:
  - React
toc: true
toc_sticky: true
# last_modified_at : 2022-04-12T00:30:00
---

React의 함수형 컴포넌트에서는 컴포넌트의 상태(state) 관리를 위해 `useState`라는 hook을 사용한다. React로 개발하면서 가장 많이 사용하게 되는 hook 중 하나인데, [이전 포스트](https://rimo1031.github.io/blog/programming%20language/TIL-chat/)에서 state를 변경하는 함수인 `setState`의 동작에 대해 정확히 알고 있어야 구현이 가능한 기능이 있었어서 알게 된 점을 정리해 보려고 한다.

## `useState` hook

웹 페이지의 컴포넌트는 사용자와의 상호 작용 등의 원인에 의해 **보여지는 내용이 바뀌는 일**이 굉장히 많이 일어난다. 이 때 자신이 변화하면 그에 맞추어 화면 내용이 변화하도록 하는 변수를 state 변수라고 부르고, React의 함수형 컴포넌트에서 이 state 변수를 선언할 수 있게 하는 hook이 `useState` 이다.

`useState` hook은 state 변수와 이 변수를 변경시키는 함수인 `setState` 함수를 반환하며, 인자로는 state 변수의 초기값을 받는다. 보통 구조 분해 할당을 사용해 state 변수를 정의한다.

```javascript
  const [state, setState] = useState(0)
```
