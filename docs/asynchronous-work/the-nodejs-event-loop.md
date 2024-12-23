---
title: Node.js 이벤트 루프
layout: learn
authors: aug2uag, ovflowd
---

# Node.js 이벤트 루프
> ❗️ *번역 날짜: 2024년 12월 23일* <br>
> 공식 문서 원문은 아래를 참고하세요.<br>
>[The Node.js Event Loop](https://nodejs.org/en/learn/asynchronous-work/the-nodejs-event-loop)

## 이벤트 루프란?

이벤트 루프는 Node.js가 비동기 I/O 작업을 수행할 수 있도록 하는 것입니다. 기본적으로 단일 JavaScript 스레드가 사용되지만, 시스템 커널에 작업을 위임하여 가능한 한 비동기 작업을 수행할 수 있도록 합니다.

대부분의 최신 커널은 다중 스레드이므로 백그라운드에서 실행되는 여러 작업을 처리할 수 있습니다. 이러한 작업 중 하나가 완료되면 커널이 Node.js에게 알리고 적절한 콜백이 **poll** 큐에 추가되어 최종적으로 실행될 수 있도록 합니다. 이에 대한 자세한 설명은 이 주제의 후반부에서 자세히 설명하겠습니다.

## 이벤트 루프 설명

Node.js가 시작되면 이벤트 루프를 초기화하고 제공된 입력 스크립트(또는 [REPL][]로 드랍됨, 이 문서에서 다루지 않음)를 처리하여 비동기 API 호출, 타이머 스케줄링, 또는 `process.nextTick()`을 호출할 수 있습니다. 그런 다음 이벤트 루프를 처리하기 시작합니다.

다음 다이어그램은 이벤트 루프의 작업 순서를 간략하게 설명한 것입니다.

```
   ┌───────────────────────────┐
┌─>│           timers          │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │
   └───────────────────────────┘
```
> 각 상자는 이벤트 루프의 "단계(phase)"를 나타냅니다.

각 단계에는 실행할 콜백의 FIFO 큐가 있습니다. 각 단계는 고유한 방식으로 특별하지만, 일반적으로 이벤트 루프가 특정 단계에 진입하면 해당 단계에 특화된 작업을 수행한 다음, 해당 단계의 큐에 있는 콜백을 큐가 소진되거나 최대 콜백 수가 실행될 때까지 실행합니다. 큐가 소진되거나 콜백 제한에 도달하면 이벤트 루프는 다음 단계로 이동하는 식입니다.

이러한 작업들이 _더 많은_ 작업을 예약할 수 있고 **poll** 단계에서 처리되는 새로운 이벤트들이 커널에 의해 큐에 추가되기 때문에, 폴링 이벤트가 처리되는 동안에도 poll 이벤트가 큐에 추가될 수 있습니다. 결과적으로 오래 실행되는 콜백은 타이머의 임계값보다 훨씬 더 오래 poll 단계가 실행되도록 할 수 있습니다. 자세한 내용은 [**timers**](#timers)와 [**poll**](#poll) 섹션을 참조하세요.

> Windows와 Unix/Linux 구현 사이에 약간의 차이가 있지만, 이 설명에서는 중요하지 않습니다. 가장 중요한 부분은 여기에 있습니다. 실제로는 7~8단계가 있지만, 우리가 관심을 가져야 할 부분 - Node.js가 실제로 사용하는 부분 - 은 위에서 설명한 것들입니다.

## 단계 개요

- **timers**: 이 단계는 `setTimeout()`과 `setInterval()`로 예약된 콜백을 실행합니다.
- **pending callbacks**: 다음 루프 반복으로 연기된 I/O 콜백을 실행합니다.
- **idle, prepare**: 내부적으로만 사용됩니다.
- **poll**: 새로운 I/O 이벤트를 검색합니다; I/O 관련 콜백을 실행합니다(close 콜백, 타이머로 예약된 것들, 그리고 `setImmediate()`를 제외한 거의 모든 것); 적절한 경우 node는 여기서 블록됩니다.
- **check**: `setImmediate()` 콜백이 여기서 호출됩니다.
- **close callbacks**: 일부 close 콜백들, 예: `socket.on('close', ...)`

이벤트 루프의 각 실행 사이에 Node.js는 비동기 I/O나 타이머를 기다리고 있는지 확인하고, 없다면 깔끔하게 종료됩니다.

## 상세 단계 설명

### timers

타이머는 제공된 콜백이 실행 _될 수 있는_ **임계값**을 지정하며, 사람이 _원하는_ **정확한** 시간을 지정하는 것이 아닙니다. 타이머 콜백은 지정된 시간이 지난 후 가능한 한 빨리 실행되도록 예약됩니다. 하지만 운영 체제 스케줄링이나 다른 콜백의 실행으로 인해 지연될 수 있습니다.

> 각 상자는 이벤트 루프의 "단계(phase)"를 나타냅니다.

각 단계에는 실행할 콜백의 FIFO 큐가 있습니다. 각 단계는 고유한 방식으로 특별하지만, 일반적으로 이벤트 루프가 특정 단계에 진입하면 해당 단계에 특화된 작업을 수행한 다음, 해당 단계의 큐에 있는 콜백을 큐가 소진되거나 최대 콜백 수가 실행될 때까지 실행합니다. 큐가 소진되거나 콜백 제한에 도달하면 이벤트 루프는 다음 단계로 이동하는 식입니다.

이러한 작업들이 _더 많은_ 작업을 예약할 수 있고 **poll** 단계에서 처리되는 새로운 이벤트들이 커널에 의해 큐에 추가되기 때문에, 폴링 이벤트가 처리되는 동안에도 poll 이벤트가 큐에 추가될 수 있습니다. 결과적으로 오래 실행되는 콜백은 타이머의 임계값보다 훨씬 더 오래 poll 단계가 실행되도록 할 수 있습니다. 자세한 내용은 [**timers**](#timers)와 [**poll**](#poll) 섹션을 참조하세요.

> Windows와 Unix/Linux 구현 사이에 약간의 차이가 있지만, 이 설명에서는 중요하지 않습니다. 가장 중요한 부분은 여기에 있습니다. 실제로는 7~8단계가 있지만, 우리가 관심을 가져야 할 부분 - Node.js가 실제로 사용하는 부분 - 은 위에서 설명한 것들입니다.

## 단계 개요

- **timers**: 이 단계는 `setTimeout()`과 `setInterval()`로 예약된 콜백을 실행합니다.
- **pending callbacks**: 다음 루프 반복으로 연기된 I/O 콜백을 실행합니다.
- **idle, prepare**: 내부적으로만 사용됩니다.
- **poll**: 새로운 I/O 이벤트를 검색합니다; I/O 관련 콜백을 실행합니다(close 콜백, 타이머로 예약된 것들, 그리고 `setImmediate()`를 제외한 거의 모든 것); 적절한 경우 node는 여기서 블록됩니다.
- **check**: `setImmediate()` 콜백이 여기서 호출됩니다.
- **close callbacks**: 일부 close 콜백들, 예: `socket.on('close', ...)`

이벤트 루프의 각 실행 사이에 Node.js는 비동기 I/O나 타이머를 기다리고 있는지 확인하고, 없다면 깔끔하게 종료됩니다.

## 단계 상세

### timers

타이머는 제공된 콜백이 실행 _될 수 있는_ **임계값**을 지정하며, 사람이 _원하는_ **정확한** 시간을 지정하는 것이 아닙니다. 타이머 콜백은 지정된 시간이 지난 후 가능한 한 빨리 실행되도록 예약됩니다. 하지만 운영 체제 스케줄링이나 다른 콜백의 실행으로 인해 지연될 수 있습니다.

> 기술적으로는 [**poll** 단계](#poll)가 타이머가 실행되는 시점을 제어합니다.

예를 들어, 100ms 임계값으로 실행되도록 타이머를 예약했고, 스크립트가 비동기적으로 파일을 읽는데 95ms가 걸린다고 가정해 보겠습니다:

```js
const fs = require('node:fs');

function someAsyncOperation(callback) {
  // 이 작업은 95ms가 걸린다고 가정합니다
  fs.readFile('/path/to/file', callback);
}

const timeoutScheduled = Date.now();

setTimeout(() => {
  const delay = Date.now() - timeoutScheduled;

  console.log(`${delay}ms have passed since I was scheduled`);
}, 100);

// 95ms가 걸리는 비동기 작업을 수행합니다
someAsyncOperation(() => {
  const startCallback = Date.now();

  // 10ms 동안 아무 작업도 하지 않습니다
  while (Date.now() - startCallback < 10) {
    // 아무 작업도 하지 않습니다
  }
});
```

이벤트 루프가 **poll** 단계에 진입할 때 큐가 비어있는 상태입니다
(`fs.readFile()`이 아직 완료되지 않음). 따라서 가장 가까운 타이머의 임계값에 도달할 때까지 남은 시간(ms)만큼 대기합니다. 95ms 동안 대기하는 동안 `fs.readFile()`이 파일 읽기를 완료하고, 완료하는 데 10ms가 걸리는 콜백이 **poll** 큐에 추가되어 실행됩니다.
콜백이 완료되면 큐에 더 이상 콜백이 없으므로, 이벤트 루프는 가장 가까운 타이머의 임계값에 도달했음을 확인하고 **timers** 단계로 돌아가 타이머의 콜백을 실행합니다. 이 예제에서는 타이머가 예약된 시점부터 콜백이 실행될 때까지의 총 지연 시간이 105ms가 됨을 볼 수 있습니다.

> **poll** 단계가 이벤트 루프를 고갈시키는 것을 방지하기 위해, [libuv][]
> (Node.js의 이벤트 루프와 플랫폼의 모든 비동기 동작을 구현하는 C 라이브러리)는
> 더 많은 이벤트를 폴링하기 전에 시스템에 따른 하드 최대값을 가지고 있습니다.

### pending callbacks

이 단계는 TCP 에러와 같은 일부 시스템 작업에 대한 콜백을 실행합니다. 예를 들어 TCP 소켓이 연결을 시도할 때 `ECONNREFUSED`를 받으면, 일부 *nix 시스템에서는 에러를 보고하기 전에 대기하려고 합니다. 이는 **pending callbacks** 단계에서 실행되도록 큐에 추가됩니다.

### poll

**poll** 단계에는 두 가지 주요 기능이 있습니다:

1. I/O를 위해 얼마나 오래 블록하고 폴링할지 계산한 다음
2. **poll** 큐에 있는 이벤트들을 처리합니다.

이벤트 루프가 **poll** 단계에 진입할 때 _예약된 타이머가 없다면_, 다음 두 가지 중 하나가 발생합니다:

- _만약 **poll** 큐가 **비어있지 않다면**_, 이벤트 루프는 큐가 소진되거나 시스템에 따른 하드 제한에 도달할 때까지 콜백들을 동기적으로 실행하며 큐를 순회합니다.

- _만약 **poll** 큐가 **비어있다면**_, 다음 두 가지 중 하나가 발생합니다:

  - `setImmediate()`로 스크립트가 예약되어 있다면, 이벤트 루프는 **poll** 단계를 종료하고 **check** 단계로 이동하여 예약된 스크립트들을 실행합니다.

  - `setImmediate()`로 스크립트가 **예약되어 있지 않다면**, 이벤트 루프는 콜백이 큐에 추가되기를 기다린 다음 즉시 실행합니다.

**poll** 큐가 비어있으면 이벤트 루프는 _시간 임계값에 도달한_ 타이머들이 있는지 확인합니다. 하나 이상의 타이머가 준비되어 있다면, 이벤트 루프는 **timers** 단계로 돌아가 해당 타이머들의 콜백을 실행합니다.

### check

이 단계에서는 **poll** 단계가 완료된 직후에 콜백을 실행할 수 있습니다. **poll** 단계가 유휴 상태가 되고 `setImmediate()`로 스크립트가 큐에 추가되어 있다면, 이벤트 루프는 대기하지 않고 **check** 단계로 진행할 수 있습니다.

`setImmediate()`는 실제로 이벤트 루프의 별도 단계에서 실행되는 특별한 타이머입니다. **poll** 단계가 완료된 후 콜백을 실행하도록 예약하는 libuv API를 사용합니다.

일반적으로 코드가 실행되면서 이벤트 루프는 결국 **poll** 단계에 도달하여 들어오는 연결이나 요청 등을 기다립니다. 하지만 `setImmediate()`로 콜백이 예약되어 있고 **poll** 단계가 유휴 상태가 되면, **poll** 이벤트를 기다리지 않고 **poll** 단계를 종료하고 **check** 단계로 진행합니다.

### close callbacks

소켓이나 핸들이 갑자기 닫히면(예: `socket.destroy()`), `'close'` 이벤트가 이 단계에서 발생합니다. 그렇지 않으면 `process.nextTick()`을 통해 발생합니다.

## `setImmediate()` vs `setTimeout()`

`setImmediate()`와 `setTimeout()`은 비슷하지만, 호출되는 시점에 따라 다르게 동작합니다.

- `setImmediate()`는 현재 **poll** 단계가 완료되면 스크립트를 실행하도록 설계되었습니다.
- `setTimeout()`은 지정된 최소 시간(ms)이 경과한 후에 스크립트가 실행되도록 예약합니다.

타이머가 실행되는 순서는 호출되는 컨텍스트에 따라 달라집니다. 두 타이머가 메인 모듈 내에서 호출되면, 타이밍은 프로세스의 성능에 영향을 받게 됩니다(이는 시스템에서 실행 중인 다른 애플리케이션의 영향을 받을 수 있습니다).

예를 들어, I/O 사이클 내에 있지 않은(즉, 메인 모듈) 다음 스크립트를 실행하면, 두 타이머가 실행되는 순서는 프로세스의 성능에 따라 결정되므로 비결정적입니다:

```js
// timeout_vs_immediate.js
setTimeout(() => {
  console.log('timeout');
}, 0);

setImmediate(() => {
  console.log('immediate');
});
```

```bash
$ node timeout_vs_immediate.js
timeout
immediate

$ node timeout_vs_immediate.js
immediate
timeout
```

하지만 두 호출을 I/O 사이클 내로 이동하면, immediate 콜백이 항상 먼저 실행됩니다:

```js
// timeout_vs_immediate.js
const fs = require('node:fs');

fs.readFile(__filename, () => {
  setTimeout(() => {
    console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('immediate');
  });
});
```

```bash
$ node timeout_vs_immediate.js
immediate
timeout

$ node timeout_vs_immediate.js
immediate
timeout
```

I/O 사이클 내에서 예약된 경우, 존재하는 타이머의 수와 관계없이 `setImmediate()`가 항상 모든 타이머보다 먼저 실행된다는 점이 `setTimeout()`보다 `setImmediate()`를 사용하는 주된 장점입니다.

## `process.nextTick()`

### `process.nextTick()` 이란?

`process.nextTick()`이 비동기 API의 일부임에도 다이어그램에 표시되지 않은 것을 눈치채셨을 수 있습니다. 이는 `process.nextTick()`이 기술적으로 이벤트 루프의 일부가 아니기 때문입니다. 대신, `nextTickQueue`는 이벤트 루프의 현재 단계와 관계없이 현재 작업이 완료된 후에 처리됩니다. 여기서 _작업_이란 기본 C/C++ 핸들러로부터의 전환과 실행되어야 할 JavaScript의 처리로 정의됩니다.

다이어그램을 다시 살펴보면, 특정 단계에서 `process.nextTick()`을 호출할 때마다 `process.nextTick()`에 전달된 모든 콜백은 이벤트 루프가 계속되기 전에 처리됩니다. 이는 재귀적인 `process.nextTick()` 호출을 통해 **I/O를 "기아 상태"로 만들 수 있기 때문에** 좋지 않은 상황을 초래할 수 있습니다. 이는 이벤트 루프가 **poll** 단계에 도달하는 것을 방해합니다.

### 왜 이것이 허용되는 걸까요?

왜 이런 것이 Node.js에 포함되어 있을까요? 이는 API가 비동기일 필요가 없는 경우에도 항상 비동기여야 한다는 설계 철학의 일부입니다. 다음 코드 예제를 살펴보세요:

```js
function apiCall(arg, callback) {
  if (typeof arg !== 'string')
    return process.nextTick(
      callback,
      new TypeError('argument should be string'),
    );
}
```

이 코드는 인자를 검사하고 올바르지 않은 경우 콜백에 오류를 전달합니다. API가 최근에 업데이트되어 `process.nextTick()`에 콜백 이후의 인자들을 전달할 수 있게 되었습니다. 이를 통해 콜백에 전달된 인자들이 그대로 전파되므로 함수를 중첩할 필요가 없어졌습니다.

우리가 하고 있는 것은 사용자의 나머지 코드가 실행되도록 허용한 _후에만_ 사용자에게 오류를 전달하는 것입니다. `process.nextTick()`을 사용함으로써 `apiCall()`이 항상 사용자의 나머지 코드 _이후_ 그리고 이벤트 루프가 진행되기 _전에_ 콜백을 실행하도록 보장합니다. 이를 위해 JS 호출 스택이 풀린 다음 즉시 제공된 콜백을 실행하도록 허용되며, 이를 통해 `process.nextTick()`을 재귀적으로 호출할 때 v8의 `RangeError: Maximum call stack size exceeded` 오류에 도달하지 않고도 호출할 수 있습니다.

이러한 철학은 잠재적으로 문제가 될 수 있는 상황을 초래할 수 있습니다.
다음 코드 조각을 예로 들어보겠습니다:

```js
let bar;

// 이는 비동기 시그니처를 가지고 있지만, 콜백을 동기적으로 호출합니다
function someAsyncApiCall(callback) {
  callback();
}

// 콜백은 `someAsyncApiCall`이 완료되기 전에 호출됩니다.
someAsyncApiCall(() => {
  // someAsyncApiCall이 완료되지 않았기 때문에 bar에는 아직 어떤 값도 할당되지 않았습니다
  console.log('bar', bar); // undefined
});

bar = 1;
```

사용자가 `someAsyncApiCall()`을 비동기 시그니처를 가지도록 정의했지만, 실제로는 동기적으로 동작합니다. 이 함수가 호출되면 `someAsyncApiCall()`에 제공된 콜백이 실제로 비동기 작업을 수행하지 않기 때문에 이벤트 루프의 동일한 단계에서 호출됩니다. 그 결과, 스크립트가 완전히 실행되지 않아 `bar` 변수가 아직 스코프에 없을 수 있음에도 콜백이 이 변수를 참조하려고 시도합니다.

콜백을 `process.nextTick()`에 배치함으로써, 스크립트는 여전히 완전히 실행될 수 있으며, 이를 통해 콜백이 호출되기 전에 모든 변수, 함수 등이 초기화될 수 있습니다. 또한 이벤트 루프가 계속 진행되는 것을 막을 수 있다는 장점이 있습니다. 이벤트 루프가 계속되기 전에 사용자에게 오류를 알리는 것이 유용할 수 있습니다. 다음은 `process.nextTick()`을 사용한 이전 예제입니다:

```js
let bar;

function someAsyncApiCall(callback) {
  process.nextTick(callback);
}

someAsyncApiCall(() => {
  console.log('bar', bar); // 1
});

bar = 1;
```

다른 실제 예제를 살펴보겠습니다:

```js
const server = net.createServer(() => {}).listen(8080);

server.on('listening', () => {});
```

포트만 전달되면 포트가 즉시 바인딩됩니다. 따라서 `'listening'` 콜백은 즉시 호출될 수 있습니다. 문제는 `.on('listening')` 콜백이 그때까지 설정되지 않는다는 것입니다.

이를 해결하기 위해 `'listening'` 이벤트는 `nextTick()`에서 큐에 추가되어 스크립트가 완전히 실행될 수 있도록 합니다. 이를 통해 사용자가 원하는 모든 이벤트 핸들러를 설정할 수 있습니다.

## `process.nextTick()` vs `setImmediate()`

사용자 입장에서는 비슷해 보이는 두 가지 호출이 있지만, 이름이 혼란스럽습니다.

- `process.nextTick()`은 같은 단계에서 즉시 실행됩니다
- `setImmediate()`는 이벤트 루프의 다음 반복 또는 '틱'에서 실행됩니다

본질적으로 이 이름들은 서로 바뀌어야 했습니다. `process.nextTick()`이 `setImmediate()`보다 더 즉시 실행되지만, 이는 과거의 유산이며 바뀔 가능성이 낮습니다. 이러한 변경은 npm의 많은 패키지들을 손상시킬 수 있습니다. 매일 새로운 모듈들이 추가되고 있어서, 기다리는 시간이 길어질수록 잠재적인 손상도 더 많이 발생할 수 있습니다. 혼란스럽긴 하지만 이름 자체는 바뀌지 않을 것입니다.

> 개발자들이 모든 경우에 `setImmediate()`를 사용하는 것을 권장합니다. 이해하기가 더 쉽기 때문입니다.

## 왜 `process.nextTick()`을 사용할까요?

두 가지 주요 이유가 있습니다:

1. 이벤트 루프가 계속되기 전에 사용자가 오류를 처리하고, 불필요한 리소스를 정리하거나,
   필요한 경우 요청을 다시 시도할 수 있도록 합니다.

2. 때로는 콜 스택이 풀린 후 이벤트 루프가 계속되기 전에 
   콜백이 실행되도록 해야 할 필요가 있습니다.

사용자의 기대를 충족시키는 것이 한 가지 예시입니다. 간단한 예제를 보겠습니다:

```js
const server = net.createServer();
server.on('connection', (conn) => {});

server.listen(8080);
server.on('listening', () => {});
```

`listen()`이 이벤트 루프의 시작 부분에서 실행되지만 listening 콜백이 `setImmediate()`에 배치된다고 가정해보겠습니다. 호스트 이름이 전달되지 않는 한, 포트 바인딩은 즉시 발생합니다. 이벤트 루프가 진행되려면 **poll** 단계에 도달해야 하는데, 이는 listening 이벤트보다 먼저 connection 이벤트가 발생할 수 있는 가능성이 있다는 것을 의미합니다.

또 다른 예시는 `EventEmitter`를 확장하고 생성자 내부에서 이벤트를 발생시키는 것입니다:

```js
const EventEmitter = require('node:events');

class MyEmitter extends EventEmitter {
  constructor() {
    super();
    this.emit('event');
  }
}

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('an event occurred!');
});
```

생성자에서 즉시 이벤트를 발생시킬 수 없습니다.
스크립트가 사용자가 해당 이벤트에 콜백을 할당하는 시점까지 
처리되지 않았기 때문입니다. 따라서 생성자 내부에서 
`process.nextTick()`을 사용하여 생성자가 완료된 후 이벤트를 
발생시키는 콜백을 설정할 수 있으며, 이는 예상된 결과를 제공합니다:

```js
const EventEmitter = require('node:events');

class MyEmitter extends EventEmitter {
  constructor() {
    super();

    // 핸들러가 할당되면 nextTick을 사용하여 이벤트를 발생시킵니다
    process.nextTick(() => {
      this.emit('event');
    });
  }
}

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('an event occurred!');
});
```

[libuv]: https://libuv.org/
[REPL]: https://nodejs.org/api/repl.html#repl_repl
