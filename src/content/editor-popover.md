---
layout: post
title: '[React] Popover 컴포넌트 만들기'
author: [Beanba]
tags: ['Web Frontend']
image: img/popover.jpg
date: '2021-06-08T23:46'
draft: false
---

최근 3개월 동안 회사에서 에디터(웹 빌더) 개발을 진행했다.<br />
에디터는 특정 Block에 마우스를 hover하면, 툴 박스 버튼 그룹이 나타나고, 툴 박스 버튼을 클릭하면 설정 창이 Popover 형식으로 나타나도록 디자인 되었다.

우선, Material-UI 사이트의 Popover 컴포넌트가 어떻게 구현되어 있는지 확인했다.

Material-UI 사이트의 Popover 컴포넌트는 다음과 같이 설명되어있다.

> Things to know when using the Popover component:<br />
> The component is built on top of the Modal component.<br />
> The scroll and click away are blocked unlike with the Popper component.

[참고 | Material-UI Popover](https://material-ui.com/components/popover/#popover)

Popper 컴포넌트가 나타나있는 동안 클릭과 스크롤이 막힌다는 내용이었다. Modal 컴포넌트를 기반으로 만들었기 때문인 것 같았다.<br />
에디터에서 사용할 Popover 컴포넌트는 설정 창(Popper)이 나타나있는 동안 스크롤과 클릭이 막히면 안되기 때문에 활용할 수 없었다.

에디터에서 필요한 Popover 기능을 정리하면 다음과 같다.

1. Block에 마우스를 hover 하면 수정 버튼이 나타난다.
2. 수정 버튼을 클릭하면 설정 창(Popper)이 나타난다.
3. 설정 창(Popper)이 나타나있는 동안 클릭과 스크롤이 가능해야 한다.
4. 설정 창(Popper) 밖의 영역을 클릭하면 설정 창(Popper)이 없어져야 한다.

## 간단한 Popover 컴포넌트 만들기

위 조건을 만족하는 Popover를 직접 만들어보기로 했다.

먼저, React 공식 문서에 [정리(clean-up)를 이용하는 Effects](https://ko.reactjs.org/docs/hooks-effect.html#effects-without-cleanup) 부분을 읽어보면 도움이 많이 된다.
useEffect에서 함수를 반환하면 해당 함수는 다음 렌더링 전에 실행된다는 것이 핵심이다.

> Popover 컴포넌트 데모

    - 마우스를 hover하면 opener가 나타나고 opener를 클릭하면 popper가 나타난다.
    - popper를 제외한 영역을 클릭하면 opener와 popper가 사라진다.

<iframe src="https://codesandbox.io/embed/popover-8zt81?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hiden;"
     title="popover"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

>

Popover 컴포넌트의 코드를 살펴보면
Popper 컴포넌트가 화면에 나타나면 useEffect안에 이벤트 리스너가 `pageClickEvent`를 콜백함수로 가지는 클릭 이벤트를 등록한다.
돔 최상단에 클릭 이벤트가 등록되었기 때문에 화면에 어디를 클릭해도 `pageClickEvent` 콜백함수가 실행된다.

또한, useEffect에서 해당 클릭 이벤트를 제거하는 함수를 리턴하고 있기 때문에 클릭 이벤트가 발생해서 화면이 다시 렌더링 되면 클릭 이벤트가 제거된다.

```javascript
export const Popover =
  React.memo <
  Props >
  (props => {
    const { onClose, children } = props;

    const settingsWindowRef = useRef < HTMLDivElement > null;

    useEffect(() => {
      const pageClickEvent = (e: any) => {
        if (!settingsWindowRef.current.contains(e.target)) {
          onClose();
        }
      };

      window.addEventListener('click', pageClickEvent, true);

      return () => {
        window.removeEventListener('click', pageClickEvent, true);
      };
    });

    return <Wrapper ref={settingsWindowRef}>{children}</Wrapper>;
  });
```

Popper(설정 창)을 useRef를 사용해서 pageClickEvent 함수에서 제외하는 이유는
설정 창 안 내용을 클릭했을 때는 Popper(설정 창)이 닫히면 안되기 때문이다.

에디터 프로젝트에서는 Popper(설정 창)이 사라지는 타이밍에 캐시 업데이트 하는 함수가 실행되어야 했었는데
Popover 컴포넌트에 해당 함수를 props(onClose)로 전달해서 `pageClick`함수 안에 넣어주었다.
