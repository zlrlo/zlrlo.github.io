---
layout: post
title: '[React] 무한 스크롤 만들기 (React with GraphQL Pagination)'
author: [Beanba]
tags: ['dev']
image: img/writing.jpg
date: '2021-09-27T16:33'
draft: false
---

_이 포스트에서는 프론트엔드 관련 코드만을 다룹니다._ <br/>
_**Relay’s Cursor Based Connection Pattern**으로 pagination을 구현합니다._

## 데이터 가져오기

### GraphQL query 작성

- 서버에서 pagination 데이터를 가져오기 위한 query를 작성한다.

```tsx
const graphql = gql`
  query PaginationUserBookmarksOnFeed($first: Int, $after: PaginationCursor) {
    paginationUserBookmarks(first: $first, after: $after) {
      pageInfo {
        hasNextPage
        endCursor
      }
      edges {
        cursor
        node {
          id
          interest {
            id
            interest
          }
          tags {
            id
            tag
          }
         ...
        }
      }
    }
  }
`;
```

### GraphQL query 실행

- 위에서 작성한 쿼리를 functional component안에서 Apollo-Client가 만들어준 `use[PaginationUserBookmarksOnFeed]Query()` hook으로 실행시킨다.

```tsx
const {
  data: feedData,
  loading: isFeedDataLoading,
  fetchMore,
} = usePaginationUserBookmarksOnFeedQuery({
  variables: {
    first: 4,
  },
});
```

- `first` 는 한번의 요청에 가져올 데이터 갯수를 의미한다.
- 참고) `notifyOnNetworkStatusChange` 속성을 true로 설정하고 networkStatus 값을 받으면 데이터를 가져오는 동안 로딩 처리를 할 수 있다. ex. spinner

### fetchMore[FeedData]() 코드 작성

- pageInfo에 hasNextPage 값이 true라면 pageInfo.endCursor 다음 데이터 4개를 가져오도록 함수를 작성한다.
- hasNextPage 값이 false일 경우 다음 데이터가 없다는 것을 의미하므로 fetchMore() 함수를 실행시키지 않는다.

```tsx
const fetchMoreFeedData = () => {
  if (pageInfo?.hasNextPage) {
    fetchMore({
      variables: { first: 4, after: pageInfo.endCursor },
    });
  }
};
```

### Apollo Client cache typePolicies 설정

- typePolicies 설정에 아래 코드와 같이 위에서 작성한 query 이름에 `relayStylePagination()` 함수를 넣어준다.

  ```tsx
  new ApolloClient({
    link: authLink.concat(httpLink),
    cache: new InMemoryCache({
      typePolicies: {
        Query: {
          fields: {
            paginationUserBookmarks: relayStylePagination(),
          },
        },
      },
    }),
  });
  ```

- typePolices 설정은 GraphQL 스키마에 대한 연속 호출 데이터가 기존 캐시와 병합되고 저장되는 방법을 정의하는 곳이다.

  ex)

  - **호출 1**: 데이터 0번에서 9번까지 가져오기

  - **호출 2**: 데이터 10번에서 19번까지 가져오기

  - 호출 1번을 통해 가져온 데이터와 호출 2번을 통해 가져온 데이터를 어떻게 병합해서 캐시에 저장할 것인지를 정의한다.

- `relayStylePagination()` 함수는 **Cursor-based pagination**에 맞게 간편하게 사용할 수 있도록 미리 정의된 라이브러리이다.

## Intersection Observer API 적용하기

> The Intersection Observer API provides a way to asynchronously observe changes in the intersection of a target element with an ancestor element or with a top-level document's viewport.

### Intersection Observer 생성하기

```tsx
const options: IntersectionObserverInit = {
  threshold: 0.5,
};

const intersectionObserver = new IntersectionObserver(handleIntersection, options);
```

- options에 root 설정을 하지 않으면 브라우저 뷰포트가 기본값이 된다.
- threshold 0.5 설정을 하면 뷰포트에 target이 50% 교차되면 callback(handleIntersection) 함수가 동작하게 된다.

### Target 설정하기

- `cursor === pageInfo?.endCursor` 인 경우(서버에서 가져온 마지막 데이터인 경우), ref 값을 target으로 설정해준다.

```tsx
export const HomeFeed = () => {
  const { entries, pageInfo, fetchMoreFeedData } = useDataAccessFeed();

  const target = useRef<HTMLDivElement>(null);

  return (
    <div className="...">
      {entries?.map(({ id, cursor, ...}, index) => {
        return (
          <div key={id} ref={cursor === pageInfo?.endCursor ? target : null} className="...">
            <ShadowCard ... />
          </div>
        );
      })}
    </div>
  );
};
```

### Callback(`handleIntersection`) 작성하기

- `IntersectionObserverEntry.isIntersecting` : true이면 뷰포트에 교차되는 변화가 일어나고 있다는 것을 의미하고, false이면 뷰포트에 교차되지 않는 변화가 일어나고 있다는 것을 의미한다.
- 따라서 `entry.isIntersecting` true일 경우 `fetchMore[FeedData]()` 함수를 실행시키고, 새로운 target을 observer에 등록한다.

```tsx
const handleIntersection = (
  [entry]: IntersectionObserverEntry[],
  observer: IntersectionObserver,
) => {
  if (!entry.isIntersecting) {
    return;
  }

  fetchMoreFeedData();

  observer.unobserve(entry.target);
  if (target.current) {
    observer.observe(target.current);
  }
};
```

### 최종 코드

```tsx
export const HomeFeed = () => {
  const { entries, pageInfo, fetchMoreFeedData } = useDataAccessFeed();

  const target = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const options: IntersectionObserverInit = {
      threshold: 0.5,
    };

    const handleIntersection = ([entry]: IntersectionObserverEntry[], observer: IntersectionObserver) => {
      if (!entry.isIntersecting) {
        return;
      }

      fetchMoreFeedData();

      observer.unobserve(entry.target);
      if (target.current) {
        observer.observe(target.current);
      }
    };

    const intersectionObserver = new IntersectionObserver(handleIntersection, options);

    if (target.current) {
      intersectionObserver.observe(target.current);
    }

    return () => intersectionObserver && intersectionObserver.disconnect();
  }, [fetchMoreFeedData]);

  return (
    <div className="...">
      {entries?.map(({ id, cursor, ... }, index) => {
        return (
          <div key={id} ref={cursor === pageInfo?.endCursor ? target : null} className="...">
            <ShadowCard ... />
          </div>
        );
      })}
    </div>
  );
};
```

## 주의할 점

- fetchMore 함수를 실행하면 다음 데이터가 불러와지는데, 이때 기존 cache 데이터에 제대로 병합이 되지 않는다면 이 [링크](https://stackoverflow.com/questions/66415243/merge-previous-data-while-scrolling-on-relay-pagination-graphql)를 읽어보는 것이 좋다.
- Apollo는 쿼리 인수에 after가 포함되어 있으면 before 이전 페이지를 병합한다. 따라서 first, after, before, last가 어떤 객체의 속성이 아닌 인수로 선언되어야 한다.
  ex) `paginationUserBookmarks(first: $first, after: $after)`
- 누군가 나 처럼 이 문제에 부딪혀 고통받지 않도록..😇

## 느낀 점

무한 스크롤 기능은 여러 페이지에서 사용될 것 같아서 시간날 때 hook으로 분리하는 작업을 해야겠다...

## Reference

- [Infinite Scroll in React with GraphQL Pagination](https://levelup.gitconnected.com/infinite-scroll-in-react-with-graphql-pagination-40bfc2294430)
- [Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)
