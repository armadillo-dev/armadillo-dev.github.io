---
title: "[React] The Component lifecycle"
date: 2019-03-15 16:00:00 -0400
categories: javascript
tags: react
---

React에서 사용되는 Component lifecycle method의 종류와 사용처를 정리하고자 한다.
비록 16.8 버전 이후 [Hook][link-hook] 기능의 추가로 인해 Class Component의 사용량은 떨어질 것이라 생각되지만 정리에 의의를 두자.

## Lifecycle Methods Diagram

각각의 lifecycle method들을 설명하기 전에 먼저 전체 다이어 그램을 통해 구조를 설명하고자 한다.
lifecycle은 크게 Mounting, Updating, Unmounting으로 구분되며 각 항목에는 여러개의 method가 존재한다.

![React lifecycle moethods diagram](/asserts/images/react-component-lifecycle-diagram.png)
*React lifecycle moethods diagram*

## constructor()

```js
constructor(props)
```

`constructor`는 다음 두 가지 목적을 위해 주로 사용된다.

1. 컴포넌트의 `state` 초기값을 정의한다( `this.state = {...}` 형태로 사용됨).
2. 컴포넌트에서 사용되는 이벤트 핸들러를 바인딩 한다(`this.someEventHandler = this.someEventHandler.bind(this)` 형태로 사용됨).

### constructor() 사용 예시

```js
constructor(props) {
  super(props);
  this.state = {
    color: 'red',
  };
  this.eventHandler = this.eventHandler.bind(this);
}
```

## componentDidMount()

```js
componentDidMount()
```

`componentDidMount()`는 컴포넌트가 최초로 마운팅 된 이후 1회 실행된다. 다음과 같은 목적을 위해 주로 사용된다.

1. 이벤트 리스너 등록 및 ajax 처리
2. `setTimeout`, `setInterval` 실행

`componentDidMount()` 내부에서 `this.setState()`를 실행 할 경우 추가적인 랜더링이 발생하지만, 브라우저가 화면을 갱신하기 이전에 수행된다. 때문에 `state`의 초기값 지정은 `constructor()` 내부에서 처리하는 것을 권장한다.

### componentDidMount() 사용 예시

```js
componentDidMount() {
  this.fetchSomeData();
}
```



## 참고 링크

- [React.Component - The Component Lifecycle][link-lifecycle]
- [React lifecycle moethods diagram][link-diagram]

[link-lifecycle]: https://reactjs.org/docs/react-component.html#the-component-lifecycle
[link-diagram]: http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/
[link-hook]: https://reactjs.org/docs/hooks-intro.html