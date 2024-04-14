
---
title: "useReducer를 활용한 React 상태 관리"
excerpt: " "
categories:
  - Programming Language
tags:
  - React
toc: true
toc_sticky: true
# last_modified_at : 2022-04-12T00:30:00
---

React에서 상태 관리를 할 때, 함수형 컴포넌트에서라면 대부분 `useState` hook을 사용해서 state 변수를 정의할 것이다. `useState` hook은 state 변수를 쉽고 편하게 관리할 수 있지만, 컴포넌트의 상태 구조가 복잡해지면 `useState` 만으로 상태 관리 로직을 구현했을 때 코드가 함께 복잡해지는 단점을 가지고 있다. 이 문제를 극복하기 위해 등장한 것이 `useReducer` hook이다.

## `useReducer` hook

`useReducer`도 `useState`와 동일하게 컴포넌트의 상태 관리에 사용하는 hook인 만큼, 기본적인 사용법 역시 `useState`와 비슷하다. `useReducer` hook은 **reducer 함수**와 **state의 초기값**을 인자로 받고, **state 변수**와 **dispatch 함수**의 튜플을 반환한다.

```javascript
const MyComponent = () => {
  const [User, dispatch] = useReducer(reducer, { name: 'John', age: 25, address: 'Chicago' })
}
```

`useReducer` hook은 조금 더 많은 인자와 반환값을 가지는데, 이 중에 `useState`와 다른 `reducer` 함수, `dispatch` 함수를 알아보도록 하자.

### `reducer` 함수

**state를 업데이트하는 로직**이 포함된 함수이다. reducer 함수는 *직전 state*와 *action*값을 인수로 받고, *action* 변수에 따라 *업데이트될 state*를 반환한다.

`action` 변수는 사용자가 어떤 행동을 취했는지와 그 행동의 context를 담은 일반 JavaScript 객체이다. action 객체의 구조에는 제한 사항이 없지만, 최소한 어떤 종류의 행동을 했는지를 알려주는 `type` field는 반드시 사용하도록 권장하고 있다.

### `dispatch` 함수

**state를 업데이트하도록 명령하는 함수**이다. `useState`에서의 `setState` 역할을 담당한다. `setState` 함수는 업데이트될 state값을 인자로 받지만, `dispatch` 함수는 `reducer` 함수에서 사용하는 `action` 객체를 인자로 받는다.

## `useState`와 `useReducer`의 차이점

`useState`와 `useReducer`는 모두 React에서 state 변수의 업데이트를 위해 사용하지만, 그 동작 방식에는 큰 차이가 있다. `useState`는 **무엇을 할 것인지**를 직접적으로 명시하지만 `useReducer`의 `dispatch` 함수는 **사용자가 방금 무슨 일을 했는지**를 React에 알려준다. 예를 들어 간단한 게시판 애플리케이션에서 게시글을 추가, 수정, 삭제하는 이벤트 핸들러를 작성한다고 하자. `useState`를 활용해 작성한다면,

```javascript
const [posts, setPosts] = useState([])

const handleAddPost = (newPost) => {
  setPosts([...posts, newPost])
}

const handleChangePost = (postId, newPost) => {
  setPosts(posts.map((post) => postId === post.id ? newPost : post))
}

const handleDeletePost = (postId) => {
  setPosts(posts.filter((post) => post.id !== postId))
}
```

와 같이 작성할 수 있을 것이다. 이 때 각각의 이벤트 핸들러에서는 추가, 변경, 삭제 상호작용에 대해 **state를 다음과 같이 설정하라.**는 식으로 이벤트 핸들러 안에서 state를 어떻게 변경할 지를 명시해 주고 있다. 반면 `useReducer`를 사용해서 같은 로직을 구현하면 다음과 같다.

```javascript
const reducer = (posts, action) => {
  switch (action.type) {
    case 'ADD': {
      return [...posts, action.newPost]
    }
    case 'CHANGE': {
      return posts.map((post) => action.postId === post.id ? action.newPost : post)
    }
    case 'DELETE': {
      return posts.filter((post) => post.id !== action.postId)
    }
  }
}

const [posts, dispatch] = useReducer(reducer, [])

const handleAddPost = (newPost) => {
  dispatch({ type: 'ADD', newPost: newPost })
}

const handleChangePost = (postId, newPost) => {
  dispatch({ type: 'CHANGE', postId: postId, newPost: newPost })
}

const handleDeletePost = (postId) => {
  dispatch({ type: 'DELETE', postId: postId })
}

```

`useReducer`를 사용하면 실제로 state를 업데이트하는 로직은 **reducer 함수 안으로 이관**되고 이벤트 핸들러 안에는 **사용자가 게시글을 추가 / 변경 / 삭제 했다는 action을 취했다**를 React에 알려주는 로직만 남는다. 

## `useState`와 `useReducer`, 어느 것이 좋은가?

같은 기능을 수행하는 hook인 만큼 `useState`와 `useReducer` 중 어느 것을 사용하는 것이 더 좋은지 비교하는 것은 피할 수 없을 것이다. React 공식 Docs와 여러 개발 블로그들에서 언급하는 각각의 장점에는 아래와 같은 것들이 있었다.

### `useReducer`의 장점

1. 관심사의 분리

사실 대부분의 이벤트 핸들러 함수는 사용자가 버튼을 누르거나, 마우스를 올리는 등 상호작용을 했을 때 **"사용자가 이런 상호작용을 했대!"** 라고 React에 알려주는 일까지만을 수행하는 것이 목적이지 **"이 상호작용이 발생하면 React의 state가 이렇게 업데이트 되어야 해."** 라는 내부 로직에까지 관여할 필요는 없다. `useReducer` hook을 사용하면 사용자의 상호작용을 React에 전달하는 역할은 이벤트 핸들러가 담당하고, 특정 상호작용이 발생했을 때 내부 state를 업데이트하는 역할은 reducer 함수가 담당하는 식으로 관심사의 분리가 가능해진다.

2. state 업데이트 구조가 복잡할 때

예를 들어 컴포넌트의 state는 하나인데 이 state를 업데이트하는 이벤트 핸들러가 매우 많다고 가정하자. `useState`의 `setState` 함수를 사용하면 각각의 수많은 `setState` 함수들이 모두 state를 어떻게 업데이트해야 하는지, 즉 state의 업데이트 로직을 포함해야 한다. 이는 곧 비슷비슷한 로직들이 수많은 이벤트 핸들러에 모두 존재한다는 것을 의미하고, 개별 이벤트 핸들러의 코드 양이 많아지면서 가독성 측면에서도 문제가 생길 수 있다.
`useReducer`를 활용하면 state 업데이트 로직을 모두 하나의 함수로 통합하고, 각각의 이벤트 핸들러에서는 `dispatch` 함수만을 실행하게 해서 디버깅도 간편하고 코드의 가독성도 높아지는 구조를 만들 수 있다.


### `useState`의 장점

1. 코드 크기

간단한 state의 업데이트에는 `useState`가 `useReducer`보다 더 짧은 코드를 작성할 수 있다. `useReducer`는 state 업데이트 로직을 reducer 함수에 따로 작성하고 dispatch 함수로 state 업데이트를 실행하는 로직을 별도로 추가해 주어야 하지만, `useState`를 활용하면 state 업데이트 로직의 정의와 실행을 한 번에 할 수 있기 때문에 작성해야 하는 코드의 양이 줄어든다.

2. 사용의 간편함

`useReducer`는 reducer와 dispatch 함수를 사용하는 구조가 복잡하다고 느껴질 수 있고, 사용법도 상대적으로 더 어렵다고 생각이 든다. 반면 `useState`는 state 변수를 정의하고, `setState` 함수의 인자로 state 변수를 변경한다는 매우 직관적인 사용법을 가지고 있어 간단한 state 관리에서는 `useReducer`보다 우위를 가진다고 할 수 있다.

이처럼 `useState`와 `useReducer`는 모두 일장일단이 있는 상호 보완적 관계라고 할 수 있다. 컴포넌트 구조를 디자인할 때 사용되는 state의 구조와 상황에 맞추어 적절한 hook을 사용하는 것이 좋은 코드 작성에 도움이 되리라 생각한다.

## 여담: 왜 이름이 `useReducer`인가?

React 공식 문서에서는 자바스크립트 배열의 `reduce()` 메서드에서 이름을 따왔다고 설명하고 있다. `reduce()` 함수는 인자로 reducer 콜백 함수를 받는데, 이 함수는 배열 내의 모든 요소에 대해 순차적으로 한 번씩 실행되면서 그 실행 결과를 하나의 변수에 저장하다가 배열 순회가 끝나면 그 변수에 저장된 값을 반환한다. 연산 결과를 하나의 변수에 누적(reduce) 한다고 해서 이런 이름이 붙었는데, `useReducer` hook도 이전 state에 대해 특정 action의 함수를 실행한 뒤 다음 state를 반환하는 일을 반복적으로 수행한다고 해서 같은 이름이 붙었다고 한다.