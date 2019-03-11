---
title: "[ES6] destructuring과 default parameter를 이용한 항샹된 함수 매개변수 정의 방법"
date: 2019-03-11 16:00:00 -0400
categories: javascript
tags: javascript es6 function
---

ES6에서 추가된 기능인 [destructuring][link-destructuring]과 [default prameter][link-default-parameter]를 이용해 단계적으로 함수의 매개변수를 향상시켜서 정의하는 방법을 알아보자.

## Before

우선 아래와 같은 함수가 존재한다고 가정해보자.

```js
function setElementBackgroundColor(selector, color, parent) {
  const parentDOM = parent || document;
  const backgroundColor = color || 'red';
  const el = parentDOM.querySelector(selector);
  
  if (!el) throw 'Cannot find Element';
  
  el.style.backgroundColor = backgroundColor;
}
```

위 함수는 다음과 같이 사용 할 수 있다.

```js
setElementBackgroundColor('#el'); // id가 el인 엘리먼트를 document에서 찾은 뒤 배경색을 red로 변경

setElementBackgroundColor('#el', 'blue'); // id가 el인 엘리먼트를 document에서 찾은 뒤 배경색을 blue로 변경

const container = document.getElementById('container');
setElementBackgroundColor('#el', 'blue', container); // id가 el인 엘리먼트를 container에서 찾은 뒤 배경색을 blue로 변경

setElementBackgroundColor('#el', null, container); // id가 el인 엘리먼트를 container에서 찾은 뒤 배경색을 red로 변경
```

## Before 함수의 문제점

현재 정의된 ``setElementBackgroundColor`` 함수에는 몇가지 문제점이 존재한다.

### 1. 함수의 사용자는 ``color``와 ``parent`` 매개변수의 목적을 파악하기 어렵다

``setElementBackgroundColor('#el', 'blue');``와 같은 형태로 함수를 사용할 경우 사용자는 ``blue``가 의미하는 것이 무엇인지 해당 함수의 정의 부분을 찾아보아야 이 매개변수의 역할이 무엇인지 파악할 수 있다. 물론 예제의 함수는 간단한 역할을 수행하므로 크게 혼동을 주지는 않겠지만, 복잡한 함수의 경우 의미를 파악하기 힘들다.

### 2. ``color``의 값은 기본값을 사용하고, ``parent``값만 지정하고 싶을 경우 ``null``로 두번째 인자를 넘겨줘야 한다

``setElementBackgroundColor('#el', null, container);``와 같이 사용할 경우, 두번째 인자를 반드시 `null`로 지정해 주어야한다. optional 매개변수가 많아질수록 이러한 단점은 더욱 부각된다. ``func(null, null, null, null, 1)``과 같은 형태가 된다면, 함수 사용자의 혼란을 야기시킬 수 있다.

### 3. `color`와 `parent` 매개변수를 입력하지 않았을 경우 기본값을 별도로 지정해주어야 한다

현재 `color`와 `parent` 값은 기본값 지정을 위해 각각 `parentDOM`, `backgroundColor`라는 변수를 새로 정의하고 있다.

## Destructuring을 이용한 개선

> 구조 분해 할당(destructuring assignment) 구문은 배열이나 객체의 속성을 해체하여 그 값을 개별 변수에 담을 수 있게 하는 JavaScript 표현식입니다.

자세한 설명은 [링크 참조][link-destructuring]!

함수의 optional 매개변수를 객체 형태로 전달 받으면 사용자에게 해당 인자가 어떤 목적을 가지고 있는지 전달 할 수 있다.
예제 함수의 경우 다음과 같이 변경 할 수 있다.

```js
function setElementBackgroundColor(selector, options) {
  const parentDOM = options.parent || document;
  const backgroundColor = options.color || 'red';
  const el = parentDOM.querySelector(selector);
  
  if (!el) throw 'Cannot find Element';
  
  el.style.backgroundColor = backgroundColor;
}

const container = document.getElementById('container');
setElementBackgroundColor('#el', { color: 'blue', parent: container });
```

개선된 함수에서는 `color`와 `parent`라는 값을 사용처에서 입력 받기 때문에 인자가 의미하는 바를 한 눈에 인지할 수 있게 된다.
하지만 `options`에 어떤 항목이 포함되어 있는지 전체 코드를 확인하기 전에는 알 수 없다는 단점이 있다. 이 때 Destructuring을 이용하면 보다 명확한 전달이 가능하다.

```js
function setElementBackgroundColor(selector, { color, parent }) {
  const parentDOM = parent || document;
  const backgroundColor = color || 'red';
  const el = parentDOM.querySelector(selector);
  
  if (!el) throw 'Cannot find Element';
  
  el.style.backgroundColor = backgroundColor;
}
```

이제 `setElementBackgroundColor` 함수에는 `options`에는 `color`, `parent` 두 개의 항목이 존재한다는 것을 명확히 알 수 있게 되었다.
또한, `parent`의 값만 지정하고 싶을 경우 `{ parent: someDomNode }`와 같이 color를 입력하지 않고 사용이 가능하다.

## Default parameter를 이용한 개선

> 기본 함수 매개변수(default function parameter)를 사용하면 값이 없거나 undefined가 전달될 경우 매개변수를 기본값으로 초기화할 수 있습니다.

자세한 설명은 [링크 참조][link-default-parameter]

destructuring을 이용해 함수 구조가 개선되었지만, 아직 부족한 점이 있다. 기본값을 지정하기 위해 별도의 변수를 사용해야 한다는 점이다.

```js
const parentDOM = parent || document;
const backgroundColor = color || 'red';
```

이는 default parameter를 이욯해 해결 할 수 있다.

```js
function setElementBackgroundColor(selector, { color = 'red', parent = document } = {}) {
  const el = parent.querySelector(selector);
  if (!el) throw 'Cannot find Element';
  
  el.style.backgroundColor = color;
}
```

위의 코드에서 `{ color = 'red', parent = document } = {}` 형식으로 기본값을 지정해 준것을 확인 할 수 있다.
기본값을 지정해 두면 함수 내부에서는 별도 로직을 없앨 수 있는 장점이 있다.

또한, `{...} = {}` 형태의 코드가 있는데, 이는 `setElementBackgroundColor('#el')`과 같이 두번째 인자를 아예 생략하기 위함이다.

[link-destructuring]: https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment
[link-default-parameter]: https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/Default_parameters