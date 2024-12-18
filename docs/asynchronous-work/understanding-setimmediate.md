---
title: setImmediate() 이해하기
layout: learn
authors: flaviocopes, MylesBorins, LaRuaNa, ahmadawais, clean99, ovflowd
---

> ❗️ _번역 날짜: 2024년 12월 17일_ <br />
> 공식 문서 원문은 아래를 참고하세요.<br /> > [Understanding setImmediate()](https://nodejs.org/en/learn/asynchronous-work/understanding-setimmediate#understanding-setimmediate)

# setImmediate() 이해하기

코드를 비동기적으로 실행하지만 가능한 한 빠르게 실행하고 싶을 때, Node.js에서 제공하는 `setImmediate()` 함수를 사용할 수 있습니다:

```js
setImmediate(() => {
  // run something
});
```

setImmediate() 의 인자로 전달된 함수는 이벤트 루프의 다음 반복(iteration)에 실행되는 **콜백** (**callback**)입니다.

`setImmediate()` 과 `setTimeout(() => {}, 0)` (0초의 타임아웃을 넘기는) 는 무엇이 다르며, `process.nextTick()` 과 `Promise.then()` 는 무엇이 다를까요?

`process.nextTick()` 에 전달된 함수는 이벤트 루프의 현재 작업 이후 **현재 반복(iteration)** 이 끝난 직후 실행됩니다.

즉, 항상 `setTimeout` 이나 `setImmediate`.보다 먼저 실행됩니다.

`setTimeout()` 0ms의 지연시간을 가진 콜백은 `setImmediate()` 와 매우 유사합니다. The 실행 순서는 여러 요인에 따라 달라질 수 있지만, 두 콜백 모두 이벤트 루프의 다음 반복에 실행됩니다.

- `process.nextTick` 콜백은 `process.nextTick queue` 에 추가됩니다.
- `Promise.then()` 콜백은 `promises microtask queue`에 추가됩니다.
- `setTimeout`,`setImmediate` 콜백은 `macrotask queue` 에 추가됩니다.

이벤트 루프는 `process.nextTick queue` 의 작업을 먼저 수행하고, `promises microtask queue`, `macrotask queue` 의 순서로 작업을 수행합니다.

아래 코드는 `setImmediate()`, `process.nextTick()` 그리고 `Promise.then()` 간의 실행 순서를 보여줍니다.

```js
const baz = () => console.log('baz');
const foo = () => console.log('foo');
const zoo = () => console.log('zoo');

const start = () => {
  console.log('start');
  setImmediate(baz);
  new Promise((resolve, reject) => {
    resolve('bar');
  }).then((resolve) => {
    console.log(resolve);
    process.nextTick(zoo);
  });
  process.nextTick(foo);
};

start();

// start foo bar zoo baz
```

이 코드는 `start()` 를 가장 먼저 호출할 것이고, 이후 `process.nextTick queue` 의 `foo()` 를 호출할 것입니다. 그런 다음 `promises microtask queue` 의 `bar` 를 출력하고 동시에 `process.nextTick queue` 에 `zoo()` 를 추가합니다. 이제 방금 추가된 `zoo()` 를 호출하고 `macrotask queue` 의 `baz()` 가 마지막으로 호출됩니다.

위의 원칙은 CommonJS 환경에서 유효하지만, `mjs` 파일과 같은 ES Modules 환경에서는 실행 순서가 다를 수 있음을 유념해야합니다.

```js
// start bar foo zoo baz
```

이는 로드되고있는 ES Module이 비동기적 작업으로 감싸져 실행되기 때문입니다. 따라서 전체 스크립트가 이미 `promises microtask queue` 에 존재하게 됩니다. 그래서 Promise가 즉시 해결(resolve) 되면 해당 콜백이 `microtask` 에 추가됩니다. Node.js는 `microtask` 큐를 비울 때까지 다른 큐로 이동하지 않으므로, `bar` 가 먼저 출력되는 것입니다.
