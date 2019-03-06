---
title: "[javascript] Event bubbling & capturing"
date: 2018-10-18 11:22:00 -0400
categories: javascript
tags: javascript event
---

javascript event handling에 있어서 필수적인 event bubbling & capturing에 대해 알아보자.

## 1. event bubbling

javascript의 event는 상위 dom element로 전달된다. 예를 들어 다음 코드의 경우 랜더링 된 버튼을 클릭하면 btn -> div -> main 순으로 alert 가 출력된다.

### event-bubbling.js

```js
const handleClick = (e) => {
  alert(e.currentTarget.id);
};

document.getElementById('btn').addEventListener('click', handleClick); 
document.getElementById('div').addEventListener('click', handleClick); 
document.getElementById('main').addEventListener('click', handleClick); 
```

### event-bubbling.html

```html
<main id="main">
  <div id="div">
    <button id="btn">click</button>
  </div>
</main>
```

이러한 특징을 이용해 다수의 element에 대한 event handling을 할 수 있다. 이러한 방법을 이벤트 위임(event delegation)이라고 한다. 예를들어 다수의 파일 목록이 있고 특정 파일을 선택하면 해당 파일의 이름을 출력하고 싶다고 가정해보자.

이벤트 위임을 이용하지 않을 경우 각 요소에 click event를 바인딩 해주어야 한다.
아래 예제의 경우 총 5개의 event listener가 등록되며, 아이템이 추가될 경우 해당 아이템에 대한 이벤트를 다시 추가해주어야 한다.

### add-event-listener-foreach.js

```js
const handleClick = (e) => {
  console.log(e.target);
  alert(e.target.innerHTML);
};

document.querySelectorAll('a').forEach((item) => {
  item.addEventListener('click', handleClick);
});
```

### add-event-listener-foreach.html

```html
<ul id="contianer">
  <li><a href="#1">file name #1</a></li>
  <li><a href="#2">file name #2</a></li>
  <li><a href="#3">file name #3</a></li>
  <li><a href="#4">file name #4</a></li>
  <li><a href="#5">file name #5</a></li>
</ul>
```

위의 예제를 이벤트 위임을 이용해 처리하면 다음과 같이 변경 할 수 있다.

### event-delegation.js

```js
const handleClick = (e) => {
  console.log(e.target);
  alert(e.target.innerHTML);
};

document.getElementById('container').addEventListener('click', handleClick);
```

### event-delegation.html

```html
<ul id="container">
  <li><a href="#1">file name #1</a></li>
  <li><a href="#2">file name #2</a></li>
  <li><a href="#3">file name #3</a></li>
  <li><a href="#4">file name #4</a></li>
  <li><a href="#5">file name #5</a></li>
</ul>
```

**이러한 이벤트 위임의 장점은 다음과 같다.**

1. 추가/수정되는 요소에 자동으로 이벤트를 바인딩 할 수 있다.
2. event handling이 수월해진다.

## 2. event capturing

event capturing이란 evnet의 전파 순서를 뒤집는 것을 말한다. 보통의 event는 상위 element로 전달되지만, capturing이 적용된 이벤트는 하위 element로 전달된다.


### event-capturing.js

```js
const handleClick = (e) => {
  alert(e.currentTarget.id);
};

document.getElementById('btn').addEventListener('click', handleClick, true); 
document.getElementById('div').addEventListener('click', handleClick, true); 
document.getElementById('main').addEventListener('click', handleClick, true); 
```

### event-capturing.html

```html
<main id="main">
  <div id="div">
    <button id="btn">click</button>
  </div>
</main>
```

위에서 사용했던 예제에 capturing을 적용하면 main -> div -> btn 순으로 alert가 출력되는 것을 확인 할 수 있다.

## 참고링크

- [EventTarget.addEventListener() - MDN][link-add-event-listner]
- [왜 이벤트 위임(delegation)을 해야 하는가?][link-why-event-delegation]

[link-add-event-listner]: https://developer.mozilla.org/ko/docs/Web/API/EventTarget/addEventListener
[link-why-event-delegation]: https://github.com/nhnent/fe.javascript/wiki/August-22-August-26,-2016