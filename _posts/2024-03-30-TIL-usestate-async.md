---
title: "useState hook의 batch update"
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
const [state, setState] = useState(0);
```

React는 `setState` 함수로 인해 state 변수가 변경되면 컴포넌트를 렌더링한다.

## React에서의 렌더링

React에서는 컴포넌트를 렌더링하는 것을 시간적으로 **스냅샷을 찍는다**라고 표현한다. 컴포넌트가 최초로 마운트되거나 컴포넌트 내의 state가 변경되어 컴포넌트 렌더링이 트리거되면, React는 컴포넌트 내부의 props, 이벤트 핸들러, 지역 변수 등을 모두 **렌더링 시점에서의 state 변수 값**을 사용해서 계산한다. 이 때문에 하나의 이벤트에서 `setState`를 여러 번 호출한다고 해서 state의 변경이 여러 번 발생하지는 않는다. 예를 들어,

```javascript
const StateTest = () => {
  const [count, setCount] = useState(0);

  const handleCountUp = () => {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
  };

  return (
    <div>
      <p>{`Count: ${count}`}</p>
      <button onClick={handleCountUp}>Click!</button>
    </div>
  );
};
```

위와 같은 컴포넌트가 있다고 가정하자. 코드만 보아서는 버튼을 클릭했을 때 `setCount` 함수가 3번 실행되면서 `count` state 변수를 1씩 3번 증가시키고, 최종적으로 화면에 표시되는 값은 3일 것이라고 예상할 수 있다. 하지만 앞서 언급한 내용처럼 컴포넌트 내부의 이벤트 핸들러는 **렌더링 시점에서의 state 값을 참조**하기 때문에, 여러 개의 `setCount` 내부에서 `count` 변수를 순서대로 참조하더라도 이벤트 핸들러 함수가 초기화되었던 렌더링 시점에서의 `count` 값은 0이었기 때문에 `count`는 1만큼만 증가하게 된다.

## state의 batch 업데이트

`setState` 함수가 state 변수를 변경하면서 리렌더링을 유발한다면, 위의 `handleCountUp` 함수를 실행하면 리렌더링이 3번 발생할 것이라고 기대할 수 있다. 하지만 실제로 함수가 실행되면 리렌더링은 마지막에 한 번만 발생하는 것을 알 수 있는데 이는 React가 state 변수를 업데이트하는 방식과 관련이 있다.

지금 `handleCountUp` 함수가 `setState`를 3번만 실행하지만, 만약 10,000번을 실행한다고 가정하자. 만일 React가 모든 `setState` 함수에 대해 state 변수를 변경하며 리렌더링을 실행한다면 이 이벤트 핸들러가 한 번 실행될 때마다 리렌더링이 10,000번 일어나고 이는 퍼포먼스의 심각한 저하로 이어질 것이다. 이 현상을 방지하기 위해 React는 큐를 사용해 여러 건의 state 업데이트를 한 번에 묶어서 처리하는데, 이를 **batching** 이라고 한다. 대표적인 예시로 React는 이벤트 핸들러의 모든 코드가 실행될 때까지 기다렸다가 state 업데이트 큐에 저장된 업데이트 내역을 순회하며 한번에 state를 업데이트한다.

### state의 수정은 단 한 번만 가능한가?

React가 state의 업데이트를 batching 기법으로 처리한다면 state 값의 변경은 단 한 번만 가능한 것일까? React에서는 위의 사례처럼 다음 렌더링이 발생하기 전에 state의 값을 여러 번 업데이트해야 하는 문제 상황에 대한 해결책을 제공하고 있다.

대부분의 상황에서 `setState`의 인자로는 새로운 state 변수에 저장될 **값**을 전달하지만, state를 연속적으로 변경해야 할 상황을 위해 이전 state값을 입력값으로 받는 **함수**도 `setState` 함수의 인자로서 전달할 수 있다. 이 함수를 updater 함수라고 하고 다음과 같이 사용한다.

```javascript
  setState((이전 값) => (변경할 값))
```

앞서 언급한 대로 React는 여러 개의 state 업데이트 요청을 큐에 저장해 이를 한 번에 순차적으로 계산하는 방식으로 state를 업데이트한다. 위와 같이 `setState`의 인자로 함수를 전달하면 React는 이 요청을 state 업데이트 큐에 삽입한다. 이 때 전달된 함수의 입력값은 **큐의 직전 요청에서 계산된 state값**이 된다.

```javascript
const [count, setCount] = useState(0);

const handleCountUp = () => {
  setCount((prevCount) => prevCount + 1);
  setCount((prevCount) => prevCount + 2);
  setCount((prevCount) => prevCount + 1);
};
```

위와 같은 React 컴포넌트(의 일부)가 있다고 하자. `handleCountUp` 함수가 실행되었을 때 React에서 생성하는 state 업데이트 큐는 다음과 같다고 생각할 수 있다.

| 이전 값 |         업데이트 요청          |   반환 값   |
| :-----: | :----------------------------: | :---------: |
|   `0`   | `(prevCount) => prevCount + 1` | `0 + 1 = 1` |
|   `1`   | `(prevCount) => prevCount + 2` | `1 + 2 = 3` |
|   `3`   | `(prevCount) => prevCount + 1` | `3 + 1 = 4` |

`setState` 함수의 인자로 state 업데이트 큐의 직전 반환값을 명시적으로 사용하라는 함수를 전달했기 때문에 state값을 연속으로 변경할 수 있었다.

## 업무에서 마주한 문제점

[이전 포스트](https://rimo1031.github.io/blog/programming%20language/TIL-chat/)에서 채팅창 UI를 개발할 때 채팅 내역을 state로 관리하는 방식으로 구현하였다. 채팅을 추가하기 위해서는 `setState` 함수를 사용해야 했는데, 지난 포스트에도 있는 내용인 _최초 입장 시 채팅 2개를 연속으로 추가_ 하는 기능을 구현하기 위해서 이 내용을 정확히 알아야 했다. 채팅 2개를 추가하는 이벤트 핸들러 함수를 정의하고 `useEffect`를 사용해 컴포넌트가 최초로 마운트될 때 이 함수를 실행하게 했다. 처음에는 updater 함수를 사용하지 않고 일반적인 용법대로 `setState([...이전 채팅, 새로운 채팅])`과 같이 구현했더니 두 번째 채팅이 추가되는 시점에 첫 번째 채팅이 지워지는 문제가 있었다. 이 문제가 바로 위에서 설명한 내용 때문에 일어났던 것이었다. `setState` 함수의 인자로 updater 함수를 사용하도록 변경해서 이 문제를 해결할 수 있었다.

## 느낀 점

프론트엔드에 거의 문외한이었던 내가 빠른 시간 내에 업무에 적응할 수 있었던 것은 잘한 일이라고 생각하지만 (물론 회사 동료분들의 멋진 서포트를 빼놓을 수 없음!!), 급하게 공부를 하다 보니 놓치고 지나갔던 내용들이 많다는 생각이 들었다. `useState`는 React로 프론트엔드 개발을 한다면 빼놓을 수 없는 기본적인 기능인데도 이런 내용을 몰랐다는 것을 조금 반성하게 되었다. 이 글을 작성하면서 [React 공식 Docs](react.dev)를 많이 참조했는데, 최근에 한 번 개편이 되더니 한국어 번역도 잘 되어 있고 내용이 더욱 좋아진 것 같다는 생각이 들었다. 앞으로 React Docs와 회사에서 사용하는 [Next.js Docs](https://nextjs.org/docs)를 정독하면서 놓쳤던 부분을 공부하고, 공부한 내용을 블로그에도 꾸준히 업데이트해 나가려고 한다.
