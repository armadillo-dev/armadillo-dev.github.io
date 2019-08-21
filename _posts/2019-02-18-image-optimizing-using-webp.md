---
title: "[html & css]  WebP 파일을 이용한 이미지 최적화"
date: 2019-02-18 11:36 -0400
categories: html css
tags: html css image webp
---

## WebP 파일이란?

WebP 파일은 구글이 웹사이트의 트래픽 감소 및 로딩 속도 단축을 겨냥해 개발한 이미지 포맷이다. 기존 웹사이트에서 널리 사용되고 있는 jpeg, png, gif 등을 대체하기 위해 개발되었다. 구글이 이러한 포맷을 개발 및 배포한 목적은 전 세계의 검색 및 사이트 접속 속도가 증가할수록 구글의 광고 수입 또한 증가하기 때문이다.

## WebP 파일을 사용해야 하는 이유

WebP 파일은 기존 jpeg 파일 대비 39.8% 압축룰이 개선 되었으며, 압축률 또한 조절 할 수 있다. 항상 페이로드를 줄이기 위해 고민하는 프론트엔드 개발자들에게는 더할나위 없이 좋은 소식이라 할 수 있다 :-)

## WebP 파일 적용 방법

안타깝게도, WebP 파일은 IE, Safari 등의 브라우저에서 지원되지 않는다. 때문에 무작정 png파일을 WebP 파일로 변경하면 화면에 제대로 랜더링 되지 않는 낭패를 격을 수 있다.

![Can I use Webp 결과](/assets/images/can-i-use-webp-browser-support.png)
*Can I use Webp 결과*

### HTML에서의 WebP 파일 활용

html 상에서 WebP 파일을 사용하기 위해서는 picture 태그를 이용하는 것이 좋다.  picture 태그는 source 태그와 함께 최적화된 이미지 리소스 로딩을 할 수 있도록 한다. 보다 자세한 설명은 링크를 참고하길 바란다.

```html
<picture>
  <source srcset="some-image.webp" type="image/webp" />
  <img src="other-image.jpg" />
</picture>
```

### CSS에서의 WebP 파일 활용

css의 `background-image` 속성에 WebP 파일을 활용하기 위해서는 javascript의 도움을 받아야 한다. 위에서 언급했듯이 모든 브라우저가 WebP파일을 지원하지 않기 때문이다. 아래 예제를 보면 어떤 식으로 활용이 가능한지 알 수 있다.

#### webp.js

```js
function WebpIsSupported(callback){
    // If the browser doesn't has the method createImageBitmap, you can't display webp format
    if(!window.createImageBitmap){
        callback(false);
        return;
    }

    // Base64 representation of a white point image
    var webpdata = 'data:image/webp;base64,UklGRiQAAABXRUJQVlA4IBgAAAAwAQCdASoCAAEAAQAcJaQAA3AA/v3AgAA=';

    // Retrieve the Image in Blob Format
    fetch(webpdata).then(function(response){
        return response.blob();
    }).then(function(blob){
        // If the createImageBitmap method succeeds, return true, otherwise false
        createImageBitmap(blob).then(function(){
            callback(true);
        }, function(){
            callback(false);
        });
    });
}

window.onload = () => {
  WebpIsSupported((isSupportWebP) => {
    if (!isSupportWebP) {
      const container = document.querySelector('.container');
      container.classList.remove('support-webp');
    }
  });
}
```

#### webp.html

```html
<style>
  .box-bg-img {
    width: 300px;
    height: 300px;
    background-color: #ccc;
    background-image: url('other-image.jpg');
  }

  .support-webp .box-bg-img {
    background-image: url('some-image.webp');
  }
</style>
<div class="container support-webp">
  <div class="box-bg-img"></div>
</div>
```

## 참고링크

- [WebP - 위키피디아][link-webp-link]
- [구글 WebP 이미지 포맷 아세요? 구글 크롬에서 WebP를 PNG로 변경해 다운로드 받기][link-webp-google]
- [using `<picture>` tag with Internet Explorer 11][link-picture]

[link-webp-link]: https://ko.wikipedia.org/wiki/WebP
[link-webp-google]: https://m.post.naver.com/viewer/postView.nhn?volumeNo=9688816&memberNo=1834
[link-picture]: https://stackoverflow.com/questions/51957262/using-picture-tag-with-internet-explorer-11
