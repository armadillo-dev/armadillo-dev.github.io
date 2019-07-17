---
title: "[edge case] 웹폰트 적용에서 최적화까지"
date: 2019-07-18 09:00:00
categories: html css
tags: html css webfont
description: "웹폰트는 사용자 로컬에 폰트가 저장되어있지 않더라도 특정 폰트를 렌더링 할 수 있도록 도와준다. 이는 사용자 경험을 향상시키기 위한 다양한 방법 중 하나로 사용되고 있다. 이번 글에서는 현재 진행 중인 프로젝트에 웹폰트를 적용하며 발생한 문제점과 그 해결 방법을 공유하고자 한다."
---

## 들어가며

웹폰트는 사용자 로컬에 폰트가 저장되어있지 않더라도 특정 폰트를 렌더링 할 수 있도록 도와준다. 이는 사용자 경험을 향상시키기 위한 다양한 방법 중 하나로 사용되고 있다. 이번 글에서는 현재 진행 중인 프로젝트에 웹폰트를 적용하며 발생한 문제점과 그 해결 방법을 공유하고자 한다.

## 문제점

진행 중인 프로젝트에서 발생한 문제점은 2가지였다.

### 웹폰트 깜빡임 현상

프로젝트에서는 아이콘을 표시하기 위해 [Material Icons](https://material.io/tools/icons/?style=baseline){:target="_blank"} 웹폰트를 사용하고 있었다. 그런데 웹폰트가 로드되기 전, 아이콘에 대응되는 영단어가 표시되었다가 로딩이 완료되면 아이콘으로 바뀌는 현상이 발생했다. 서비스에 다수의 아이콘이 존재했기에 이 깜빡임이 더욱 두드러졌고, 사용자 경험을 해치고 있었다.

![웹폰트 깜빡임 현상](/asserts/images/web-font-loading-non-preload-a79a55f4-88b5-451b-a8e3-7bd19ed064dd.gif)

웹 폰트가 적용되기 전, face라는 문구가 표시된 이후에 아이콘으로 변환된다.

### 웹폰트 용량

서비스에서 본문을 표현하기 위해 사용중인 NatoSans KR 폰트는 WOFF2 기준 968KB(Black, Bold, Medium, Regular, Light)의 용량을 사용하고 있었다. 1MB에 육박하는 용량은 웹 페이지를 무겁게 만든다. 때문에 이를 개선할 여지가 있는지 검토가 필요했다.

## 웹폰트 렌더링 과정

첫번째 문제인 깜빡임 현상은 웹폰트 렌더링 과정과 연관되어 있다. 아래 그림을 보자. Paint text가 끝난 시점(텍스트가 화면에 렌더링 된 시점)에 아직 웹폰트 로딩이 완료되지 않았을 경우, 브라우저에 따라 웹폰트 렌더링을 차단한다. 이후 로딩이 완료되면 웹 폰트를 적용해서 텍스트를 렌더링한다.

![웹폰트 렌더링 과정](/asserts/images/webfont-rendering-flow.png)

[웹폰트 렌더링 과정 - Google Web Fondamentals](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/webfont-optimization){:target="_blank"}

브라우저에서 웹폰트 렌더링을 차단하는 방법은 2가지로 나뉜다. IE는 FOUT(Flash of Unstyled Text) 방식을 따르고, 그 외 브라우저는 FOIT(Flash of Invisible Text) 방식을 따른다.

- **FOIT**: 웹폰트가 적용된 텍스트 영역을 여백으로 처리하고, 웹폰트 로딩이 완료되면 해당 폰트를 적용한다. 단, 3초의 제한 시간이 있어 로딩이 3초를 넘어가면 폴백 폰트로 렌더링 한다.
- **FOUT**: 폴백 폰트를 먼저 표시하고, 웹폰트 로딩이 완료되면 해당 폰트를 적용한다.

![FOIT 방식과 FOUT 방식 비교](/asserts/images/compare-foit-with-fout.gif)

[FOIT 방식과 FOUT 방식 비교 - 네이버 D2](https://d2.naver.com/helloworld/4969726){:target="_blank"}

프로젝트에서 깜빡임 현상이 발생하는 이유도 웹폰트가 로딩되기 전에 Paint text가 발생하고, 웹폰트 로딩이 완료되면 텍스트가 아이콘으로 치환되기 때문이다. 크롬 브라우저에서 `face`라는 텍스트가 노출되는 이유는 FOIT 방식을 따르지만 웹 폰트 로딩에 3초 이상의 시간이 소요되기 때문이다.

## Preload

문제를 해결하기 위해 `preload`를 이용하기로 했다. `preload`를 이용하면 웹폰트의 로딩 우선순위를 앞으로 끌어당길 수 있다. `preload`에 대한 설명은 [링크](https://medium.com/@koh.yesl/preload-prefetch-and-priorities-in-chrome-15d77326f646){:target="_blank"}를 참고하자.

먼저 preload를 적용하기 전 리소스 다운로드 순서를 보자. 아래 이미지에서 볼 수 있듯, CSS파일을 먼저 다운로드 한 뒤, 폰트 파일을 다운로드 한다.

![preload 적용 전 리소스 다운로드 순서](/asserts/images/webfont-network-without-preload.png)

`head` 태그 내부에 다음과 같이 선언해주고 다시 결과를 살펴보자.

```html
<link rel="preload" href="webfont-path" as="font" crossorigin />
```

이번에는 웹폰트 파일을 먼저 다운로드하기 시작한다. 이를 통해 깜빡임 현상이 발생하지 않도록 유도할 수 있다. 하지만, `preload`를 이용하더라도 폰트 파일의 로딩이 현저하게 느리면 동일한 현상이 발생하게 된다. 또한  IE는 `preload`를 지원하지 않는 문제점이 있다.

![preload 적용 후 리소스 다운로드 순서](/asserts/images/webfont-loading-with-preload.png)

## `font-display` 속성

`preload`가 웹폰트를 미리 다운로드한다면, `font-display`는 웹폰트 렌더링 차단 시점에 텍스트를 어떻게 보여줄 것인지 선택할 수 있다.

```css
font-display: auto | block | swap | fallback | optional;
```

- `auto`: 브라우저의 기본 동작을 따른다.
- `block`: FOIT 방식으로 렌더링 한다.
- `swap`: FOUT 방식으로 렌더링 한다.
- `fallback`: 100ms 동안은 텍스트를 렌더링 하지 않고, 그 후 폴백 폰트로 렌더링 한다. 그리고 약 2초 이내에 웹 폰트 다운로드가 완료되면 웹 폰트로 렌더링 한다. 하지만 2초 이내에 로딩이 완료되지 않으면 계속 폴백 함수로 렌더링 된다.
- `optional`: `fallback`과 유사하게 동작한다. 다만, 웹폰트 로딩이 완료되었을 때 네트워크 상태에 따라 웹폰트와 폴백 폰트 중 한 가지를 골라 렌더링 한다.

`font-display` 속성을 지정했지만, IE는 `font-display` 속성을 지원하지 않는다 :-(

## Web font loader

`preload`와 `font-display`를 모두 지원하지 않는 IE에 대응하기 위해 [Web font loader](https://github.com/typekit/webfontloader){:target="_blank"}를 사용하기로 했다. Web font loader를 이용하면 구글에서 제공하는 폰트를 손쉽게 로드하고 조작할 수 있다(구글이 지원하지 않는 웹폰트도 사용할 수 있다.).

간단한 사용법은 다음과 같다. 아래 코드만 작성하면 Material Icons 폰트를 로드할 수 있다.

```html
<script src="https://ajax.googleapis.com/ajax/libs/webfont/1.6.26/webfont.js"></script>
<script>
  WebFont.load({
    google: {
      families: ['Material Icons'] // Material Icons 폰트 로드
    }
  });
</script>
```

Web font loader는 지정한 폰트의 다운로드가 완료되면 `html` 태그에 `wf-active`라는 클래스를 추가한다. 이를 통해 웹 폰트 로딩 전/후를 구분하여 스타일을 적용할 수 있다.

```css
// 웹폰트 다운로드 전
.material-icons {
  visibility: hidden;
}

// 웹폰트 다운로드 완료!
.wf-active .material-icons {
  visibility: visible;
}
```

Web font loader까지 적용한 뒤, `face` 글자가 노출되지 않는 것을 확인할 수 있다. 예제는 크롬 브라우저이지만, IE도 지원된다.

![Web font loader적용 후 깜빡임 현상 사라짐](/asserts/images/web-font-loading-after-webfontloader.gif)

Web font loader적용 후, 문구가 노출되지 않고 바로 아이콘이 보여진다.

## 웹폰트 형식

깜빡임 현상을 해결한 뒤에는 웹 폰트의 용량을 줄이기 위한 방법을 찾아보았다.

웹 폰트는 TTF, OTF, EOT, WOFF, WOFF2까지 다양한 형식이 존재한다. 이 중에 WOFF2 형식이 가장 높은 압축률을 자랑한다. 때문에 WOFF2 형식을 기본으로 제공하고, 이를 지원하지 않는 브라우저를 위해 WOFF, TTF 등의 형식을 순차적으로 제공하는 것이 바람직하다.

![브라우저 별 웹폰트 지원 형식](/asserts/images/browser-support-webfont-type.png)

[브라우저 별 웹폰트 지원 형식 - w3schoools.com](https://www.w3schools.com/Css/css3_fonts.asp){:target="_blank"}

웹폰트를 정의할 때 어떤 타입으로 제공할 것인지 우선순위를 지정할 수 있다.

```css
@font-face {
  font-family: 'Noto Sans KR';
  font-style: normal;
  font-weight: 400;
  src: local('Noto Sans KR Regular'), // 사용자의 피씨에 해당 폰트가 있을 경우 로컬에 있는 폰트 사용
    url(woff2-font-path) format('woff2'), // woff2 형식 파일 먼저 로딩
    url(woff-font-path) format('woff'); // woff2 형식을 지원하지 않을 경우 woff 형식 파일 로딩
}
```

## 서브셋 폰트 사용

서브셋 폰트는 실제 사용할 글자만 남기고 불필요한 글자를 제거한 폰트 파일을 일컫는다.

한글은 모든 자음과 모음을 조합하면 총 11,172자로 구성되어 있다. 이는 26개의 알파벳으로 구성된 영문에 비해 훨씬 많은 조합이며, 이를 표현하기 위해서는 보다 큰 용량이 필요하다. 하지만 실제 서비스에서는 모든 조합이 필요하지 않다. `걅햟`과 같은 문자는 사용될 여지가 거의 없다. 때문에 2,350자로 구성된 서브셋 폰트를 제공해 폰트 파일의 용량을 줄이는 것이 바람직하다.

서브셋 폰트를 만드는 방법은 [이 글](http://indivdot.github.io/%EC%9B%B9/2016/04/02/webfont.html#section-4){:target="_blank"}을 참고하자.

Noto sans KR 파일과 서브셋 파일의 용량은 WOFF2 기준(Black, Bold, Medium, Regular, Light) 각각968KB,  664KB로 30%의 차이를 보인다.

## `unicode-range`  속성

`unicode-range` 속성을 이용하면 특정 문자열만 웹폰트로 지정할 수 있다. 웹 페이지에 지정된 범위의 문자열이 없으면 웹폰트를 다운로드 하지 않는다. 다국어를 지원하는 웹사이트일 경우 현재 선택된 언어에 따라 각각의 언어만 지원하는 웹폰트를 다운로드 할 수 있다.

```css
@font-face {
  font-family: 'foo font';
  font-weight: 400;
  src: local('foo font'),
    url(woff2-foo-font-ko-path) format('woff2'),
    url(woff-foo-font-ko-path) format('woff'),
  unicode-range: U+1100-U+11FF; //한글만 다운로드
}

@font-face {
  font-family: 'foo font';
  font-weight: 400;
  src: local('foo font'),
    url(woff2-foo-font-path) format('woff2'),
    url(woff-foo-font-path) format('woff'),
  unicode-range: U+000-5FF; //라틴어만 다운로드
}
```

[구글 폰트에서 제공하는 웹폰트](https://fonts.google.com/) CSS 파일을 보면, 아래와 같이 `uincode-range`가 쓰인 것을 확인할 수 있다. 이는 머신 러닝을 토대로 웹 페이지의 주제에 따라 사용되는 패턴을 발견하고, 그 패턴에 따라 한글을 나눠서 실제 사용되는 문자만 다운로드 할 수 있게 해준다.

```css
/* [0] */
@font-face {
  font-family: 'Noto Sans KR';
  font-style: normal;
  font-weight: 400;
  font-display: swap;
  src: local('Noto Sans KR Regular'), local('NotoSansKR-Regular'), url(https://fonts.gstatic.com/s/notosanskr/v11/PbykFmXiEBPT4ITbgNA5Cgm203Tq4JJWq209pU0DPdWuqxJFA4GNDCBYtw.0.woff2) format('woff2');
  unicode-range: U+f9ca-fa0b, U+ff03-ff05, U+ff07, U+ff0a-ff0b, U+ff0d-ff19, U+ff1b, U+ff1d, U+ff20-ff5b, U+ff5d, U+ffe0-ffe3, U+ffe5-ffe6;
}

/* [2] */
@font-face {
  font-family: 'Noto Sans KR';
  font-style: normal;
  font-weight: 400;
  font-display: swap;
  src: local('Noto Sans KR Regular'), local('NotoSansKR-Regular'), url(https://fonts.gstatic.com/s/notosanskr/v11/PbykFmXiEBPT4ITbgNA5Cgm203Tq4JJWq209pU0DPdWuqxJFA4GNDCBYtw.2.woff2) format('woff2');
  unicode-range: U+d723-d728, U+d72a-d733, U+d735-d748, U+d74a-d74f, U+d752-d753, U+d755-d757, U+d75a-d75f, U+d762-d764, U+d766-d768, U+d76a-d76b, U+d76d-d76f, U+d771-d787, U+d789-d78b, U+d78d-d78f, U+d791-d797, U+d79a, U+d79c, U+d79e-d7a3, U+f900-f909, U+f90b-f92e;
}
```

덕분에 웹폰트 전체 용량보다 작은 리소스만 다운로드 하게 된다. 아래는 Noto Sans KR 파일을 여러개로 분리해 다운로드 받은 모습이다.

![구글 웹폰트 로딩 화면, 한 개의 폰트가 여러개로 분리되어 있다.](/asserts/images/webfont-loading-unicode-range.png)

## 마무리

아이콘 깜빡임 현상으로 시작된 공부가 꽤나 길어졌다. 그만큼 웹폰트를 최적화할 수 있는 다양한 방법이 존재한다. 신규 프로젝트를 진행할 때에도 앞서 언급했던 기법들을 적용해 사용자 경험을 향상시킬 수 있을 것 같다.

## 참고 링크

- [웹 폰트 사용과 최적화의 최근 동향 - 네이버 D2](https://d2.naver.com/helloworld/4969726){:target="_blank"}
- [웹폰트 최적화 - Google Web Fundamentals](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/webfont-optimization?hl=ko){:target="_blank"}
- [웹폰트 경량화](http://indivdot.github.io/%EC%9B%B9/2016/04/02/webfont.html){:target="_blank"}
