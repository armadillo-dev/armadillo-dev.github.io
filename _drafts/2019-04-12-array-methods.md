---
title: "[ES6] 배열(array) 관련 메서드 소개 및 활용"
date: 2019-04-13 00:00
categories: es6 javascript
tags: es6 array
description: "ES6에 들어서면서 배열과 관련된 많은 메서드를 활용 할 수 있게 되었다. 관련 메서드들을 소개하고 각각의 활용 사례를 살펴보자."
---

배열은 개발에서 빠질 수 없는 기초적인 자료구조이다. 때문에 프로그래밍 언어는 배열과 관련된 다양한 메서드들을 제공한다. 자바스크립트도 예외는 아니다.

본 게시물에서는 자바스크립트에서 제공하는 배열 관련 메서드들을 소개하고 각각의 메서드를 어떤 용도로 사용할 수 있는지 살펴보도록 한다.

## forEach

```js
arr.forEach(callback[, thisArg]);
```

## map

```js
arr.map(callback(currentValue[, index[, array]])[, thisArg])
```

## reduce

```js
arr.reduce(callback[, initialValue])
```

## find

```js
arr.find(callback[, thisArg])
```

## findIndex

```js
arr.findIndex(callback[, thisArg])

```

## some

```js
arr.some(callback[, thisArg])
```

## every

```js
arr.every(callback[, thisArg])
```

## 참고자료

- [`Array.prototype.forEach()`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach){:target="_blank"}
- [`Array.prototype.map()`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/map){:target="_blank"}
- [`Array.prototype.reduce()`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce){:target="_blank"}
- [`Array.prototype.find()`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/find){:target="_blank"}
- [`Array.prototype.findIndex()`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/findindex){:target="_blank"}
- [`Array.prototype.some()`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/some){:target="_blank"}
- [`Array.prototype.every()`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/every){:target="_blank"}