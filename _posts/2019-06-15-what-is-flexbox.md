---
title: "[CSS] Flexbox 이해하기"
date: 2019-06-15 23:00:00
categories: css
tags: html css layout flexbox
description: "필자는 웹 페이지의 레이아웃을 구성하기 위해 float와 position 등의 속성을 주로 이용했다. 그러나 Flexbox의 등장 이후 사용 빈도가 현저히 줄어들었다. 이 글에서는 Flexbox에 대해 설명하고, 예제를 통해 어떻게 활용 할 수 있는지 알아보고자 한다."
---

필자는 웹 페이지의 레이아웃을 구성하기 위해 `float`와 `position` 등의 속성을 주로 이용했다. 그러나 Flexbox의 등장 이후 사용 빈도가 현저히 줄어들었다. 이 글에서는 Flexbox에 대해 설명하고, 예제를 통해 어떻게 활용 할 수 있는지 알아보고자 한다.

## Flex container & Flex item

flexbox는 flex container와 flex item으로 구성된다. **flex containe**r는 flex item들을 감싸서 컨테이너 역할을 하는 DOM 엘리먼트로 `display: flex | flex-inline` 속성을 통해 지정할 수 있다. **flex item**은 ****flex container 하위 DOM 엘리먼트를 뜻한다.

![flex container와 flex item들](/asserts/images/img-flex-container-and-items.png)

flex container와 flex item들

## Flexbox에 존재하는 두 개의 축

flexbox에는 main axis와 cross axis 두 개의 축이 존재한다. **flex item은 main axis의 방향에 따라 정렬된다.** 그래서 축 설정에 따라 렌더링 결과가 달라진다. 때문에 두 축에 대해 정확히 이해하고 있어야 한다.

축의 방향은 flex container의  `flex-direction` 속성을 이용해 결정할 수 있다.

### `flex-direction: row`일 경우

main axis는 왼쪽에서 오른쪽으로(→), cross axis는 위에서 아래로(↓) 진행된다. flex item들은 한 줄로 정렬되며, 첫번째 요소가 가장 왼편에 존재하게 된다.

![flex-direction: row일 경우](/asserts/images/flex-direction-row.png)

### `flex-direction: column`일 경우

main axis는 위에서 아래로(↓), cross axis는 왼쪽에서 오른쪽으로(→) 진행된다. flex item들은 한 줄에 한 개의 요소만 존재하고 블럭 형태로 쌓이게 된다. 이때 가장 첫번째 요소가 가장 위에 존재한다.

![flex-direction: column일 경우](/asserts/images/flex-direction-column.png)

## Flexbox 주요 CSS 속성

flex container와 flex item은 각자 다른 CSS속성을 사용할 수 있다. 각 속성을 알아보자.

- **flex container**: `flex-direction`, `justify-content`, `align-items`, `flex-wrap` 등
- **flex item**: `flex-grow`, `flex-shrink`, `flex-basis`, `flex`, `order` 등

### `flex-direction`

```css
flex-direction: row|row-reverse|column|column-reverse
```

위에서 설명했듯, `flex-direction`은 flexbox 양 축의 방향을 결정한다. `row`와 `column`이 주로 사용되며 `row-reverse`, `column-reverse`는 각각 `row`, `column`의 역방향으로 flex item이 정렬된다. 이는 DOM 구조와 화면에 보여지는 순서가  달라짐을 뜻한다. 이러한 불일치는 스크린 리더의 도움을 받는 사람들에게 부정적인 영향을 끼칠 수 있다.

### `justify-content`

```css
justify-content: center|flex-start|flex-end|space-around|space-between
```

`justify-content` 속성은 flex item들의 main axis 상 위치/여백을 결정한다. 주로 사용되는 속성 값은 다음과 같다.

- `flex-start`: main axis의 시작 지점부터 flex item이 시작된다.
- `center`: flex item이 main axis 중앙으로 정렬된다.
- `flex-end`: flex item이 main axis 끝 지점부터 정렬된다.
- `space-around`: flex item이 동일한 여백으로 정렬된다. 이 때 flex container의 시작과 끝 지점에는 flex item 사이 여백의 1/2에 해당하는 여백이 들어간다.
- `space-between`: flex item이 동일한 여백으로 정렬된다. `space-around`와는 다르게 flex container의 시작과 끝에는 여백이 존재하지 않는다.

<iframe width="100%" height="700" src="//codepen.io/armadillo-dev167/embed/xoZLYv/?height=265&theme-id=0&default-tab=result"></iframe>

### `align-items`

```css
align-items: flex-start|center|flex-end|stretch
```

`justify-content` 속성이 main axis에 영향을 준다면, `align-items`는 cross axis 상 위치/여백을 조정하는 속성이다. `flex-start`, `flex-end`, `center`는 바라보는 축만 다를뿐, `justify-content`와 동일한 기능을 수행한다. `stretch`는  flex item의 높이를 cross axis의 길이와 동일하게 세팅한다.

<iframe width="100%" height="800" src="//codepen.io/armadillo-dev167/embed/GboQeX/?height=265&theme-id=0&default-tab=result"></iframe>

### `flex-wrap`

```css
flex-wrap: nowrap|wrap|wrap-reverse
```

flexbox는 main axis를 기준으로 1개 라인만 생성된다. 그러나 `flex-wrap` 속성을 `wrap`으로 지정하면 flex container main axis 길이가 flex items 전체 길이보다 작을 경우 다음 라인으로 레이아웃이 밀리게 된다.

<iframe width="100%" height="450" src="//codepen.io/armadillo-dev167/embed/qzbYbw/?height=265&theme-id=0&default-tab=result"></iframe>

### `flex-grow`

```css
flex-grow: <number>
```

`flex-grow` 속성을 지정하면 flex container의 길이가 소속된 flex item들의 길이보다 클 경우, flex item의 길이가 확장되는 비율을 지정할 수 있다. 기본값인 0으로 설정되어 있으면 flex item본연의 길이가 유지된다. 아래 예제를 보면 `flex-grow` 값이 각각 1, 2, 3으로 지정되었을 경우 각 item의 크기도 1, 2, 3배로 지정되는 것을 확인 할 수 있다.

<iframe width="100%" height="300" src="//codepen.io/armadillo-dev167/embed/NZxMbg/?height=265&theme-id=0&default-tab=result"></iframe>

### `flex-shrink`

```css
flex-shrink: <number>
```

`flex-shrink`는 `flex-grow`와 반대로 flex item의 길이가 축소되는 비율을 지정할 수 있다. 아래 예제를 보면 flwx-shrink값이 0인 경우, flex item의 크기(120px + margin 20px)가 flex container를 초과했을 때 크기가 줄어들지 않는다. 이와 반대로 flex-shrink값이 지정되어 있을 경우 비율에 맞게 각 flex item의 길이가 줄어드는 것을 확인 할 수 있다.

<iframe width="100%" height="320" src="//codepen.io/armadillo-dev167/embed/NZxzKz/?height=265&theme-id=0&default-tab=result"></iframe>

### `flex-basis`

```css
flex-basis: <width>
```

앞서 살펴보았듯, flex item은 `flex-grow`, `flex-shrink`에 의해 그 크기가 가변적이다. `flex-basis`는 이러한 크기 변화가 일어나기 전의 초기 값을 지정한다. `<width>` 위치에는 px, em, % 등 크기를 표현하는 모든 단위를 사용할 수 있다.

### `flex`

```css
flex: flex-grow flex-shrink flex-basis
```

`flex`을 이용하면 위에서 살펴본 `flex-grow`, `flex-shrink`, `flex-basis` 속성을 한 번에 지정할 수 있다. flex는 자주 사용되는 몇가지 셋팅 값을 키워드로 미리 지정하여 제공한다.

- `initial`: 0 1 auto - 크기의 축소만 발생하며 기본 크기는 `width`, `height`등의 속성에 의해 결정된다.
- `auto`: 1 1 auto - 크기의 확장/축소가 일어나며 기본 크기는 `width`, `height`등의 속성에 의해 결정된다.
- `none`: 0 0 auto - 크기의 확장/축소가 일어나지 않으며 기본 크기는 `width`, `height`등의 속성에 의해 결정된다.

### `order`

```css
order: <number>
```

`order` 속성을 지정하면 해당 순서에 따라 flex item의 정렬 순서가 변경된다. 아래 예제를 보면 `order: 3` flex item은 실제 DOM 엘리먼트 순서는 가장 첫번째이다. 하지만 `order`  값이 가장 크기 때문에 가장 마지막 위치에 랜더링된다.

<iframe width="100%" height="200" src="//codepen.io/armadillo-dev167/embed/RzrJxx/?height=265&theme-id=0&default-tab=result"></iframe>

## Flex 활용 예제

### 컨텐츠 수평/수직 중앙 정렬

기존에는 수평/수직 중앙 정렬을 위해 `position, top, left, transform, line-height, margin` 등의 속성을 이용했다. flexbox를 이용하면 다음과 같이 손쉽게 구현할 수 있다.

```css
.flex-container {
  display: flex;
  justify-content: center; // 수평 정렬
  align-items: center; // 수직 정렬
}
```

<iframe width="100%" height="400" src="//codepen.io/armadillo-dev167/embed/bPEKmZ/?height=265&theme-id=0&default-tab=result"></iframe>

### `margin: auto`를 활용한 콘텐츠 정렬

flex item에 `margin: auto;`를 지정하면 flex container에 남아있는 여백을 모두 마진으로 잡는다. 이를 이용해 보다 편리하게 레이아웃을 구성할 수 있다.

```css
.flex-container {
  display: flex;
}

.flex-item {
  margin-right: auto; // 남아있는 여백을 모두 margin-right로 사용
}
```

<iframe width="100%" height="200" src="//codepen.io/armadillo-dev167/embed/EBPpyv/?height=265&theme-id=0&default-tab=result"></iframe>

### 무한 수평 스크롤 레이아웃

`flex-wrap` 속성이 `nowrap`일 경우 flex container는 flex item의 길이가 아무리 커도 1개의 라인으로 표시한다. 이러한 특징을 이용해 수평 스크롤 레이아웃을 구성할 수 있다.

```css
.flex-container {
  display: flex;
  overflow-x: scroll; // flex item의 갯수가 많아졌을 경우 스크롤 발생
  width: 400px; //일정 길이의 너비 지정
}
```

<iframe width="100%" height="200" src="//codepen.io/armadillo-dev167/embed/PrZaLx/?height=265&theme-id=0&default-tab=result"></iframe>

## 마무리

flexbox에 익숙해지면 이전보다 훨씬 간결하게 레이아웃을 구성할 수 있다. 심지어 반응형 레이아웃에 대한 대응까지 수월해진다. 프론트앤드 개발자에게 flexbox는 강력한 무기로 자리 잡았다. 다음 글에서는 프론트앤드의 또 다른 무기, Grid layout에 대해 다뤄보려고 한다.

## 참고 링크

- [Basic concepts of flexbox / MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flexible_Box_Layout/Basic_Concepts_of_Flexbox){:target="_blank"}
