---
title: "Next.js에서의 페이지 렌더링"
excerpt: " "
categories:
  - Programming Language
tags:
  - [CSS]
toc: true
toc_sticky: true
# last_modified_at : 2022-04-12T00:30:00
---

기본적으로, Next.js는 모든 페이지를 미리 렌더링 (pre-rendering) 한다. Next.js는 **모든** 페이지들에 대한 HTML 파일을 미리 생성하고, 생성된 HTML에 해당 페이지가 작동하기 위한 최소한의 JS 코드를 결합한다. 이렇게 만들어진 HTML 페이지는 React 프레임워크조차 적용되어 있지 않은 100% 정적인 페이지이다. 이 페이지가 브라우저에서 로딩되면 페이지 곳곳에 이벤트 핸들러가 부착되는 등의 interaction을 생성하는 작업이 일어나고, 우리가 처음 의도했던 SPA 어플리케이션을 완성한다. 이 과정을 React에서는 Hydration (수화, 수분 공급) 라고 부른다. 

## Pre-rendering

Next.js에서는 Static Generation, Server-side Rendering 두 가지 방식의 pre-rendering 방식을 지원한다. 두 방식의 차이점은 pre-rendering이 일어나는 **시점**에 있다. Static Generation은 pre-rendering HTML 파일을 **빌드 시에 생성**해서 매 요청마다 이 파일을 재활용하지만, Server-side Rendering은 HTML을 **매 요청마다 재생성**한다. Next.js에서는 개발자가 페이지의 특성에 따라 어떤 pre-rendering 방식을 사용할지 선택할 수 있게 한다. 하지만, Static Generation 방식으로 pre-rendering된 HTML은 별도 설정이 없어도 CDN 등에 캐시되어 퍼포먼스 등에서 우위를 점할 수 있다.

## Next.js에서의 렌더링 방식

### Server-side Rendering (SSR)

Next.js에서 Server-side Rendering을 사용하려면, 각 페이지의 컴포넌트에서 `getServerSideProps`라는 이름의 함수를 정의한다. 이 함수 내에서 API를 통해 데이터를 fetch해 컴포넌트에 props로 넘겨주는 등의 작업을 수행할 수 있다. `getServerSideProps`는 해당 페이지에 대한 모든 요청 때마다 실행된다.

```javascript
export async function getServerSideProps() {
  // Fetch data from external API
  const res = await fetch(`https://.../data`)
  const data = await res.json()
 
  // Pass data to the page via props
  return { props: { data } }
}
```

### Static Site Generation (SSG)

SSG 방식을 사용하는 페이지는 Next.js 애플리케이션을 빌드하는 시점에 생성되어 해당 페이지에 대한 모든 요청에 대해 재사용되고, CDN에 캐시될 수 있다.

#### 데이터 fetch가 없는 경우

기본적으로, Next.js는 외부로부터 데이터를 받아오지 않는 모든 페이지를 SSG 방식으로 pre-render한 후 재사용한다.

#### 데이터 fetch가 있는 경우

페이지 **내용**이 외부 데이터에 의존하는 경우와 페이지 **경로**가 외부 데이터에 의존하는 경우 두 가지로 나눌 수 있다. 각각의 경우에 `getStaticProps`와 `getStaticPaths` 함수를 사용하고, 사용 방법은 SSR에서의 `getServerSideProps`와 비슷하다. `getStaticProps`는 `getServerSideProps`와 동일하게 외부 데이터를 받아와 컴포넌트에서 사용할 수 있도록 props로 넘겨주는 역할을 하고, `getStaticPaths`는 외부에서 받아온 데이터를 토대로 pre-render할 경로의 배열을 생성해 반환한다. 생성된 경로들은 Next.js의 Dynamic Routing을 통해 라우팅된다.

### Incremental Static Regeneration (ISR)

SSG의 변형이라고 생각할 수 있는 렌더링 방식이다. 기본적으로 SSG의 형태로 동작하지만, 애플리케이션을 빌드할 때만 페이지를 생성할 수 있는 SSG와 달리 **주기적으로 Static Generation된 페이지를 업데이트**할 수 있는 기능을 제공한다. SSG 방식으로 렌더링하는 페이지는 대부분 외부 데이터에 의존하지 않거나, 의존하더라도 외부 데이터가 거의 변하지 않는 페이지일 것이다. 대표적으로 블로그 페이지를 들 수 있는데, 의존하는 데이터가 거의 변하지 않기는 하지만 매우 느린 주기로 업데이트될 때가 있다. (블로그 제목이나 내용을 수정하는 등) 순수한 SSG를 적용한 페이지라면 한번 애플리케이션을 빌드한 뒤 이를 업데이트할 방법이 없으므로 애플리케이션 전체를 다시 빌드해야 하는 불편함을 유발한다. 이 문제를 해결하기 위해 SSG에 `revalidate` 옵션을 적용, 해당 페이지의 렌더링된 HTML 파일을 일정 주기마다 업데이트하게 할 수 있다. 이 옵션이 적용된 페이지는 기본적으로 SSG 형식으로 동작하지만, revalidate 주기가 지난 뒤의 첫 번째 요청에 대해서는 `getStaticProps` 함수를 Server-side에서 다시 실행시켜 페이지를 다시 렌더링한다. 마치 SSR처럼 동작한다고 볼 수도 있는데, 차이점은 이렇게 렌더링된 페이지를 애플리케이션 내부에 저장해 다음 revalidate 주기가 지나기 전까지는 다시 SSG 방식으로 돌아간다는 것이다. 미리 페이지를 만들어 놓고 캐시해 응답 속도를 높이는 SSG의 장점과, 외부 데이터를 바로바로 반영할 수 있는 SSR의 장점을 절충한 렌더링 방식이라고 볼 수 있다.