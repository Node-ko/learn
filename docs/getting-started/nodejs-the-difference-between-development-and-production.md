---
title: Node.js에서 개발 환경과 프로덕션 환경의 차이
layout: learn
authors: flaviocopes, MylesBorins, fhemberger, LaRuaNa, ahmadawais, RenanTKN, mcollina
---

# Node.js에서 개발 환경과 프로덕션 환경의 차이
> ❗️ *번역 날짜: 2024년 12월 2일* <br>
> 공식 문서 원문은 아래를 참고하세요.<br>
>[Node.js, the difference between development and production](https://nodejs.org/en/learn/getting-started/nodejs-the-difference-between-development-and-production)

**Node.js 자체에는 개발 환경과 프로덕션 환경 간의 차이가 없습니다.** 즉, Node.js를 프로덕션 환경에서 실행하기 위해 특별한 설정을 적용할 필요가 없습니다.

하지만 npm 레지스트리에 있는 일부 라이브러리는 `NODE_ENV` 변수를 사용하며, 기본적으로 `development` 설정을 사용합니다. 따라서 항상 `NODE_ENV=production` 설정으로 Node.js를 실행하는 것이 좋습니다. 이는 [12 요소 방법론](https://12factor.net/)을 따라 애플리케이션을 구성하는 일반적인 방법입니다.

항상 `NODE_ENV=production` 설정으로 Node.js를 실행하세요.

[12 요소 방법론](https://12factor.net/)을 사용하여 애플리케이션을 구성하는 일반적인 방법입니다.

## Express 프레임워크에서의 NODE_ENV

[Express](https://expressjs.com/) 프레임워크에서 `NODE_ENV`를 `production`으로 설정하면 다음과 같은 효과를 얻을 수 있습니다:

- **로깅 수준 최소화**: 불필요한 로그를 줄여 성능을 향상시킵니다.
- **성능 최적화**: 캐싱을 강화하여 응답 속도를 높입니다.

이 설정은 보통 다음 명령어를 통해 적용합니다.

```bash
export NODE_ENV=production
```

쉘에서 직접 설정하는 것 외에도, 쉘 구성 파일(예: Bash의 `.bash_profile` 또는 `.bashrc`)에 설정을 추가하는 것이 좋습니다. 그렇지 않으면 시스템을 재시작할 때 설정이 유지되지 않습니다.

또한, 애플리케이션을 실행할 때 환경 변수를 함께 설정할 수도 있습니다:

```bash
NODE_ENV=production node app.js
```

예를 들어, Express 애플리케이션에서는 환경에 따라 다른 오류 처리기를 설정할 수 있습니다.

```js
if (process.env.NODE_ENV === 'development') {
  app.use(express.errorHandler({ dumpExceptions: true, showStack: true }));
}

if (process.env.NODE_ENV === 'production') {
  app.use(express.errorHandler());
}
```

또 다른 예로, [Pug](https://pugjs.org)는 [Express](https://expressjs.com)에서 사용되는 템플릿 엔진으로, `NODE_ENV`가 `production`으로 설정되지 않으면 디버그 모드로 컴파일됩니다. Express 뷰는 개발 모드에서는 매 요청마다 컴파일되지만, 프로덕션 모드에서는 캐싱됩니다. 이 외에도 다양한 라이브러리에서 `NODE_ENV`를 활용하여 환경에 맞는 최적화를 수행합니다.

**이 환경 변수는 외부 라이브러리에서 널리 사용되는 관례이지만, Node.js 자체에는 특별한 영향을 미치지 않습니다.**

## 왜 NODE_ENV가 안티패턴(Anti-Pattern)인가요?

**환경(Environment)** 이란 엔지니어가 소프트웨어 제품을 빌드, 테스트, 배포 및 관리할 수 있는 디지털 플랫폼 또는 시스템을 의미합니다. 일반적으로 다음 네 가지 단계 또는 유형의 환경에서 애플리케이션이 실행됩니다.

- Development (개발)
- Testing (테스트)
- Staging (스테이징)
- Production (프로덕션)

`NODE_ENV`의 주요 문제는 개발자가 최적화와 소프트웨어 동작을 환경 변수와 직접 결합하는 것입니다. 그 결과는 다음과 같은 코드가 됩니다:

```js
if (process.env.NODE_ENV === 'development') {
  // ...
}

if (process.env.NODE_ENV === 'production') {
  // ...
}

if (['production', 'staging'].includes(process.env.NODE_ENV)) {
  // ...
}
```

이 코드는 한눈에 보기에는 문제가 없어 보일 수 있지만, 프로덕션과 스테이징 환경을 다르게 처리하게 되어 신뢰할 수 있는 테스트를 어렵게 만듭니다. 예를 들어, 테스트는 `NODE_ENV`가 `development`로 설정되었을 때는 성공할 수 있지만, `production`으로 설정되었을 때는 실패할 수 있습니다.

따라서, `NODE_ENV`를 `production`이 아닌 다른 값으로 설정하여 환경에 따른 동작을 분기하는 것은 *안티패턴*으로 간주됩니다. 이는 코드의 복잡성을 증가시키고, 예기치 않은 동작을 초래할 수 있기 때문입니다.
