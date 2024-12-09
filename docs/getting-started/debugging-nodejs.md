---
title: Node.js 디버깅
layout: learn
---

# Node.js 디버깅

> ❗️ _번역 날짜: 2024년 12월 09일_ <br>
> 공식 문서 원문은 아래를 참고하세요.<br> > [Debugging Node.js](https://nodejs.org/en/learn/getting-started/debugging)

이 가이드는 Node.js 애플리케이션과 스크립트를 디버깅하는 방법을 소개합니다.

## 인스펙터 활성화

`--inspect` 스위치와 함께 시작하면 Node.js 프로세스는 디버깅 클라이언트를 위한 리스닝 상태가 됩니다. 기본적으로 `127.0.0.1:9229` 호스트와 포트에서 리스닝합니다. 각 프로세스는 고유한 [UUID](https://tools.ietf.org/html/rfc4122)도 할당됩니다.

인스펙터 클라이언트는 호스트 주소, 포트, UUID를 알고 지정해야 연결할 수 있습니다. 전체 URL은 다음과 같은 형식을 가집니다:
`ws://127.0.0.1:9229/0f2c936f-b1cd-4ac9-aab3-f63b0f33d55e`.

Node.js는 또한 `SIGUSR1` 신호를 받을 때 디버깅 메시지에 대한 리스닝을 시작합니다. (Windows에서는 `SIGUSR1`이 지원되지 않습니다.) Node.js 7 이하에서는 레거시 디버거 API가 활성화됩니다. Node.js 8 이상에서는 인스펙터 API가 활성화됩니다.

## 보안 고려사항

디버거는 Node.js 실행 환경에 대한 전체 액세스 권한을 가지고 있기 때문에, 이 포트에 연결할 수 있는 악의적인 행위자는 Node.js 프로세스의 대리로 임의의 코드를 실행할 수 있습니다. 디버거 포트를 공용 및 사설 네트워크에 노출하는 보안 위험을 이해하는 것이 중요합니다.

### 디버그 포트를 공용으로 노출하는 것은 안전하지 않습니다

디버거가 공용 IP 주소 또는 `0.0.0.0`에 바인딩된 경우, IP 주소에 도달할 수 있는 모든 클라이언트는 제한 없이 디버거에 연결하고 임의 코드를 실행할 수 있습니다.

기본적으로 `node --inspect`는 `127.0.0.1`에 바인딩됩니다. 외부 연결을 허용하려면 명시적으로 공용 IP 주소 또는 `0.0.0.0` 등을 지정해야 합니다. 그러나 이는 상당한 보안 위협에 노출될 수 있으므로 적절한 방화벽 및 액세스 제어를 보장해야 합니다.

[원격 디버깅 활성화](#원격-디버깅-활성화) 섹션을 참고하여 원격 디버거 클라이언트가 안전하게 연결할 수 있는 방법에 대한 조언을 확인하세요.

### 로컬 애플리케이션은 인스펙터에 완전한 액세스 권한을 가집니다

인스펙터 포트를 기본값인 `127.0.0.1`에 바인딩하더라도, 로컬 머신에서 실행되는 모든 애플리케이션은 제한 없이 액세스할 수 있습니다. 이는 로컬 디버거가 편리하게 연결할 수 있도록 설계된 것입니다.

### 브라우저, WebSocket 및 동일 출처 정책

웹 브라우저에서 열린 웹사이트는 브라우저 보안 모델 하에서 WebSocket 및 HTTP 요청을 수행할 수 있습니다. 고유한 디버거 세션 ID를 얻기 위해 초기 HTTP 연결이 필요합니다. 동일 출처 정책은 웹사이트가 이 HTTP 연결을 생성하지 못하도록 방지합니다. [DNS 재바인딩 공격](https://en.wikipedia.org/wiki/DNS_rebinding)에 대한 추가적인 보안을 위해 Node.js는 연결의 ‘Host’ 헤더가 IP 주소 또는 정확히 localhost를 지정했는지 확인합니다.

이러한 보안 정책은 호스트 이름을 지정하여 원격 디버그 서버에 연결하는 것을 허용하지 않습니다. 이 제한을 우회하려면 IP 주소를 지정하거나 아래에 설명된 대로 SSH 터널을 사용하는 방법을 사용할 수 있습니다.

## 인스펙터 클라이언트

`node inspect myscript.js`로 최소 CLI 디버거를 사용할 수 있습니다. Node.js 인스펙터에 연결할 수 있는 상용 및 오픈 소스 도구도 있습니다.

### Chrome DevTools 55+, Microsoft Edge

- **옵션 1**: Chromium 기반 브라우저에서 `chrome://inspect`를 열거나 Edge에서 `edge://inspect`를 엽니다. Configure 버튼을 클릭하여 대상 호스트와 포트가 나열되어 있는지 확인합니다.
- **옵션 2**: `/json/list`의 출력에서 `devtoolsFrontendUrl`을 복사하거나 --inspect 힌트 텍스트를 복사하여 Chrome에 붙여넣습니다.

자세한 내용은 [Chrome DevTools Frontend](https://github.com/ChromeDevTools/devtools-frontend) 및 [Microsoft Edge Insider](https://www.microsoftedgeinsider.com)를 참조하세요.

### Visual Studio Code 1.10+

- 디버그 패널에서 설정 아이콘을 클릭하여 `.vscode/launch.json`을 엽니다. 초기 설정으로 "Node.js"를 선택합니다.

자세한 내용은 [Visual Studio Code](https://github.com/microsoft/vscode)를 참조하세요.

### Visual Studio 2017+

- "Debug > Start Debugging"을 선택하거나 F5를 누릅니다.
- [자세한 지침](https://github.com/Microsoft/nodejstools/wiki/Debugging)

### JetBrains WebStorm 및 기타 JetBrains IDE

- 새 Node.js 디버그 구성을 생성하고 **Debug** 버튼을 누릅니다. Node.js 7+에서는 기본적으로 `--inspect`가 사용됩니다. 이를 비활성화하려면 IDE 레지스트리에서 `js.debugger.node.use.inspect`의 선택을 해제하십시오. WebStorm 및 기타 JetBrains IDE에서 Node.js를 실행하고 디버깅하는 방법에 대해 자세히 알아보려면 [WebStorm 온라인 도움말](https://www.jetbrains.com/help/webstorm/running-and-debugging-node-js.html)을 참조하세요.

### chrome-remote-interface

- [Inspector Protocol](https://chromedevtools.github.io/debugger-protocol-viewer/v8/) 엔드포인트와의 연결을 쉽게 하기 위한 라이브러리.

자세한 내용은 [chrome-remote-interface](https://github.com/cyrus-and/chrome-remote-interface)를 참조하세요.

### Gitpod

- `Debug` 보기에서 Node.js 디버그 구성을 시작하거나 `F5`를 누릅니다. [자세한 지침](https://medium.com/gitpod/debugging-node-js-applications-in-theia-76c94c76f0a1).

자세한 내용은 [Gitpod](https://www.gitpod.io)를 참조하세요.

### Eclipse IDE with Eclipse Wild Web Developer extension

- `.js` 파일에서 "Debug As... > Node program"을 선택하거나,
- 디버거를 실행 중인 Node.js 애플리케이션(`--inspect`로 이미 시작됨)에 연결하는 디버그 구성을 생성합니다.

자세한 내용은 [Eclipse IDE](https://eclipse.org/eclipseide)를 참조하세요.

## 명령줄 옵션

다음 표는 다양한 런타임 플래그가 디버깅에 미치는 영향을 나열합니다:

| 플래그                             | 의미                                                                                                                                                                           |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| --inspect                          | 인스펙터 에이전트를 활성화합니다. 기본 주소 및 포트(127.0.0.1:9229)에서 리스닝합니다.                                                                                          |
| --inspect=[host:port]              | 인스펙터 에이전트를 활성화합니다. 지정된 주소 또는 호스트(기본값: 127.0.0.1)에 바인딩하고, 지정된 포트(기본값: 9229)에서 리스닝합니다.                                         |
| --inspect-brk                      | 인스펙터 에이전트를 활성화하고 기본 주소 및 포트(127.0.0.1:9229)에서 리스닝하며, 사용자 코드가 시작되기 전에 중단합니다.                                                       |
| --inspect-brk=[host:port]          | 인스펙터 에이전트를 활성화하고 지정된 주소 또는 호스트(기본값: 127.0.0.1)에 바인딩하고, 지정된 포트(기본값: 9229)에서 리스닝하며, 사용자 코드가 시작되기 전에 중단합니다.      |
| --inspect-wait                     | 인스펙터 에이전트를 활성화하고 기본 주소 및 포트(127.0.0.1:9229)에서 리스닝하며, 디버거가 연결될 때까지 대기합니다.                                                            |
| --inspect-wait=[host:port]         | 인스펙터 에이전트를 활성화하고 지정된 주소 또는 호스트(기본값: 127.0.0.1)에 바인딩하고, 지정된 포트(기본값: 9229)에서 리스닝하며, 디버거가 연결될 때까지 대기합니다.           |
| node inspect script.js             | `--inspect` 플래그로 사용자의 스크립트를 실행하기 위해 하위 프로세스를 생성하며, CLI 디버거를 실행하는 메인 프로세스를 사용합니다.                                             |
| node inspect --port=xxxx script.js | `--inspect` 플래그로 사용자의 스크립트를 실행하기 위해 하위 프로세스를 생성하며, CLI 디버거를 실행하는 메인 프로세스를 사용합니다. 지정된 포트(기본값: 9229)에서 리스닝합니다. |

## 원격 디버깅 시나리오 활성화

디버거를 공용 IP 주소에서 리스닝하도록 설정하지 않는 것을 권장합니다. 원격 디버깅 연결을 허용해야 하는 경우 SSH 터널을 사용하는 것이 좋습니다. 아래는 참고용 예제입니다. 원격 액세스를 허용하기 전에 관련된 보안 위험을 반드시 이해하십시오.

Node.js를 실행 중인 원격 머신 `remote.example.com`에서 디버깅을 원한다고 가정합니다. 해당 머신에서 Node.js 프로세스를 로컬호스트에서만 리스닝하도록 시작해야 합니다(기본 설정).

```bash
node --inspect server.js
```

이제 로컬 머신에서 디버그 클라이언트 연결을 시작하려면 SSH 터널을 설정할 수 있습니다:

```bash
ssh -L 9221:localhost:9229 user@remote.example.com
```

이 SSH 터널 세션은 로컬 머신의 포트 9221에 대한 연결을 원격 머신 `remote.example.com`의 포트 9229로 포워딩합니다. 이제 Chrome DevTools 또는 Visual Studio Code와 같은 디버거를 `localhost:9221`에 연결하면, Node.js 애플리케이션이 로컬에서 실행 중인 것처럼 디버깅할 수 있습니다.

## 레거시 디버거

**레거시 디버거는 Node.js 7.7.0부터 사용 중단되었습니다. 대신 `--inspect` 및 Inspector를 사용하세요.**

Node.js 7 이하 버전에서 **--debug** 또는 **--debug-brk** 스위치로 시작하면 기본적으로 TCP 포트 `5858`에서 중단된 V8 디버깅 프로토콜 명령을 수신합니다. 이 프로토콜을 지원하는 모든 디버거 클라이언트는 실행 중인 프로세스에 연결하여 디버깅할 수 있습니다. 다음은 몇 가지 인기 있는 클라이언트입니다.

V8 디버깅 프로토콜은 더 이상 유지되거나 문서화되지 않습니다.

### 내장 디버거

`node debug script_name.js` 명령을 실행하여 내장 명령줄 디버거에서 스크립트를 시작합니다. 스크립트는 `--debug-brk` 옵션으로 시작된 별도의 Node.js 프로세스에서 실행되며, 초기 Node.js 프로세스는 `_debugger.js` 스크립트를 실행하고 대상에 연결합니다. 자세한 내용은 [문서](https://nodejs.org/dist/latest/docs/api/debugger.html)를 참조하세요.

### node-inspector

Chromium에서 사용하는 [Inspector Protocol](https://chromedevtools.github.io/debugger-protocol-viewer/v8/)을 Node.js의 V8 디버거 프로토콜로 변환하는 중간 프로세스를 사용하여 Chrome DevTools에서 Node.js 애플리케이션을 디버깅합니다. 자세한 내용은 [node-inspector](https://github.com/node-inspector/node-inspector)를 참조하세요.
