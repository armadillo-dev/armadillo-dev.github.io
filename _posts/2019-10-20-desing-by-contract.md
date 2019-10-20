---
title: "계약에 의한 설계(Design by Contract)로 견고한 서비스 만들기"
date: 2019-10-20 11:00:00
categories: design programming
tags: design contract programming
description: "개발자는 서비스 신뢰도를 높이기 위해(이외의 이유도 존재한다.) 다양한 테스트 케이스를 작성하고 검증한다. 그러나, 100개의 테스트 케이스를 작성해도 101번째 경우의 수에서 에러가 발생하는 것이 부지기수다. 어떻게하면 위와 같은 상황이 발생하지 않도록 견고한 서비스를 만들 수 있을까? 오늘은 그 해법중 하나인 계약에 의한 설계에 대해 이야기해보려고 한다."
---

{% include figure image_path="/assets/images/img-contract.jpg" alt="계약(출처: 픽사베이)" caption="계약(출처: 픽사베이)" %}

> 본 게시물은 [오브젝트 도서](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=193681076){:target="blank"}와 [코드스피츠 강의](https://www.youtube.com/playlist?list=PLBNdLLaRx_rLOQsMCWxvawSUVI6cXoMsk){:target="blank"}를 통해 배운 내용을 정리한 게시물입니다.

개발자는 서비스 신뢰도를 높이기 위해(이외의 이유도 존재한다.) 다양한 테스트 케이스를 작성하고 검증한다. 그러나, 100개의 테스트 케이스를 작성해도 101번째 경우의 수에서 에러가 발생하는 것이 부지기수다.

어떻게하면 위와 같은 상황이 발생하지 않도록 견고한 서비스를 만들 수 있을까? 오늘은 그 해법중 하나인 계약에 의한 설계에 대해 이야기해보려고 한다.

## 계약의 의미

먼저 일상 생활에서 쓰이는 계약에 대해 알아보자. 표준국어대사전에서는 **관련되는 사람이나 조직체 사이에서 서로 지켜야 할 의무에 대하여 글이나 말로 정하여 둠. 또는 그런 약속**이라고 정의하고 있다. 계약은 다음과 같은 특징이 있다.

- 각 계약 당사자는 계약으로부터 **이익**을 기대하고, 이익을 얻기 위해 **의무**를 이행해야 한다.
- 각 계약 당사자의 이익과 의무는 계약서에 **문서화** 된다.

이익을 제공하는 쪽은 상대방이 어떤 식으로 의무를 다했는지에 대해서는 관심이 없다. 그저 의무를 다 했을 때 이익을 제공하면 그만이다. 반대로, 의무를 이행하는 쪽은 약속된 이익이 보장되지 않으면 의무를 다할 이유가 없다.

이처럼 계약은 당사자 간 협력을 명확하게 정의하고 커뮤니케이션할 수 있도록 돕는다. 이러한 아이디어는 프로그래밍 세계에도 적용할 수 있다.

## 프로그래밍에서의 계약

앞서 정의한 계약을 프로그래밍에 적용하면 **협력에 참여하는 두 객체 사이의 의무와 이익을 문서화한 것**이라고 정의할 수 있겠다.

- 협력에 참여하는 각 객체는 계약으로부터 이익을 기대하고 **이익**을 얻기위해 **의무**를 이행한다.
- 협력에 참여하는 각 객체의 이익과 의무는 객체의 인터페이스 상에 **문서화** 된다.

일반적으로 우리는 인터페이스를 통해 협력에 필요한 정보들을 제공한다. 다음의 코드를 보면 `someMethod`의 세부 내용을 보지 않더라도 1개의 `number` 타입 변수를 입력 받고 `SomeClass` 인스턴스를 반환하며,  이 메서드를 외부에서도 호출할 수 있다는 사실을 알 수 있다.

```typescript
//typescript로 작성된 메서드
public(가시성) someMethod(메서드 이름)(number num(파라미터)): SomeClass(반환형) {
  ...
}
```

계약에 의한 설계는 여기서 한 발 더 나아가 보다 상세한 제약 조건을 명시한다.

## 사전 조건(precondition)

사전 조건은 메서드가 실행되기 전 보장되어야 할 조건이다. 때문에 사전 조건을 만족시킬 책임은 클라이언트에게 있다. 사전 조건이 충족되지 않을 경우 메서드는 실행을 멈추고 이 사실을 클라이언트에게 알린다.

이해를 돋기 위해 다음 코드를 보자.

```typescript
function sum(num1: number, num2: number): number {
  // num1에 대한 검증
  if (typeof num1 !== 'number' || Number.isNaN(num1)) {
    throw new Error('invalid num1')
  }

  // num2에 대한 검증
  if (typeof num2 !== 'number' || Number.isNaN(num2)) {
    throw new Error('invalid num2')
  }

  return num1 + num2
}

sum(1, 'aa'); // error - invalid num2
sum(NaN, 1); // error - invalid num1
sun(1, 2) // 3
```

`sum` 함수는 2 개의 숫자를 받아 합을 반환하는 기능을 수행한다. 실제 로직은 `num1 + num2` 부분으로 매우 간단하다. 하지만 인자로 받는 `num1`, `num2`에 대한 검증을 통과하지 못하면 실제 로직은 수행되지 않는다.

또한, `num1`과 `num2`는 `sum`함수를 사용하는 측에서 주입시키는 값이다. 사전 조건을 만족시킬 책임이 클라이언트에게 있다는 것은 해당 함수를 호출하는 쪽에서 모든 사전 조건을 통과할 수 있도록 인자를 넘겨야 함을 뜻한다.

사전 조건의 장점은 로직을 수행하기 전 검증을 완료하기 때문에, 실제 로직에서는 whitelist인 값만 고려하면 된다는 점이다.

## 사후 조건(postcondition)

사전 조건은 메서드의 실행 이후 메서드가 보장해야 할 조건이다. 사전 조건과는 반대로 사후 조건을 만족시킬 책임은 서버에게 있다. 클라이언트가 사전 조건을 모두 만족시켰는데도 사후 조건을 만족시키지 못한다면, 서버는 이를 클라이언트에게 알려야 한다.

다음 코드를 보자.

```typescript
function divide(num1: number, num2: number) {
  const result = num1 / num2
  
  // result에 대한 검증
  if (result > num1) {
    throw new Error('invalid result')
  }
  
  return result
}

divide(1, 0.5) // error - invalid result
divide(10, 2) // 5
```

`divide` 함수는 두개의 숫자를 받아 나누기를 수행한다. `result`는 나눗셈이 수행된 결과를 가지는 변수다. 클라이언트로 반환하기 전에 기존 숫자가 결과값보다 크면 에러를 발생시킨다. 이처럼 사후 조건은 조건이 로직 수행 이후 충족되지 않으면 클라이언트에게 이 사실을 알린다. 

사후 조건의 장점은 클라이언트가 값을 리턴 받는데 성공하면, 이 값을 신뢰할 수 있다는 것이다. 서버의 내부 로직은 신경쓰지 않아도 된다.

## 불변식(Invariant)

불변식은 객체에서 항상 참이라고 가정되는 조건이다. 메서드 실행 도중에는 불변식을 불만족시킬 수도 있으나, 실행 전/후에는 항상 불변식을 만족시켜야 한다. 불변식은 스스로의 상태를 검증하므로 사전 조건보다 먼저 검증되어야 한다.

Google Analytics와 같은 로거를 주입받아 로깅 작업을 하는 클래스가 있다고 가정해보자.

```typescript
class Logger {
  private logger

  initialize(logger) {
    this.logger = logger // GA, pixel ...
  }

  send() {
    if (!this.logger) {
        throw new Error('logger is undefined')
    }

    if (!logger.isConnected()) {
      throw new Error('logger is not connected')
    }

    ...
  }
}
```

로그를 쌓는 `send` 메서드는 실행 전에 `logger`를 주입받지 않으면 에러를 발생시킨다. 또한, `logger`를 주입했더라도, 네트워크 상태가 연결되어 있지 않다면 로직을 수행하지 않는다. 이처럼 불변식은 클라이언트에게 전달받는 메시지(인자)와 상관 없이 객체 자신의 상태를 검증한다.

## 마무리

사전 조건, 사후 조건, 불변식은 계약에 의한 설계의 전부가 아니다. 필자의 실력이 전체 내용을 담지 못해 이정도 선에서 글을 마무리하려고 한다. 언제가 될지는 모르겠지만 조금 더 자세한 설명을 할 수 있게 되면 글을 업데이트 해야겠다.

이 글을 통해 궁금증이 생겼다면, [무료로 공개된 강의](https://youtu.be/U1vySD_wG78){:target="blank"}나 [도서](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=193681076){:target="blank"}를 구입해서 정독해보기를 바란다.
