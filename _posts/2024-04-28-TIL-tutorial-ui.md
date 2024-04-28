---
title: "튜토리얼 웹 페이지 UI 만들기"
excerpt: " "
categories:
  - Programming Language
tags:
  - [JavaScript, React]
toc: true
toc_sticky: true
# last_modified_at : 2022-04-12T00:30:00
---

회사에서 신규 서비스에 사용될 튜토리얼 기능을 제작하는 업무를 맡게 되었다. 업무를 위해 리서치를 진행하는데 튜토리얼 UI 제작에 대한 레퍼런스가 생각보다 많지 않은 것 같아 이번 기회에 작업 내용을 정리해 보려고 한다.

## 튜토리얼 UI?

정확한 용어가 따로 있는지는 모르겠지만, 많은 애플리케이션에서 볼 수 있는 튜토리얼 UI가 있다. 튜토리얼 단계를 진행하면서 현재 단계에서 설명하고자 하는 부분을 제외한 배경을 어둡게 처리하고 설명하려는 부분 옆에 말풍선 형태의 툴팁이 뜨는 UI를 많이 본 적이 있을 것이다.

<p align="center" style="color:gray">
  <!-- 마진은 위아래만 조절하는 것이 정신건강에 좋을 듯 하다. 이미지가 커지면 깨지는 경우가 있는 듯 하다.-->
  <img style="margin:50px 0 10px 0" src="https://rimo1031.github.io/assets/img/tutorial-example.png"  width=400 />
  (출처 : 요즘IT)
</p>

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
