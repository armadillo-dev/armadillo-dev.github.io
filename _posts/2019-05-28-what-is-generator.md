---
title: "[Javascript] Generator 이해하기"
date: 2019-05-28 00:00:00
categories: javascript
tags: javascript generator
description: "자바스크립트는 손쉽게 well-formed iterable을 생성할 수 있는 Generator 함수를 제공한다. 이번에는 Generator는 무엇이고, 어떤 특징이 있는지 알아보자."
---
[이전 글](https://armadillo-dev.github.io/javascript/what-is-iterable-and-iterator/){:target="_blank"}에서는 Iterable과 iterator에 대한 개념을 정리해보았다(이번 글을 읽기 전에 먼저 읽어보는 것을 추천한다). 이전 글의 끝에서 말했듯, 자바스크립트는 손쉽게 well-formed iterable을 생성할 수 있는 Generator 함수를 제공한다. 이번에는 Generator는 무엇이고, 어떤 특징이 있는지 알아보자.

## Generator 정의

Generator는 Generator 함수를 실행 했을 때 반환되는 객체로, well-formed iterable로 평가된다.

### Generator 함수

```js
function* [name]([param1[, param2[, ..., paramN]]]) { // *을 이용해 정의
    statements
}
```

Generator 함수는 일반적인 함수와 달리 `function` 뒤에 `*`를 붙여서 정의한다. **Generator 함수는 화살표 함수를 이용해 정의 할 수 없다.**

### `yield`

Generator 함수에서 보이는 `yield`는 생성된 Generator 객체가 반복을 수행하면서 반활 할 값(IteratorResult)을 결정한다. 더이상 `yield`가 존재하지 않을 때까지 반복을 수행하고, 그 이후에 `next` 메서드를 호출하면 `{ value: undefined, done: true }`를 반환한다. 특이한 점은 **Generator 객체에서 `next` 메서드를 호출하면 이전 호출 다음에 존재하는 `yield` 까지만 실행된다**는 점이다.

```js
function* someGeneratorFunction() {
  console.log('1번 실행')
  yield 1
  console.log('2번 실행')
  yield 2
  console.log('3번 실행')
  yield 3
}

const iter = someGeneratorFunction()

iter.next() // 1번 실행, { value: 1, done: false }

for (num of iter) console.log(num) // 2번 실행, 2, 3번 실행, 3
```

위의 `someGeneratorFunction` 함수를 통해 만든 `iter`에서 `next` 메서드를 실행하면, 첫번째 `yield` 우항에 있는 `1`이 IteratorResult의 `value`로 담겨 반환되며, `1번 실행` 부분만 콘솔에 출력된다.

### `yield*`

`yield`  키워드 뒤에 `*`를 붙이면 또 다른 iterator를 `yield` 시킬 수 있게 된다. 이를 통해 중첩된 Generator 구조를 만들 수 있다.

```js
function* someGeneratorFunction() {
  const iter = otherGeneratorFunction()
  yield 0
  yield* iter
}

function* otherGeneratorFunction() {
  yield 1
  yield 2
}

const gen = someGeneratorFunction()
gen.next() // { value: 0, done: false }
gen.next() // { value: 1, done: false }
gen.next() // { value: 2, done: false }
gen.next() // { value: undefined, done: true }
```

### `return`

Generator 함수 내부에서 `return`을 실행하거나, Generator 인스턴스에서  `return` 메서드를 실행하면 값이 반환되며 `done` 값이 `true`로 변경된다. 때문에 더 이상 반복이 실행되지 않는다. **주의할 점은 `for..of`, 전개 연산자 등에서는 `return` 키워드가 나오기 전까지의 `yield`만 취급한다는 것이다.**

```js
function* gen() {
  yield 1
  return 2
  yield 3
}

const iter = gen()
iter.next() // { value: 1, done: false }
iter.next() // { value: 2, done: true }
iter.next() // { value: undefined, done: true }

const iter2 = gen()
for (let num of iter2) console.log(num) // 1 <-- 2는 출력되지 않는다.
```

### `throw`

Generator 함수 내부에서 `throw`를 실행하거나, Generator 인스턴스에서 `throw` 메서드를 실행하면 `Uncaught` 에러가 발생하며 `done` 값이 `true`로 변경된다. 

```js
function* gen() {
  yield 1
  throw '에러 발생!!'
  yield 3
}

const iter = gen()
iter.next() // { value: 1, done: false }
iter.next() // Uncaught 에러 발생!!
iter.next() // { value: undefined, done: true }
```

## Generator 특징

### 문장을 값으로 전환

Generator를 이용하면 문장을 값으로 만들 수 있다. 이를 통해 프로그래머가 원하는 어떠한 값이든 반복이 가능하게 만들 수 있다.

아래는 원하는 범위 내에서 홀수 값만 입력 받는 예제이다. Generator 함수 내에 표현된 문장(`for`문)을 이용해 100까지 존재하는 홀수 iterator를 값으로 전환했다.

```js
function* odds(limit = 10) {
  for(let i = 1; i < limit; i += 2) {
    yield i
  }
}

const iter = odds(100)
    for (num of iter) console.log(num) // 1, 3, 5, ..., 99
```

### 지연평가(Lazy evaluation)

Generator는 어떠한 값에 대하여 평가하는 시점을 사용자가 선택할 수 있다. 아래 코드에서 사용자는 `gen` 함수를 실행했지만, 로그에는 아무런 내용도 출력되지 않는다.  `iter.next()`를 실행해야지 로그에 `실행되지 않는다.` 영역이 출력된다.

자바스크립트에서는 Generator의 이러한 특징을 이용해 [지연 평가](https://ko.wikipedia.org/wiki/%EB%8A%90%EA%B8%8B%ED%95%9C_%EA%B3%84%EC%82%B0%EB%B2%95){:target="_blank"}를 구현 할 수 있다.

```js
function* gen(){
  console.log('실행되지 않는다.') //iter.next()를 실행해야 출력된다.
  yield 1
  yield 2
  yield 3
}

const iter = gen()
//iter.next()
```

## 마무리

이번 글을 통해 Generator에 대한 개념과 특징에 대해 정리해보았다. 개인적으로 Generator를 이용한 지연 평가는 처음 접했을 때 신세계를 맛보는 기분이었다. 그래서 다음 글에서는 지연 평가에 대해 자세히 다뤄보고자 한다.

## 참고자료

- [ECMAScript® 2015 Language Specification / Generator Function section](https://www.ecma-international.org/ecma-262/6.0/#sec-generator-objects){:target="_blank"}
- [MDN / 반복기 및 생성기](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Iterators_and_Generators){:target="_blank"}
- [인프런 / 함수형 프로그래밍과 JavaScript ES6+](https://www.inflearn.com/course/functional-es6/dashboard){:target="_blank"}