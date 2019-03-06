---
title: "[lodash] throttle vs debounce"
date: 2018-10-25 10:54:00 -0400
categories: javascript
tags: javascript lodash
---

## 1.throttle

> ```js
> _.throttle(func, [wait=0], [options={}])
> ```

``_.throttle``은 특정 함수(func)를 일정 주기(wait) 내에 한 번만 호출되도록 throttled된 함수를 만들어주는 lodash의 함수이다. 이 함수는 주로 짧은 시간 내에 무수히 많이 수행 될 수 있는 이벤트(scroll, mousemove, mouseover)로 인한 성능 저하를 막기 위해 사용된다.

아래 예제를 실행시킨 뒤, 마우스를 움직여보면 event handler가 실행 됨에 따라 count 값이 매우 빠르게 증가하는 것을 확인 할 수 있다.

### without-throttle.js

```js
const container = document.querySelector('.container');
let cnt = 0;
container.addEventListener('mousemove', (e) => {
  cnt++;
  container.innerText = `COUNT: ${cnt}`;
});
```

### without-throttle.html

```html
<style>
  .container {
    height: 250px;
    padding: 20px;
    font-size: 1.2rem;
    background-color: #eee;
  }
</style>
<div class="container"></div>
```

이렇듯 무리한 핸들러의 호출은 전체 사이트의 과부하를 불러 일으키게 된다. 따라서 ``_.throttle``을 이용해 특정 주기 내에 1회만 실행되도록 코드를 수정 할 수 있다.

다음 예제는 1초에 한 번만 함수가 실행되도록 코드를 고친 예제이다.
두 예제의 차이를 느낄 수 있다.

### with-throttle.js

```js
const container = document.querySelector('.container');
let cnt = 0;
container.addEventListener('mousemove', _.throttle((e) => {
  cnt++;
  container.innerText = `COUNT: ${cnt}`;
}, 1000));
```

## 2. deboune

> ```js
> _.debounce(func, [wait=0], [options={}])﻿
> ```

``_.debounce``는 특정 함수(func)의 호출을 grouping한 뒤, 일정 주기(wait)가 지날 때까지 해당 함수의 호출이 없을 때 1회만 호출하도록 하는 함수를 만들어주는 lodash의 함수이다. ``_.throttle``과 마찬가지로 이벤트의 중복 호출을 방지하기 위해 주로 사용 되지만, 그 사용처에는 차이가 있다.

``_.debounce``는 검색 폼에서 유용하게 사용된다. 입력폼에 값을 입력할 때마다 ajax 통신을 통해 검색 결과를 화면에 갱신해준다고 가정해보자.

``_.debounce``를 사용하지 않을 경우 '검색'이라고 검색을 한다면 'ㄱ', '거', '검', '검ㅅ', '검새', '검색' 순으로 총 6번의 검색이 일어난다. 실 사용자가 원하는 결과는 '검색'에 대한 검색 결과일 것이기 때문에 불필요한 이벤트 호출을 막을 수 있다.

다음은 ``_.debounce`` 적용 전후를 비교한 예제이다.

### debounce.js

```js
const input = document.getElementById('input');
const input2 = document.getElementById('input2');
const normal = document.getElementById('normal');
const debounce = document.getElementById('debounce');

const handlerWithoutDebounce = (e) => {
  const li = document.createElement('li');
  li.innerHTML = e.target.value;
  normal.appendChild(li);
};

const handlerWithDebounce = (e) => {
  const li = document.createElement('li');
  li.innerHTML = e.target.value;
  debounce.appendChild(li);
}

input.addEventListener('keyup', handlerWithoutDebounce);
input2.addEventListener('keyup', _.debounce(handlerWithDebounce, 300));
```

### debounce.html

```html
<style>
  .container {
  display: flex;
}

.content {
  width: 50%;
}
</style>
<div class="container">
  <div class="content">
    <h2>without debounce</h2>
    <input id="input" />
    <ul id="normal"></ul>
  </div>
  <div class="content">
    <h2>with debounce</h2>
    <input id="input2" />
    <ul id="debounce"></ul>
  </div>
</div>
```

양쪽의 검색창을 비교하면 ```_.debounce```의 효과를 체감 할 수 있을 것이다.

### 참고링크

- [lodash - throttle][link-throttle]
- [lodash - debounce][link-debounce]

[link-throttle]: https://lodash.com/docs#throttle
[link-debounce]: https://lodash.com/docs#debounce