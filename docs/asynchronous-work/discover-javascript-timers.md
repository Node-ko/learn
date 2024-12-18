---
title: JavaScript 타이머 알아보기
layout: learn
authors: flaviocopes, MylesBorins, LaRuaNa, amiller-gh, ahmadawais, ovflowd
---

# JavaScript 타이머 알아보기

> ❗️ _번역 날짜: 2024년 12월 17일_ <br>
> 공식 문서 원문은 아래를 참고하세요.<br> > [Discover JavaScript Timers](https://nodejs.org/en/learn/asynchronous-work/discover-javascript-timers#discover-javascript-timers)

## `setTimeout()`

JavaScript 코드를 작성할 때 함수 실행을 지연시키고 싶을 수 있습니다.

이 작업은 `setTimeout`의 역할입니다. 나중에 실행할 콜백 함수를 지정하고, 몇 밀리초(ms) 후에 실행할지 값을 설정합니다:

```js
setTimeout(() => {
  // 2초 후 실행
}, 2000);

setTimeout(() => {
  // 50밀리초 후 실행
}, 50);
```

이 구문은 새로운 함수를 정의합니다. 이 안에서 원하는 다른 함수를 호출할 수 있으며, 기존 함수 이름과 매개변수를 전달할 수도 있습니다:

```js
const myFunction = (firstParam, secondParam) => {
  // 무언가 실행
};

// 2초 후 실행
setTimeout(myFunction, 2000, firstParam, secondParam);
```

`setTimeout`은 타이머 ID를 반환합니다. 일반적으로 사용되지 않지만, 이 ID를 저장하고 원하는 경우 이 예약된 함수 실행을 삭제할 수 있습니다:

```js
const id = setTimeout(() => {
  // 2초 후 실행 예정
}, 2000);

// 실행을 취소
clearTimeout(id);
```

### 0 지연

타임아웃 지연을 `0`으로 설정하면 콜백 함수는 가능한 한 빨리 실행되지만, 현재 함수 실행 이후에 실행됩니다:

```js
setTimeout(() => {
  console.log('after ');
}, 0);

console.log(' before ');
```

이 코드는 다음과 같이 출력됩니다:

```bash
before
after
```

이는 CPU를 차단하는 작업을 방지하고 무거운 계산을 수행하는 동안 다른 함수가 실행될 수 있도록 대기열에 함수를 추가하는 데 특히 유용합니다.

> 일부 브라우저(IE 및 Edge)는 동일한 기능을 수행하는 `setImmediate()` 메서드를 구현하지만, 이는 표준이 아니며 [다른 브라우저에서 사용할 수 없습니다](https://caniuse.com/#feat=setimmediate). 그러나 이는 Node.js에서는 표준 함수입니다.

## `setInterval()`

`setInterval`은 `setTimeout`과 비슷한 함수로, 차이점은 콜백 함수가 한 번 실행되는 대신 특정 시간 간격(밀리초)마다 영구적으로 실행된다는 점입니다:

```js
setInterval(() => {
  // 2초마다 실행
}, 2000);
```

위 함수는 실행 중 멈추라는 명령을 내리지 않는 한 2초마다 실행됩니다. 이 경우 `setInterval`이 반환한 interval ID를 사용하여 `clearInterval`을 호출해 중지할 수 있습니다:

```js
const id = setInterval(() => {
  // 2초마다 실행
}, 2000);

clearInterval(id);
```

종종 `clearInterval`은 `setInterval` 콜백 함수 내에서 호출되어, 다시 실행해야 할지 또는 중지해야 할지 스스로 결정합니다. 예를 들어, 다음 코드는 `App.somethingIWait` 값이 `arrived`가 될 때까지 실행됩니다:

```js
const interval = setInterval(() => {
  if (App.somethingIWait === 'arrived') {
    clearInterval(interval);
  }
  // 그렇지 않으면 작업을 계속 진행
}, 100);
```

## 재귀적 `setTimeout`

`setInterval`은 함수가 실행 완료 시간을 고려하지 않고 n 밀리초마다 함수를 시작합니다.

함수가 항상 동일한 실행 시간을 가지면 문제가 없습니다:

![setInterval 정상 작동](/static/images/learn/javascript-timers/setinterval-ok.png)

하지만 네트워크 상태에 따라 실행 시간이 달라질 수 있습니다:

![setInterval 실행 시간 변동](/static/images/learn/javascript-timers/setinterval-varying-duration.png)

이 경우 긴 실행 시간이 다음 실행과 겹칠 수 있습니다:

![setInterval 겹침](/static/images/learn/javascript-timers/setinterval-overlapping.png)

이를 방지하려면 콜백 함수 실행이 완료되었을 때 재귀적인 `setTimeout`을 호출하도록 예약할 수 있습니다:

```js
const myFunction = () => {
  // 무언가 실행

  setTimeout(myFunction, 1000);
};

setTimeout(myFunction, 1000);
```

이로써 다음과 같은 시나리오를 달성할 수 있습니다:

![재귀적 setTimeout](/static/images/learn/javascript-timers/recursive-settimeout.png)

`setTimeout`과 `setInterval`은 [Timers 모듈](https://nodejs.org/api/timers.html)을 통해 Node.js에서도 사용할 수 있습니다.

Node.js는 `setImmediate()`도 제공하며, 이는 `setTimeout(() => {}, 0)`을 사용하는 것과 동등하며, 주로 Node.js 이벤트 루프와 함께 사용됩니다.
