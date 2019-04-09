---
title: "[Refactoring] 7장 객체 간의 기능 이동 요약"
date: 2019-04-09 22:00
categories: book
tags: book refactoring
description: "객체 설계에서 가장 중요한 일 중 하나가 바로 '기능을 어디에 넣을지 판단'하는 것이다."
---

> 본 게시물은 마틴 파울러의 [리팩토링: 코드 품질을 개선하는 객체지향 사고법][link-book] 도서를 읽고 정리한 내용입니다.

객체 설계에서 가장 중요한 일 중 하나가 바로 '기능을 어디에 넣을지 판단'하는 것이다.

## 1. 메서드 이동(Move Method)

클래스에 기능이 너무 많거나 클래스가 다른 클래스와 과하게 연동되어 의존성이 지나칠 때는 메서드를 옮기는 것이 좋다. 메서드를 옮기면 클래스가 간결해지고 여러 기능을 더 명확히 구현할 수 있다.

### Before

```js
class Account {
  ...
  overdraftCharge() {
    if (this.type.isPremium()) { // type 객체의 메서드를 호출하고 있다.
      let result = 10;
      if (this.dayOverdrawn > 7) result += (this.dayOverdrawn - 7) * 0.85;
      return result;
    } else {
      return this.dayOverdrawn * 1.75;
    }
  }

  bankCharge() {
    let result = 4.5
    if (this.dayOverdrawn > 0) result += overdraftCharge();
    return result;
  }
  ...
}
```

### After

```js
class AccountType {
  overdraftCharge(daysOverdrawn) {
    if (this.isPremium()) {
      let result = 10;
      if (dayOverdrawn > 7) result += (dayOverdrawn - 7) * 0.85;
      return result;
    } else {
      return dayOverdrawn * 1.75;
    }
  }
  ...
}

class Account {
  overdraftCharge() {
    this.type.overdraftCharge(this.dayOverdrawn)
  }
  ...
}
```

## 2. 필드 이동(Move Field)

어떤 필드가 자신이 속한 클래스보다 다른 클래스에서 더 많이 사용될 때는 대상 클래스 안에 새 필드를 선언하고 그 필드 참조 부분을 전부 새 필드 참조로 수정한다.

### Before

```js
class Account {
  constructor() {
    this.type;
    this.interestRate;
  }

  interestForAmountDays(amount, days) {
    return this.interestRate * amount * days / 365
  }
  ...
}
```

### After

```js
class AccountType {
  constructor() {
    this.interestRate;
  }

  setInterestRate(arg) {
    this.interestRate = arg
  }

  getInterestRate() {
    return this.interestRate;
  }
  ...
}

class Account {
  constructor() {
    this.type;
    this.interestRate;
  }

  interestForAmountDays(amount, days) {
    return this.type.getInterestRate() * amount * days / 365
  }
  ...
}
```

## 3. 클래스 추출(Extract Class)

클래스는 확시랗게 추상화되어야 하며, 두세 가지의 명확한 기능을 담당해야 한다. 두 클래스가 처리해야 할 기능이 하나의 클래스에 들어있을 땐 새 클래스를 만들고 기존 클래스의 관련 필드와 메서드를 새 클래스로 옮긴다.

### Before

```js
class Person {
  getName() {
    return this.name;
  }

  getTelephoneNumber() {
    return `(${this.officeAreaCode}) ${this.officeNumber}`;
  }

  getOfficeAreaCode() {
    return this.officeAreaCode;
  }

  getOfficeNumber() {
    return this.officeNumber;
  }

  setOfficeAreaCode(arg) {
    this.officeAreaCode = arg;
  }

  setOfficeNumber(arg) {
    this.officeNumber = arg;
  }
  ...
}
```

## After

```js
class TelephoneNumber {
  getTelephoneNumber() {
    return `(${this.areaCode}) ${this.number}`;
  }

  getAreaCode() {
    return this.areaCode;
  }

  getNumber() {
    return this.number;
  }

  setAreaCode(arg) {
    this.areaCode = arg
  }

  setNumber(arg) {
    this.number = arg
  }
}

class Person {
  constructor() {
    this.officeTelephone = new Telephone();
  }

  getTelephoneNumber() {
    return this.officeTelephone.getTelephoneNumber();
  }

  getOfficeTelephone() {
    return this.officeTelephone;
  }
}
```

## 4. 클래스 내용 직접 삽입(Inline Class)

클래스 내용 직접 삽입은 클래스 추출과 반대다. 클래스가 더이상 제 역할을 수행하지 못하여 존재할 이유가 없을 때, 모든 기능을 다른 클래스로 합쳐 넣고 원래의 클래스는 삭제한다.

### Before

```js
class TelephoneNumber {
  getTelephoneNumber() {
    return `(${this.areaCode}) ${this.number}`;
  }

  getAreaCode() {
    return this.areaCode;
  }

  getNumber() {
    return this.number;
  }

  setAreaCode(arg) {
    this.areaCode = arg
  }

  setNumber(arg) {
    this.number = arg
  }
}

class Person {
  constructor() {
    this.officeTelephone = new Telephone();
  }

  getTelephoneNumber() {
    return this.officeTelephone.getTelephoneNumber();
  }

  getOfficeTelephone() {
    return this.officeTelephone;
  }
}
```

### After

```js
class Person {
  getName() {
    return this.name;
  }

  getTelephoneNumber() {
    return `(${this.officeAreaCode}) ${this.officeNumber}`;
  }

  getOfficeAreaCode() {
    return this.officeAreaCode;
  }

  getOfficeNumber() {
    return this.officeNumber;
  }

  setOfficeAreaCode(arg) {
    this.officeAreaCode = arg;
  }

  setOfficeNumber(arg) {
    this.officeNumber = arg;
  }
  ...
}
```

## 5. 대리 객체 은폐(Hide Delegate)

객체에서 핵심 개념중 하나가 바로 **캡슐화**다. 캡슐화란 객체가 시스템의 다른 부분에 대한 정보의 일부만 알 수 있게 은폐하는 것을 뜻한다. 객체를 캡슐화하면 무언가를 변경할 때 그 변화를 전달해야 할 객체가 줄어드므로 변경하기 쉬워진다. 클라이언트가 객체의 대리 클래스를 호출할 땐 대리 클래스를 감추는 메서드를 서버에 작성한다.

### Before

```js
class Person {
  constructor() {
    this.department = new Department()
  }

  getDepartment() {
    this.department;
  }

  setDepartment(arg) {
    this.department = arg
  }
}

class Department {
  constructor(manager) { //manager: Person
    this.manager = manager;
  }

  getManager() {
    return this.manager;
  }
}
```

### After

```js
class Person {
  constructor() {
    this.department = new Department()
  }
  
  getManager() {
    return this.department.getManager();
  }

  setDepartment(arg) {
    this.department = arg
  }
}

class Department {
  constructor(manager) { //manager: Person
    this.manager = manager;
  }

  getManager() {
    return this.manager;
  }
}
```

### 6. 과잉 중개 메서드 제거(Remove Middle Man)

클래스에 자잘한 위임이 너무 많을 땐 대리 객체를 클라이언트가 직접 호출하게 한다.

### Before

```js
class Person {
  constructor() {
    this.department = new Department()
  }
  
  getManager() {
    return this.department.getManager();
  }

  setDepartment(arg) {
    this.department = arg
  }
}

class Department {
  constructor(manager) { //manager: Person
    this.manager = manager;
  }

  getManager() {
    return this.manager;
  }
}
```

### After

```js
class TelephoneNumber {
  getTelephoneNumber() {
    return `(${this.areaCode}) ${this.number}`;
  }

  getAreaCode() {
    return this.areaCode;
  }

  getNumber() {
    return this.number;
  }

  setAreaCode(arg) {
    this.areaCode = arg
  }

  setNumber(arg) {
    this.number = arg
  }
}

class Person {
  constructor() {
    this.officeTelephone = new Telephone();
  }

  getTelephoneNumber() {
    return this.officeTelephone.getTelephoneNumber();
  }

  getOfficeTelephone() {
    return this.officeTelephone;
  }
}
```

## 7. 외래 클래스에 메서드 추가(Introduce Foreign Method)

사용중인 서버 클래스에 메서드를 추가해야 하는데, 그 클래스를 수정할 수 없을 땐 클라이언트 클레스 안에 서버 클래스의 인스턴스를 첫 번째 인자로 받는 메서드를 작성한다. `Date` 객체와 같이 수정 불가능한 클래스에 적용 할 수 있다.

서버 클레스에 수많은 외래 메서드를 작성해야 하거나 하나 이상의 외래 메서드를 여러 클래스가 사용해야 할 때는 국소적 상속확장 클래스 사용 기법을실시한다.

### Before

```js
const date = new Date(prevEnd.getYear(), prevEnd.getMonth(), prevEnd.getDay() + 1);
```

### After

```js
const date = nextDay(prevEnd);

nextDay(arg) {
  return new Date(arg.getYear(), arg.getMonth(), arg.getDay() + 1);
}
```

## 8. 국소적 상속확장 클래스 사용

사용중인 서버 클래스에 여러 개의 메서드를추가해야 하는데 클래스를 수정할 수 없을 떈 새 클래스를 작성하고 그 안에 필요한 여러 개의 메서드를 작성한다. 이 상속확장 클래스를 원본 클래스의 하위클래스나 래퍼 클래스로 만든다.

### Before

```js
const date = new Date(prevEnd.getYear(), prevEnd.getMonth(), prevEnd.getDay() + 1);
```

### After

```js
class MfDateSub extends Date {
  nextDay() {
    return new Date(this.getYear(), this.getMonth(), this.getDay() + 1);
  }
}

or

class MfDateSub {
  constructor(dateString) {
    this.original = new Date(dateString);
  }

  getYear() {
    this.original.getYear();
  }

  getMonth() {
    this.original.getMonth();
  }

  getDay() {
    this.original.getDay();
  }

  nextDay() {
    return new Date(this.getYear(), this.getMonth(), this.getDay() + 1);
  }
}
```

## 관련 게시물
- [[Refactoring] 6장 메서드 정리 요약](/book/refactoring-06-composing-methods/)


[link-book]: https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=20793053