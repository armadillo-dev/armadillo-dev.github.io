---
title: "[Javascript] IntersectionObserver API 설명과 사용 방법"
date: 2019-04-30 23:00:00
categories: javascript
tags: javascript Intersection​Observer
description: "IntersectionObserver는 DOM 엘리먼트가 뷰포트상에 노출되었는지 여부를 비동기적으로 감시하는 API이다. 이번 게시물을 통해 기존 방법의 문제점과 IntersectionObserver를 이용하면 어떻게 문제점을 해결 할 수 있는지 알아보자."
---

## Intersection​Observer 란?

`IntersectionObserver`는 DOM 엘리먼트가 뷰포트상에 노출되었는지 여부를 비동기적으로 감시하는 API이다. MDN에서는 `IntersectionObserver`를 통해 다음과 같은 것들을 할 수 있다고 설명한다.

- 페이지 스크롤에 따른 이미지/콘텐츠의 레이지 로딩 구현
- 무한 스크롤 웹사이트 구현
- 광고 수익 측정을 위한 광고 문구 노출 여부 계산
- 사용자가 결과를 볼지 유무에 따라 애니메이션 또는 작업 실행 여부 결정

물론 위의 사항들은 `IntersectionObserver` 없이 기존에도 구현이 가능했지만, 기존 방법에는 문제점이 있었다. 이번 게시물을 통해 기존 방법의 문제점과 `IntersectionObserver`를 이용하면 어떻게 문제점을 해결 할 수 있는지 알아보자.

## 기존 이미지/콘텐츠 레이지 로딩 방법

쇼핑몰의 상품 세부 정보 페이지와 같이 다수의 이미지가 중심이 되는 페이지에서 화면상에 존재하는 모든 이미지를 한번에 불러오면, 불필요한 네트워크 비용이 증가하고 이로 인해 사용자 경험을 저하 시킬 수 있다. 때문에 현재 화면에 보이는 이미지만 선별적으로 로딩하는 기법이 필요하다.

```js
// viewport 안에 있는지 여부 판별
function isInViewport(element) {
  const viewportHeight = document.documentElement.clientHeight;
  const viewportWidth = document.documentElement.clientWidth;
  const rect = element.getBoundingClientRect();

  if (!rect.width || !rect.height) return false;

  const top = rect.top >= 0 && rect.top < viewportHeight;
  const bottom = rect.bottom >= 0 && rect.bottom < viewportHeight;
  const left = rect.left >= 0 && rect.left < viewportWidth;
  const right = rect.right >= 0 && rect.right < viewportWidth;

  return (top || bottom) && (left || right);
}

// 이벤트 처리
const images = Array.from(document.querySelectorAll('img'))
document.addEventListener('scroll', () => {
  images.forEach(image => {
    console.log(image, isInViewport(image))
    if (isInViewport(image)) {
      image.onload = () => images.splice(images.indexOf(image), 1);
      image.src = 'image-path';
    }
  });
});
```

### 기존 방법의 문제점

기존에는 대상 이미지 또는 엘리먼트가 뷰포트 내에 존재하는지 유무를 판별하기 위해 [`element.getBoundingClientRect`](https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect){:target="_blank"} 함수를 이용했다. 이 함수는 때때로 유용하지만, 성능적 이슈가 발생한다. `element.getBoundingClientRect`가 호출되면 브라우저는 해당 엘리먼트의 크기와 위치값을 계산한다. 그리고 이 과정에서 reflow가 발생하게 된다. 간헐적인 사용이라면 크게 무리가 되지 않지만 아래 예제와 같이 scroll 이벤트와 함께 사용될 경우 메인 스레드가 과부하 될 수 있다.

또한, `iframe`를 사용 할 때, 부모 창의 도메인이 `iframe` 내부의 도메인과 다를 경우 부모 창의로의 접근이 허용되지 않아 `iframe` 자체적으로 동적 로딩 기능을 구현 할 수 없었다.

## Intersection​Observer의 등장

```js
const observer = new Intersection​Observer(callback[, options]);
```

기존 방식의 문제점을 해결하기 위해 `Intersection​Observer`가 등장했다. `Intersection​Observer`는 `element.getBoundingClientRect`와 달리 reflow를 일으키지 않는다. 또한 `iframe` 내부에서도 가시성을 판단 할 수 있다.

## IntersectionObserver 사용 예제

### 1. 동적 이미지 로딩

`Intersection​Observer`를 이용하면 동적 이미지 로딩을 손쉽게 구현 할 수 있다. 구현 방법은 대상 `element`가 뷰포트에 들어왔을 때 이미지 경로를 변경 한 뒤 `unobserve`시킨다. 다음의 코드는 간단한 예제 코드이다.

```js
const observer = new IntersectionObserver((entries, observer) => {
  entries.forEach(entry => {
    if (!entry.isIntersecting) return

    const { target, { dataset: { src }}} = entry
    target.style.backgroundImage = src
    observer.unobserve(target)
  })
})

const items = [
  { id: 1, src: 'some-image-path' },
  { id: 2, src: 'some-image-path2' },
  { id: 3, src: 'some-image-path3' },
  { id: 4, src: 'some-image-path4' },
  ...
]

const templates = items
  .map(item => (`<div class="item" data-src=${item.src}></div>`))
  .join('')

document.getElementById('container').innerHTML = templates
Array
  .from(document.querySelectorAll('.item'))
  .forEach(item => observer.observe(item))
```

아래 예제는 이 글을 쓰면서 참고한 [레진 코믹스의 기술 블로그에 있는 예제](https://tech.lezhin.com/2017/07/13/intersectionobserver-overview){:target="_blank"}이다. 개발자 도구의 네트워크 탭을 열어서 확인해보면, 스크롤을 내려야지만 이미지 로딩을 한다는 것을 알 수 있다.

<iframe width="90%" height="400" src="https://cdn.rawgit.com/fallroot/intersectionobserver-examples/master/demo-lazyload.html"></iframe>

![IntersectionObserver를 이용한 이미지 동적 로딩](/assets/images/lazy-loading-with-intersection-observer.gif)

### 2. 무한 스크롤

무한 스크롤 기능도 `IntersectionObserver`를 이용하면 쉽게 구현 할 수 있다. 컨테이너 역할을 하는 DOM의 하단에 추가 로딩을 위한 `element`를 추가 한뒤, 해당 `element`가 뷰포트에 접근했을 때 추가 컴포넌트 또는 이미지를 로딩 하면 된다. 이미지 동적 로딩 때와는 달리, 계속해서 추가 로딩을 해야하기 때문에 `unobserve`는 하지 않는다.

#### html

```html
<div class="container">
  <div id="items"></div>
  <div id="loader"></div>
</div>
```

#### javascript

```js
const count = 20
let index = 0

const loadItems = () => {
  const fragment = document.createDocumentFragment()

  for (let i = index; i < index + count; i += 1) {
    const item = document.createElement('div')
    item.innerText = `${i + 1}`
    item.classList.add('item')
    fragment.appendChild(item)
  }
  
  document.getElementById('items').appendChild(fragment)
  index += count
}

const observer = new IntersectionObserver(([loader]) => {
  if(!loader.isIntersecting) return

  loadItems()
})

loadItems()
observer.observe(document.getElementById('loader'))
```

아래 예제는 마찬가지로 [레진 코믹스의 기술 블로그에 있는 예제](https://tech.lezhin.com/2017/07/13/intersectionobserver-overview){:target="_blank"}이다. 스크롤을 내리면 계속해서 추가 노드가 생성되는 것을 확인 할 수 있다.

<iframe width="90%" height="300" src="https://cdn.rawgit.com/fallroot/intersectionobserver-examples/master/demo-infinitescroll.html"></iframe>

## Polyfill

아쉽게도 `IntersectionObserver`는 IE에서 지원되지 않는다. 하지만 [W3C에서 작성한 Polyfill](https://github.com/w3c/IntersectionObserver){:target="_blank"}을 적용하면 동일한 인터페이스로 해당 기능을 사용 할 수 있다.

![Can I use IntersectionObserver](/assets/images/can-i-use-intersection-observer.png)

## 참고자료

- [IntersectionObserver를 이용한 이미지 동적 로딩 기능 개선](https://tech.lezhin.com/2017/07/13/intersectionobserver-overview){:target="_blank"}
- [Intersection​Observer](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserver){:target="_blank"}
- [Can I use IntersectionObserver](https://caniuse.com/#search=intersection%20observer){:target="_blank"}
- [IntersectionObserver Polyfill](https://github.com/w3c/IntersectionObserver){:target="_blank"}
