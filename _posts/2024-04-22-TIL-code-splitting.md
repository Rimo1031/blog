---
title: "Next.js에서 코드 스플리팅 적용하기"
excerpt: " "
categories:
  - Programming Language
tags:
  - Next.js
toc: true
toc_sticky: true
# last_modified_at : 2022-04-12T00:30:00
---

## 코드 스플리팅이란?

웹 애플리케이션의 규모가 커지고 구조가 복잡해지면서, 자연스럽게 애플리케이션을 구성하는 코드의 양도 커지게 되었다. 이에 따라 사용자가 웹 페이지에 진입했을 때 다운로드받아야 하는 파일의 크기가 커지게 되면서 웹 페이지의 초기 로딩 시간이 길어지는 문제가 발생하게 되었다. 이는 웹 페이지가 주는 사용자 경험에 매우 큰 영향을 끼치는 문제로, 구글이 웹 페이지의 성능을 측정하는 기준이 되는 Core Web Vitals에도 중요한 평가 항목 중 하나로 포함되어 있다.

그러나 웹 페이지를 구성하는 모든 코드가 사용자에게 첫 화면의 콘텐츠를 보여주기 위해 필요한 것은 아닐 때가 있다. 예를 들어 사용자가 특정 버튼을 눌렀을 때 표시되는 팝업 창을 구성하는 코드가 있을 수 있다. 만약 이 코드를 분리할 수 있다면 최초 로딩되는 코드의 크기를 줄여 로딩 시간을 줄이는 효과를 얻을 수 있을 것이다. 이처럼 특정 코드를 분리하여 실제 실행 시에 불러오도록 처리하는 최적화 기법을 코드 스플리팅 (Code Splitting) 이라고 한다.

## Next.js에서의 코드 스플리팅

Next.js에서는 `next/dynamic`이라는 자체 라이브러리로 코드 스플리팅을 지원한다. React에서 코드 스플리팅을 위해 지원하는 `React.lazy()`와 `Suspense`를 통합한 라이브러리로, 코드 스플리팅을 보다 더 쉽게 할 수 있게 해 준다.

`next/dynamic` 라이브러리에서 제공하는 `dynamic` 함수로 컴포넌트의 import문을 감싸면 해당 컴포넌트가 Dynamic Import 처리되면서 컴포넌트가 포함된 페이지의 최초 번들에 포함되지 않고, 컴포넌트가 실제로 렌더링되어야 할 때 코드 로딩을 시작한다.

```javascript
import dynamic from 'next/dynamic'

const DynamicModal = dynamic(() => import('../components/Modal/Modal'))

export default function Main() {
  return (
    <>
    <DynamicModal />
    ...
    </>
  )
}
```

### 로딩 UI

`dynamic` 함수의 `loading` 옵션을 사용하면 Dynamic Import 처리된 컴포넌트가 로딩되는 동안 대신 표시할 fallback UI를 정의할 수 있다.

```javascript
import dynamic from 'next/dynamic'

const DynamicModal = dynamic(() => import('../components/Modal/Modal'), {
  loading: () => <p>Loading...</p>
})
```

### SSR 비활성화

컴포넌트가 `window`, `document`와 같은 브라우저 API를 사용할 때, 이 컴포넌트가 서버 사이드에서도 렌더링되면 오류가 발생할 수 있다. `dynamic` 함수의 `ssr` 옵션을 false로 지정하면 SSR 과정에서 그 컴포넌트가 렌더링되지 않게 처리할 수 있다.

```javascript
import dynamic from 'next/dynamic'

const DynamicNoSSR = dynamic(() => import('../components/NoSSR'), {
  ssr: false
})
```

### 외부 라이브러리의 Lazy Loading

외부 라이브러리를 특정 상황에서만 필요로 할 때, 그 부분에서 해당 라이브러리를 import함으로써 필요할 때만 라이브러리를 로드하게 처리할 수 있다. 다음은 Next.js에서 제공하는 외부 라이브러리의 Dynamic Loading 예제인데, input의 onChange 핸들러에서 `fuse.js`라는 라이브러리를 import하는 것을 볼 수 있다.

```javascript
import { useState } from 'react'
 
const names = ['Tim', 'Joe', 'Bel', 'Lee']
 
export default function Page() {
  const [results, setResults] = useState()
 
  return (
    <div>
      <input
        type="text"
        placeholder="Search"
        onChange={async (e) => {
          const { value } = e.currentTarget
          // Dynamically load fuse.js
          const Fuse = (await import('fuse.js')).default
          const fuse = new Fuse(names)
 
          setResults(fuse.search(value))
        }}
      />
      <pre>Results: {JSON.stringify(results, null, 2)}</pre>
    </div>
  )
}
```

React로 대표되는 SPA 아키텍처의 웹 어플리케이션이 유행하면서, 최초 로드되는 번들의 크기를 줄이기 위한 코드 스플리팅과 같은 최적화 기법이 더욱 중요해졌다. 그러나 과도한 코드 스플리팅의 사용은 역으로 사용자 경험을 해칠 수 있다. 하나의 코드를 여러 번 분리하는 것이다 보니 자연스레 웹 서버에 번들 파일을 요청하는 횟수가 늘어나고, 여러 개의 작은 파일 각각에 대한 응답 시간을 모두 더해 보면 정작 큰 번들 파일 하나를 받아오는 시간과 큰 차이가 없거나 오히려 시간이 더 늘어나는 역효과까지 발생할 수 있기 때문이다. 코드 스플리팅을 적용하기 전에 정말로 이 코드가 스플리팅이 필요한지 점검하고, 코드 사이즈가 크면서 최초 로딩에 사용하지 않는 경우에만 코드 스플리팅을 적절하게 사용하는 것이 최적화를 통해 사용자 경험을 유의미하게 향상하는 길이 될 것이다.