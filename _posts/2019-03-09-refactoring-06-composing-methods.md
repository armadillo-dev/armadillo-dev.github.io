---
title: "[Refactoring] 6장 메서드 정리 요약"
date: 2019-03-09 08:31 -0400
categories: book
tags: book refactoring
description: "리팩토링의 주된 작업은 코드를 포장하는 메서드를 적절히 정리하는 것이다. 거의 모든 문제점은 장황한 메서드로 인해 생긴다. 장황한 메서드에는 많은 정보가 들어있는데, 마구 얽힌 복잡한 로직에 이 정보들이 묻혀버린다."
---

> 본 게시물은 마틴 파울러의 [리팩토링: 코드 품질을 개선하는 객체지향 사고법][link-book] 도서를 읽고 정리한 내용입니다.

리팩토링의 주된 작업은 코드를 포장하는 메서드를 적절히 정리하는 것이다. 거의 모든 문제점은 장황한 메서드로 인해 생긴다. 장황한 메서드에는 많은 정보가 들어있는데, 마구 얽힌 복잡한 로직에 이 정보들이 묻혀버린다.

## 1. 메서드 추출(Extract Method)

메서드 추출 기법은 가장 많이 사용되는 기법이다. 메서드가 너무 길거나 코드에 주석을 달아야만 의도를 이해할 수 있을 때, 그 코드를 별도의 메서드로 만든다. 이 때 메서드 명은 원리가 아닌 기능을 나타내는 이름으로 짓는다.

### Before

```js
class SomeClass {
  printOwing(amount) {
    pringBanner();

    // 세부 정보 출력
    console.log(`name: ${this.name}`);
    console.log(`amount: ${amount}`);
  }
  ...
}
```

### After

```js
class SomeClass {
  printOwing(amount) {
    pringBanner();
    printDetails(amount);
  }

  printDetails() {
    console.log(`name: ${this.name}`);
    console.log(`amount: ${amount}`);
  }
  ...
}
```

## 2. 메서드 내용 직접 삽입(Inline Method)

메서드명에 모든 기능이 반영될 정도로 메서드 기능이 단순할 때에는 해당 메서드를 없애 불필요한 인다이렉션을 제거한다. 해당 메서드를 호출하는 모든 부분을 메서드 내용으로 교채해주면 된다.

### Before

```js
class SomeClass {
  getRating() {
    return (moreThanFiveLateDeliveries()) ? 2 : 1;
  }

  moreThanFiveLateDeliveries() {
    return this.numberOfLateDeliveries > 5;
  }
  ...
}
```

### After

```js
class SomeClass {
  getRating() {
    return (this.numberOfLateDeliveries > 5) ? 2 : 1;
  }
  ...
}
```

## 3. 임시변수 내용 직접 삽입(Inline Temp)

간단한 수식을 대입받는 임시변수로 인해 다른 리펙토링 기법 적용이 힘들 때, 그 임시변수를 참조하는 부분을 전부 수식으로 치환한다. 임시변수 내용 직접 삽입 기법은 임시변수를 메서드 호출로 전환 기법과 함께 사용되는 경우가 대부분이다.

### Before

```js
const basePrice = anOrder.basePrice();
return (basePrice > 1000);
```

### After

```js
return (anOrder.basePrice() > 1000);
```

## 4. 임시변수를 메서드 호출로 전환(Replace Temp with Query)

임시변수는 일시적이고 해당 메서드 내에서만 사용 할 수 있다. 그래서 임시변수에 접근하기 위해 코드가 길어지게 된다. 임시변수를 메서드 호출로 수정하면 클래스 안 모든 메서드가 접근 할 수 있으므로 클래스가 훨씬 깔끔해진다.

### Before

```js
const basePrice = this.quantity * this.itemPrice;
if (basePrice > 1000) {
  return basePrice * 0.95;
} else {
  return basePrice * 0.98;
}
```

### After

```js
if (basePrice() > 1000) {
  return basePrice() * 0.95;
} else {
  return basePrice() * 0.98;
}
...
basePrice() {
  return this.quantity * this.itemPrice;
}
```

## 5. 직관적 임시변수 사용(Introduce Explaining Variable)

수식이 너무 복잡할 때, 수식의 결과나 수식의 일부분을 용도에 부합하는 직관적 이름의 임시변수에 대입한다. 다만, 이 방법보다 메서드 추출을 사용하길 권장한다.

### Before

```js
if (
  (platform.toUpperCase().indexOf('MAC') > -1) &&
  (browser.toUpperCase().indexOf('IE') > -1) &&
  wasInitialized() && resize > 0
) {
  ...
}
```

### After

```js
const isMacOS = platform.toUpperCase().indexOf('MAC') > -1;
const isIEBrowser = browser.toUpperCase().indexOf('IE') > -1;
const wasResized = resize > 0;

if (isMacOS && isIEBrowser && wasInitialized() && wasResized) {
  ...
}
```

### 6. 임시변수 분리(Split Temporary Variable)

루프 변수(``for(let i = 0; i < 10; i ++)``에서의 ``i``)나 값 누적용 임시변수가 아닌 임시변수에 여러번 값이 대입될 땐, 각 대입마다 다른 임시변수를 사용한다. 값이 두 번 이상 대입된다는 건 그 변수가 메서드안에서 여러 용도로 사용된다는 반증이다.

### Before

```js
let temp = 2 * (height + width);
console.log(temp);
temp = height * width;
console.log(temp);
```

### After

```js
const perimeter = 2 * (height + width);
console.log(perimeter);
const area = height * width;
console.log(area);
```

## 7. 매개변수로의 값 대입 제거(Remove Assignments to Parameters)

매개변수로 전달받은 객체에 어떠한 처리를 하든 상관 없고 일상적인 작업이지만, 다른 객체 참조로 변경하는 것은 절대로 안된다. 매개변수로 값을 대입하는 코드가 있을 땐 매개변수 대신 임시변수를 사용하게 수정한다.

### Before

```js
const discount = function(inputVal, quantity, yearToDate) {
  if (inputVal > 50) inputVal -= 2;
}
```

### After

```js
const discount = function(inputVal, quantity, yearToDate) {
  let result = inputVal;
  if (inputVal > 50) result -= 2;
}
```

## 8. 메서드를 메서드 객체로 전환(Replae Method with Method Object)

지역변수 때문에 메서드 추출을 적용 할 수 없는 긴 메서드가 있을 때, 그 메서드 자체를 객체로 전환해서 모든 지역변수를 개체의 필드로 변경한다. 이후 메서드를 개체 안의 여러 메서드로 쪼갠다.

### Before

```js
class Order {
  price() {
    const primaryBasePrice;
    const secondaryBasePrice;
    const tertiaryBasePrice;
    ...
  }
}
```

### After

```js
class Order {
  price() {
    return new PriceCalculator(this).compute();
  }
}

class PriceCalculator {
  constructor(order) {
    this.primaryBasePrice;
    this.secondaryBasePrice;
    this.tertiaryBasePrice;
    this.order = order;
  }

  compute() {
    ...
  }
}
```

## 9. 알고리즘 전환

어떤 기능을 수행하기 위해 비교적 간단한 방법이 있다면, 복잡한 방법을 좀 더 간단한 방법으로 교체해야 한다. 이 작업을 수행하기 위해서는 메서드를 최대한 잘게 쪼개야 수월하다.

### Before

```js
const foundPerson(people) {
  for (let i = 0; i < people.length; i += 1) {
    if (people[i] === 'Don') {
      return 'Don';
    }

    if (people[i] === 'John') {
      return 'John';
    }

    if (people[i] === 'Kent') {
      return 'Kent';
    }
  }
}
```

### After

```js
const foundPerson(people) {
  const candidates = ['Don', 'John', 'Kent'];
  for (let i = 0; i < people.length; i += 1) {
    if (candidates.include(people[i])) {
      return people[i];
    }
  }
  return '';
}
```

[link-book]: https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=20793053