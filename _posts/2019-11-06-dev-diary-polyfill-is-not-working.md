---
title: "[개발 일기 #1] Polyfill 적용기"
date: 2019-11-06 22:30
categories: dev-diary programming
tags: dev-diary polyfill TextEncoder Intersection​Observer 
description: "프로젝트에서 websocket 통신을 위해 사용중인 stompjs 라이브러리는 TextEncoder와 TextDecoder를 사용한다. TextEncoder와 TextDecoder는 IE 계열 브라우저와 모바일 하위 OS에서 지원되지 않기 때문에 Polyfill 적용이 필요하다. "
---

> 실무에서 경험한 삽질기를 다루기 위한 개발 일기입니다. 각 잡고 포스팅하기엔 너무 가볍고, 기록하지 않으면 잊어버릴 것 같은 내용을 다룹니다.

프로젝트에서 websocket 통신을 위해 사용중인 `stompjs` 라이브러리는 `TextEncoder`와 `TextDecoder`를 사용한다. `TextEncoder`와 `TextDecoder`는 IE 계열 브라우저와 구형 모바일 OS에서 지원되지 않기 때문에 Polyfill 적용이 필요하다.

`stompjs` 공식 사이트에서는 스크립트 태그를 이용해 Polyfill을 적용할 것을 추천하고 있었다.

```html
<script src="https://cdn.jsdelivr.net/npm/text-encoding@0.6.4/lib/encoding.min.js"></script>
```

그러나, 이 방법은 `TextEncoder`와 `TextDecoder`를 지원하는 브라우저에서도 불필요한 리소스 다운로드를 유도한다.  때문에 `nuxt-polyfill` 모듈을 이용하기로 했다.  `nuxt-polyfill`은 특정 조건을 만족했을 때에만 폴리필 파일을 로드하게끔 도와주는 Nuxt 모듈이다.

작성한 코드는 다음과 같다.

```typescript
//nuxt.config.ts
export default {
  ...
  polyfill: {
    features: [
      { 
        required: 'text-encoding',
        // detect 함수의 결과값이 false인 경우에만 폴리필 적용
        detect: () => 'TextEncoder' in window, 
      }
    ]
  }
  modules: ['nuxt-polyfill'],
}
```

적용을 마치고 이슈가 됐었던 iOS 10.2 버전 단말에서 테스트를 진행했다. 그런데 여전히 TextEncoder가 정의되지 않아 에러가 발생했다. 개발자 도구에서 확인했을 때 polyfill 파일은 정상적으로 로드되고 있었다. 심지어 같은 모듈로 polyfill을 적용한 `IntersectionObserver`는 정상적으로 동작하고 있었다. 그래서 동공 지진만 일어나고 있었는데...😇  

혹시 몰라 `script` 태그를 이용해 직접 삽입했더니 정상적으로 동작했다. 차라리 동작을 안하면 polyfill의 문제인가 싶을텐데 ㅎㅎ... 잠깐동안 그냥 스크립트 태그를 이용해 처리할까 했지만, 불필요한 리소스를 다운로드 시켜야 한다는 점이 너무 마음에 안들었다.

한동안의 삽질 끝에 `TextEncoder` polyfill 파일 소스를 직접 확인해보니 다음과 같이 코드가 작성되어 있었다. 즉시 실행 함수에 인자로 넘겨주는 `this`에 `widnow`가 들어가기를 기대했으나, `nuxt-polyfill`의 실행 컨텍스트 때문에 다른 값이 들어가서 이런 일이 발생하지 않았을까 추측 할 수 있었다.

```javascript
//text-encoding.js
(function(global) {
  ...

  if (typeof module !== "undefined" && module.exports) {
    module.exports = {
      TextEncoder: global['TextEncoder'],
      TextDecoder: global['TextDecoder'],
      EncodingIndexes: global["encoding-indexes"]
    };
  }
}(this || {}))
```

다행히 `nuxt-polyfill`에서는 `install` 옵션을 통해 polyfill 파일 적용 이후 후처리 작업을 할 수 있도록 지원하고 있었고, 다음과 같이 코드를 수정 했더니 정상 동작하는 모습을 볼 수 있었다.

```typescript
//nuxt.config.ts
export default {
  ...
  polyfill: {
    features: [
      { 
        required: 'text-encoding',
        detect: () => 'TextEncoder' in window, 
        // install 함수는 첫번째 인자로 polyfill 파일에서 export한 값을 받는다.
        install: (polyfill) => {
          window.TextEncoder = polyfill.TextEncoder
          window.TextDecoder = polyfill.TextDecoder
        }
      }
    ]
  }
  modules: ['nuxt-polyfill'],
}
```

적용을 완료하고 왜 polyfill에서 `window`에 직접 삽입하지 않고 굳이 `this`를 넘겨서 처리했을까 생각해보니 `TextEncoder`/`TextDecoder`는 nodejs 환경에서도 동작해야하기 때문인것 같다.

실제로 브라우저 환경에서만 사용되는 `IntersectionObserver`는 polyfill 파일에서 window에 직접 기능을 추가하고 있었다.

```javascript
//intersection-observer.js
...
window.IntersectionObserver = IntersectionObserver;
window.IntersectionObserverEntry = IntersectionObserverEntry;
```

작업을 마무리하고 여유로운 마음으로 퇴근할 수 있었다.

여담으로, 사실 나는 polyfill 방식을 썩 좋아하진 않는다. 특정 기능을 지원하지 않는 브라우저는 오래된(= 모던 브라우저 대비 성능이 떨어지는) 브라우저다. 결국 polyfill은 안그래도 느린 브라우저에게 더 많은 짐을 지어주는 꼴이 된다.

이런 문제를 해결하기 위한 방법으로, 구형 브라우저에게는 더 적은 기능을 제공할 수 있을 것이다. 하지만 이런 방법이 현실 세계에서 받아들여지기는 쉽지 않을 것 같다.

## 참고자료

- [StompJs v5: Polyfillsl / StompJS Family](https://stomp-js.github.io/guide/stompjs/rx-stomp/ng2-stompjs/2018/06/29/pollyfils-for-stompjs-v5.html#polyfills){:target="_blank"}
- [Nuxt polyfill](https://github.com/Timkor/nuxt-polyfill){:target="_blank"}