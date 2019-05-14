# [ES6] Iterable과 Iterator 이해하기

자바스크립트의 레퍼런스를 살펴보면 Iterable, Iterator라는 단어를 자주 접할 수 있다. 또한 ES6에 새롭게 추가된 `Set`, `Map` 객체는 Iterable로 평가된다. 심지어 항상 사용하는 `String`, `Array` 또한  Iterable로 평가된다.

이쯤되자, Iterable과 Iterator에 대해 명확한 이해가 필요하다고 생각했다. Iterable, Iterator가 무엇이기에 다양한 영역에서 사용되 것인지 정리해보았다.

## Iteration protocol

Iteration protocol은 어떠한 객체든 특정 조건을 만족하면 Iterable 또는 Iterator로 평가 받을 수 있도록 하는 규약이다.

## Iterable

Iterable은 반복 가능한 객체를 의미한다. Iterable로 평가된 값은 [`for...of`문](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/for...of){:target="_blank"}, [전개 문법(Spread syntax)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax){:target="_blank"}, [구조 분해 할당(Destructuring)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment){:target="_blank"} 등에 사용 할 수 있다.

**Iterable이 되기 위해서는 다음 사항을 준수해야 한다.**

1. 객체 내에 `[Symbol.iterator]`(= `@@iterator`) 메서드가 존재해야 한다.
2. `[Symbol.iterator]` 는 **Iterator 객체**를 반환해야 한다.

```js
const iterable = {
  [Symbol.iterator]() {
    return someIteratorObject
  }
  ...
}

for(item of iterable) {
  console.log(item) // work
}
```

## Iterator

Iterator는 Iterable 객체에서 반복을 실행하는 반복기를 뜻한다. Iterable 객체가 반복 하면서 어떠한 값을 반환 할 것인지 결정하게 된다.

**Iterator가 되기 위해서는 다음 사항을 준수해야 한다.**

1. 객체 내에 `next` 메서드가 존재해야 한다.
2. `next` 메서드는 **IteratorResult 객체**를 반환해야 한다.
3. IteratorResult 객체는 `done: boolean`과 `value: any` 프로퍼티를 가진다.
4. 이전 `next` 메서드의 호출의 결과로 `done` 값이 `true`를 리턴했다면, 이후 호출에 대한 `done` 값도 `true`여야 한다.

위와 같이 Iterator 객체가 반환하는 IteratorResult 객체는 `done`과 `value` 프로퍼티를 가진다. `done`은 Iterator의 반복이 모두 완료되었는지를 판별한다. Iterator는 `done` 값이 `true`가 될 때까지 반복을 수행한다. `value`는 각 반복 수행을 하면서 반환하는 값이다.

```js
const iterable = {
  [Symbol.iterator]() {
    let i = 0
    // iterator 객체
    return {
      next() {
        while(i < 10) { // i가 10이 될 때까지 반복기 수행
          return { value: i++, done: false }
        }
        return { done: true } // i 가 10이 되면 반복 종료(value 값 생략 가능)
      }
    }
  }
}

for(let num of iterable) console.log(num) // 1, 2, ..., 9
```

## Well-formed iterable

Iterator이면서 Iterable인 객체를 well-formed iterable이라고 한다. 쉽게 말하면 `iter[Symbol.iterable]() === iter`일 경우를 well-formed iterable이라고 한다.

```js
const wellFormedIterable = { // Iterator 객체
  next() {
    return someIteratorResultObject
  }

  // Iterator 객체에 Symbol.iterator 메서드가 존재하며, 
  // 해당 메서드가 자기 자신(iterator)을 반환한다.
  [Symbol.iterator]() {
    return this
  }
  ...
}
```

well-form iterable의 장점은, 자기 자신의 상태를 기억 할 수 있다는 것이다. 이로 인해 Iterator는 얼마나 반복을 진행했는지 기억 할 수 있다.

    const iterable = {
      i: 0,
      next(){
        while(this.i < 10) {
          return { value: this.i++, done: false }
        }
        return { done: true }
      },
      [Symbol.iterator]() {
        return this
      }
    }
    
    const iter = iterable[Symbol.iterator]()
    
    console.log(iter.next()) // { value: 0, done: false }
    console.log(iter.next()) // { value: 1, done: false }
    console.log(iter.next()) // { value: 2, done: false }
    
    // 앞에서 next()를 호출했기 때문에 3부터 시작된다.
    for (num of iter) console.log(num) // 3, 4, ..., 9 
    

## Iterable 만들어보기

원하는 만큼 홀수를 출력하는 iterable 객체를 만들어보자. 만들어진 객체는 앞서 말했듯 [`for...of`문](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/for...of){:target="_blank"}, [전개 문법(Spread syntax)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax){:target="_blank"}, [구조 분해 할당(Destructuring)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment){:target="_blank"} 등과 함께 사용 할 수 있다. 물론 `next` 메서드를 직접 호출하는 것도 가능하다.

<iframe height="265" style="width: 100%;" scrolling="no" title="Custom iterable" src="//codepen.io/armadillo-dev167/embed/mYrKPQ/?height=265&theme-id=0&default-tab=js,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
</iframe>

## 마무리

이번 글을 통해 Iterator와 Iterable에 대한 개념을 정리 해볼 수 있었다. 하지만, 직접 Iterator/Iterable 객체를 만드는 것은 번거로운 일이다. 그래서 자바스크립트는 이용해 손쉽게 well-formed iterable을 생성 할 수 있도록 **[제네레이터 함수](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/function*){:target="_blank"}**를 제공한다. 다음에는 제네레이터에 대해 알아보는 포스팅을 하고자 한다.

## 참고문서

- ECMAScript® 2015 Language Specification / Iteration section - [https://www.ecma-international.org/ecma-262/6.0/#sec-iteration](https://www.ecma-international.org/ecma-262/6.0/#sec-iteration)
- MDN / iteration protocol - [https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Iteration_protocols](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Iteration_protocols)