---
title: "[Refactoring] 8장 데이터 체계화"
date: 2019-05-04 18:00:00
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

const row = new Performance('armadillo', 15)
```

## 6. 관측 데이터 복제(Duplicate Observed Data)

도메인 메서드가 데이터에 접근해야 할 땐, 데이터를 도메인 객체로 복사하고, 양층의 데이터를 동기화하는 observer를 작성해서 비즈니스 로직 처리 코드와 사용자 인터페이스 처리 코드를 분리한다.

![관측 데이터 복제 다이어그램](/assets/images/refectoring-duplicate-observed-data.png)

## 7. 클래스의 단방향 연결을 양방향으로 전환(Change Unidirectional Association to Bidirectional)

두 클래스가 서로의 기능을 사용해야 하는데 한 방향으로만 연결되어 있을 땐, 역포인터를 추가하고 서로 업데이트 할 수 있게 접근 한정자(private, protected, public)를 수정한다. **이 리펙토링 기법은 그 방법이 까다로워 반드시 테스트를 거쳐야 한다.**

### Before

```js
class Order {
  constructor() {
    this._customer = null
  }

  get customer() {
    return this._customer
  }

  set customer(customer) {
    this._customer = customer
  }
}

// Order 클래스를 참조하는 코드가 존재하지 않음
class Customer {
  ...
}
```

### After

```js
class Order {
  ...
  get customer() {
    this._customer
  }

  set customer(customer) {
    if (this._customer !== null) {
      this._customer.friendOrders().delete(this)
    }

    this._customer = customer

    if (this._customer !== null) {
      this._customer.friendOrders().add(this)
    }
  }
}

//Customer 클래스에서도 Order 클래스를 참조
class Customer {
  constructor() {
    this._orders = new Set()
  }

  friendOrders() {
    return this._orders
  }

  addOrder(order){
    this._orders.add(order)
  }

  removeOrder(order) {
    this._orders.delete(order)
  }
}
```

## 8. 클래스의 양방향 연결을 단방향으로 전환(Change Bidirectional Association to Unidirectional)

두 클래스가 양방향으로 연결되어 있는데, 한 클래스가 다른 클래스의 기능을 더 이상 사용하지 않을 땐 불필요한 방향의 연결을 끊는다. 이를 통해 전체 복잡도와 결합도를 줄인다.

### Before

```js
// Customer 클래스가 존재해야지만 있을 수 있음
class Order {
  ...
  get customer() {
    return this._customer
  }

  set customer(customer) {
    if (this._customer !== null) {
      this._customer.friendOrders().delete(this)
    }

    this._customer = customer

    if (this._customer !== null) {
      this._customer.friendOrders().add(this)
    }
  }

  get discountPrice() {
    return this.getGrossPrice() * (1 - this._customer.getDiscount())
  }
}

class Customer {
  constructor() {
    this._orders = new Set()
  }

  getPriceFor(order) {
    return order.getDiscountPrice()
  }
  ...
}
```

### After

```js
// _this.customer 삭제
class Order {
  ...
  get discountPrice(customer) {
    return this.getGrossPrice() * (1 - customer.getDiscount())
  }
}

class Customer {
  ...
  getPriceFor(order) {
    return order.getDiscountPrice(this)
  }
  ...
}
```

## 9. 마법 숫자를 기호 상수로 전환(Replace Magic Number with Symbolic Constant)

특별한 의미를 가진 리터럴 숫자가 있을 땐, 의미를 살린 이름의 상수로 작성한다.

### Before

```js
function potentialEnergy(mass, height) {
  return mass * 9.81 * height
}
```

### After

```js
const GRAVITATIONAL_CONSTANT = 9.81

function potentialEnergy(mass, height) {
  return mass * GRAVITATIONAL_CONSTANT * height
}
```

## 10. 필드 캡슐화(Encapsulate Field)

데이터를 `public` 타입으로 만들면 데이터가 있는 객체가 모르는 사이에 다른 객체가 데이터 값을 읽고 변경할 수 있다. `public` 필드가 있을 땐 그 필드를 `private`로 전환하고 필드용 읽기, 쓰기 메서드를 작성한다.

### Before

```ts
//Typescript
public _name: string
```

### After

```ts
priviate _name:string
public get name(): string { return this._name }
public set name(value: string) { this._name = value }
```

## 11. 컬렉션 캡슐화(Encapsulate Collection)

메서드가 컬렉션을 반환할 땐 그 메서드가 읽기전용 뷰를 반환하게 수정하고 추가 메서드와 삭제 메서드를 작성한다.

### Before

```js
class Person {
  constructor () {
    this._courses = []
  }

  getCourses() {
    return this._courses
  }

  setCourses(courses) {
    this._courses = courses
  }
}
```

### After

```js
class Person {
  ...
  getCourses() {
    return JSON.parse(JSON.stringify(this._courses))
  }

  addCourse(course) {
    this._courses.push(course)
  }

  removeCourse(course) {
    this._courses.splice(this._courses.indexof(course), 1)
  }
}
```

## 12. 레코드를 데이터 클래스로 전환(Replace Record with Data Class)

레코드 구조를 이용한 인터페이스를 제공해야 할 땐 레코드 구조를 저장할 덤 데이터 객체를 작성하자.

## 13. 분류 부호를 클래스로 전환

기능에 영향을 미치는 숫자형 분류 부호가 든 클래스가 있을 땐 새 클래스로 바꾸자. 분류 부호를 사용하는 메서드들은 상징적인 이름이 아닌 숫자만을 인자로 받는다. 때문에 코드를 이해하기 힘들어지게 된다.

### Before

```js
// typescript
class Person {
  public static readonly O: number = 0
  public static readonly A: number = 1
  public static readonly B: number = 2
  public static readonly AB: number = 3

  private _bloodGroup: number

  constructor(bloodGroup: number) {
    this._bloodGroup = bloodGroup
  }
}
```

### After

```ts
class BloodGroup {
  public static readonly O: BloodGroup = new BloodGroup(0)
  public static readonly A:BloodGroup = new BloodGroup(1)
  public static readonly B:BloodGroup = new BloodGroup(2)
  public static readonly AB:BloodGroup = new BloodGroup(3)
  public static readonly _values:BloodGroup[] = [
    BloodGroup.O,
    BloodGroup.A,
    BloodGroup.B,
    BloodGroup.AB
  ]

  private _code:number

  constructor (code) {
    this._code = code
  }

  private getCode (): number {
    return this._code
  }

  private static code(index: number) {
    return this._values[index]
  }
}

class Person {
  ...
  constructor(bloodGroup: BloodGroup) {
    this._bloodGroup = bloodGroup
  }

  get bloodGroup() {
    return this._bloodGroup
  }

  set bloodGroup(bloodGroup: BloodGroup) {
    this._bloodGroup = bloodGroup
  }
}
```

## 14. 분류 부호를 하위 클래스로 전환(Replace Type Code with Subclasses)

클래스 기능에 영향을 주는 변경 불가 분류 부호가 있을 땐 분류 부호를 하위 클래스로 만든다. 기능에 영향을 주지 않을 경우에는 분류 부호를 클래스로 전환 기법을 실시한다.

### Before 

```ts
// typescript
class Employee {
  private _type:number
  static readonly ENGINEER = 0
  static readonly SALESMAN = 1
  static readonly MANAGER = 2

  consturcotr(type: number) {
    this._type = type
  }
  ...
}
```

### After 

```ts
class Employee {
  private _type:number
  static readonly ENGINEER = 0
  static readonly SALESMAN = 1
  static readonly MANAGER = 2

  private consturcotr(type: number) {
    this._type = type
  }

  create (type: number): Employee {
    switch(type) {
      case Employee.ENGINEER:
        return new Engineer()
      case Employee.SALESMAN:
        return new Salesman()
      case Employee.MANAGER:
        return new Manager()
      default:
        throw new Error('허용되지 않는 분류 부호')
    }
  }

  get type {
    return this._type
  }
  ...
}

class Engineer extends Employee {
get type(): number {
  return Employee.ENGINEER
}
}

class Salesman extends Employee {
get type(): number {
  return Employee.SALESMAN
}
}

class Manager extends Employee {
get type(): number {
  return Employee.MANAGER
}
}
```

## 15. 분류 부호를 상태/전략 패턴으로 전환(Replace Type Code with State/Strategy)

분류 분호가 클래스의 기능에 영향을 주지만, 하위 클래스로 전환할 수 없을 땐 그 분류 부호를 상태 객체로 만든다. **이 방법은 분류 부호가 객체 수명주기 동안 변할 때나 다른 이유로 하위 클래스를 만들 수 없을 때 사용한다.**

### Before

```ts
// typescript
class Employee {
  private _type:number
  static readonly ENGINEER = 0
  static readonly SALESMAN = 1
  static readonly MANAGER = 2

  consturcotr(type: number) {
    this._type = type
  }

  payAmount() {
    switch(this._type) {
      case Employee.ENGINEER:
        return this._monthlySalary
      case Employee.SALESMAN:
        return this._monthlySalary + this._commission
      case Employee.MANAGER:
        return this._monthlySalary + this._bonus
      default:
        throw new Error('허용되지 않는 분류 부호')
    }
  }
  ...
}
```

### After

```ts
class Employee {
  ...
  private _type: EmployeeType

  payAmount() {
    switch(this.type) {
      case EmployeeType.ENGINEER:
        return this._monthlySalary
      case EmployeeType.SALESMAN:
        return this._monthlySalary + this._commission
      case EmployeeType.MANAGER:
        return this._monthlySalary + this._bonus
      default:
        throw new Error('허용되지 않는 분류 부호')
    }
  }

  get type () {
    return this._type.getTypeCode()
  }

  set type(type) {
    this._type = EmployeeType.newType(type)
  }
}

class EmployeeType {
  static readonly ENGINEER = 0
  static readonly SALESMAN = 1
  static readonly MANAGER = 2

  newType(code: number) {
    switch(code) {
      case EmployeeType.ENGINEER:
        return new Engineer()
      case EmployeeType.SALESMAN:
        return new Salesman()
      case EmployeeType.MANAGER:
        return new Manager()
      default:
        throw new Error('허용되지 않는 분류 부호')
    }
  }
}

class Engineer extends Employee {
  get typeCode() {
    return EmployeeType.ENGINEER
  }
}

class Salesman extends Employee {
  get typeCode() {
    return EmployeeType.SALESMAN
  }
}

class Manager extends Employee {
  get typeCode() {
    return EmployeeType.MANAGER
  }
}
```

## 16. 하위클래스를 필드로 전환(Replace Subclasses with Fields)

여러 하위클래스가 상수 데이터를 반환하는 메서드만 다를 땐 각 하위클래스의 메서드를 상위 클래스 필드로 전환하고 하위 클래스는 전부 삭제한다.

### Before 

```ts
// typescript
abstract class Person {
  abstract isMale(): boolean
  abstract getCode(): string
}

class Male extends Person {
  isMale(): boolean {
    return true
  }

  getCode(): string {
    return 'M'
  }
}

class Famale extends Person {
  isMale(): boolean {
    return false
  }

  getCode(): string {
    return 'F'
  }
}
```

### After

```ts
class Person {
  constructor (isMale, code) {
    this._isMale = isMale
    this._code = code
  }

  createMale() {
    return new Person(true, 'M')
  }

  createFamle() {
    return new Person(false, 'F')
  }

  isMale() {
    return this._isMale
  }

  getCode() {
    return this._code
  }
}
```

## 관련 게시물

- [[Refactoring] 6장 메서드 정리 요약](/book/refactoring-06-composing-methods/)
- [[Refactoring] 7장 객체 간의 기능 이동 요약](/book/refactoring-07-moving-features-between-objects/)
- [[Refactoring] 8장 데이터 체계화](/book/refactoring-08-organizing-data/)

[link-book]: https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=20793053
