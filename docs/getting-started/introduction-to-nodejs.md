---
title: Node.js 소개
layout: learn
authors: flaviocopes, potch, MylesBorins, RomainLanz, virkt25, Trott, onel0p3z, ollelauribostrom, MarkPieszak, fhemberger, LaRuaNa, FrozenPandaz, mcollina, amiller-gh, ahmadawais, saqibameen, dangen-effy, aymen94, benhalverson
---

# Node.js 소개

> ❗️ _번역 날짜: 2024년 12월 09일_ <br>
> 공식 문서 원문은 아래를 참고하세요.<br> > [Introduction to Node.js](https://nodejs.org/en/learn/getting-started/introduction-to-nodejs)

Node.js는 오픈 소스이자 크로스 플랫폼 JavaScript 런타임 환경입니다. 다양한 프로젝트에서 사용할 수 있는 인기 있는 도구입니다!

Node.js는 Google Chrome의 핵심인 V8 JavaScript 엔진을 브라우저 외부에서 실행합니다. 이를 통해 Node.js는 매우 뛰어난 성능을 발휘합니다.

Node.js 애플리케이션은 단일 프로세스에서 실행되며, 요청마다 새로운 스레드를 생성하지 않습니다. Node.js는 표준 라이브러리에서 비동기 I/O 기본 요소를 제공하여 JavaScript 코드가 차단되는 것을 방지합니다. 일반적으로 Node.js의 라이브러리는 비차단 패러다임으로 작성되어 있으며, 차단 동작은 예외적인 경우에만 발생합니다.

Node.js가 네트워크 읽기, 데이터베이스 액세스, 파일 시스템 접근과 같은 I/O 작업을 수행할 때 스레드를 차단하고 CPU 사이클을 낭비하며 대기하는 대신, 응답이 반환되면 작업을 재개합니다.

이러한 구조 덕분에 Node.js는 스레드 동시성을 관리할 필요 없이 단일 서버에서 수천 개의 동시 연결을 처리할 수 있습니다. 스레드 동시성 관리는 종종 많은 버그의 원인이 될 수 있습니다.

또한 Node.js는 브라우저에서 JavaScript를 작성하던 수백만 명의 프론트엔드 개발자가 클라이언트 측 코드뿐만 아니라 서버 측 코드도 작성할 수 있도록 해주는 독특한 장점이 있습니다. 이를 위해 완전히 새로운 언어를 배울 필요가 없습니다.

Node.js에서는 새로운 ECMAScript 표준을 문제없이 사용할 수 있습니다. 모든 사용자가 브라우저를 업데이트하기를 기다릴 필요가 없기 때문입니다. Node.js 버전을 변경하거나 특정 실험적 기능을 플래그로 활성화하여 사용할 ECMAScript 버전을 결정할 수 있습니다.

## Node.js 애플리케이션 예제

Node.js의 가장 간단한 예제는 웹 서버입니다:

#### CommonJS 방식 (cjs)

```javascript
const { createServer } = require('node:http');

const hostname = '127.0.0.1';
const port = 3000;

const server = createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

#### ES 모듈 방식 (mjs)

```javascript
import { createServer } from 'node:http';

const hostname = '127.0.0.1';
const port = 3000;

const server = createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

위 코드를 실행하려면 파일을 `server.js`로 저장한 후 터미널에서 `node server.js`를 실행하십시오. ES 모듈 방식 코드를 사용하는 경우, 파일을 `server.mjs`로 저장하고 `node server.mjs`를 실행해야 합니다.

이 코드는 Node.js의 [`http` 모듈](https://nodejs.org/api/http.html)을 먼저 포함합니다.

Node.js는 네트워킹에 대한 일류 지원을 포함하여 환상적인 [표준 라이브러리](https://nodejs.org/api/)를 제공합니다.

`http`의 `createServer()` 메서드는 새로운 HTTP 서버를 생성하여 반환합니다.

서버는 지정된 포트와 호스트 이름에서 수신 대기하도록 설정됩니다. 서버가 준비되면 콜백 함수가 호출되어 서버가 실행 중임을 알려줍니다.

새 요청이 수신될 때마다 [`request` 이벤트](https://nodejs.org/api/http.html#http_event_request)가 호출되며, 두 개의 객체가 제공됩니다: 요청 객체([`http.IncomingMessage`](https://nodejs.org/api/http.html#http_class_http_incomingmessage))와 응답 객체([`http.ServerResponse`](https://nodejs.org/api/http.html#http_class_http_serverresponse))입니다.

이 두 객체는 HTTP 호출을 처리하는 데 필수적입니다.

첫 번째 객체는 요청 세부 정보를 제공합니다. 이 간단한 예제에서는 사용하지 않지만, 요청 헤더와 요청 데이터를 액세스할 수 있습니다.

두 번째 객체는 호출자에게 데이터를 반환하는 데 사용됩니다.

다음 코드를 통해:

```javascript
res.statusCode = 200;
```

응답 상태 코드를 200으로 설정하여 성공적인 응답임을 나타냅니다.

Content-Type 헤더를 설정합니다:

```javascript
res.setHeader('Content-Type', 'text/plain');
```

그리고 `end()`에 콘텐츠를 인수로 추가하여 응답을 종료합니다:

```javascript
res.end('Hello World\n');
```
