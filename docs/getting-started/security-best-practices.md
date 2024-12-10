---
title: 보안 모범 사례
layout: learn
authors: RafaelGSS, UlisesGascon, fraxken, facutuesca, mhdawson, arhart, naugtur, anonrig
---

# 보안 모범 사례

> ❗️ _번역 날짜: 2024년 12월 09일_ <br>
> 공식 문서 원문은 아래를 참고하세요.<br> > [Security Best Practices](https://nodejs.org/en/learn/getting-started/security-best-practices)

## 의도

이 문서는 현재 [위협 모델][threat model]을 확장하고 Node.js 애플리케이션을 보호하는 방법에 대한 광범위한 지침을 제공합니다.

## 문서 내용

- 모범 사례: 모범 사례를 간단하게 요약하여 보여줍니다. [해당 이슈][security guidance issue]나 [해당 지침][nodejs guideline]을 시작점으로 사용할 수 있습니다. 이 문서는 Node.js에 특정한 내용임을 유의해야 하며, 보다 포괄적인 내용을 원한다면 [OSSF 모범 사례][OSSF Best Practices]를 고려하세요.
- 공격 설명: 위협 모델에서 언급된 공격을 평이한 영어로 설명하고, 가능하다면 일부 코드 예제와 함께 문서화합니다.
- 서드파티 라이브러리: 위협(타이포스쿼팅 공격, 악성 패키지 등)을 정의하고 Node.js 모듈 종속성 관련 모범 사례를 제공합니다.

## 위협 목록

### HTTP 서버 서비스 거부(CWE-400)

이 공격은 애플리케이션이 들어오는 HTTP 요청을 처리하는 방식으로 인해 설계된 목적으로 사용할 수 없게 되는 경우입니다. 이 요청은 악의적인 행위자가 의도적으로 제작하지 않아도, 잘못 구성되었거나 결함이 있는 클라이언트가 서버에 특정 패턴의 요청을 보내서 서비스 거부를 일으킬 수 있습니다.

HTTP 요청은 Node.js HTTP 서버에서 수신되며 등록된 요청 핸들러를 통해 애플리케이션 코드에 전달됩니다. 서버는 요청 본문의 내용을 구문 분석하지 않습니다. 따라서 요청 본문 내용으로 인해 발생하는 서비스 거부는 Node.js 자체의 취약점이 아니며, 이는 애플리케이션 코드가 올바르게 처리해야 할 책임입니다.

웹 서버가 소켓 오류를 적절히 처리하도록 설정해야 합니다. 예를 들어, 서버를 오류 핸들러 없이 생성하면 서비스 거부에 취약해질 수 있습니다.

```cjs
const net = require('node:net');

const server = net.createServer(function (socket) {
  // socket.on('error', console.error) // 이것이 서버 충돌을 방지합니다
  socket.write('Echo server
');
  socket.pipe(socket);
});

server.listen(5000, '0.0.0.0');
```

```mjs
import net from 'node:net';

const server = net.createServer(function (socket) {
  // socket.on('error', console.error) // 이것이 서버 충돌을 방지합니다
  socket.write('Echo server
');
  socket.pipe(socket);
});

server.listen(5000, '0.0.0.0');
```

잘못된 요청(_bad request_)이 수행되면 서버가 충돌할 수 있습니다.

내용이 아닌 방식으로 발생하는 서비스 거부 공격의 예로는 [Slowloris][]가 있습니다. 이 공격에서는 HTTP 요청이 느리고 단편적으로 전송되며, 한 번에 한 조각씩 전달됩니다. 요청이 완전히 전달될 때까지 서버는 진행 중인 요청에 전용 리소스를 유지합니다. 이러한 요청이 충분히 동시에 전송되면 동시 연결의 최대치를 초과하여 서비스 거부가 발생합니다. 이 공격은 요청 내용이 아니라 요청의 타이밍 및 패턴에 따라 달라집니다.

**완화 방안**

- 요청을 수신하고 Node.js 애플리케이션으로 전달하는 리버스 프록시를 사용합니다. 리버스 프록시는 캐싱, 로드 밸런싱, IP 차단 등의 기능을 제공하여 서비스 거부 공격의 효과를 줄일 수 있습니다.
- 서버 시간 초과를 올바르게 구성하여 유휴 상태의 연결이나 요청이 너무 느리게 도착하는 연결을 중단할 수 있도록 합니다. [`http.Server`][`http.Server`]의 `headersTimeout`, `requestTimeout`, `timeout`, `keepAliveTimeout`과 같은 다양한 시간 초과 설정을 참조하세요.
- 호스트별 및 전체 열려 있는 소켓 수를 제한합니다. [http 문서][http docs]에서 `agent.maxSockets`, `agent.maxTotalSockets`, `agent.maxFreeSockets`, `server.maxRequestsPerSocket`을 참조하세요.

### DNS 재바인딩(CWE-346)

이 공격은 [--inspect 스위치][--inspect switch]를 사용하여 디버깅 인스펙터가 활성화된 Node.js 애플리케이션을 대상으로 할 수 있습니다.

웹 브라우저에서 열린 웹사이트는 WebSocket 및 HTTP 요청을 생성할 수 있으므로 로컬에서 실행 중인 디버깅 인스펙터를 대상으로 할 수 있습니다. 이는 일반적으로 최신 브라우저에서 구현된 [동일 출처 정책][same-origin policy]에 의해 방지됩니다. 동일 출처 정책은 악성 웹사이트가 로컬 IP 주소에서 요청된 데이터를 읽지 못하도록 차단합니다.

그러나 DNS 재바인딩을 통해 공격자는 요청이 로컬 IP 주소에서 시작된 것처럼 보이도록 출처를 임시로 제어할 수 있습니다. 이를 위해 웹사이트와 해당 IP 주소를 해석하는 DNS 서버를 모두 제어합니다. 자세한 내용은 [DNS Rebinding 위키][DNS Rebinding wiki]를 참조하세요.

**완화 방안**

- `process.on('SIGUSR1', ...)` 리스너를 연결하여 _SIGUSR1_ 신호에서 인스펙터를 비활성화합니다.
- 프로덕션 환경에서 인스펙터 프로토콜을 실행하지 마십시오.

### 민감 정보 노출(CWE-552)

현재 디렉토리에 포함된 모든 파일과 폴더는 패키지 게시 중 npm 레지스트리에 푸시됩니다.

`.npmignore` 및 `.gitignore`를 사용하여 차단 목록을 정의하거나 `package.json`에서 허용 목록을 정의하여 이 동작을 제어할 수 있습니다.

**완화 방안**

- `npm publish --dry-run`을 사용하여 게시할 파일 목록을 확인합니다. 패키지 게시 전에 콘텐츠를 검토하십시오.
- `.gitignore` 및 `.npmignore`와 같은 무시(ignore) 파일을 작성하고 유지 관리하는 것도 중요합니다. 이러한 파일을 통해 게시하지 말아야 할 파일/폴더를 지정할 수 있습니다. `package.json`의 [files 속성][files property]을 사용하면 반대로 허용 목록을 정의할 수 있습니다.
- 노출이 발생한 경우, [패키지 게시 취소][unpublish the package]를 수행하십시오.

### HTTP 요청 스머글링(CWE-444)

이 공격은 두 HTTP 서버(일반적으로 프록시와 Node.js 애플리케이션)를 포함합니다. 클라이언트는 프론트엔드 서버(프록시)를 거쳐 백엔드 서버(애플리케이션)로 리디렉션되는 HTTP 요청을 보냅니다. 프론트엔드와 백엔드가 모호한 HTTP 요청을 다르게 해석하면, 공격자는 프론트엔드에서 보이지 않지만 백엔드에서 보이는 악성 메시지를 보낼 수 있습니다. 이는 사실상 프록시 서버를 "우회"하여 전달됩니다.

[CWE-444][]에서 더 상세한 설명과 예제를 확인할 수 있습니다.

이 공격은 Node.js가 HTTP 요청을 다른 HTTP 서버와 다르게 해석하는 경우에 의존하므로, 성공적인 공격은 Node.js, 프론트엔드 서버 또는 둘 모두의 취약성 때문일 수 있습니다. 요청이 HTTP 명세서(예: [RFC7230][])와 일치하도록 Node.js에서 해석된다면 이는 Node.js의 취약성으로 간주되지 않습니다.

**완화 방안**

- HTTP 서버를 생성할 때 `insecureHTTPParser` 옵션을 사용하지 마십시오.
- 프론트엔드 서버를 구성하여 모호한 요청을 정규화합니다.
- Node.js 및 선택한 프론트엔드 서버의 새로운 HTTP 요청 스머글링 취약성을 지속적으로 모니터링합니다.
- 가능하면 HTTP/2를 끝단까지 사용하고 HTTP 다운그레이드를 비활성화하십시오.

### 타이밍 공격을 통한 정보 노출(CWE-208)

이 공격은 애플리케이션이 요청에 응답하는 데 걸리는 시간을 측정하여 민감한 정보를 알아내는 공격입니다. 이 공격은 Node.js에 국한되지 않고 거의 모든 런타임을 대상으로 할 수 있습니다.

타이밍 공격은 애플리케이션이 시크릿을 타이밍 민감한 작업(예: 분기)에서 사용하는 경우 가능합니다. 예를 들어, 애플리케이션에서 인증을 처리할 때 이메일과 비밀번호를 자격 증명으로 사용하는 기본 인증 방법이 있습니다. 이때 데이터베이스(DBMS)에서 사용자 정보를 검색하고 내장된 문자열 비교 기능을 사용하여 응답 시간이 길어질 수 있습니다. 이를 통해 공격자는 비밀번호의 길이와 값을 추측할 수 있습니다.

**완화 방안**

- Crypto API는 `timingSafeEqual` 함수를 노출하여 실제 값과 예상 민감 값을 상수 시간 알고리즘으로 비교할 수 있습니다.
- 비밀번호 비교의 경우, 기본 crypto 모듈에서 사용할 수 있는 [scrypt][]를 활용하십시오.
- 더 일반적으로, 변수 시간 작업에서 시크릿 사용을 피하십시오. 여기에는 시크릿으로 분기하거나, 공격자가 동일한 인프라(예: 동일한 클라우드 머신)에서 위치할 수 있는 경우 메모리에서 시크릿을 인덱스로 사용하는 것도 포함됩니다. JavaScript에서 상수 시간 코드를 작성하는 것은 어렵습니다(JIT 때문). 암호화 애플리케이션의 경우, 기본 crypto API 또는 WebAssembly(네이티브에서 구현되지 않은 알고리즘)를 사용하십시오.

### 악성 서드파티 모듈(CWE-1357)

현재 Node.js에서는 모든 패키지가 네트워크 접근과 같은 강력한 리소스에 접근할 수 있습니다.
게다가, 파일 시스템에 접근할 수 있으므로 데이터를 어디든 보낼 수 있습니다.

Node 프로세스에서 실행되는 모든 코드는 `eval()`(또는 그와 동등한 기능)을 사용하여 추가 임의 코드를 로드하고 실행할 수 있습니다.
파일 시스템 쓰기 권한이 있는 모든 코드도 로드된 새 파일이나 기존 파일을 작성하여 동일한 작업을 수행할 수 있습니다.

Node.js는 로드된 리소스를 신뢰할 수 없는 상태로 또는 신뢰할 수 있는 상태로 선언할 수 있는 실험적인[¹][experimental-features]
[정책 메커니즘][policy mechanism]을 제공합니다. 하지만, 이 정책은 기본적으로 활성화되어 있지 않습니다.
종속성 버전을 고정하고 일반적인 워크플로 또는 npm 스크립트를 사용하여 자동 취약성 검사를 실행하세요.
패키지를 설치하기 전에, 해당 패키지가 유지 관리되고 있으며 예상했던 모든 콘텐츠를 포함하는지 확인하세요.
주의해야 할 점은 GitHub 소스 코드가 항상 게시된 코드와 동일하지 않을 수 있으므로 *node_modules*에서 이를 확인해야 합니다.

#### 공급망 공격

Node.js 애플리케이션에 대한 공급망 공격은 종속성(직접 또는 전이적) 중 하나가 손상되었을 때 발생합니다.
이는 종속성 명세에 대한 너무 느슨한 정의(원하지 않는 업데이트 허용) 또는 명세의 일반적인 오타([타이포스쿼팅][typosquatting] 취약점)를 통해 발생할 수 있습니다.

상위 패키지에 대한 제어권을 가진 공격자는 악성 코드를 포함한 새 버전을 게시할 수 있습니다.
Node.js 애플리케이션이 안전한 버전을 엄격히 지정하지 않고 해당 패키지를 의존한다면, 패키지가 최신 악성 버전으로 자동 업데이트되어 애플리케이션이 손상될 수 있습니다.

`package.json` 파일에 지정된 종속성은 정확한 버전 번호 또는 범위를 가질 수 있습니다.
그러나 종속성을 정확한 버전에 고정하더라도 전이적 종속성 자체는 고정되지 않습니다.
이는 여전히 원치 않는/예상치 못한 업데이트에 애플리케이션이 취약하다는 것을 의미합니다.

가능한 공격 벡터:

- 타이포스쿼팅 공격
- Lockfile 중독
- 손상된 유지보수자
- 악성 패키지
- 종속성 혼동

**완화 방안**

- `--ignore-scripts` 옵션을 사용하여 npm이 임의의 스크립트를 실행하지 못하도록 방지합니다.
  - 또한, `npm config set ignore-scripts true`를 사용하여 전역적으로 비활성화할 수 있습니다.
- 종속성 버전을 범위나 변경 가능한 소스에서 가져오는 버전이 아닌 특정 변경 불가능한 버전으로 고정합니다.
- 모든 종속성(직접 및 전이적)을 고정하는 lockfile을 사용합니다.
  - [Lockfile 중독 완화][Mitigations for lockfile poisoning]를 참고하세요.
- CI를 사용하여 [`npm-audit`][]와 같은 도구로 새 취약성을 자동으로 확인합니다.
  - [`Socket`][]와 같은 도구를 사용하여 정적 분석으로 네트워크 또는 파일 시스템 접근과 같은 위험한 동작을 찾을 수 있습니다.
- `npm install` 대신 [`npm ci`][]를 사용합니다.
  이는 _package.json_ 대신 lockfile을 무시하는 대신 lockfile과 _package.json_ 파일 간의 불일치가 오류를 발생시킵니다.
- _package.json_ 파일에서 종속성 이름에 오류/오타가 있는지 신중히 확인하세요.

### 메모리 접근 위반(CWE-284)

메모리 기반 또는 힙 기반 공격은 메모리 관리 오류와 악용 가능한 메모리 할당기의 조합에 의존합니다.
모든 런타임과 마찬가지로, 프로젝트가 공유 머신에서 실행되는 경우 Node.js도 이러한 공격에 취약합니다.
보안 힙을 사용하면 포인터 초과 및 언더런으로 인해 민감한 정보가 유출되는 것을 방지하는 데 유용합니다.

안타깝게도, 보안 힙은 Windows에서 사용할 수 없습니다.
Node.js [보안 힙 문서][secure-heap documentation]에서 더 많은 정보를 확인할 수 있습니다.

**완화 방안**

- 애플리케이션에 따라 `--secure-heap=n`을 사용하며, *n*은 할당된 최대 바이트 크기입니다.
- 프로덕션 앱을 공유 머신에서 실행하지 마십시오.

### 원숭이 패치(Monkey Patching) (CWE-349)

원숭이 패치는 실행 중에 속성을 수정하여 기존 동작을 변경하려는 것을 말합니다. 예시:

```js
// eslint-disable-next-line no-extend-native
Array.prototype.push = function (item) {
  // 글로벌 [].push 동작을 재정의
};
```

**완화 방안**

`--frozen-intrinsics` 플래그는 모든 기본 JavaScript 객체와 함수를 재귀적으로 동결하는 실험적인[¹][experimental-features] 플래그를 활성화합니다.
따라서, 다음 코드 스니펫은 `Array.prototype.push`의 기본 동작을 재정의할 수 **없습니다**.

```js
// eslint-disable-next-line no-extend-native
Array.prototype.push = function (item) {
  // 글로벌 [].push 동작을 재정의
};

// 에러 발생:
// TypeError <Object <Object <[Object: null prototype] {}>>>:
// Cannot assign to read only property 'push' of object ''
```

하지만, `globalThis`를 사용하여 여전히 새로운 글로벌 변수를 정의하고 기존 글로벌 변수를 교체할 수 있음을 언급하는 것이 중요합니다.

```console
> globalThis.foo = 3; foo; // 새로운 글로벌 변수를 정의 가능
3
> globalThis.Array = 4; Array; // 그러나 기존 글로벌 변수를 교체할 수도 있음
4
```

따라서, `Object.freeze(globalThis)`를 사용하여 어떤 글로벌 변수도 교체되지 않도록 보장할 수 있습니다.

### 프로토타입 오염 공격 (CWE-1321)

프로토타입 오염은 \_\_proto\__, \_constructor_, \_prototype\_ 및 기본 프로토타입에서 상속된 기타 속성의 사용을 악용하여
JavaScript 언어 항목에 속성을 수정하거나 삽입할 수 있는 가능성을 나타냅니다.

```js
const a = { a: 1, b: 2 };
const data = JSON.parse('{"__proto__": { "polluted": true}}');

const c = Object.assign({}, a, data);
console.log(c.polluted); // true

// 잠재적인 DoS
const data2 = JSON.parse('{"__proto__": null}');
const d = Object.assign(a, data2);
d.hasOwnProperty('b'); // Uncaught TypeError: d.hasOwnProperty is not a function
```

이것은 JavaScript 언어에서 유래된 잠재적인 취약점입니다.

**예시**:

- [CVE-2022-21824][] (Node.js)
- [CVE-2018-3721][] (서드파티 라이브러리: Lodash)

**완화 방안**

- [안전하지 않은 재귀 병합][insecure recursive merges]을 피하세요. [CVE-2018-16487][] 참조.
- 외부/신뢰할 수 없는 요청에 대해 JSON 스키마 검증을 구현하세요.
- `Object.create(null)`을 사용하여 프로토타입이 없는 객체를 생성하세요.
- 프로토타입을 동결합니다: `Object.freeze(MyObject.prototype)`.
- `--disable-proto` 플래그를 사용하여 `Object.prototype.__proto__` 속성을 비활성화하세요.
- 속성이 프로토타입이 아닌 객체에 직접 존재하는지 `Object.hasOwn(obj, keyFromObj)`를 사용하여 확인하세요.
- `Object.prototype`에서 메서드를 사용하는 것을 피하세요.

### 통제되지 않은 검색 경로 요소 (CWE-427)

Node.js는 [모듈 해석 알고리즘][Module Resolution Algorithm]을 따라 모듈을 로드합니다.
따라서, 요청된(require) 모듈의 디렉터리는 신뢰할 수 있다고 가정합니다.

이는 다음과 같은 애플리케이션 동작이 예상된다는 것을 의미합니다.
다음 디렉터리 구조를 가정합니다:

- _app/_
  - _server.js_
  - _auth.js_
  - _auth_

server.js가 `require('./auth')`를 사용하면 모듈 해석 알고리즘을 따라 _auth.js_ 대신 *auth*를 로드합니다.

**완화 방안**

실험적인[¹][experimental-features] [정책 메커니즘과 무결성 확인][policy mechanism with integrity checking]을 사용하면 위의 위협을 방지할 수 있습니다.
위의 디렉터리에 대해, 다음과 같은 `policy.json`을 사용할 수 있습니다:

```json
{
  "resources": {
    "./app/auth.js": {
      "integrity": "sha256-iuGZ6SFVFpMuHUcJciQTIKpIyaQVigMZlvg9Lx66HV8="
    },
    "./app/server.js": {
      "dependencies": {
        "./auth": "./app/auth.js"
      },
      "integrity": "sha256-NPtLCQ0ntPPWgfVEgX46ryTNpdvTWdQPoZO3kHo0bKI="
    }
  }
}
```

따라서, _auth_ 모듈을 요청할 때 시스템은 무결성을 확인하고 예상과 일치하지 않으면 오류를 발생시킵니다.

```console
» node --experimental-policy=policy.json app/server.js
node:internal/policy/sri:65
      throw new ERR_SRI_PARSE(str, str[prevIndex], prevIndex);
      ^

SyntaxError [ERR_SRI_PARSE]: Subresource Integrity string "sha256-iuGZ6SFVFpMuHUcJciQTIKpIyaQVigMZlvg9Lx66HV8=%" had an unexpected "%" at position 51
    at new NodeError (node:internal/errors:393:5)
    at Object.parse (node:internal/policy/sri:65:13)
    at processEntry (node:internal/policy/manifest:581:38)
    at Manifest.assertIntegrity (node:internal/policy/manifest:588:32)
    at Module._compile (node:internal/modules/cjs/loader:1119:21)
    at Module._extensions..js (node:internal/modules/cjs/loader:1213:10)
    at Module.load (node:internal/modules/cjs/loader:1037:32)
    at Module._load (node:internal/modules/cjs/loader:878:12)
    at Module.require (node:internal/modules/cjs/loader:1061:19)
    at require (node:internal/modules/cjs/helpers:99:18) {
  code: 'ERR_SRI_PARSE'
}
```

참고로, 정책 변조를 방지하기 위해 항상 `--policy-integrity`를 사용하는 것이 좋습니다.

## 프로덕션 환경에서의 실험적 기능

프로덕션 환경에서 실험적 기능을 사용하는 것은 권장되지 않습니다.
실험적 기능은 필요에 따라 중대한 변경 사항을 겪을 수 있으며, 그 기능이 안정적으로 보장되지 않습니다.
그러나 피드백은 매우 환영합니다.

## OpenSSF 도구

[OpenSSF][]는 특히 npm 패키지를 게시할 계획이 있는 경우 매우 유용한 여러 이니셔티브를 주도하고 있습니다. 이러한 이니셔티브는 다음을 포함합니다:

- [OpenSSF Scorecard][]: Scorecard는 일련의 자동화된 보안 위험 검사를 사용하여 오픈 소스 프로젝트를 평가합니다. 이를 통해 코드베이스의 취약성과 종속성을 사전 평가하고, 취약성을 수락할지에 대한 정보를 바탕으로 결정을 내릴 수 있습니다.
- [OpenSSF Best Practices Badge Program][]: 프로젝트는 각 모범 사례를 준수하는 방법을 설명하여 자발적으로 자가 인증할 수 있습니다. 이를 통해 프로젝트에 추가할 수 있는 배지가 생성됩니다.

[threat model]: https://github.com/nodejs/node/security/policy#the-nodejs-threat-model
[security guidance issue]: https://github.com/nodejs/security-wg/issues/488
[nodejs guideline]: https://github.com/goldbergyoni/nodebestpractices
[OSSF Best Practices]: https://github.com/ossf/wg-best-practices-os-developers
[Slowloris]: https://en.wikipedia.org/wiki/Slowloris_(computer_security)
[`http.Server`]: https://nodejs.org/api/http.html#class-httpserver
[http docs]: https://nodejs.org/api/http.html
[--inspect switch]: /learn/getting-started/debugging
[same-origin policy]: /learn/getting-started/debugging
[DNS Rebinding wiki]: https://en.wikipedia.org/wiki/DNS_rebinding
[files property]: https://docs.npmjs.com/cli/v8/configuring-npm/package-json#files
[unpublish the package]: https://docs.npmjs.com/unpublishing-packages-from-the-registry
[CWE-444]: https://cwe.mitre.org/data/definitions/444.html
[RFC7230]: https://datatracker.ietf.org/doc/html/rfc7230#section-3
[policy mechanism]: https://nodejs.org/api/permissions.html#policies
[typosquatting]: https://en.wikipedia.org/wiki/Typosquatting
[Mitigations for lockfile poisoning]: https://blog.ulisesgascon.com/lockfile-posioned
[`npm ci`]: https://docs.npmjs.com/cli/v8/commands/npm-ci
[secure-heap documentation]: https://nodejs.org/dist/latest-v18.x/docs/api/cli.html#--secure-heapn
[CVE-2022-21824]: https://www.cvedetails.com/cve/CVE-2022-21824/
[CVE-2018-3721]: https://www.cvedetails.com/cve/CVE-2018-3721/
[insecure recursive merges]: https://gist.github.com/DaniAkash/b3d7159fddcff0a9ee035bd10e34b277#file-unsafe-merge-js
[CVE-2018-16487]: https://www.cve.org/CVERecord?id=CVE-2018-16487
[scrypt]: https://nodejs.org/api/crypto.html#cryptoscryptpassword-salt-keylen-options-callback
[Module Resolution Algorithm]: https://nodejs.org/api/modules.html#modules_all_together
[policy mechanism with integrity checking]: https://nodejs.org/api/permissions.html#integrity-checks
[experimental-features]: #experimental-features-in-production
[`Socket`]: https://socket.dev/
[OpenSSF]: https://openssf.org/
[OpenSSF Scorecard]: https://securityscorecards.dev/
[OpenSSF Best Practices Badge Program]: https://bestpractices.coreinfrastructure.org/en
