---
title: "[html & css]  WebP 파일을 이용한 이미지 최적화"
date: 2019-03-07 22:35 -0400
categories: javascript
tags: javascript variable const let
---

개인적으로 `const`, `let`을 통한 변수 선언은 ECMAScript2015가 발표되면서 Javascript에 가져온 큰 변화 중 하나라고 생각하다. 기존에는 모든 변수를 `var`를 이용해 선언해야 했다. 이들 사이에는 어떤 차이가 있는지 알아보자.

## 1. 호이스팅(hoisting)

`const`와 `let`은 호이스팅이 되지 않는다. 이는 변수가 선언되기 전에 해당 변수를 호출할 수 없음을 의미한다.

```js
console.log(var1); //undefined
var var1 = 'var';

console.log(var2); //Uncaught ReferenceError: var2 is not defined
let var2 = 'let';

console.log(var3); //Uncaught ReferenceError: var3 is not defined
const var3 = 'const';
```

`var`의 경우 선언하기 전에 호출하면 에러가 발생하지 않는다. 하지만 `const` 또는 `let`으로 선언된 변수는 에러를 출력한다.

## 2. scope

### 2.1 var - function scope
기존에 사용되었던 `var`는 function scope를 가진다. function scope란, 한 번 선언된 변수는 함수 단위 내에서 자유롭게 사용할 수 있음을 의미한다. 다음 코드를 보면 더욱 쉽게 이해 할 수 있다.


```js
for(var i = 0; i < 10; i++){
  console.log('i is ', i);
}
console.log('after for i is ', i); //after for i is 10

function func(){
  for(var j = 0; j < 10; j++){
    console.log('j is ', j);
  }
}
func();
console.log('after for j is ', j); //Uncaught ReferenceError: j is not defined
```

위 코드에서 `for`문을 마친 뒤에도 `i`는 10이라는 값을 계속 유지하고 있는것을 알 수 있다.
하지만 함수를 통해 생성된 `j`는 출력을 시도하면 Uncaught ReferenceError가 발생한다.

`j`는 `func()`안에서 선언되었기 때문에, 사용 범위가 `func()`로 제한되었기 때문이다. `i`의 경우 호이스팅이 일어나 영역의 제한 없이 자유롭게 사용이 가능하다.

### 2.2 const, let - block scope

`var`와는 달리 `const`와 `let`은 block scope를 가진다. function scope와의 차이는 이름에서 유추 할 수 있듯이 사용 범위가 블록 단위로 제한된다는 점이다. 다음 코드를 보자.

```js
//let
for(let i = 0; i < 10; i++){
    console.log('i is ', i);
}
console.log('after for i is ', i); //Uncaught ReferenceError: i is not defined

function func(){
    for(let j = 0; j < 10; j++){
        console.log('j is ', j);
    }
}
func();
console.log('after for j is ', j); //Uncaught ReferenceError: j is not defined
```

function scope를 설명하기 위한 코드에서 변수 선언 부분을 `let`으로만 변경했다. 그 결과, `var`를 사용했을 때에는 출력이 되었던 `i`가 이번에는 Uncaught ReferenceError를 발생시킨다(`const`도 동일한 범위로 작동한다.).

## 3. 재할당 불가
`const`의 경우 한 번 선언되고 나면 그 값을 변경하는 것이 불가능하다. 때문에 변경이 불필요한 경우 `const`를 사용하는 것을 추천한다.

```js
//const
const str = 10;
str = 11; //Uncaught SyntaxError: Identifier 'str' has already been declared
```
강제로 재할당을 시도하면 에러를 출력한다.

## 결론

아래와 같은 이유로 `var`의 사용을 멈추고 `const`와 `let`을 사용하길 권장한다.

1. `const`, `let`은 선언하지 않고 호출했을 경우 에러를 출력한다.
2. `const`, `let`은 block scope를 가지므로 function scope를 가지는 `var`보다 실수를 방지 할 수 있다.
3. `const`는 재할당이 불가하므로 변경이 불필요한 값들은 `const`로 선언해 재할당으로 인항 오류를 근절 할 수 있다.