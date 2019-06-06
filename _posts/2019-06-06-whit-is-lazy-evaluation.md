---
title: "[Javascript] 지연 평가(Lazy evaluation) 를 이용한 성능 개선"
date: 2019-06-06 13:00:00
categories: javascript
tags: javascript lazy-loading
description: "컴퓨터 프로그래밍에서 느긋한 계산법(Lazy evaluation)은 계산의 결과 값이 필요할 때까지 계산을 늦추는 기법이다. 위키피디아에서 지연 평가를 설명하는 문장이다. 지연 평가는 필요 할 때까지 계산을 늦추면서 불필요한 계산을 줄일 수 있다. 이러한 지연 평가는 3가지 이점을 가지고 있다."
---

이전 글에서 Iterable/Iterator와 Generator에 대해 알아보았다(*이 글은 이전 글을 읽었다는 가정 하에 작성되었다.*). Generator를 설명하는 글 말미에서 말했듯, 이번 글에서는 지연 평가란 무엇이고 어떻게 성능 개선에 활용할 수 있는지 알아보도록 한다.

## 지연 평가란?

> 컴퓨터 프로그래밍에서 느긋한 계산법(Lazy evaluation)은 계산의 결과 값이 필요할 때까지 계산을 늦추는 기법이다. - [Wikipedia / 지연 평가](https://ko.wikipedia.org/wiki/%EB%8A%90%EA%B8%8B%ED%95%9C_%EA%B3%84%EC%82%B0%EB%B2%95){:target="_blank"}

위키피디아에서 지연 평가를 설명하는 문장이다. 지연 평가는 필요 할 때까지 계산을 늦추면서 불필요한 계산을 줄일 수 있다. 이러한 지연 평가는 3가지 이점을 가지고 있다.

1. 불필요한 계산을 하지 않으므로 빠른 계산이 가능하다.
2. 즉시 평가되지 않기 때문에 무한 자료 구조를 사용 할 수 있다.
3. 복잡한 수식에서 오류 상태를 피할 수 있다.

이번 글에서는 1번 항목을 중심으로 설명하고자 한다.

## 지연 평가 동작 방식

위해 엄격한 평가(strict evaluation)의 동작 방식과 비교를 통해 지연 평가의 동작 방식을 알아보자. 엄격한 평가는 지연 평가의 반대말로, 수행되는 즉시 계산의 결과를 도출하는 동작 방식을 뜻한다.

**0~5로 이루어진 배열에서 10을 더한 뒤, 홀수 2개만 추출하는 로직을 예제로 살펴보자.**

### 엄격한 평가

#### 예제 코드

```js
const arr = [0, 1, 2, 3, 4, 5]
const result = arr.map(num => num + 10).filter(num => num % 2).slice(0, 2)
console.log(result) // [11, 13]
```

#### 동작 방식

엄격한 평가는 평가 흐름이 왼쪽에서 오른쪽으로 흐른다. 아래의 도표를 보면 `map`, `filter`, `slice` 각각의 계산이 모두 종료되어야 다음 단계를 수행하는 것을 알 수 있다. 최종 결과물(**[11, 13]**)을 보면 `4`와 `5`에 대한 계산은 불필요함을 알 수 있다.

따라서 **총 계산 횟수는 14번(`map` 6번, `filter` 6번, `slice` 2번)이 된다.**

![즉시 평가. 평가 흐름이 왼쪽에서 오른쪽(→)으로 흐른다.](/asserts/images/strict-evaluation-process.png)
*즉시 평가. 평가 흐름이 왼쪽에서 오른쪽(→)으로 흐른다.*

### 지연 평가

#### 예제 코드

다음은 지연 평가를 활용해 예제를 수행한 코드이다. 사용된 함수들은 잠시 뒤에 어떻게 구현되었는지 살펴보고, 지금은 작성된 코드만 살펴보자. `result`부분의 코드는 오른쪽 아래에서부터 왼쪽 위로 읽으면 된다.

```js
const arr = [0, 1, 2, 3, 4, 5]
const result = _.take(2,
  L.filter(num => num % 2,
    L.map(num => num + 10, arr)))
console.log(result) // [11, 13]
```

#### 동작 방식

지연 평가는 평가 흐름이 위에서 아래로 흐른다. 아래의 도표를 보면 배열의 각 원들이 `map`, `filter`, `take` 함수를 차례대로 수행한다는 것을 알 수 있다. `3`까지 평가가 완료되었을 때 이미 원하는 결과가 나왔기 때문에, `4`와 `5`에 대한 계산은 하지 않는다.

따라서 **총 계산 횟수는 10번(`map` 4번, `filter` 4번, `take` 2번)이 된다.**

![지연 평가. 평가 흐름이 위에서 아래로(↓) 흐른다.](/asserts/images/lazy-evaluation-process.png)
*지연 평가. 평가 흐름이 위에서 아래로(↓) 흐른다.*

이처럼 지연 평가는 연산 횟수 자체를 줄여 보다 좋은 성능을 준다. 평소 '어떻게 하면 연산을 보다 빠르게 할 수 있을까'에 대한 생각은 했지만, 이런 식의 접근은 생각해보지 못해서 굉장히 새롭고 신세계를 접하는 기분이었다 :)

지연 평가는 대상이 크면 클수록 그 효과를 발휘한다. 다음은 위의 예제 코드에서 배열의 원소 개수를 1,000,000개로 수정한 뒤 적용한 [성능 비교 결과](https://jsperf.com/lazy-evaluation-vs-strict-evaluation-armadillo-ko/1){:target="_blank"}이다. 그래프에서 보듯 지연 평가가 압도적인 성능을 자랑한다.

![lazy evaluation vs strict evaluation](/asserts/images/lazy-evalutation-vs-strict-evaluation.png)
*lazy evaluation vs strict evaluation*

## 지연 평가 활용 함수 만들기

위의 예제에서 사용한 지연 평가용 함수를 직접 작성해보자. `L` 객체 안에 각각의 함수들을 정의했다. 각각의 함수들이 Generator를 이용한 것을 볼 수 있다. Generator 함수를 호출하면 Generator 객체를 반환한다. 이 Generator 객체는 바로 내부 동작을 수행하지 않고 `next` 메서드를 호출할 때 비로소 연산을 수행한다. 이러한 특징이 지연 평가를 가능케 한다.

```js
// 지연 평가 함수 모음
const L = {}

L.map = function* (f, iter) {
  for (let item of iter) {
    yield f(item)
  }
}

L.filter = function* (f, iter) {
  for (let item of iter) {
    if(f(item)) yield item
  }
}

L.take = function* (limit, iter) {
  for (let item of iter) {
    yield item
    if (!--limit) break
  }
}

// 엄격한 평가 함수 모음
const _ = {}

_.take = function(limit, iter) {
  const result = []
  for (let item of iter) {
    result.push(item)
    if (result.length === limit) return result
  }
}
```

## Lodash를 이용한 지연평가

[Loadash](https://lodash.com/){:target="_blank"}에서는 `_.chain` , `_.value` 함수를 이용해 지연 평가를 지원한다. `chain` 함수의 인자로 사용할 데이터를 넘기면 해당 데이터를 `lodash` 객체로 감싼다. 이후 제공되는 지연 평가 함수들을 이용해 원하는 결과로 가공한다. 최종적으로 `value` 함수를 통해 wrapping되어 있는 결과를 실제 값으로 치환한다.

위의 예제를 lodash를 이용해 풀어보면 다음과 같다.

```js
const arr = [0, 1, 2, 3, 4, 5]
const result = _.chain(arr)
  .map(num => num + 10)
  .filter(num => num % 2)
  .take(2)
  .value()
console.log(result) // [11, 13]
```

lodash는 위에서 직접 작성했던 것보다 훨씬 다양한 종류의 함수를 제공하고 있기 때문에 문서를 보면서 각자의 역할에 맞게 사용 할 수 있다.

## 관련 글

- [[Javascript] Iterable과 Iterator 이해하기](https://armadillo-dev.github.io/javascript/what-is-iterable-and-iterator/){:target="_blank"}
- [[Javascript] Generator 이해하기](https://armadillo-dev.github.io/javascript/what-is-generator/){:target="_blank"}

## 참고 링크

- [인프런 - 자바스크립트로 알아보는 함수형 프로그래밍 / 유인동](https://www.inflearn.com/course/%ED%95%A8%EC%88%98%ED%98%95-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/dashboard){:target="_blank"}
- [Lodash의 지연 평가 소개 by Filip Zawada](https://edykim.com/ko/post/introduction-to-lodashs-delay-evaluation-by-filip-zawada/){:target="_blank"}
- [Lodash - chain](https://lodash.com/docs/4.17.11#chain){:target="_blank"}
