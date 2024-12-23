---
title: Node.js의 Event emitter
layout: learn
authors: flaviocopes, MylesBorins, fhemberger, LaRuaNa, ahmadawais, ovflowd
---

> ❗️ _번역 날짜: 2024년 12월 17일_ <br />
> 공식 문서 원문은 아래를 참고하세요.<br /> > [The Node.js Event emitter](https://nodejs.org/en/learn/asynchronous-work/the-nodejs-event-emitter#the-nodejs-event-emitter)

# Node.js의 Event emitter

브라우저에서 JavaScript를 사용해 봤다면, 마우스 클릭, 키보드 버튼 입력, 마우스 움직임에 반응하기 등 정말 많은 상호작용이 이벤트를 통해 처리된다는 것을 알고 있을 것입니다.

백엔드에서는 Node.js가 [`events` module](https://nodejs.org/api/events.html) 을 통해 유사한 시스템을 구축할 수 있는 옵션을 제공합니다.

이 모듈에서 특히 중요한 것은 이벤트를 처리하기 위한 `EventEmitter` 클래스입니다.

다음과 같이 초기화할 수 있습니다.

```js
const EventEmitter = require('node:events');

const eventEmitter = new EventEmitter();
```

```js
import EventEmitter from 'node:events';

const eventEmitter = new EventEmitter();
```

이 객체는 여러 메서드를 노출하며, 그중에서 중요한 `on` 과 `emit` 메서드가 있습니다.

- `emit`: 이벤트를 **발생**시키는 데 사용됩니다.
- `on`: 이벤트가 발생했을 때 실행할 **콜백 함수**를 추가하는 데 사용됩니다.

예를 들어, `start` 라는 이벤트를 생성하고, 콘솔에 출력함으로써 이벤트에 반응해봅시다.

```js
eventEmitter.on('start', () => {
  console.log('started');
});
```

이후 다음을 실행하면

```js
eventEmitter.emit('start');
```

이벤트 핸들러 함수가 트리거되며 콘솔에 로그가 출력됩니다.

`emit()` 메서드에 추가적인 인자를 전달하여 이벤트 핸들러에 값을 넘길 수 있습니다.

```js
eventEmitter.on('start', (number) => {
  console.log(`started ${number}`);
});

eventEmitter.emit('start', 23);
```

여러 개의 인자 전달하기

```js
eventEmitter.on('start', (start, end) => {
  console.log(`started from ${start} to ${end}`);
});

eventEmitter.emit('start', 1, 100);
```

EventEmitter 객체는 이벤트에 상호작용할 수 있는 몇 가지 다른 메서드도 제공합니다:

- `once()`: 한 번만 실행되는(one-time) 이벤트 리스너를 추가합니다.
- `removeListener()` / `off()`: 이벤트에서 특정 리스너를 제거합니다.
- `removeAllListeners()`: 특정 이벤트에 등록된 모든 리스너를 제거합니다.

이 메서드들에 대한 더 자세한 내용은 [공식 문서](https://nodejs.org/api/events.html) 에서 확인할 수 있습니다.
