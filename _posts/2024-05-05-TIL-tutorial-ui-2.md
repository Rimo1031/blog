---
title: "튜토리얼 웹 페이지 UI 만들기 (2)"
excerpt: " "
categories:
  - Programming Language
tags:
  - [JavaScript, React]
toc: true
toc_sticky: true
# last_modified_at : 2022-04-12T00:30:00
---

지난 주에 이어서 업무로 제작한 튜토리얼 UI의 구현 과정을 정리해 보려고 한다.

## 브라우저 창의 크기가 변경될 때

실제 유저가 사용할 때 자주 일어나는 일은 아니겠지만, 기능을 제작하다 보니 튜토리얼 진행 중에 브라우저의 창 사이즈가 변경되면 튜토리얼 팝업의 위치가 망가지는 현상이 있었다. 이는 튜토리얼 팝업의 위치를 결정하는 로직이 step을 넘어가는 시점에만 실행되게 구현되어 있었기 때문이다. 그래서 step을 넘어가지는 않았지만 다른 이유로 현재 focus되어 있는 영역의 위치가 변경된다 해도 튜토리얼 팝업의 위치는 그대로이고, UI가 올바르게 보이지 않는 현상이 발생한 것이다. 이를 방지하기 위해 `resize` 이벤트에 튜토리얼 팝업의 모달을 다시 계산하는 이벤트 핸들러를 추가하였다. 이 때, 아무런 조치 없이 단순하게 함수를 추가하면 브라우저 창의 크기가 1px 단위로 변경될 때마다 무수히 많은 이벤트 핸들러가 실행되며 성능 저하를 유발할 수 있다. 이 현상을 방지하기 위해 **스로틀링**을 적용하였다. `setTimeout` 함수를 사용해 100ms마다 `resize` 이벤트의 핸들러가 실행되도록 설정해 이벤트 핸들러의 호출 횟수를 줄일 수 있었다.

```javascript
useEffect(() => {
  window.addEventListener("resize", () => {
    setTimeout(() => {
      setModalPosition(); // 튜토리얼 모달의 위치를 계산하는 함수
    }, 100); // 100ms마다 실행 => 성능 저하 방지 (스로틀링)
  });

  return () => window.removeEventListener("resize");
}, []);
```

이 기능을 구현하면서 state의 특성을 제대로 이해하지 못해 잠깐 헤맸던 경험이 있었다. 처음 resize 이벤트 핸들러를 구현하고 나서 테스트를 하는데, 이벤트 핸들러 자체는 잘 동작하지만 1단계가 아닌 단계에서 resize 이벤트를 발생시켰을 때 focus되는 영역이 1단계의 영역으로 되돌아가 버리는 문제가 있었다. 이 문제는 결론부터 말하면 **이벤트 핸들러는 React State의 변경사항을 감지하지 못한다**는 특성에서 생기는 문제였다.

이벤트 핸들러는 **최초 렌더링 시**에 초기화되고, 이후 React에 의한 re-render 과정에서 **업데이트되지 않는다.** 즉 이벤트 핸들러 내부에서 state가 사용된다고 해도 state의 업데이트에 따라 그 값이 바뀌지 않고, 최초 렌더링 시 이벤트 핸들러 함수가 초기화되며 받아온 state의 초기값만 계속 참조한다는 사실을 의미한다. 당연히 튜토리얼 step의 초기값은 1단계이므로 이벤트 핸들러가 실행될 때마다 계속 1단계 영역을 focus하도록 설정되고 있었던 것이다. 이 문제는 현재 진행중인 step을 관리하는 방식을 `useState`에서 `useRef`로 변경해서 해결할 수 있었다. React state의 특성에 대해 알 수 있었던 또 하나의 경험이 된 트러블슈팅이었다.

## '일주일 간 보지 않기' 기능

튜토리얼 기능 명세서에 있었던 '일주일 간 보지 않기' 기능을 구현하는데 조금 고민이 있었다. 사용자가 해당 버튼을 클릭하면 튜토리얼을 7일간 표출하지 않아야 하는데, 사용자가 버튼을 눌렀는지의 여부와 눌렀다면 언제 눌렀는지를 어떻게 저장할 것인지를 선택해야 했다. 처음에는 이 내용을 DB에 저장하고 API를 만들어서 백엔드 서버와 통신하도록 구현하려고 했다. 이 방법이 가장 확실하게 사용자별로 튜토리얼 비활성화 여부를 관리할 수 있는 방법이라고 생각했기 때문이었다. 그러나 이 기능과 관련해서 다른 팀원들과 논의를 해보면서 생각을 바꾸게 되었다. DB를 이용하면 가장 확실하게 관리할 수 있게 되는 것은 맞지만, 튜토리얼 내부의 부가적인 기능을 위해 DB 구조를 수정하고 API까지 새로 만들어야 하는 상황은 조금 과할 수 있다는 의견이 있었다. 팀 내에서 이 기능이 현재 개발해야 하는 기능 안에서 중요한 비중을 차지하고 있다면 그렇게 할 가치가 있지만, 사실 일주일 간 보지 않기 기능이 튜토리얼 기능 전체에서 크게 중요한 기능은 아닐 뿐더러 사용자의 튜토리얼 비활성화 여부가 DB에서 관리해야 할 정도로 엄격하게 사용자 단위로 구분되어 관리되어야 할 데이터는 아닌 것 같다는 의견도 주셔서 구현 방식을 다시 생각해보게 되었다. 확실히 DB를 이용하는 방식은 얻을 수 있는 가치에 비해 비용이 너무 크다는 생각이 들어 브라우저의 로컬 스토리지를 이용하는 방식으로 선회하였다.

UI에서 '일주일 간 보지 않기' 버튼을 누르면 튜토리얼을 강제로 비활성화하고 로컬 스토리지에 현재 날짜의 7일 뒤 날짜를 만료일로서 저장한다. 튜토리얼 UI가 처음 렌더링될 때 로컬 스토리지에 저장된 값을 검사해 현재 날짜가 만료일보다 이전이면 튜토리얼을 표시하지 않는 식으로 '일주일 간 보지 않기' 기능을 구현했다.

```javascript
const onClickDisableFor7Days = () => {
  const expDate = localStorage.getItem("key");
  if (new Date() < new Date(expDate)) {
    setShowTutorial(false);
  }
  return;
};
```

이 밖에도 좋은 튜토리얼 UI를 만들기 위해 적용했던 로직들이 있다. 처음 튜토리얼 기능을 만들 때 "최대한 공용으로 쓸 수 있도록 하자"라고 다짐하면서 기능 구현에 들어갔는데, 스스로 판단하기엔 아직 이르지만 특정 페이지에 의존하지 않고 독립적으로 다양한 페이지에 녹아들 수 있도록 모듈화가 잘 되었다고 생각한다. 나중에 다른 페이지에서도 튜토리얼 기능이 만들어져야 할 때 다른 팀원들이 불편함 없이 간단하게 튜토리얼을 적용할 수 있다면 그보다 보람찬 일이 없을 것 같다.
