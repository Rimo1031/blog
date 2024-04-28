---
title: "튜토리얼 웹 페이지 UI 만들기 (1)"
excerpt: " "
categories:
  - Programming Language
tags:
  - [JavaScript, React]
toc: true
toc_sticky: true
# last_modified_at : 2022-04-12T00:30:00
---

이번에 신규 서비스에 사용될 튜토리얼 기능을 제작하는 업무를 맡게 되었다. 업무를 위해 리서치를 진행하는데 튜토리얼 UI 제작에 대한 레퍼런스가 생각보다 많지 않은 것 같아 이번 기회에 작업 내용을 정리해 보려고 한다.

## 튜토리얼 UI?

정확한 용어가 따로 있는지는 모르겠지만, 많은 애플리케이션에서 볼 수 있는 튜토리얼 UI가 있다. 튜토리얼 단계를 진행하면서 현재 단계에서 설명하고자 하는 부분을 제외한 배경을 어둡게 처리하고 설명하려는 부분 옆에 말풍선 형태의 툴팁이 뜨는 UI를 많이 본 적이 있을 것이다.

![튜토리얼 UI]({{site.url}}{{site.baseurl}}/assets/img/tutorial-example.png)

이와 같은 UI를 구현해 보려고 한다.

## 구현 과정

### 튜토리얼 배경 처리

가장 먼저 해야 할 것은 튜토리얼 진행 중에 배경을 어둡게 처리하고 페이지 내의 원하는 위치만 밝게 표시하는 것이었다. 위의 이미지처럼 배경을 어둡게 처리하는 것을 dimmed라고 부르는데, CSS로 어렵지 않게 구현할 수 있다.

```css
.wrapper {
  background-color: black;
  opacity: 0.5;
}
```

이제 원하는 영역만 dimmed 처리하지 않고 원래 색상대로 밝게 표시하는 기능이 필요하다. 이 또한 여러가지 방법이 있는데, CSS의 `mix-blend-mode` 속성을 활용해 보기로 하였다.

밝게 표시하기를 원하는 영역의 배경 색상을 RGB(128,128,128)로 지정하고, 어두운 배경의 `mix-blend-mode` 속성을 `hard-light`로 설정하면 원하는 영역만 원래 색상대로 표시되게 할 수 있다.

```css
.wrapper {
  background-color: black;
  opacity: 0.5;
  mix-blend-mode: hard-light;

  .focus {
    background-color: rgb(128, 128, 128);
  }
}
```

`mix-blend-mode`는 서로 겹쳐진 두 요소의 색상을 어떻게 배합할지를 결정하는 옵션이라고 하는데, 원래 포토샵 등에서 디지털 이미지를 편집할 때 사용하는 용어라고 한다. 자세한 내용은 [위키피디아](https://ko.wikipedia.org/wiki/블렌드_모드)를 참고!

![화면 특정 영역 focus하기]({{site.url}}{{site.baseurl}}/assets/img/tutorial-step1.png)

### React 환경에서 튜토리얼 배경 렌더링하기

React에서는 기본적으로 컴포넌트들이 트리 형태의 구조를 이루게 된다. 그러나 코드 상에서의 컴포넌트 위치와 실제 DOM 트리 상에서의 위치를 다르게 렌더링해야할 필요가 생길 수 있다. 튜토리얼 UI를 비롯한 각종 모달(팝업) 창들을 예시로 들 수 있는데, 모달 컴포넌트들은 각자의 부모 컴포넌트 아래에 위치하지만, 실제로 렌더링될 때는 부모 컴포넌트와는 독립적으로 렌더링되어야 한다. React는 이러한 수요를 위해 `createPortal` 이라는 방법을 제공한다.

`createPortal`을 사용하면 자식 컴포넌트를 부모로부터 "탈출"시킬 수 있다. 첫 번째 인자로 렌더링할 컴포넌트를, 두 번째 인자로 그 컴포넌트의 부모가 될 컴포넌트를 지정하면 **현재 컴포넌트의 위치와 관계없이 두 번째 인자로 제공된 컴포넌트의 자식**으로 현재 컴포넌트의 DOM 트리 상 위치를 이동시킨다.

튜토리얼 UI의 경우는 React 트리 상에서의 위치만 특정 페이지 컨테이너의 하위에 위치하고, 실제로는 페이지 레이아웃과 완전히 독립적으로 렌더링되어야 한다. 만약 페이지의 하위에 위치한다면 그 페이지의 CSS 스타일 등에 튜토리얼 UI가 영향을 받을 수 있다. 이를 방지하기 위해 튜토리얼 UI의 배경과 말풍선을 최상위 Layout 컴포넌트 바로 아래에 렌더링되도록 `createPortal`을 사용하였다.

```jsx
...
const layout = document.getElementById('layout')
return (
  <>
    {createPortal(<div className={styles.wrapper}>
      <div className={styles.focus} />
    </div>, layout)}
  </>
)
```

### 포커스할 영역 위치 지정하기

튜토리얼 단계를 진행할 때마다 포커스되는 영역의 위치와 크기가 달라져야 한다. 이를 위해 step이 바뀌면 포커스할 영역 element에 미리 id를 할당하고 단계마다 각 단계에 대응하는 영역을 찾아 그 element의 위치, 크기를 읽어온 뒤 그에 맞추어 밝은 영역의 위치, 크기를 변경하도록 하였다.

```javascript
  useEffect(() => {
    const focusArea = document.getElementById(scenario[step].focusId).getBoundingClientRect()
    setFocusPosition(focusArea)
  }, [step])
  ...

  return (
  <>
    {createPortal(<div className={styles.wrapper}>
      <div className={styles.focus} style={{...focusPosition}} />
    </div>, layout)}
  </>
)
```

이 밖에도 구현했던 내용이 많지만, 포스트 길이상 다음 포스트로 분할하려고 한다.

## Reference

- 튜토리얼 UI 예시 이미지 : 요즘IT (https://yozm.wishket.com)
- [React 튜토리얼 컴포넌트 만들기](https://velog.io/@saemmmm/React-튜토리얼-컴포넌트-만들기)
