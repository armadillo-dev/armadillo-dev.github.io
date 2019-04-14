---
title: "[ES6] 배열(array) 관련 메서드 소개 및 활용"
date: 2019-04-13 00:00
categories: es6 javascript
tags: es6 array
description: "ES6에 들어서면서 배열과 관련된 많은 메서드를 활용 할 수 있게 되었다. 관련 메서드들을 소개하고 각각의 활용 사례를 살펴보자."
---

배열은 개발에서 빠질 수 없는 기초적인 자료구조이다. 때문에 프로그래밍 언어는 배열과 관련된 다양한 메서드들을 제공한다. 자바스크립트도 마찬가지로 활용도 높은 메서드들을 제공하고 있다.

본 게시물에서는 자바스크립트에서 제공하는 배열 관련 메서드들을 소개하고 각각의 메서드를 어떤 용도로 사용할 수 있는지 살펴보도록 한다.

## `forEach`

```js
arr.forEach(currentValue[, index[, array]])[, thisArg]);
```

`forEach`는 배열의 원소를 순회하면서 `callback` 함수를 실행한다. `callback` 함수는 `currentValue`, `index`, `array` 세 개의 인자를 사용 할 수 있다. `currentValue`에는 각 원소의 값이 들어가므로 `for`문과 유사한 형태로 사용 할 수 있다.

주로 반복문을 대체하는 역할로 사용된다.

### `forEach` vs `for`

```js
const arr = [1, 2, 3]

for (let i = 0; i < arr.length; i += 1) {
  console.log(arr[i])
}

arr.forEach(num => console.log(num))
```

위 예제는 `for`문과 `forEach`를 이용해 똑같은 로직을 수행한다. 하지만, `forEach`를 사용하는 쪽이 훨씬 간결하다. 때문에 간단한 반복문이라면 `forEach`를 사용하는 것을 추천한다.

### `forEach` 사용 예제

```js
// 1
const arr = [1, 2, 3]
arr.forEach(num => console.log(num))

// 2
const itemIds = [1, 2, 3]
const removeItem = id => someAPI.removeItem(id)
itemIds.forEach(removeItem)
```

1번 예제는 각 원소의 값을 로그에 출력한다.

2번 예제의 경우 원소를 순회하며 특정 함수를 호출한다. 사실아래 설명할 `map`으로 `Promise` 배열을 받은 뒤 `Promise.all`로 삭제하는게 더 낫지만, 이 게시물의 취지는 forEach에 대한 설명이므로 더 `Promise`에 대한 자세한 내용은 [이 게시물](https://armadillo-dev.github.io/javascript/javascript-tips-of-promise/)을 참고하자.

## `map`

```js
arr.map(callback(currentValue[, index[, array]])[, thisArg])
```

`map`은 배열의 원소를 순회하며 `callback`함수를 실행한다. 그리고 `callback`함수의 반환값으로 구성된 새로운 배열을 반환한다.

주로 배열의 구성 원소를 이용해 새로운 배열을 생성 할 때 사용된다.

## `map` 사용 예제

```js
// 1
const arr = [1, 2, 3]
const plusFiveArr = arr.map(num => num + 5) // [6, 7, 8]

// 2
const itemIds = [1, 2, 3]
const removeItem = (id) => someAPI.removeItem(id)
Promise.all(itemIds.map(removeItem))
```

1번 예제는 각 배열 원소에 5를 더해 새로운 배열을 생성한다.

2번 예제는 배열 원소를 이용해 다수의 API 호출이 필요한 경우 주로 사용되는 방식이다. `map`을 이용해 전체 API 호출을 하나의 배열에 담는다. 그 이후 `Promise.all`을 이용해 API 호출을 병렬 처리 할 수 있다.

## `reduce`

```js
arr.reduce(callback(accumulator, currentValue[, index[, array]])[, initialValue])
```

`reduce`는 각 배열의 원소를 순회하며 `callback` 함수를 실행시킨다. `callback` 함수는 `accumulator`, `currentValue`, `index`, `array`를 인자로 받는다.

여기서 중요한 포인트는 `accumulator`이다. `accumulator`는 현재까지 누적된 결과값을 뜻한다. `callback` 함수에서 리턴한 값은 다음 `callback` 함수의 `accumulator` 값이 된다. 이를 통해 누적된 값을 계산 할 수 있다. `initialValue` 값을 입력하면 첫번째 `callback`함수 실행에서 `accumulator` 값이 된다.

주로 배열을 하나의 값으로 만들 때 사용된다. 이를 다른식으로 표현하면 벡터(Vector)를 스칼라(Scalar)로 만든다고 할 수 있다.

### `reduce`의 동작 원리

`reduce`를 처음 접하면 그 동작원리가 정확히 이해되지 않을 수 있다.
다음 예제를 통해 동작 원리를 알아보자.

```js
const arr = [1, 2, 3]
const callback = (accumulator, currentValue, index, array) => accumulator + currentValue
const sum = arr.reduce(callback, 0) // 6
```

위의 예제에서 `callback` 함수는 총 3번 호출된다. 각 호출에 따른 인자와 반환값은 다음과 같다.

 callback 호출 횟수 | accumulator | currentValue | index | array       | 반환 값
--------------------|-------------|--------------|-------|-------------|---------
 1번째               | 0           | 1            | 0     | `[1, 2, 3]` | 1       
 2번째               | 1           | 2            | 1     | `[1, 2, 3]` | 3       
 3번째               | 3           | 3            | 2     | `[1, 2, 3]` | 6       

위의 표를 보면 이전 `callback` 함수의 반환 값이 다음 `callback` 함수의 `accumulator` 값으로 사용되는 것을 알 수 있다. 마지막 `callback` 호출 후 반환되는 6을 `sum` 변수에 대입하게 된다.

### `reduce` 사용 예제

```js
// 1
const arr = [1, 2, 3]
const sum = arr.reduce((acc, curr) => acc + curr, 0) // 6

// 2
const accounts = [
  { id: 'armadillo1', age: 27, gender: 'male' },
  { id: 'armadillo2', age: 33, gender: 'male' },
  { id: 'armadillo3', age: 21, gender: 'female' },
  { id: 'armadillo4', age: 40, gender: 'female' },
  { id: 'armadillo5', age: 13, gender: 'female' },
]
const groupByGender = accounts.reduce((acc, curr) => {
  const key = curr.gender
  if (!acc.hasOwnProperty(key)) {
    acc[key] = []
  }
  acc[key].push(curr)
  return acc
}, {})

console.log(groupByGender) // { male: [...], female: [...] }
```

1번 예제는 전체 배열 원소의 합계 값을 구한다.

2번 예제는 계정(`account`)을 순회하면서 성별(`gender`)로 그루핑 하는 작업을 수행한다.

## `find`

```js
arr.find(callback[, thisArg])
```

`find`는 `callback` 함수가 `truthy` 값을 반환 할 때까지 배열의 원소를 순회하며 실행시킨다. `callback` 함수가 `truthy`를 반환 할 경우 해당 `callback`에 해당하는 원소 값을 리턴 시킨다. 만약 모든 `callback`이 `falsy` 값을 반환하면 `undefined`를 반환한다.

주로 특정 조건을 만족하는 원소를 찾을 때 사용된다.

### `find` 사용 예제

```js
// 1
const arr = [10, 20, 30]
const result = arr.find(num => num > 10) // 20

// 2
const accounts = [
  { id: 'armadillo1', age: 27, gender: 'male' },
  { id: 'armadillo2', age: 33, gender: 'male' },
  { id: 'armadillo3', age: 21, gender: 'female' },
  { id: 'armadillo4', age: 40, gender: 'female' },
  { id: 'armadillo5', age: 13, gender: 'female' },
]

const armadillo3 = accounts.find(account => account.id === 'armadillo3') // { id: 'armaidllo3' ... }
```

1번 예제는 10보다 큰 숫자를 찾는다. `20`과 `30` 모두 10보다 큰 숫자이지만, `20`가 먼저 `callback` 함수로 실핼되기 때문에 최종 결과는 20이 된다.

2번 예제는 여러 개정을 가진 배열에서 특정 계정을 찾는 작업을 수행한다.`

## `some`

```js
arr.some(callback[, thisArg])
```

`some` 함수는 `callback` 함수에서 조건을 충족하는 원소가 하나라도 있으면 `true`를, 모든 원소가 조건을 충족시키지 못한다면 `false`를 반환한다. 한 개의 원소라도 조건을 충족하면 즉시 `true`를 반환하고 순회를 종료하기 때문에 [`every`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/every){:target="_blank"}보다 퍼포먼스가 좋다.

주로 배열에서 특정 조건을 만족하는지 여부를 판별 할 때 사용된다.

### `some` 사용 예제

```js
// 1
const arr = [1, 2, 3]
const hasGreaterThanTwo = arr.some(num => num > 2) // true


// 2
const arr2 = [10, 20, 30, 40]
const hasGreaterThanTen = arr2.some((num) => {
  console.log(num)
  return num > 10
}) // 10이 출력되고 true 반환
```

1번 예제는 2보다 큰 숫자를 가지고 있는지 판별한다. 원소 중 3이 있으므로 `true`를 반환한다.

2번 예제는 10보다 큰 숫자를 가지고 있는지 판별한다. 1번 예제와 다른 점은 `callback` 함수가 실행될 때마다 로그를 남기는 것이다. 실행 결과를 보면 10만 로그에 찍히는 것을 알 수 있다. 이처럼 `some` 메서드는 전체 원소를 순회하지 않고 `true`값이 반환되면 그 즉시 실행을 종료한다.

## 결론

javascript에서는 다양한 메서드를 통해 배열을 조작 할 수 있다. 이러한 메서드들을 잘 활용하면 기존의 반복문을 대부분 제거 할 수 있고, 코드 또한 간결해진다.

이 게시물에서 다루지 못한 다양한 메서드들안 [이 링크](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array#%EB%A9%94%EC%84%9C%EB%93%9C_2){:target="_blank"}에서 확인할 수 있다.

## 참고자료

- [`Array.prototype.forEach()`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach){:target="_blank"}
- [`Array.prototype.map()`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/map){:target="_blank"}
- [`Array.prototype.reduce()`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce){:target="_blank"}
- [`Array.prototype.find()`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/find){:target="_blank"}
- [`Array.prototype.findIndex()`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/findindex){:target="_blank"}
- [`Array.prototype.some()`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/some){:target="_blank"}
- [`Array.prototype.every()`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/every){:target="_blank"}