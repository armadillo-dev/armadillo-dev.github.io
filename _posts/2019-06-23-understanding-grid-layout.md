---
title: "[CSS] Grid layout 이해하기"
date: 2019-06-23 18:00:00
categories: css
tags: html css layout grid
description: "Grid layout은 Flexbox와 마찬가지로, 레이아웃을 보다 수월하게 구성할 수 있도록 도와준다. 특히 열과 행으로 구성된 레이아웃에 탁월함을 보인다. 이 글에서는 Grid layout에 대해 설명하고, 예제를 통해 어떻게 활용할 수 있는지 알아보자."
---

Grid layout은 [지난 글](https://armadillo-dev.github.io/css/what-is-flexbox/){:target="_blank"}에서 다뤘던 Flexbox와 마찬가지로, 레이아웃을 보다 수월하게 구성할 수 있도록 도와준다. 특히 열과 행으로 구성된 레이아웃에 탁월함을 보인다. 이 글에서는 Grid layout에 대해 설명하고, 예제를 통해 어떻게 활용할 수 있는지 알아보자.

## Grid layout 구성요소

Grid layout은 행과 열로 구성된다. 표 모양을 떠올리면 쉽게 이해할 수 있다. Grid layout에는 Grid container, Grid item, Grid track, Grid line, Grid cell, Grid area와 같은 구성요소가 존재한다. 각각의 요소를 그림으로 표현하면 다음과 같다.

![그리드 레이아웃 구성요소](/asserts/images/grid-elemtns.png)

- **Grid container:** 전체 Grid layout을 감싸는 역할을 수행한다. `display: grid | inline-grid` 속성을 통해 지정할 수 있다.
- **Grid item:** Grid container에 속해있는 하위 DOM 요소를 뜻한다.
- **Grid track:** Grid layout에 존재하는 **행 또는 열**을 의미한다. Grid track의 개수는 명시적으로 지정할 수도, 암묵적으로 늘어나게 할 수도 있다.
- **Grid line:** Grid track을 구분하는 선을 의미한다. 선의 번호는 위에서 아래로(↓), 왼쪽에서 오른쪽으로(→) 메겨진다.  그리드 라인의 번호는 **1부터 시작**된다는 점을 주의하자.
- **Grid cell:** 그리드 레이아웃에서 가장 작은 단위 요소이며, 테이블의 셀과 유사하다.
- **Grid area:** 다수의 Grid cell로 이루어진 영역을 뜻한다. Grid area는 항상 사각형의 모양을 가져야한다(ㄴ자, ㄱ자 형태 불가능).

## Grid layout 주요 CSS 속성

Grid layout은 Grid container와 Grid item에 부여할 수 있는 속성이 구분되어 있다.

- Grid layout: `grid-template-columns`, `grid-template-rows`, `grid-template-areas`, `grid-gap` 등
- Grid item: `gird-column`, `grid-row`, `grid-area` 등

### grid-template-columns

```css
grid-template-columns: none | <track-list> | <auto-track-list>
```

`grid-template-columns`를 이용해 열의 개수 및 크기를 지정할 수 있다. 예를들어 `100px`로 이루어진 3개의 열을 구성하고 싶을 경우 `grid-template-columns: 100px 100px 100px`로 값을 지정하면 된다. 이를 보다 간결하게 `grid-template-columns: repeat(3, 100px)`  형태로 작성해도 동일한 결과를 얻는다.

<iframe height="550" style="width: 100%;" scrolling="no" title="Gridlayout grid-template-columns" src="//codepen.io/armadillo-dev167/embed/PrWbXz/?height=265&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/armadillo-dev167/pen/PrWbXz/'>Gridlayout grid-template-columns</a> by armadillo
  (<a href='https://codepen.io/armadillo-dev167'>@armadillo-dev167</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

위의 예제를 보면 `fr`이라는 새로운 단위가 등장한다. `fr`은 그리드 트랙에 남아있는 여백의 크기를 `fr`앞에 기입된 숫자의 비율대로 나누어, 각 열 또는 행에 할당한다. 이를 이용해 반응형 레이아웃을 손쉽게 구현할 수 있다.

### grid-template-rows

```css
grid-template-rows: none | <track-list> | <auto-track-list>
```

`grid-template-rows`는 행의 개수 및 크기를 지정할 수 있다. 작성 방법은 `grid-template-columns`와 동일하다.

<iframe height="510" style="width: 100%;" scrolling="no" title="Gridlayout grid-template-columns" src="//codepen.io/armadillo-dev167/embed/GbrYPp/?height=265&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/armadillo-dev167/pen/PrWbXz/'>Gridlayout grid-template-columns</a> by armadillo
  (<a href='https://codepen.io/armadillo-dev167'>@armadillo-dev167</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### grid-template-areas

```css
grid-template-areas: none | <string>
```

그리드는 행과 열을 이용한 레이아웃이라고 앞서 언급했다. `grid-template-areas`는 각각의 행과 열에 이름을 붙여 해당 영역에 특정 그리드 아이템이 위치하도록 한다. 이 속성은 뒤에 설명할 `grid-area`와 함께 쓰인다.

`grid-template-areas`에서 전체 레이아웃을 구성하면, `grid-area`을 이용해 레이아웃의 각 요소를 정의할 수 있다.

<iframe height="360" style="width: 100%;" scrolling="no" title="Gridlayout grid-template-columns" src="//codepen.io/armadillo-dev167/embed/zVZLzy/?height=265&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/armadillo-dev167/pen/PrWbXz/'>Gridlayout grid-template-columns</a> by armadillo
  (<a href='https://codepen.io/armadillo-dev167'>@armadillo-dev167</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### grid-gap

```css
grid-gap: <row-gap> <column-gap>?
```

일반적으로 요소 간 여백을 주기 위해서는 `margin`을 사용한다. 그리드 레이아웃에서는 `grid-gap`을 이용해 각 그리드 사이 여백을 설정할 수 있다. 주의할 점은 그리드 **사이**의 여백이기 때문에 그리드 컨테이너의 외곽은 여백이 지정되지 않는다는 점이다. 이렇게 설계된 이유는 그리드 컨테이너에 `padding`값을 이용할 수 있기 때문인 것 같다.

<iframe height="440" style="width: 100%;" scrolling="no" title="Gridlayout grid-template-columns" src="//codepen.io/armadillo-dev167/embed/OeWBeR/?height=265&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/armadillo-dev167/pen/PrWbXz/'>Gridlayout grid-template-columns</a> by armadillo
  (<a href='https://codepen.io/armadillo-dev167'>@armadillo-dev167</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### grid-column

```css
grid-column: <grid-line> [/ <grid-line>] ?
```

`grid-column`은 **Grid line 번호**를 이용해 해당 그리드 아이템이 몇개의 열을 차지할지 설정할 수 있다. 예를 들어 `grid-column: 1 / 3`일 경우 1번 grid line부터 3번 grid line까지의 열을 차지하게 된다. `span` 키워드를 이용해 특정 grid line이 아닌 칸의 개수를 지정할 수도있다.  `grid-column: span 2`으로 설정하면 해당 gird item이 시작되는 열에서 2칸을 더 차지하게 된다.

<iframe height="160" style="width: 100%;" scrolling="no" title="Gridlayout grid-template-columns" src="//codepen.io/armadillo-dev167/embed/MMpZyR/?height=265&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/armadillo-dev167/pen/PrWbXz/'>Gridlayout grid-template-columns</a> by armadillo
  (<a href='https://codepen.io/armadillo-dev167'>@armadillo-dev167</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### grid-row

```css
grid-row: <grid-line> [/ <grid-line>] ?
```

`grid-row`는 열이 아닌 행에 대한 grid line을 지정한다는 것만 빼면 `gird-column`과 동일하게 동작한다.

<iframe height="330" style="width: 100%;" scrolling="no" title="Gridlayout grid-template-columns" src="//codepen.io/armadillo-dev167/embed/mZWagj/?height=265&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/armadillo-dev167/pen/PrWbXz/'>Gridlayout grid-template-columns</a> by armadillo
  (<a href='https://codepen.io/armadillo-dev167'>@armadillo-dev167</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### grid-area

```css
grid-area: <grid-line> [/ <grid-line>] ? [/ <grid-line>] ? [/ <grid-line>] ? | <custom-ident>
```

`grid-area`는 `grid-column`과 `grid-row`를 함께 사용할 수 있는 속성이다. `grid-area: 1 / 4 / 1 / 1`과 같이 `grid-area: 시작 열 / 끝 열 / 시작 행 / 끝 행` 순으로 지정할 수 있다. 또한 앞서 보았던 `grid-areas` 에서 지정한 영역의 이름으로 지정하면 해당 영역을 차지하게 된다.

<iframe height="330" style="width: 100%;" scrolling="no" title="Gridlayout grid-template-columns" src="//codepen.io/armadillo-dev167/embed/zVZygN/?height=265&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/armadillo-dev167/pen/PrWbXz/'>Gridlayout grid-template-columns</a> by armadillo
  (<a href='https://codepen.io/armadillo-dev167'>@armadillo-dev167</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## Grid layout 활용 예제

### 전체 페이지 레이아웃 구성하기

grid-area를 이용하면 손쉽게 전체 페이지 레이아웃을 구성할 수 있다.

<iframe height="350" style="width: 100%;" scrolling="no" title="Gridlayout grid-template-columns" src="//codepen.io/armadillo-dev167/embed/BgWbEm/?height=265&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/armadillo-dev167/pen/PrWbXz/'>Gridlayout grid-template-columns</a> by armadillo
  (<a href='https://codepen.io/armadillo-dev167'>@armadillo-dev167</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### `auto-fit`, `minmax`를 이용한 반응형 레이아웃

`grid-template-columns`에 `auto-fix`, `minmax`를 이용해 가변적으로 크기가 변하는 레이아웃을 구현할 수 있다.  여기서 잠깐 auto-fit, minmax에 대해 설명하면 다음과 같다.

- `minmax`: grid item의 최소/최대 사이즈를 설정할 수 있다. 예를 들어 `minmax(300px, 1fr)`일 경우 해당 grid item의 크는 최소 `300px`로 설정되며  브라우저의 크기가 커지면 `브라우저 크기 / 그리드 아이템 개수`로 늘어나게 된다.
- `auto-fit`: 앞서 살펴보았던  `grid-template-columns`, `grid-template-rows`는 행 또는 열의 개수를 고정시켜놓았지만 auto-fit을 이용하면 가변적으로 그 개수를 조정할 수 있다. `minmax`에 의해 그리드 아이템의 크기가 가변일 경우 해당 grid container가 수용 가능한 개수만큼 행 또는 열의 개수를 자동으로 조정한다. 이와 비슷한 속성으로 `auto-fill`이 있다([설명 링크](https://css-tricks.com/auto-sizing-columns-css-grid-auto-fill-vs-auto-fit/){:target="_blank"}).

브라우저 크기를 변경해보면, 아래 예제에서 한 줄에 표시되는 grid item의 개수가 달라지는 것을 확인할 수 있다.

<iframe height="265" style="width: 100%;" scrolling="no" title="Gridlayout grid-template-columns" src="//codepen.io/armadillo-dev167/embed/VJpRJj/?height=265&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/armadillo-dev167/pen/PrWbXz/'>Gridlayout grid-template-columns</a> by armadillo
  (<a href='https://codepen.io/armadillo-dev167'>@armadillo-dev167</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

## 마무리

Grid layout은 기존에는 어려웠을, 혹은 귀찮았을 다양한 레이아웃을 간결하게 구현할 수 있도록 많은 기능을 제공하고 있다. 그리드 레이아웃이 보편화돼서 프론트앤드 개발자/퍼블리셔 분들이 보다 편한 삶을 살 수 있었으면 한다.

관련된 속성이 워낙 많고, 활용 방안도 다양하기 때문에 본 글에서는 필자가 중요하다고 생각하는 몇 가지 사례를 들어 설명했다. [Gridy by Example](https://gridbyexample.com/examples/){:target="_blank"}에서는 Grid layout을 통해 만들 수 있는 다양한 예제를 제공하고 있다. 이 글에서 흥미를 느꼈다면 해당 사이트에서 보다 많은 정보를 얻길 바란다.
