---
layout: post
title: '[Git] Stash'
author: [Mirabel]
tags: ['dev']
image: img/github.jpeg
date: '2021-06-19T20:07'
draft: false
---

> commit하기 껄끄러울 때 사용하는 stash

## Stash?

작업 파일들을 **스택**에 잠시 저장할 수 있도록 하는 것, stash 명령을 사용하면 워킹 디렉토리에서 수정한 파일들만 저장한다.

## Stash 원리

- `git stash` 를 실행하면 스택에 새로운 Stash가 만들어진다.

## Stash 사용 방법

`git stash` : <br />스택에 새로운 Stash가 만들어진다.<br /><br />
`git stash list` : <br />저장한 Stash를 확인한다.<br /><br />
`git stash apply <이름>` : <br />Stash 적용하기, 이름이 없으면 Git은 가장 최근의 Stash를 적용한다.<br />

    꼭 깨끗한 워킹 디렉토리나 Stash 할 때와 같은 브랜치에 적용해야 하는 것은 아니다.
    만약 충돌이 있으면 알려준다.

`git stash apply —index` : <br />Stash를 적용할 때 Staged 상태였던 파일을 Staged 상태까지 적용한다.<br /><br />
`git stash drop` : <br />Stash를 제거한다.<br /><br />
`git stash pop` : <br />Stash를 적용하고 나서 바로 스택에서 제거해준다.<br /><br />
`git stash -u` : <br />기본적으로 `git stash` 는 추적 중인 파일만 저장한다. 추적 중이지 않은 파일을 같이 저장하려면 `-u` 옵션을 붙여준다.<br /><br />

## Stash를 적용한 브랜치 만들기

- 수정한 파일에 Stash를 적용하면 충돌이 일어날 수 있고 그러면 또 충돌을 해결해야 한다.
- `git stash branch <브랜치>` 명령을 실행하면 Stash 할 당시의 커밋을 Checkout 한 후 새로운 브랜치를 만들고 여기에 적용한다. 이 모든 것이 성공하면 Stash를 삭제한다.

## Reference

[문서] [Stashing과 Cleaning](https://git-scm.com/book/ko/v2/Git-%EB%8F%84%EA%B5%AC-Stashing%EA%B3%BC-Cleaning)
