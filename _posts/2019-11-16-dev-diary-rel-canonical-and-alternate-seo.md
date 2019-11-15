---
title: '[개발 일기 #2] rel="canonical"과 rel="alternate"을 이용한 SEO 최적화'
date: 2019-11-16 00:00
categories: dev-diary seo
tags: dev-diagry seo rel-canonical
description: "데스크톱 페이지에는 rel=alternate와 media 속성을 이용해 모바일 페이지의 URL을 기록하고, 모바일 페이지에서는 rel=canonical을 이용해 데스크탑 페이지의 URL과 연결시켜주면 된다."
---

> 실무에서 경험한 삽질기를 다루기 위한 개발 일기입니다. 각 잡고 포스팅하기엔 너무 가볍고, 기록하지 않으면 잊어버릴 것 같은 내용을 다룹니다.

문제의 발단은 이러했다. 최근 진행한 프로젝트는 데스크탑과 모바일 사이트가 별도 URL로 분리되어 있다.  데스크톱 사이트가 `https://some-site.com`라면, 모바일 사이트는 `https://m.some-site.com`같은 식이었다. 모바일 사이트를 막 오픈 한 시점에 PO(Product Owner) 분이 다음 문의를 하셨다.

> 모바일 기기에서 검색하면 모바일 사이트가 노출되고, 데스크탑으로 검색했을 때는 데스트탑 사이트가 노출되게 할 수 있을까요?

질문을 받은 순간 내 표정은 딱 이랬다. 😳 애초에 이런 요구사항 자체를 생각해본 적이 없어서 당황스러웠다. 이런 요구사항을 생각해본 적이 없었고, 이게 되나? 라는 생각이 들었다.

## HTTP redirect

해결법으로 가장 먼저 떠오른 방법은 접속한 사용자의 `user-agent` 값을 기반으로 모바일/데스크탑 사용자를 구분해 모바일 사용자는 모바일 사이트로, 데스크탑 사용자는 데스크탑 사이트로 HTTP redirect 시키는 것이었다.

하지만, 이 방법은 도메인이 다른 두 사이트 간 쿠키 공유는 불가능하다는 문제가 있었다. 가령 데스크탑 사이트에 로그인 된 상태로 모바일 기기에서 데스크탑 사이트에 접근하면 모바일 사이트에서는 쿠키로 저장된 토큰이 존재하지 않아 로그인이 풀리고, 이로 인해 접근 불가능한 경우가 생길 수 있다.

그런 사유로 다른 방법을 찾아보기로 했다.

## `rel="canonical"`과 `rel="alternate"`

그 다음 찾은 방법이 바로 `rel="canonical"`과 `rel="alternate"`를 이용하는 것이었다.

### `rel="canonical"`

`rel="canonical"`은 표준 페이지를 설정하는 역할을 한다. 표준 페이지란 같은 내용을 표기하는 다수의 URL이 존재할 때 대표되는 URL을 뜻한다. 다음과 같은 블로그 URL이 있다고 가정해보자.

```html
https://blog.com/articles/title-of-article
https://blog.com/articles/title-of-article?page=1
https://blog.com/aritlces?id=1
https://blog.com?type=article&id=1
```

위 4개의 URL은 모두 같은 내용을 의미하지만 서로 다른 URL을 가지고 있다. 이 때, 해당 페이지에 다음 태그를 추가해 표준 페이지를 설정할 수 있다.

```html
<!-- https://blog.com/articles/title-of-article을 표준 페이지로 지정 -->
<link rel="canonical" href="https://blog.com/articles/title-of-article" />
```

표준 페이지를 설정했을 때 얻는 장점은 검색엔진이 중복되는 URL에 접근했을 때, 표준 페이지를 선택해서 크롤링 한다. 때문에 표준 페이지로 지정된 URL이 검색 결과에 노출될 확률이 높아진다.

### `rel="alternate"`

`rel="canonical"`가 표준 페이지를 정의한다면, `rel="alternate"`는 다국어로 정의된 페이지/보조 페이지를 정의한다. 다국어 처리가 된 웹사이트가 존재한다고 가정해보자.

각각의 언어는 대략 다음과 같은 형태의 URL을 가질 것이다.

```html
https://blog.com/ <-- 한국어(기본)
https://blog.com/en
https://blog.com/de
```

이 때 각 페이지의 `header` 태그에 다음과 같이 `rel="alternate"` 태그를 추가할 수 있다. 이렇게 하면 각 언어 별 페이지를 설정해 두면 검색엔진은 사용자의 언어와 일치하는 페이지를 검색 결과에 노출시켜준다.

```html
<link rel="alternate" hreflang="ko" href="https://blog.com/" />
<link rel="alternate" hreflang="en" href="https://blog.com/en" />
<link rel="alternate" hreflang="de" href="https://blog.com/de" />
```

## 그래서?

위에서 설명한 `rel="canonical"`과 `rel="alternate"`를 이용하면 데스크톱과 모바일 사이트를 분리해낼 수 있다. 사실 `rel="alternate"`에는 추가로 설정할 수 있는 값이 있는데, 바로 `media` 속성이다. `media` 속성은 CSS의 media query와 동일한 값을 입력할 수 있다.

```html
<link rel="alternate" href="https://blog.com/" media="only screen and (max-width: 640px" />
```

검색엔진은 `media` 속성의 조건을 만족할 경우 보조 페이지를 결과에 노출시킨다. 때문에 데스크톱 페이지에는 `rel="alternate"`와 media 속성을 이용해 모바일 페이지의 URL을 기록하고, 모바일 페이지에서는 `rel="canonical"`을 이용해 데스크탑 페이지의 URL과 연결시켜주면 된다.

```html
<!-- 데스크탑 페이지 -->
<link rel="alternate" href="https://m.blog.com/" media="only screen and (max-width: 640px" />

<!-- 모바일 페이지 -->
<link rel="canonical" href="https://blog.com" />
```

## 참고 자료

- [별도 URL / Google 검색](https://developers.google.com/search/mobile-sites/mobile-seo/separate-urls){:target="_blank"}
- [중복 URL 통합 / Search Console 고객센터](https://support.google.com/webmasters/answer/139066?hl=ko){:target="_blank"}
- [Google에 페이지의 현지화된 버전 알리기 / Search Console 고객센터](https://support.google.com/webmasters/answer/189077?hl=ko){:target="_blank"}
- [표준 페이지 설정, link rel=canonical / 검색엔진 최적화](http://www.seo-korea.com/%ED%91%9C%EC%A4%80-%ED%8E%98%EC%9D%B4%EC%A7%80-%EC%84%A4%EC%A0%95-link-rel-canonical/){:target="_blank"}