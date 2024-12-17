---
title: 블로킹과 논블로킹의 개요
layout: learn
authors: ovflowd, HassanBahati
---

# 블로킹과 논블로킹의 개요
> ❗️ *번역 날짜: 2024년 12월 17일* <br>
> 공식 문서 원문은 아래를 참고하세요.<br>
> [Overview of Blocking vs Non-Blocking](https://nodejs.org/en/learn/asynchronous-work/overview-of-blocking-vs-non-blocking)

이 개요에서는 Node.js에서의 **블로킹** 과 **논블로킹**의 차이점에 대해 다룹니다. 이 과정에서 이벤트 루프와 libuv에 대해 언급하지만, 이에 대한 사전 지식은 필요하지 않습니다. 독자는 자바스크립트 언어와 Node.js의 [callback pattern](/learn/asynchronous-work/javascript-asynchronous-programming-and-callbacks)에 대한 기본적인 이해가 있다고 가정합니다.

> "I/O"는 주로 [libuv](https://libuv.org/)에서 지원하는 시스템 디스크 및 네트워크와의 상호 작용을 의미합니다.

## 블로킹

**블로킹**이란 Node.js 프로세스에서 추가적인 자바스크립트 실행이 비-자바스크립트 작업이 완료될 때까지 기다려야 하는 상황을 말합니다. 이는 **blocking** 작업이 진행되는 동안 이벤트 루프가 자바스크립트 실행을 계속할 수 없기 때문에 발생합니다.

Node.js에서 CPU 집약적인 작업으로 인해 성능이 저하되는 자바스크립트는, I/O와 같은 비-자바스크립트 작업을 대기하는 경우와는 달리 일반적으로 **블로킹**이라고 부르지 않습니다. Node.js 표준 라이브러리의 동기 메서드 중 libuv를 사용하는 메서드가 가장 흔히 사용되는 **블로킹** 작업에 해당합니다. 또한, 네이티브 모듈도 **블로킹** 메서드를 포함할 수 있습니다.

Node.js 표준 라이브러리의 모든 I/O 메서드는 비동기 버전을 제공하며, 이는 **논블로킹** 방식으로 동작하고 콜백 함수를 지원합니다. 일부 메서드는 이름이 `Sync`로 끝나는 **블로킹** 버전도 제공합니다.

## 코드 비교

**블로킹** 메서드는 **동기적**으로 실행되며, **논블로킹** 메서드는 **비동기적**으로 실행됩니다.

파일 시스템 모듈(File System module)을 예로 들면, 다음은 **동기적** 파일 읽기 방식입니다:

```js
const fs = require('node:fs');

const data = fs.readFileSync('/file.md'); // 파일을 읽을 때까지 여기에서 실행이 멈춥니다.
```

다음은 이에 해당하는 **비동기적** 예제입니다:

```js
const fs = require('node:fs');

fs.readFile('/file.md', (err, data) => {
  if (err) throw err;
});
```

첫 번째 예제는 두 번째 예제보다 간단해 보이지만, 파일 전체가 읽힐 때까지 두 번째 줄이 실행을 **블로킹**하여 추가적인 자바스크립트 실행이 멈추는 단점이 있습니다. 동기식 버전에서는 오류가 발생하면 이를 처리해야 하며, 그렇지 않으면 프로세스가 종료됩니다. 반면, 비동기식 버전에서는 오류를 발생시킬지 여부를 작성자가 결정해야 합니다.

예제를 조금 확장해 봅시다:

```js
const fs = require('node:fs');

const data = fs.readFileSync('/file.md'); // 여기에서 파일이 읽힐 때까지 실행이 멈춥니다
console.log(data);
moreWork(); // console.log가 표시된 후에 실행됩니다
```

다음은 유사하지만 완전히 동일하지 않은 비동기적 예제입니다:

```js
const fs = require('node:fs');

fs.readFile('/file.md', (err, data) => {
  if (err) throw err;
  console.log(data);
});
moreWork(); // console.log가 표시되기 전에 실행됩니다.
```

위의 첫 번째 예제에서는 `console.log`가 `moreWork()`보다 먼저 호출됩니다. 두 번째 예제에서는 `fs.readFile()`가 **논블로킹**이기 때문에 자바스크립트 실행이 계속 진행될 수 있고, 그 결과 `moreWork()`가 먼저 호출됩니다. 파일 읽기가 완료될 때까지 기다리지 않고 `moreWork()`를 실행할 수 있는 능력은 더 높은 처리량을 가능하게 하는 중요한 설계 선택입니다.

## 동시성과 처리량

Node.js에서 자바스크립트 실행은 단일 스레드에서 이루어지므로, 동시성은 다른 작업을 완료한 후 이벤트 루프가 자바스크립트 콜백 함수를 실행할 수 있는 기능을 의미합니다. 동시적으로 실행될 것으로 예상되는 코드는 I/O와 같은 비-자바스크립트 작업이 발생하는 동안 이벤트 루프가 계속 실행될 수 있도록 해야 합니다.

예를 들어, 웹 서버에 대한 각 요청이 완료되는 데 50ms가 걸리고, 그 50ms 중 45ms는 비동기적으로 처리할 수 있는 데이터베이스 I/O 작업이라고 가정해봅시다. **논블로킹** 비동기 작업을 선택하면 각 요청당 45ms를 다른 요청을 처리하는 데 할애할 수 있게 됩니다. **블로킹** 메서드 대신 **논블로킹** 메서드를 사용함으로써 처리 용량에 큰 차이가 발생합니다.

이벤트 루프는 많은 다른 언어에서 동시 작업을 처리하기 위해 추가 스레드를 생성하는 모델과는 다릅니다.

## 블로킹 코드와 논블로킹 코드를 혼합하는 위험성

I/O 작업을 처리할 때 피해야 할 몇 가지 패턴이 있습니다. 예시를 살펴보겠습니다:

```js
const fs = require('node:fs');

fs.readFile('/file.md', (err, data) => {
  if (err) throw err;
  console.log(data);
});
fs.unlinkSync('/file.md');
```

위의 예제에서 `fs.unlinkSync()`는 `fs.readFile()`보다 먼저 실행될 가능성이 높아, 실제로 파일을 읽기 전에 `file.md`가 삭제될 수 있습니다. 이를 개선한 방법은 완전히 **논블로킹** 방식으로, 올바른 순서대로 실행될 수 있도록 보장됩니다. 아래와 같이 작성할 수 있습니다:

```js
const fs = require('node:fs');

fs.readFile('/file.md', (readFileErr, data) => {
  if (readFileErr) throw readFileErr;
  console.log(data);
  fs.unlink('/file.md', unlinkErr => {
    if (unlinkErr) throw unlinkErr;
  });
});
```

위 코드는 `fs.readFile()`의 콜백 내에서 **논블로킹** 방식으로 `fs.unlink()` 호출을 배치하여 작업 순서가 올바르게 실행되도록 보장합니다.

## 추가 자료

- [libuv](https://libuv.org/)