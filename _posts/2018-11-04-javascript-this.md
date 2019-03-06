---
title: "[javascript] this 톺아보기"
date: 2018-11-04 9:33 -0400
categories: javascript this
---

javascript에서 ``this``는 호출 환경에 따라 그 대상이 달라진다. 다음과 같은 환경에서 서로 다른 ``this``를 가지게 된다.

- 함수 호출(function call)
- 메소드 호출(method call)
- 생성자 호출(constructor call)
- 간접 호출(apply, call)
- 화살표 함수에서 호출(arrow function call)

## 1. 함수 호출(function call)

함수 호출에서 ``this``는 root 객체를 가리킨다(브라우저 환경에서는 ``window``, node.js환경에서는 ``global`` 객체). 다만, 엄격 모드(``strict mode``)에서의 ``this``는 ``undefined``이다.

```js
function func() {
  console.log(this);
}

func(); //window 

function func2() {
  'use strict';
  console.log(this);
}

func2(); // undefined
```

## 2. 메서드 호출(method call)

클래스 내부의 메서드를 호출 할 경우 this는 해당 클래스 인스턴스를 가리킨다.

```js
class ClassA {
  constructor() {
    this.str = 'ClassA\'s str';
  }
  
  method() {
    console.log(this);
    console.log(this.str);
  }
}

const classA = new ClassA();
classA.method(); //ClassA { str: ... }, ClassA's str
```

개인적으로 메서드 호출에서의 실수가 가장 빈번하게 일어나게 되는 것 같다(이 글을 쓰게 된 계기도 메서드 호출에서의 혼동 때문이었다). 메서드 호출에서의 실수는 다음과 같이 분류 할 수 있다.

- ``setTimeout(class.method, 1000)`` 과 같이 함수의 인자로 method를 사용 할 경우(클래스와 분리시켜 메서드를 호출하는 경우)  ``this``는 인스턴스 객체가 아닌 ``window`` 객체를 가리킨다(엄격 모드라면 ``undefined``).
- 메서드를 특정 변수에 저장한 뒤 호출하면 ``this``는 ``undefined``가 된다. 생각하기로는 이러한 사용도 ``this``는 글로벌 객체를 가리켜야 하는 것 아닌가 생각했는데 ``undefined``가 할당된다.

```js
class ClassA {
  constructor() {
    this.str = 'ClassA\'s str';
  }
  
  method() {
    console.log(this);
  }
}

const classA = new ClassA();
const func = classA.method;

func(); // undefined

setTimeout(classA.method, 1000); // window
```

## 3. 생성자 호출(constructor call)

생성자 내부에서의 ``this``도 method 내부의 ``this``와 동일하게 해당 인스턴스를 가리킨다.

```js
class ClassA {
  constructor() {
    this.str = 'str';
    console.log(this); 
  }
}

const classA = new ClassA(); // ClassA { str: 'str' }
```

생성자 내부의 ``this``를 사용하며 발생할 수 있는 실수는 ``new`` 키워드를 생략할 때이다. 이 때의 ``this``는 ``window`` 객체를 가리키게 된다.

## 4. 간접 실행(apply, call, + bind)

javascript는 ``apply()`` 또는 ``call()`` 메서드를 통해 함수를 간접 실행 할 수 있다. 두 메서드는 첫 번째 인자로 어떤 객체를 ``this``로 사용할지 전달 받는다.

```js
fun.call(thisArg[, arg1[, arg2[, …]]])
fun.apply(thisArg, [argsArray])
```

두 메서드를 사용 할 경우 ``this``는 첫번째 인자로 전달된 객체를 가리킨다. 이와 비슷하게 ``bind()`` 메서드를 이용해서 ``this``를 주입 시킬 수도 있다.

## 5. 화살표 함수(arrow function)

화살표 함수의 ``this``는 상위 scope의 ``this``를 가리킨다.

```js
const func = () => {
  console.log(this);
}

func() // window

class ArrowFunctionClass {
  method = () => {
    console.log(this);
  };
}

const instance = new ArrowFunctionClass();
instance.method(); //ArrowFunctionClass {...}

```