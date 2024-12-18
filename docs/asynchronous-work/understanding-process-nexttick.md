---
title: process.nextTick() 이해하기
layout: learn
authors: flaviocopes, MylesBorins, LaRuaNa, ahmadawais, ovflowd, marksist300
---

# process.nextTick() 이해하기

> ❗️ _번역 날짜: 2024년 12월 18일_ <br>
> 공식 문서 원문은 아래를 참고하세요.<br> > [Understanding process.nextTick()](https://nodejs.org/en/learn/asynchronous-work/understanding-processnexttick)

Node.js 이벤트 루프를 이해하려고 할 때, 중요한 부분 중 하나는 `process.nextTick()`입니다.
런타임이 이벤트를 처리하기 위해 JavaScript로 다시 호출할 때마다 이를 tick이라고 부릅니다.

`process.nextTick()`에 함수를 전달하면, 현재 작업이 완료된 직후, 이벤트 루프의 다음 단계로 이동하기 전에 이 함수를 호출하도록 엔진에 지시합니다:

```js
process.nextTick(() => {
  // 무언가 실행
});
```

이벤트 루프는 현재 함수 코드를 처리하는 데 바쁩니다. 이 작업이 끝나면, JS 엔진은 해당 작업 중 `nextTick` 호출에 전달된 모든 함수를 실행합니다.

이는 JS 엔진에 현재 함수 이후에 비동기적으로(즉시) 함수를 처리하라고 지시하는 방법이며, 대기열에 추가하지 않습니다.

`setTimeout(() => {}, 0)`을 호출하면 해당 함수는 다음 tick의 끝에 실행되지만, `nextTick()`을 사용하면 호출이 우선순위를 가져서 다음 tick의 시작 직전에 실행됩니다.

다음 이벤트 루프 반복에서 코드가 이미 실행되었는지 확인하려면 `nextTick()`을 사용하십시오.

#### 이벤트 순서 예제:

```js
console.log('Hello => number 1');

setImmediate(() => {
  console.log('Running before the timeout => number 3');
});

setTimeout(() => {
  console.log('The timeout running last => number 4');
}, 0);

process.nextTick(() => {
  console.log('Running at next tick => number 2');
});
```

#### 예제 출력:

```bash
Hello => number 1
Running at next tick => number 2
Running before the timeout => number 3
The timeout running last => number 4
```

출력 결과는 실행마다 다를 수 있습니다.
