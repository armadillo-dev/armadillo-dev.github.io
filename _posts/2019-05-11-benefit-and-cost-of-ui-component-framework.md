---
title: "UI Component Framework의 장단점(feat. Vuetify)"
date: 2019-05-11 18:00:00
categories: UI/UX
tags: ui ux component vuetify
description: "회사에서 진행중인 프로젝트에 Vuetify를 도입하면서 개인적으로 느낀 UI Component Framework의 장단점을 정리한다."
---

![Vuetify](/asserts/images/img-vuetify-a43a19c3-79e3-4284-b311-0a085d9505ab.png)

> 회사에서 진행중인 프로젝트에 Vuetify를 도입하면서 개인적으로 느낀 UI Component Framework의 장단점을 정리한다.

## UI Component Framework란?

UI Component Framework는 버튼, 폼 양식, 다이얼로그 등과 같이 자주 사용되는 UI를 컴포넌트 단위로 미리 구현해서 제공하는 프레임워크이다. 사용자는 프레임워크에서 제공하는 컴포넌트를 기반으로 UI를 구성 할 수 있다.

예시로는 [Vuetify](https://vuetifyjs.com/ko/){:target="_blank"}, [Bootstrap](https://getbootstrap.com/){:target="_blank"}, [Material UI](https://material-ui.com/){:target="_blank"} 등이 있다.

## UI Component Framework의 장점

### 1. 생산성

생산성은 UI Component Framework의 최대 장점이 아닐까 생각한다. 단 몇 줄의 코드만 작성하면 멋진 UI의 컴포넌트들을 사용 할 수 있다. 사용자는 HTML, CSS에 대한 깊은 이해가 없어도 빠르게 화면을 구성 할 수 있다.

<iframe height="265" style="width: 100%;" scrolling="no" title="Vuetify example" src="//codepen.io/armadillo-dev167/embed/Mdaxpg/?height=265&theme-id=0&default-tab=html,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
</iframe>
*Vuetify를 이용해 구현한 간단한 예제*

또한, 대부분의 UI Component Framework에서는 그리드 시스템을 제공하고 있다. 이를 이용하면 레이아웃을 쉽게 구성 할 수 있으며, 모바일/태블릿 환경에 대한 대응도 수월하다.

그리고 프론트엔드 개발자라면 필할 수 없는 브라우저 이슈를 해결해준다. 현재 Vuetify의 경우 IE11까지 지원하고 있다. 작성일(2019년 5월) 기준 **한국의 IE 10 이하 점유율은 1%내외**이다. 전세계를 기준으로 할 경우 그 수치는 더욱 낮아진다. 때문에 대부분의 경우 브라우저 관련 이슈는 발생하지 않는다고 봐도 무방하다.

![Vuetify에서 지원하는 브라우저](/asserts/images/img-vuetify-browser-support-af70c280-7c11-47ee-b734-5f9e7614cace.png)
*Vuetify에서 지원하는 브라우저*

![statcounter의 브라우저 점유율 자료](/asserts/images/bwoser-market-share-in-south-korea-c7d9dbfd-07bb-4615-a58e-f52773cf7630.png)
[*statcounter의 브라우저 점유율 자료*](http://gs.statcounter.com/browser-version-market-share/all/south-korea/#monthly-201804-201904){:target="_blank"}

### 2. 심미성

나를 포함한 대부분의 개발자는 직접 하는 디자인에 어려움을 겪는다. 디자이너가 아니기 때문에 당연한 결과이다. UI Component Framework는 일정 수준 이상의 UI/UX를 제공한다( Vuetify의 경우 Metarial Design에 영향을 받아 제공되는 모든 컴포넌트가 Metarial Design 스펙을 따르고 있다). 스타일과 테마 가이드 라인만 잘 따라도 충분히 멋진 화면을 구성 할 수 있다. [Vuetify를 이용한 샘플 사이트](https://demos.creative-tim.com/vuetify-material-dashboard/#/dashboard?ref=vuetifyjs.com){:target="_blank"}

또한, 버튼 클릭 시 파장이 퍼지는 듯한 효과(ripple)와 같이 다양한 마이크로 애니메이션/트랜지션 효과가 내장되어 있다. 해당 효과들은 속성 값만 전달하면 완성된다. 때문에 사용자는 손쉽게 다양한 효과로 사용자 경험을 증대 시킬 수 있다. 보다 다양한 효과는 [여기서](https://vuetifyjs.com/ko/framework/transitions){:target="_blank"} 확인 할 수 있다.

## UI Component Framework의 단점

### 1. 커스터마이징

아무리 자유도가 높다고 해도 직접 만드는 것 보다는 떨어질 수밖에 없다. 이는 UI Component Framework 고유의 UI를 유지하기 위해 수반되는 어쩔 수 없는 사항이다. 예를 들어, 앞서 말했듯이 Vuetify는 Meterial Design 스펙을 따르고 있다. 때문에 이에 반하는(?) UI를 구성하려고 하면 커스터마이징에 어려움을 겪을 수 있다.

내가 겪었던 사례는 다음과 같다.

1. `v-select`의 높이값 조정: `v-select`에는 `height`속성이 있어 10으로 지정했으나 높이가 변하지 않았다. 확인 결과 `v-select` 내부에 `min-height` 속성이 들어가 있어 일정 수준 이하로 높이 지정이 불가능하다.
2. input 계열의 컴포넌트(`v-text-field`, `v-checkbox`, `v-radio` 등)는 기본적으로 상하 `margin`, `padding` 값이 존재한다. 이는 레이블과 디테일 메시지를 표시하기 위한 여백인데, `hide-details` 속성을 이용해 아래 여백은 지울 수 있으나, 윗 여백은 직접 css를 작성해 지울 수밖에 없다.

<iframe height="265" style="width: 100%;" scrolling="no" title="Vuetify customizing" src="//codepen.io/armadillo-dev167/embed/RmrmLP/?height=265&theme-id=0&default-tab=html,result" frameborder="no" allowtransparency="true" allowfullscreen="true"></iframe>

### 2. 예상과는 다른 동작

UI Component Framework가 본인의 생각과 다르게 동작하는 경우가 종종 발생할 수 있다. 물론 정리된 문서를 살펴보면 대부분의 문제는 해결된다. 하지만, 인간은 습관의 동물이기에 모든 문서를 살펴보기 보다는 익숙한 형태로 코드를 작성하기 쉽다. 때문에 이러한 동작은 예상치 못한 버그의 원인이 된다.

내가 겪었던 사례는 다음과 같다.

1. `v-select`의 `items` 속성값으로 정수형 배열을 넘긴뒤, `v-select`의 옵션을 엔터키로 조작하려고 하면 정상 동작하지 않는다. 내부적으로 넘겨진 값에 `toLowerCase`를 실행하기 때문이다.
2. `v-checkbox`에는 `checked` 속성이 존재하지 않는다.

### 3. DOM 노드의 증가

DOM 노드의 증가는 네트워크, 런타임, 메모리 방면의 성능 이슈를 야기 시킬 수 있다([관련 글](https://developers.google.com/web/tools/lighthouse/audits/dom-size){:target="_blank"}). 때문에 불필요한 DOM 노드는 최대한 제거해야 한다. 하지만 UI Component Framework를 사용하면 의도치 않게 DOM 노드가 증가한다. 내장되어 있는 다양한 옵션에 대응하기 위해서, 또는 컴포넌트 자체의 UI를 위해서 노드들이 발생하기 때문이다.

<iframe height="265" style="width: 100%;" scrolling="no" title="v-select vs select" src="//codepen.io/armadillo-dev167/embed/xNObbE/?height=265&theme-id=0&default-tab=html,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
</iframe>

*`v-select`와 `select` 비교*

아래는 `v-select`를 표현하기 위해 필요한 DOM 구조이다. `option` 영역이 제외 되었는대도 10개 이상의 DOM노드가 사용되고 있다.

![v-select에 필요한 DOM 노드들](/asserts/images/v-select-dom-tree-3320bfed-77fa-4c6f-87bc-8ef108c05e1c.png)
*`v-select`에 필요한 DOM 노드들*

## 마무리

UI Component Framework는 우리에게 높은 생산성을 제공해준다. 덕분에 빠른 개발 속도와 아름다운 UI/UX를 얻을 수 있다. 하지만 커스터마이징의 어려움과 본인과 다른 설계자의 생각으로 의도하지 않은 동작을 할 때도 있다. 장점은 취하고, 단점은 최대한 영향을 받지 않도록 적용/사용 해야 할 것 같다.

그런 의미에서 개인적으로는 커스터마이징이 별로 필요하지 않은 서비스(사내 인트라넷, 관리자 페이지 등)에 적용했을 때 가장 큰 효과가 발휘 될 것이라고 생각하다.