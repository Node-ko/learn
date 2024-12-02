---
title: V8 JavaScript 엔진
layout: learn
authors: flaviocopes, smfoote, co16353sidak, MylesBorins, LaRuaNa, andys8, ahmadawais, karlhorky, aymen94
---

# V8 JavaScript 엔진
> ❗️ *번역 날짜: 2024년 12월 2일* <br>
> 공식 문서 원문은 아래를 참고하세요.<br>
>[The V8 JavaScript Engine](https://nodejs.org/en/learn/getting-started/the-v8-javascript-engine)

V8은 Google Chrome에서 사용되는 JavaScript 엔진입니다. 이는 JavaScript 코드를 분석하고 실행하여 Chrome 브라우저에서 웹 페이지를 구동합니다.

V8은 JavaScript 엔진으로, JavaScript 코드를 파싱하고 실행합니다. DOM 및 기타 Web Platform APIs는 브라우저가 제공하는 런타임 환경을 구성합니다.

특히, JavaScript 엔진이 호스팅되는 브라우저와 독립적으로 동작한다는 점이 매력적입니다. 이 중요한 기능 덕분에 Node.js의 인기가 증가했습니다. V8은 2009년에 Node.js를 지원하기 위해 선택되었으며, Node.js의 인기가 급증하면서 V8은 이제 JavaScript로 작성된 많은 서버 측 코드를 지원하는 엔진으로 자리잡았습니다.

Node.js 생태계는 커지고 있으며, V8 덕분에 데스크톱 애플리케이션도 지원할 수 있게 되었습니다. 또한 Electron과 같은 프로젝트가 이를 더욱 확장하고 있습니다.

## 다른 JavaScript 엔진

다른 브라우저들은 자체 JavaScript 엔진을 가지고 있습니다:

- Firefox는 [**SpiderMonkey**](https://spidermonkey.dev)를 가지고 있습니다.
- Safari는 [**JavaScriptCore**](https://developer.apple.com/documentation/javascriptcore) (또한 Nitro라고도 함)를 가지고 있습니다.
- Edge는 원래 [**Chakra**](https://github.com/Microsoft/ChakraCore)를 기반으로 했지만, 최근에는 [Chromium](https://support.microsoft.com/en-us/help/4501095/download-the-new-microsoft-edge-based-on-chromium)을 기반으로 재구축되었고 V8 엔진을 사용합니다.

그 외에도 많은 엔진들이 존재합니다.

모든 엔진들은 [ECMA ES-262 표준](https://www.ecma-international.org/publications/standards/Ecma-262.htm)을 구현합니다. 이것은 JavaScript에서 사용되는 표준입니다.

## 성능 경쟁

V8은 C++로 작성되었고, 지속적으로 개선되고 있습니다. 이것은 포터블하며 Mac, Windows, Linux 및 여러 다른 시스템에서 실행됩니다.

이 V8 소개에서는 V8의 구현 세부 내용은 언급하지 않을 것입니다: 이러한 내용들은 공식 사이트에서 찾을 수 있습니다 (예: [V8 공식 사이트](https://v8.dev/)), 그리고 시간이 지남에 따라 자주 변경됩니다.

V8은 항상 다른 JavaScript 엔진들과 함께 발전하고 있습니다. 이는 웹과 Node.js 생태계의 성능을 높이기 위해서입니다.

웹에서는 연속적으로 성능 경쟁이 이루어지고 있습니다. 이는 사용자와 개발자에게 큰 이점을 줍니다. 매년 더 빠르고 최적화된 소프트웨어의 성능 향상을 얻을 수 있기 때문입니다.

## 컴파일

JavaScript는 일반적으로 해석되는 언어로 간주되지만, 최근의 JavaScript 엔진들은 더 이상 해석만 하는 것이 아니라 컴파일합니다.

이는 2009년에 Firefox 3.5에 스파이더몽키 JavaScript 컴파일러가 추가되었을 때부터 발생하고 있습니다. 모든 사람들이 이 아이디어를 따랐습니다.

JavaScript는 V8에 의해 **just-in-time** (JIT) **compilation**을 통해 내부적으로 컴파일되어 실행 속도를 높입니다.

이것은 2004년에 Google Maps가 도입된 이후 JavaScript가 일반적으로 몇 십 줄의 코드를 실행하는 것에서 브라우저에서 수천 또는 수백만 줄의 코드를 실행하는 완전한 애플리케이션으로 진화하는 것을 의미합니다.

우리의 애플리케이션들은 이제 브라우저 내에서 몇 시간 동안 실행될 수 있으며, 단순한 형식 검증 규칙이나 간단한 스크립트가 아닙니다.

이 *새로운 세상* 에서, JavaScript를 컴파일하는 것은 엄청난 의미가 있습니다. 왜냐하면 JavaScript 코드가 실행되기 전에 잠깐의 준비 과정(컴파일)이 필요하지만, 완료되면 순수하게 해석된 코드보다 훨씬 더 성능이 좋아질 것이기 때문입니다.
