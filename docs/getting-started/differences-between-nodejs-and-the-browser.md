---
title: Node.js와 브라우저의 차이점
layout: learn
authors: flaviocopes, ollelauribostrom, MylesBorins, fhemberger, LaRuaNa, ahmadawais, karlhorky
---

# Node.js와 브라우저의 차이점
> ❗️ *번역 날짜: 2024년 12월 03일* <br>
> 공식 문서 원문은 아래를 참고하세요.<br>
> [Differences between Node.js and the Browser](https://nodejs.org/en/learn/getting-started/differences-between-nodejs-and-the-browser)

Node.js와 브라우저는 모두 JavaScript를 프로그래밍 언어로 사용합니다. 브라우저에서 실행되는 앱을 개발하는 것과 Node.js 애플리케이션을 개발하는 것은 완전히 다른 경험입니다. JavaScript를 사용한다는 점은 같지만, 몇 가지 주요 차이점이 이러한 경험을 근본적으로 다르게 만듭니다.

JavaScript를 주로 사용하는 프론트엔드 개발자의 관점에서 보면, Node.js 애플리케이션은 큰 장점을 제공합니다. 바로 프론트엔드와 백엔드를 단일 언어로 프로그래밍할 수 있다는 편리함입니다.

프로그래밍 언어를 완벽하고 깊이 있게 배우는 것이 얼마나 어려운지 잘 알고 있으며, 동일한 언어를 사용하여 클라이언트와 서버 모두에서 웹의 모든 작업을 수행함으로써 여러분은 특별한 이점을 누릴 수 있습니다.

> **변화하는 것은 생태계입니다.**

브라우저에서는 대부분 DOM 또는 쿠키와 같은 다른 웹 플랫폼 API와 상호 작용하는 경우가 많습니다. 물론 Node.js에는 이러한 것들이 존재하지 않습니다. 브라우저에서 제공하는 `document`, `window` 및 기타 모든 객체가 없습니다.

또한 브라우저에는 파일 시스템 액세스 기능과 같이 Node.js가 모듈을 통해 제공하는 멋진 API가 모두 포함되어 있지 않습니다.

또 다른 큰 차이점은 Node.js에서는 사용자가 환경을 제어할 수 있다는 점입니다. 누구나 어디서나 배포할 수 있는 오픈 소스 애플리케이션을 구축하는 경우가 아니라면, 어떤 버전의 Node.js에서 애플리케이션을 실행할지 알 수 있습니다. 방문자가 사용할 브라우저를 선택할 수 없는 브라우저 환경과 비교하면 매우 편리합니다.

즉, Node.js 버전이 지원하는 모든 최신 ES2015+ JavaScript를 작성할 수 있습니다. 자바스크립트는 매우 빠르게 발전하지만 브라우저는 업그레이드 속도가 다소 느리기 때문에 웹에서 이전 자바스크립트 / ECMAScript 릴리스를 사용해야 하는 경우가 있습니다. 브라우저에 코드를 전송하기 전에 Babel을 사용하여 코드를 ES5와 호환되도록 변환할 수 있지만, Node.js에서는 그럴 필요가 없습니다.

또 다른 차이점으로 Node.js는 CommonJS와 ES 모듈 시스템을 모두 지원하지만(Node.js v12부터), 브라우저에서는 ES 모듈 표준이 구현되기 시작했다는 점입니다.

실제로는 Node.js에서 `require()`와 `import`를 모두 사용할 수 있지만 브라우저에서는 `import`로 제한됩니다.
