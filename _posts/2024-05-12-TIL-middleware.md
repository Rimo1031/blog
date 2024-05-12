---
title: "Next.js에서의 미들웨어를 사용한 클라이언트 인증 로직 구현"
excerpt: " "
categories:
  - Programming Language
tags:
  - [JavaScript, Next.js]
toc: true
toc_sticky: true
# last_modified_at : 2022-04-12T00:30:00
---

## 미들웨어란?

넓은 의미에서의 미들웨어는 두 개의 소프트웨어 사이에 위치하면서 양쪽이 서로 데이터를 잘 주고받을 수 있도록 매개 역할을 하는 소프트웨어를 말한다. 데이터베이스와 클라이언트 사이에 위치하면서 클라이언트가 데이터베이스 내부의 데이터를 읽어올 수 있도록 API를 제공하는 백엔드 서버도 미들웨어의 일종이라고 할 수 있을 것이다. 미들웨어는 그 정의가 굉장히 느슨한 만큼 소프트웨어 개발의 여러 분야에서 널리 쓰이는 말이지만, 웹 개발에서는 주로 클라이언트와 서버 사이에서 동작하며 둘 사이의 HTTP 요청 및 응답에 개입하는 소프트웨어라는 의미로 쓰이고 있다.

## Next.js에서의 미들웨어

Next.js에서의 미들웨어도 앞서 언급한 웹 개발에서의 미들웨어의 정의와 부합하는 역할을 한다. Next.js 공식 Docs는 미들웨어의 역할에 대해 이렇게 설명한다.

> 미들웨어는 서버로 들어오는 요청이 완료되기 전에 실행되며, 들어온 요청을 rewrite, redirect하거나 요청 및 응답 헤더를 수정하는 등 응답을 조작할 수 있습니다.

미들웨어는 Next.js 서버로 들어오는 **모든 요청**에 대해 실행된다. Next.js 서버로 요청이 들어올 때 실행되는 로직의 순서는 다음과 같다.

1. `next.config.js`의 `headers` option
2. `next.config.js`의 `redirects` option
3. **미들웨어**
4. `next.config.js`의 `beforeFiles`
5. File-System Routes (`public/`, `pages/`, ...)
6. `next.config.js`의 `afterFiles`
7. Dynamic Routes (/blog/[path])
8. `next.config.js`의 `fallback`

미들웨어보다 앞서 실행되는 로직은 config에 위치한 headers, redirects 옵션뿐인 것을 생각해 보면 미들웨어는 거의 모든 요청에 대해 Next.js 서버 내에서 최우선적으로 실행되는 로직이라고 할 수 있을 것이다.

앞서 언급한 것처럼 Next.js의 미들웨어에서는 다음과 같은 작업을 수행할 수 있다.

### `redirect`

들어온 요청에 대한 경로를 다른 URL로 이동시킨다.

### `rewrites`

요청으로 들어온 URL은 유지한 채로 응답만 다른 URL의 내용으로 바꿔치기한다. 사용자가 A URL로 접근해도 실제로 받아보는 응답은 B URL의 내용이 되는 것이다.

### header 설정

Next.js 13부터 지원하는 기능으로, 미들웨어 단에서 요청 / 응답의 헤더를 조작할 수 있다.

### 새로운 응답 생성

기존의 라우팅 경로를 따르지 않고 미들웨어에서 별도의 응답을 생성해서 보여줄 수 있다.

## 미들웨어를 이용해 클라이언트 인증 구현하기

사내 어드민 페이지를 개발하는 중 미들웨어 단에서 클라이언트 인증을 구현해야 하는 상황에 맞닥뜨렸다. 어드민 페이지도 Next.js 기반인 만큼 대부분의 페이지가 JSX 페이지로 구성되어 있고, 각종 인증 모듈을 활용해 클라이언트 사이드에서 사용자 검증을 수행할 수 있었다. 그러나 어드민 페이지 내부의 한 기능이 Next.js Page router와 분리된, **완전히 정적인 HTML 파일**을 보여줘야 할 필요가 있었다. 단순히 보여주기만 하는 것이라면 `public` 폴더 안에 넣어서 라우팅되게 할 수 있지만 문제는 이 페이지 역시 어드민 페이지의 일부이기 때문에 사내 관리자만 접근할 수 있어야 한다는 것이었다. 완전히 정적인 HTML 파일인 만큼 추가로 JS를 사용해서 인증 로직을 끼워넣기도 어렵고, React의 `dangerouslySetInnerHtml` 옵션을 사용하려 해도 HTML 내부의 CSS, JS 파일 연결 부분이 제대로 작동하지 않아 사용할 수 없었다. 이 문제를 해결할 방법을 고민하다가 미들웨어 단에 인증 로직을 구현해놓고, 해당 HTML 파일로 접근하는 요청은 인증 로직을 통과하도록 구현하기로 했다.

### fake URL의 요청을 target HTML로 rewrite

인증 로직으로 보호한다고는 하지만, `public` 폴더 내부에 위치한 HTML 파일의 경로를 그대로 노출하는 것은 위험하다고 생각했다. 그래서 fake URL을 하나 만들고, 미들웨어에서 그 URL로 들어오는 요청에 대한 응답을 우리가 보여주려는 HTML 파일로 rewrite하는 방식으로 실제 경로를 숨기기로 했다.

```javascript
  export default function middleware(req, res) {
    if (req.url.includes('fake_url')) {
      const newUrl = req.url.replace('fake_url', 'real_html_path')
      return res.rewrite(new URL(newUrl, req.url))
    }
  }
```

### 실제 경로로 직접 들어오는 요청을 404 페이지로 redirect

rewrite 로직을 통해 실제 파일 경로를 숨기기는 했지만, 실제 파일의 경로로 직접 요청이 들어올 경우에는 일괄적으로 404 페이지로 redirect해서 응답을 받을 수 없게 처리했다.

```javascript
  export default function middleware(req, res) {
    ...
    if (req.url.includes('real_html_path')) {
      const newUrl = req.url.replace('real_html_path', '404')
      return res.redirect(new URL(newUrl, req.url))
    }
  }
```

### fake URL로 들어오는 요청에 대해 인증 로직 추가

어드민 사이트의 사용자 여부 판별은 jwt 토큰을 통해 진행할 수 있다. 처음에는 사용자 인증 로직 자체를 미들웨어에 위치시키려고 했으나, 인증 로직을 모두 구현하고 실행하려고 하자 특정 모듈을 실행할 수 없다는 오류 메시지와 함께 실행이 되지 않았다. 그 이유는 미들웨어가 **엣지 런타임***에서 실행되기 때문이었다.

Next.js는 두 가지 종류의 서버 런타임을 지원한다. 우리가 일반적으로 사용하는 Node.js 런타임과, 앞서 언급한 엣지 런타임이 있다. 엣지 런타임은 Node.js 런타임의 경량화 버전이라고 생각할 수 있다. 성능을 위해 최소한으로 필요한 웹 표준 API만을 모아 만들어진 아주 작은 크기의 런타임인데, 이 엣지 런타임에 jwt 토큰 복호화에 필요한 API가 존재하지 않아서 계속 인증에 실패하는 것이었다. 현재 Next.js 미들웨어는 엣지 런타임에서의 실행만을 지원하고, 일반 Node.js 런타임은 지원하지 않기 때문에 인증 로직을 미들웨어에서 실행할 수 없었다. 그래서 차선책으로 인증 기능만을 수행하는 API Routes를 하나 만들고, fake URL로 요청이 들어왔을 경우 요청 header를 그대로 해당 API Endpoint로 보내서 그 응답 결과에 따라 분기 처리하도록 적용했다.

```javascript
  export default async function middleware(req, res) {
    ...
    if (req.url.includes('fake_url')) {
      const reqHeaders = new Header(req.headers)
      const verifyResult = await fetch('/api/auth', { headers: reqHeaders })
      if (verifyResult.statusCode === 200) {
        const newUrl = req.url.replace('fake_url', 'real_html_path')
        return res.rewrite(new URL(newUrl, req.url))
      } else {
        const newUrl = req.url.replace('real_html_path', '404')
        return res.redirect(new URL(newUrl, req.url))
      }
    }
  }
```