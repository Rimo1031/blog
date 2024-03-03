---
title: "Apollo Client에서의 Pagination 처리 방법"
excerpt: " "
categories: 
  - Programming Language
tags:
  - JavaScript, Apollo Client, Pagination
toc: true
toc_sticky: true
# last_modified_at : 2022-04-12T00:30:00
---

Apollo Client는 웹 프론트엔드에서 GraphQL API를 호출하는 데 가장 널리 사용되는 라이브러리이다. 현재 업무에서도 Apollo Client를 사용 중인데, 최근에 pagination을 적극적으로 사용해야 하는 기능을 개발하게 되어 이에 대해 공부한 내용을 정리해 보려고 한다.

## Pagination

pagination은 그 이름처럼 페이지화, 즉 많은 양의 데이터를 여러 개의 조각(page)으로 나눈다는 뜻을 가지고 있다. 대표적인 예시로 블로그 페이지처럼 많은 양의 데이터를 목록으로 보여주어야 하는 상황을 들 수 있다. 수십~수백 개의 블로그 게시물 목록을 최초 로딩 시에 모두 받아와서 화면에 보여준다면 로딩 시간도 오래 걸리고 첫 화면에 보여주는 정보량이 지나치게 많아져 사용자 경험에도 좋지 않을 것이다. 이러한 상황을 막기 위해서 최초 로딩 시에는 데이터의 일부만 보여주고 더보기 버튼을 누르거나 목록 스크롤을 끝까지 내렸을 때 그 다음 데이터를 일부 보여주는 식으로 pagination을 구현하게 된다.

## Apollo Client에서의 pagination

Apollo Client에서는 백엔드 API에서 제공하는 pagination 방식에 의존하지 않는 범용적인 구조의 pagination 기능을 제공한다. 여기서는 React Apollo Client를 사용하고, pagination은 `offset`과 `size` 변수를 사용해 구분한다고 하자.

React Apollo Client에서는 query를 요청할 때 `useQuery` hook을 사용한다. 이 `useQuery` hook의 return 값을 보통 `data`, `loading`, `error` 3가지 값으로 구조 분해 할당해 사용하는데, return값에서 추가로 `fetchMore`라는 이름의 함수를 받아올 수 있다.

`fetchMore` 함수는 아무런 인자 없이 사용하면 최초에 `useQuery` hook에서 사용했던 query문, variable을 그대로 사용해 데이터를 가져오지만, 인자에 새로운 variable을 제공하면 **같은 query를 variable만 바꾸어 재요청**한다. 재요청 결과로 받아온 데이터는 `useQuery` hook의 `data` 반환값에 다시 할당되어 사용할 수 있게 된다.

예를 들어, 영상 목록을 가져오는 `getVideos`라는 query가 있다고 하자. 이 query는 `category`, `from`, `size` 3개의 variable을 가지는데, DB 내에 있는 수많은 영상 목록 중에서 특정 카테고리의 `from`번째 영상으로부터 `size` 개수만큼의 영상 목록을 가져온다.

```javascript
const GET_VIDEOS = gql`
  query GetVideos($category: String, $from: Int, $size: Int) {
    getVideos(category: $category, from: $from, size: $size) {
      videos {
        title
        views
      }
      total
    }
  }
`
```

이 query를 사용해서 영상 데이터를 불러오는 React Component를 작성하면 다음과 같을 것이다. IT 카테고리에 해당하는 영상들을 0번째부터 20개 불러오도록 작성하였다.

```javascript
const VideoList = () => {
  // 영상 데이터 최초 로딩
  const { data, loading, error, fetchMore } = useQuery(GET_VIDEOS, {
    variables: {
      category: "IT",
      from: 0,
      size: 20
    }
  })
}
```

`fetchMore` 함수를 이용해 데이터를 추가로 불러오는 함수를 작성할 수 있다. 지금까지 불러온 영상 목록의 길이로부터 추가로 20개의 영상을 더 불러오도록 variable을 설정하였다.

```javascript
const handleFetchMore = async () => {
  await fetchMore({
    variables: {
      from: data?.getVideos?.videos?.length,
      size: 20
    }
  })
}
```

## 기존 데이터와 신규 데이터 병합

`fetchMore` 함수로 분할된 데이터를 받아올 수 있지만, 따로 설정을 하지 않으면 새로 받아온 데이터가 기존 데이터를 교체하게 된다. 이는 Apollo Client가 불러온 데이터를 cache하는 방식과 관련이 있다.

Apollo Client는 기본적으로 **variable이 하나로 다르면 모두 다른 데이터**로 인식해 캐시를 별도로 저장한다. 위의 예시를 통해 설명하자면 "IT 카테고리의 0번 ~ 20번 영상"과 "IT 카테고리의 20번 ~ 40번 영상"을 아무 관계 없는 별도의 데이터로 인식해 캐시도 별도로 저장된다는 의미이다. 그러나 pagination을 사용해 추가로 불러온 데이터의 경우 **기존 데이터의 연장선**과 같기 때문에 별도로 관리되지 않고 기존 데이터에 합쳐질 필요가 있다.

### Key Arguments

Apollo Client 3.0 버전부터는 이러한 상황에서 기존 데이터와 신규 데이터를 병합하기 위해 Key Arguments라는 속성을 제공한다. Key Arguments란 **특정 query 내에서 별도의 cache로 저장될 데이터를 구분하기 위해 사용되는 variable**을 의미한다. Apollo Client는 모든 variable이 단 하나라도 변경되면 별도의 cache로 저장하기 때문에, 별다른 설정이 없다면 기본적으로 모든 query의 variable이 Key Arguments라고 할 수 있다. 만약 이 Key Arguments를 수동으로 지정하게 된다면 Key Arguments에서 빠진 variable들은 **그 값이 달라지더라도 하나의 cache에 저장**된다.

Key Arguments는 Apollo Client 생성 시점에 `cache` option을 정의하면서 설정할 수 있다.

```javascript
const cache = new InMemoryCache({
  typePolicies: {
    Query: {
      fields: {
        getVideos: {
           keyArgs: ['category'],
        }
      }
    }
  }
})
```

pagination을 적용할 때 key arguments를 사용한다면 보통 pagination variables를 제외한 나머지를 key arguments로서 설정할 것이다. Key Arguments를 설정한 뒤에는 기존에 cache된 데이터와 신규로 fetch한 데이터를 어떻게 병합할 것인지 명시해야 한다. Apollo Client에서 제공하는 `merge` 함수를
통해 데이터를 어떻게 합칠 지 정의할 수 있다. 이 때 합쳐진 데이터의 구조는 기존 데이터, 신규 데이터의 구조와 일치해야 한다. `merge` 함수의 `existing`, `incoming` 인자를 통해 각각 기존 데이터와 신규 데이터를 함수 내에서 사용할 수 있고, `merge` 함수의 반환값이 합쳐진 데이터로서 cache에 저장된다.

```javascript
const cache = new InMemoryCache({
  typePolicies: {
    Query: {
      fields: {
        getVideos: {
           keyArgs: ['category'],
           merge(existing = [], incoming) {
            const merged = { 
              videos: [ ...existing?.videos, ...incoming?.videos ],
              total: incoming?.total
            }
            return merged
           }
        }
      }
    }
  }
})
```

기존에 존재하던 `existing` 데이터의 `videos` 배열 뒤에 새로 받아온 `incoming` 데이터의 `videos` 배열이 합쳐져 저장된다.

### `updateQuery`

`fetchMore` 함수의 option으로 `updateQuery`라는 callback을 사용할 수 있다. `updateQuery`는 `fetchMore` 함수가 실행 완료 된 후 실행되고, query의 결과를 **자신의 return값으로 업데이트**하는 기능을 수행한다. 이를 이용하면, 위에서 Key Arguments의 선언과 함께 설정했던 merge 과정을 이 callback 안에서 수행하게 함으로써 pagination된 데이터의 병합을 구현할 수 있다.

```javascript
const handleFetchMore = async () => {
  await fetchMore({
    variables: {
      from: data?.getVideos?.videos?.length,
      size: 20
    },
    updateQuery: (prev, { fetchMoreResult }) => {
      return {
          ...prev,
        getVideos: {
          ...prev?.getVideos,
          videos: [ ...prev?.getVideos?.videos, ...fetchMoreResult?.getVideos?.videos ]
        }
      }
    }
  })
}
```

### Key Arguments와 `updateQuery` 중 어느 쪽을 사용해야 할까?

Key Arguments는 하나의 Apollo Client 내에서 정의되므로 한 client 내에서라면 어느 위치에서 해당 query를 사용하든지 같은 정책이 적용된다. 하지만 `updateQuery`는 특정 query의 호출 시점에 `fetchMore` 함수의 option으로서 정의되므로 한 client 내의 동일한 query라도 해당 option을 적용했는지의 여부에 따라 merge 정책이 정의될수도, 그렇지 않을 수도 있다. 이를 Apollo 재단에서는 일종의 불확실성으로 인지했는지 `updateQuery` option을 deprecate하려는 시도도 있었는데, [많은 개발자들의 항의를 받고 (...) 시도를 철회]('https://github.com/apollographql/apollo-client/issues/8915')한 것으로 보인다.

현재 Apollo Client 공식 Docs에서도 Key Arguments를 이용한 cache merge 방법만을 언급하고 있고 바로 앞에서 언급한 Github Issue에서도 Apollo 개발자가 Key Arguments를 사용하는 것을 권장한다고 언급한 것으로 볼 때,

> Key Arguments를 사용해 일관된 캐시 정책을 유지하는 것이 일반적으로 권장된다. 하지만 하나의 query에 대해 상황에 따라 다른 merge 방법을 사용할 필요가 있다면 `updateQuery`를 사용해서 캐시를 합치는 방법도 사용할 수 있다.

고 정리할 수 있을 것이다.

## Reference

- [Pagination - Apollo Client Docs (v2)](https://www.apollographql.com/docs/react/v2/data/pagination/)
- [Core Pagination API - Apollo Client Docs (v3)](https://www.apollographql.com/docs/react/pagination/core-api/)