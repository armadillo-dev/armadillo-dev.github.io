---
title: "[Refactoring] 8장 데이터 체계화"
date: 2019-04-19 22:00:00
categories: book
tags: book refactoring
description: "데이터 체계화 리펙토링을 통해 데이터 연동을 더 간편하게 할 수 있는 방법을 알아본다."
---

> 본 게시물은 마틴 파울러의 [리팩토링: 코드 품질을 개선하는 객체지향 사고법][link-book] 도서를 읽고 정리한 내용입니다.

데이터 체계화 리펙토링은 데이터 연동을 더 간편하게 해준다.

## 1. 필드 자체 캡슐화(Self Encapsulate Field)

필드용 읽기/쓰기(setter, getter) 메서드를 작성해서 두 메서드를 통해서만 필드에 접근하게 만든다. 이 방식을 사용하면 하위 클래스가 메서드에 해당 정보를 가져오는 방식을 재정의 하거나, 데이터 관리의 유연성을 확보 할 수 있다.

### Before

```js
class SomeClass {
  constructor() {
    this.low;
    this.high;
  }

  includes(number) {
    return number >= this.low && number <= this.high
  }
}
```

### After

```js
class SomeClass {
  constructor(low, high) {
    this._low = low;
    this._high = high;
  }

  get low() {
    return this._low
  }

  set low(number){
    this._low = number
  }

  get high() {
    return this._high
  }

  set high(number) {
    this._high = number
  }

  includes(number) {
    return number >= this.low && number <= this.high
  }
}
```

## 2. 데이터 값을 객체로 전환(Replace Data Value with Object)

프로젝트의 규모가 커질수록 처음에는 단순했던 데이터가 복잡해진다. 이에따라 데이터에 요구되는 항목이나 데이터도 늘어나게 된다. 데이터나 기능을 더 추가해야 할 때에는 해당 데이터를 객체로 만든다.

### Before

```js
class Order {
  constructor(customer) {
    this._customer = customer // 문자열 형식
  }

  get customer() {
    return this._customer
  }

  set customer(str) {
    this._customer = str
  }  
}
```

### After

```js
class Customer {
  constructor(name){
    this._name = name
  }

  get name() {
    return this._name
  }
}

class Order {
  constructor(customerName) {
    this._customer = new Customer(customerName)
  }

  get customerName() {
    return this._customer.name
  }

  set customer(customerName) {
    this._customer = new Customer(customerName)
  }  
}
```

### 3. 값을 참조로 전환(Change Value to Reference)

객체는 참조 객체와 값 객체로 분류 할 수 있다. 값 객체 클래스에 같은 인스턴스가 많이 들어있을 경우 해당 객체를 참조 객체로 전환한다.

### Before

```js
class Customer {
  constructor(name){
    this._name = name
  }

  get name() {
    return this._name
  }
}

class Order {
  constructor(customerName) {
    this._customer = new Customer(customerName)
  }

  get customerName() {
    return this._customer.name
  }

  set customer(customerName) {
    // 이름이 동일하더라도 다른 고객으로 판단됨(항상 new를 통해 정의하기 때문에)
    this._customer = new Customer(customerName) 
  }  
}
```

### After

```js
class Customer {
  static create(name) {
    return new Customer(name)
  }

  constructor(name){
    this._name = name
  }

  get name() {
    return this._name
  }
}

class Order {
  constructor(customerName) {
    this._customer = Customer.create(customerName)
  }

  get customerName() {
    return this._customer.name
  }

  set customer(customerName) {
    this._customer = new Customer(customerName)
  }  
}
```

## 4. 참조를 값으로 전환(Change Reference to Value)

참조 객체를 사용한 작업이 복잡해질 경우 값 객체로 전환한다. 값 객체는 변경 할 수 없어야 한다는 주요 특성이 있다.

### Before 

```js
class Currency {
  constructor(code) {
    this._code = code
  }

  get code() {
    return this._code
  }
}

// 참조 객체는 객체가 저장되어 있는 메모리 주소로 비교한다.
new Currency('USD') === new Currency('USD') // false

```

### After

```js
class Currency {
  constructor(code) {
    this._code = code
  }

  equals(currency) {
    if (!(currency instanceof Currency)) return false
    return this.code === currency.code
  }

  get code() {
    return this._code
  }
}

// 값 객체는 객체가 가지고 있는 값을 이용해 비교한다.
new Currency('USD').equals(new Currency('USD')) // true
```

## 5. 배열을 객체로 전환(Replace Array with Object)

배열은 비슷한 값 또는 객체의 컬렉션 용도로 사용해야 한다. 만약 배열을 구성하는 원소가 각 인덱스마다 특별한 의미를 가질 땐 그 배열을 객체로 전환한다.

### Before

```js
const row = ['armadillo', 15] // 0: 이름, 1: 승 수
```

### After

```js
class Performance {
  constructor(name, wins) {
    this.name = name
    this.wins = wins
  }
}

const row = new Performance('armadillo', 15);
```

## 6. 관측 데이터 복제(Duplicate Observed Data)

비즈니스 로직 처리 코드와 사용자 인터페이스 처리 코드를 분리한다.



[link-book]: https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=20793053