---
title: "[javascript]Promise를 보다 잘 활용 할 수 있는 Tip"
date: 2018-12-07 14:09 -0400
categories: javascript
tags: javascript promise async
---

``Promise``는 javascript에서 비동기 처리를 위해 사용된다. 개인적으로 es7에 들어서며 ``async/await``로 그 사용처가 줄어들었다(``Promise``로 작성된 코드는 ``async/await``로 작성된 코드에 비해 가독성이 떨어진다). 그럼에도 불구하고 여전히 특정  케이스에서는 ``Promise``를 적극 활용하고 있어 공유하고자 한다.

## 1. ``Promise.all``을 이용한 병렬 처리

``Promise.all``는 다수의 비동기 요청을 병렬로 처리 할 수 있도록 도와준다. 예를 들어, 서버에 3명의 회원 정보를 요청해 화면에 출력해야 한다고 가정해보자.  

회원에 대한 정보를 한개씩 얻어올 수도 있지만, 3명의 회원 정보를 한 번에 요청하고 병렬로 처리하면 전체 처리 속도를 단축시킬 수 있다.

![Promise 사용 유무에 따른 순차 처리와 병렬 처리](/assets/images/img-promise-parallel-vs-sequential.png)

아래의 코드를 통해 구현 방법을 살펴보자.

### promise-all.js
```js
(function(){
  const promises = []; 
  promises.push(fetch('https://jsonplaceholder.typicode.com/users/1').then(res => res.json()));
  promises.push(fetch('https://jsonplaceholder.typicode.com/users/2').then(res => res.json()));
  promises.push(fetch('https://jsonplaceholder.typicode.com/users/3').then(res => res.json()));
  Promise.all(promises).then(([user1, user2, user3]) => {
    console.log('user1:', user1);
    console.log('user2:', user2);
    console.log('user3:', user3);
  });
})();
```

위의 예제를 실행하면 차례대로 user1, 2, 3번의 정보를 콘솔 창에 출력한다.

``Promise.all``은  인자로 전달된 배열의 모든 원소들을 병렬로 요청하고, ``resolve``되기를 기다린다. 모든 인자가 ``resolve``될 경우 ``Promise.all``가 최종 ``resolve``되며, 각 인자의 리턴값이 순서대로 배열에 담겨 리턴된다.

## 2. ``Promise.race``를 이용한 ``timeout`` 설정

``Promise.race``는 ``Promise.all``과 마찬가지로 전달받은 배열의 ``Promise``객체를 병렬로 요청한다. 다만, ``Promise.all``이 모든 요청이 완료 되기를 기다리는것과 달리, 가장 먼저 완료된 요청만 리턴된다.

이러한 특징을 활용해 비동기 요청의 ``timeout``을 설정 할 수 있다.

### promise-race.js
```js
const timeout = new Promise((resolve, reject) => {
  const id = setTimeout(() => {
    clearTimeout(id);
    reject('timeout!');
  }, 500);
});

const promise = new Promise((resolve, reject) => {
  const id = setTimeout(() => {
    clearTimeout(id);
    resolve('response!');
  }, 700);
});

Promise.race([promise, timeout]).then(response => {
  console.log(response);
}).catch(err => {
  console.log(err) //timeout!
});
```

위의 예제를 실행하면 ``Promise.race``의 ``catch``에 걸려 timeout!을 콘솔창에 출력한다. 실제로 실행하고자 했던 동작(promise 함수)의 실행 시간이(700ms)가 제한 시간(500ms)보다 길었기 때문이다.

## 3. ``Promise``를 활용한 재시도(retry) 로직 구현

서버에 보낸 요청이 실패 했을 경우, 즉시 실행을 종료하고 에러를 표시하게 될 것이다. 그러나, ``Promise``를 활용하면 실행을 즉시 멈추지 않고 일정 횟수만큼 재시도를 요청 할 수 있다.

다음 예제에서 2번 라인을 주석 처리하고 3번 라인의 주석을 해제하면 5번 재시도하는 것을 확인 할 수 있다. 

### promise-retry.js

```js
const fetchUsers = async (userId) => {
  const response = await fetch('https://jsonplaceholder.typicode.com/users/');
  // const response = await fetch('https://jsonplaceholder.typicode.com/users/wrong-url');
  if (response.ok) {
    return response.json();
  } else {
    throw response;
  }
}

const retry = (func, params = [], maxRetriesCount = 5, interval = 500) => new Promise((resolve, reject) => {
  func(...params)
    .then((response) => resolve(response))
    .catch((err) => {
      if (maxRetriesCount === 0) {
        reject(err); 
        return;
      }
      setTimeout(() => {
        console.log('retry!');
        retry(func, params, maxRetriesCount - 1).then(resolve, reject);
      }, interval);
   });
});

(async function(){
  try {
    const users = await retry(fetchUsers);
    document.write(JSON.stringify(users));
    console.log('response', users);
  } catch(err) {
    console.log('error:', err);
  }
})();
```

위 예제를 조금 더 수정하면 매 실행시마다 대기 시간을 다르게 준다거나, 앞서 배웠던 ``Promise.race``를 활용하는 등의 변형도 가능하다.

위에서 설명한 것 이외에도 ``Promise``은 다양한 활용 방법을 가지고 있을 것이다. 단순하게만 사용했던 것을, 보다 깊이 이해하고 활용도를 높이는 일은 언제나 즐거운 경험이다 :-)

## 참고링크

- [Promise - MDN][link-promise]
- [Promise.all() - MDN][link-promise-all]
- [Promise.race() - MDN][link-promise-race]

[link-promise]: https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise
[link-promise-all]: https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise/all
[link-promise-race]: https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise/race
