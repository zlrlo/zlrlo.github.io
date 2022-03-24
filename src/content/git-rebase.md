---
layout: post
title: '[Git] Rebase(리베이스, 재배치)'
author: [Beanba]
tags: ['dev']
image: img/github.jpeg
date: '2021-06-19T19:16'
draft: false
---

웹 개발 회사에서 일을 막 시작했을 때, git rebase 기능을 잘못 사용해서 중복 커밋을 100개 이상 만들어 낸 적이 있다..😂
그때, 리베이스를 제대로 알고 써야겠다 생각해서 정리해둔 문서가 있었는데 블로그에도 포스팅 해보려고 한다.

## Rebase란?

리베이스는 커밋의 베이스를 똑 떼서 다른 곳으로 붙이는 것이다.

## Merge와 Rebase 차이점

코드 충돌이 발생했을 때, 내 브랜치로 먼저 머지하고, 충돌을 해결한 다음 다시 풀 리퀘스트를 보내면 충돌이 나지 않는다. 하지만 풀 리퀘스트에 충돌을 해결하느라 생긴 머지 커밋이 생긴다.
**리베이스는 깔끔하게 변경한 부분만 풀 리퀘스트를 보낼 수 있는 방법이다.**

## Rebase 원리

1. HEAD(feature)와 대상 브랜치(master)의 공통 조상을 찾는다.
2. 공통 조상 이후에 생성한 커밋들을 대상 브랜치(master) 뒤(마지막 커밋)로 재배치한다.
3. **리베이스 완료된 커밋들은 체크섬이 바뀐 새로운 커밋이다.**

## Rebase 방법

1. `git checkout feature`
2. `git rebase master`
3. `git checkout master`
4. `git merge feature` : 한 줄이 되었기 때문에 빨리 감기 병합을 수행한다.

## Rebase 주의사항

- 원격 저장소에 푸시한 브랜치는 rebase하지 않는 것이 원칙이다.
- 예를 들어 C1 커밋을 원격에 푸시하고 rebase를 하게 되면 원격에는 C1이 존재하고 로컬에는 다른 커밋인 C1`가 생성된다.
- 동일한 커밋의 사본이 여러개 존재하게 된다.

## Rebase로 특정 시점 Commit 지우는 방법

1. `git rebase -i HEAD~n` : <br />지우고 싶은 커밋이 보이는 곳 까지(n) 이동<br />
2. 지우고 싶은 커밋 키워드를 `drop`으로 수정

## Reference

[서적] 팀 개발을 위한 Git, GitHub 시작하기
