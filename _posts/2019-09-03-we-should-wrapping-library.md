---
title: "외부 라이브러리를 wrapping 해야 하는 이유"
date: 2019-09-03 10:00:00
categories: javascript
tags: library dependency
description: "우리는 프로젝트를 진행하면서 수많은 외부 라이브러리의 도움을 받는다. 많은 사람들이 외부 라이브러리를 사용할 때 wrapping 해서 사용해야 한다고 말한다. 이번 글에서는 왜 많은 사람들이 wrapping을 권고하는지 필자가 겪었던 사례를 통해 알아보고자 한다."
---

우리는 프로젝트를 진행하면서 수많은 외부 라이브러리의 도움을 받는다. 많은 사람들이 외부 라이브러리를 사용할 때 wrapping 해서 사용해야 한다고 말한다. 이번 글에서는 왜 많은 사람들이 wrapping을 권고하는지 필자가 겪었던 사례를 통해 알아보고자 한다.

## Vuetify 2.0 업데이트

필자가 담당하고 있는 프로젝트는 UI를 그릴 때 [Vuetify](https://v15.vuetifyjs.com/)를 사용하고 있다. 그런데 최근 [Vuetify 2.0이 릴리즈](https://github.com/vuetifyjs/vuetify/releases/tag/v2.0.0)되면서 많은 장점(Typescript 지원, SASS 지원 등)들이 생겼고, 현재 프로젝트에도 2.0을 도입하기로 했다.

필자는 [Upgrade guide](https://github.com/vuetifyjs/vuetify/releases?after=v2.0.4#user-content-upgrade-guide)를 참고해 프로젝트에 Vuetify 2.0 버전을 적용하는 것 까지는 성공했다. 그러나 문제는 그 다음에 발생했다. 컴포넌트의 API가 변경되어 레이아웃이 깨지거나 정상적인 동작을 하지 않는 경우가 빈번하게 발생했다. 

예를 들어 `v-btn` 컴포넌트의 경우 `flat` prop은 `text`로, `round` prop은 `rounded`로 명칭이 변경됐다.

```html
// Vuetify 1.x v-btn 컴포넌트
<v-btn
  flat
  round
  @click="onClick"
>
  Button
</v-btn>

// Vuetify 2.x v-btn 컴포넌트
<v-btn
  text
  rounded
  @click="onClick"
>
  Button
</v-btn>
```

Vuetify에서 제공하는 컴포넌트는 대부분의 컴포넌트에서 사용하고 있었다. 때문에 대량의 파일 수정이 불가피했다. 너무 많은 파일을 수정하다 보니 중간에 휴먼 에러가 발생하기도 하고, 일일이 확인하며 넘어가다 보니 작업 속도는 한없이 느렸고 작업 자체도 지루했다.

## lodash → lodash-es 변경

프로젝트에 사용하고 있었던 lodash를 트리 셰이킹 적용을 위해 lodash-es로 변경하는 작업을 하게 되었다. 이번에도 Vuetify 때와 똑같은 문제가 발생했다. lodash를 사용하는 모든 파일을 탐색한 뒤 lodash-es로 변경해주어야 했다. 변경 후에는 기능이 정상적으로 작동하는지 일일이 확인해야 했다.

```js
// before
import _ from 'lodash'

function multiArray(arr, num) {
  return _.map(arr, n => n * num)
}

// after
import _ from 'lodash-es'

function multiArray(arr, num) {
  return _.map(arr, n => n * num)
}
```

*모든 작업을 완료한 후에 [babel-plugin-lodash](https://github.com/lodash/babel-plugin-lodash)의 존재를 알았다. 😭

## 무엇이 문제인가?

위에서 살펴본 Vuetify와 lodash 두 경우 모두 **결국은 의존성 문제**이다. 외부 라이브러리를 사용하는 모든 코드 조각은 해당 라이브러리에 대한 의존성을 가지게 된다. 프로젝트 내에 이런 코드 조각이 많아질수록 의존성은 높아지고, 외부 라이브러리의 수정에 의한 여파가 크게 나타난다.

wrapper를 만들기 전/후의 의존성을 비교하면 다음과 같다.

![외부 라이브러리 직접 사용](/assets/images/img-library-dependency.png)
*외부 라이브러리를 사용한 파일들이 외부 라이브러리를 직접 바라본다.*

![Wrapper를 이용한 외부 라이브러리 사용](/assets/images/img-wrapper-library-dependency.png)
*wrapper만 외부 라이브러리를 바라본다.*

Wrapper를 사용하면 외부 라이브러리가 변경되더라도 wrapper만 수정하면 그 외의 파일은 수정이 필요 없다. 그래서 우리는 되도록 외부 라이브러리에 대한 의존성을 낮추기 위해 wrapping 과정을 거치는 것이다.

## 외부 라이브러리 wrapping 하기

생각보다 wrapping 과정은 간단하다. 외부 라이브러리에서 제공하는 API를 감싼 내부 모듈을 만들어주면 된다.

### Vuetify wrapping

Vuetify에서 제공하는 각 컴포넌트를 감싸는 내부 컴포넌트를 생성해 `base` prifix를 붙여 사용했다. 이제 Vuetify API의 수정이 발생하면 base 컴포넌트의 내부 구현만 변경하고 외부에 노출시키는 API는 그대로 유지할 수 있다.

```js
//base-button.vue
<template>
  <v-btn
    v-bind="{ ...$attrs }"
    :text="flat" <- 외부에는 1.x대 API를 유지하면서 내부적으로는 2.x API를 따름
    v-on="{ ...$listeners }
  />
</template>

<script>
  module.exports = {
    props: {
      // 외부에는 1.x대 API를 유지하면서 내부적으로는 2.x API를 따름
      flat: { type: Boolean, required: false, default: false }
    }
  }
</script>
```

### lodash wrapping

lodash도 마찬가지로 utility 파일을 만들어 wrapping할 수 있다.

```js
// util.js
import _ from 'lodash'

export function isEmpty(value) {
  return _.isEmpty(value)
}
```
