---
title: "[HTML] 좋은 사용자 경험(UX)을 제공하는 Form 만들기"
date: 2019-09-12 13:00:00
categories: html ux
tags: html ux form
description: "대부분 서비스는 사용자로부터 값을 입력받는 Form을 제공한다. 하지만 사용자는 서비스 이용을 위해 양식에 값을 입력하는 과정에서 상당한 번거로움을 느낀다. 따라서 서비스를 만드는 개발자는 최대한 좋은 사용자 경험을 제공해, 사용자의 번거로움을 덜어주어야 한다. 이번 글에서는 어떻게 하면 HTML로 좋은 사용자 경험을 제공하는 Form을 만들 수 있는지 알아보자."
---

## 들어가며

대부분 서비스는 사용자로부터 값을 입력받는 Form을 제공한다. 하지만 사용자는 서비스 이용을 위해 양식에 값을 입력하는 과정에서 상당한 번거로움을 느낀다. 따라서 서비스를 만드는 개발자는 최대한 좋은 사용자 경험을 제공해, 사용자의 번거로움을 덜어주어야 한다. 이번 글에서는 어떻게 하면 HTML로 좋은 사용자 경험을 제공하는 Form을 만들 수 있는지 알아보자.

## 알맞은 키보드 제공하기(모바일)

서비스가 요구하는 입력값과 모바일의 키보드 형태가 일치하지 않는 상황은 모바일 기기로 양식을 작성하는 과정에서 어렵지않게 마주할 수 있다.

아래와 같은 상황은 사용자를 손쉽게 곤경에 빠트릴 수 있다.  핸드폰 번호를 입력해야 하는 경우에 복잡하고 작은 쿼티 키보드에서 숫자만 찾아 입력해야하고, 이메일 주소를 입력해야 하는 경우에는 한영 전환과 특수문자 버튼을 눌러야만 작성이 가능한 번거로움이 잇따른다.

![핸드폰 번호와 이메일 주소를 입력하는 인풋이지만 쿼티 키보드가 보인다.](/assets/images/input-phone-and-email-with-qwerty-keyboard.jpeg)

*핸드폰 번호와 이메일 주소를 입력하는 인풋이지만 쿼티 키보드가 보인다.*

### `type` 속성 활용하기

`input` 태그는 이러한 불편함을 줄이기 위해 `type` 속성에 다양한 값을 지원하고 있다. `type`의 종류에 따라 가상 키보드의 형태도 달라진다. 위의 예제에서 핸드폰 번호와 이메일 주소 `input` `type`에 각각 `tel`, `email`을 부여하면 다음과 같이 알맞은 키보드가 보인다.

```html
<input type="tel" name="phone" />
<input type="email" name="email" />
```

![핸드폰 번호와 이메일 주소 인풋에 알맞은 키보드가 보인다.](/assets/images/input-phone-and-email-with-correct-keyboard.jpeg)

*핸드폰 번호와 이메일 주소 인풋에 알맞은 키보드가 보인다.*

다음은 `type` 속성에 정의할 수 있는 주요 값과 설명이다.

값 | 설명
--------|----------------------------------------------------------------------------
 text   | 일반적인 한 줄의 텍스트를 입력하기 위한 타입이다.
 number  | 숫자를 입력하기 위한 타입이다. 정수와 소수를 모두 입력할 수 있다.
 tel  | 전화번호를 입력하기 위한 타입이다.
 email  | 이메일을 입력하기 위한 타입이다. `@`와 `.com` 등의 도메인을 쉽게 입력할 수 있다.
 url  | URL을 입력하기 위한 타입이다.
 date  | 날짜를 입력하기 위한 타입이다. 날짜 입력을 위한 UI가 보여진다.

*아이폰의 경우 `type="number"`로 지정해도 쿼티 키보드가 출력된다. 이럴 때는 `pattern="\d*"`을 이용해 해결할 수 있다.*

### `inputmode` 속성 활용하기

`type` 속성을 통해 대부분의 문제를 해결할 수 있겠지만, 일부 케이스에서 동작하지 않는 경우가 있다. 금액을 입력하는 폼에서 천 단위로 `,`을 표기하고 싶을 때가 그렇다. `type="number"`일 경우 숫자와 소수점을 제외한 입력은 무시된다.

```html
<input type="number" value="123,1231" />
```

![,를 포함한 값을 억지로 주입했을 때 발생하는 경고](/assets/images/warning-input-type-number-with-comma.png)

*`,`를 포함한 값을 억지로 주입했을 때 발생하는 경고*

위의 경우, `inputmode` 속성을 활용할 수 있다. `inputmode`는 `input` 태그의 `type` 값과 무관하게 특정 키보드를 보여준다. 천 단위 `,` 처리는 Javascript를 이용해 처리하고, 키보드로는 숫자만 표기할 수 있게 된다.

```html
<input id="amount" type="text" inputmode="numeric" />
<script>
  document.getElementById('amount').addEventListener('input', (e) => {
    // 천단위 콤마 처리 함수
  })
</script>
```

![inputmode를 활용한 키보드](/assets/images/inputmode-numeric.gif)

다음은 `inputmode` 속성에 사용할 수 있는 주요 값과 설명이다.

값 | 설명
--------|----------------------------------------------------------------------------
text  | 일반적인 한 줄의 텍스트를 입력하기 위한 타입이다.
numeric | 숫자를 입력하기 위한 타입이다. 정수와 소수를 모두 입력할 수 있다.
decimal | 숫자를 입력하기 위한 타입이다. numeric과는 다르게 마이너스 기호는 입력할 수 없다.
tel | 전화번호를 입력하기 위한 타입이다.
email | 이메일을 입력하기 위한 타입이다. `@`와 `.com` 등의 도메인을 쉽게 입력할 수 있다.
url | URL을 입력하기 위한 타입이다. 

## 양식 자동 완성 기능 활용하기

브라우저는 다양한 방법의 추론으로 양식을 자동 완성되도록 돕는다. 이러한 방법의 하나로 `autocomplete` 속성이 있다. `autocomplete`는 `autocomplete` 속성값 또는 `name` 속성값에 특정 값이 들어가 있을 경우, 이전에 저장되어 있는 메타 데이터를 이용해 자동 완성을 돕는 힌트를 제공한다.

```html
<input name="email" autocomplete="email" />
```

![a만 입력했는데 저장된 이메일 주소를 힌트로 제공한다.](/assets/images/input-email-with-hint.png)

*a만 입력했는데 저장된 이메일 주소를 힌트로 제공한다.*

`autocomplete` 또는 자동 완성을 위한 `name`에 활용할 수 있는 값은 [WHATWG HTML 표준](https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#autofill){:target="_blank"}으로 정해져있다. 주요 값은 다음과 같다.

유형 | `name` | `autocomplete`
----|-----------------|-----------------
이름 | name | name, family-name, given-name
이메일 | email | email
전화번호 | phone, mobile | tel
주소 | address, city, region, postal | address-level1, address-level2, postal-code
username | username | username
비밀번호 | password | current-password, new-password

[구글이 제공하는 샘플](https://googlesamples.github.io/web-fundamentals/fundamentals/design-and-ux/input/forms/order.html){:target="_blank"}을 보면 이 기능이 얼마나 유용한지 알 수 있다.

## 자연스러운 submit 동작

대부분의 사람은 로그인 폼에서 아이디와 비밀번호를 입력하고 엔터를 치면 로그인이 될 것으로 생각한다. 이처럼 사람들은 자연스러운 submit 동작을 기대한다. 하지만 일부 서비스는 submit 버튼을 눌러야지만 동작을 수행하곤 한다.

사용자가 기대하는 submit 동작을 수행하기 위한 방법은 다음과 같다.

1. `form` 태그에 `submit` 이벤트를 등록한다.
2. 해당 `form` 태그 내부에 `type="submit"` 인 `input` 또는 `button` 태그를 만든다.

```html
<form id="form">
  <div>
    <label for="username">아이디</label>
    <input id="username" name="username" autocomplete="username" />
  </div>
  <div>
    <label for="password">비밀번호</label>
    <input id="password" name="password" type="password" autocomplete="current-password" />
  </div>
  <div>
    <button id="submit">로그인</button>
  </div>
</form>
<script>
  document.getElementById('form').addEventListener('submit', (e) => {
    e.preventDefault()
    alert('login')
  })
</script>
```

![입력 후 엔터를 치면 자연스럽게 서브밋 된다.](/assets/images/auto-submit-form.gif)

*입력 후 엔터를 치면 자연스럽게 서브밋 된다.*

이러한 동작이 누락된 폼은 대부분 `button` 태그에 `type="submit"`을 주지 않았거나 `form` 태그 자체를 사용하지 않아 발생한다. 이는 근래의 프론트앤드 개발이 SPA, 컴포넌트 기반 방식을 따르기 때문에 놓치게 되는게 아닐까 개인적으로 생각한다.

## 자동 대문자 활성화 방지(모바일)

모바일 기기로 로그인 폼을 작성하는 모습을 상상해보자. 아이디를 입력하기 위해 포커싱했을 때, 모바일 기기의 설정에 따라 대문자가 활성화된 것을 경험한 적이 있을 것이다. 대부분의 웹 서비스에서 아이디는 영어 소문자와 숫자의 조합으로 이루어져 있다. 따라서 활성화된 대문자를 풀고 아이디를 입력해야 하는 번거로움이 생긴다.

![처음 키보드가 활성화 되었을 때 대문자가 활성화 되어있다.](/assets/images/login-form-with-uppercase.gif)

*처음 키보드가 활성화 되었을 때 대문자가 활성화 되어있다.*

이러한 번거로움을 없애기 위해 `autocapitalize` 속성을 이용할 수 있다. `autocapitalize="off"`로 지정하면 모바일 기기의 설정에 상관없이 처음 키보드가 활성화되었을 때 소문자를 바로 입력할 수 있다.

```html
<input name="username" autocomplete="username" autocapitalize="off" />
```

## 마무리

지금까지 HTML을 이용해 좋은 사용자 경험을 제공하는 Form을 만드는 방법에 관해 이야기해 보았다. 어떻게 보면 사소한 것으로 생각할 수 있다. **하지만, 이런 사소한 기법들이 모여서 동작만 하는 서비스가 아닌, 잘 만들어진 서비스가 탄생한다고 생각한다.**

내가 소개한 방법 외에도 다양한 기법들이 존재한다. 이번 글을 계기로 흥미를 느낀 분들은 다른 방법들도 찾아보기를 권한다.
