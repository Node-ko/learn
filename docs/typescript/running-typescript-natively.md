---
title: TypeScript 네이티브로 실행하기
layout: learn
authors: AugustinMauroy
---

# TypeScript 네이티브로 실행하기
> ❗️ *번역 날짜: 2024년 12월 16일* <br>
> 공식 문서 원문은 아래를 참고하세요.<br>
> [Running TypeScript Natively](https://nodejs.org/en/learn/typescript/run-natively)

> **⚠️주의⚠️:** 이 글의 모든 콘텐츠는 Node.js 실험적 기능을 사용합니다. 이 문서에 언급된 기능을 지원하는 Node.js 버전을 사용하고 있는지 확인하세요. 그리고 실험적 기능은 향후 버전의 Node.js에서 변경될 수 있다는 점을 기억하세요.

이전 글에서는 트랜스파일과 러너를 사용하여 TypeScript 코드를 실행하는 방법을 배웠습니다. 이번 글에서는 Node.js 자체를 사용하여 TypeScript 코드를 실행하는 방법을 알아보겠습니다.

## Node.js로 TypeScript 코드 실행하기

V22.6.0부터 Node.js는 “타입 스트리핑”을 통해 일부 TypeScript 구문을 실험적으로 지원합니다. 유효한 TypeScript 코드를 먼저 트랜스파일할 필요 없이 Node.js에서 바로 작성할 수 있습니다.

`--experimental-strip-types` 플래그는 Node.js가 실행하기 전에 TypeScript 코드에서 타입 주석을 제거하도록 지시합니다.

```bash
node --experimental-strip-types example.ts
```

이제 끝입니다! 이제 TypeScript 코드를 먼저 트랜스파일할 필요 없이 Node.js에서 바로 실행할 수 있으며, TypeScript 사용하여 유형 관련 오류를 포착할 수 있습니다.

V22.7.0에서는 이 실험적 지원이 `--experimental-transform-types` 플래그를 추가하여 `enum` 및 `namespace`와 같은 TypeScript 전용 구문을 변환하는 데까지 확장되었습니다.

```bash
node --experimental-strip-types --experimental-transform-types another-example.ts
```

향후 버전에서는 명령줄 플래그가 필요 없는 TypeScript 지원할 예정입니다.

## 제한 사항

이 글을 쓰는 시점에서 Node.js의 TypeScript 대한 실험적 지원에는 몇 가지 제한 사항이 있습니다.

자세한 내용은 [API 문서](https://nodejs.org/docs/latest/api/typescript.html#typescript-features)에서 확인할 수 있습니다.

## 중요 참고 사항

이 기능을 가능하게 해준 모든 기여자에게 감사드립니다. 이 기능이 조만간 Node.js LTS 버전에서 안정적으로 제공될 수 있기를 바랍니다.

이 기능은 실험적인 기능으로 몇 가지 제한 사항이 있으므로 사용 사례에 적합하지 않은 경우 다른 기능을 사용하거나 수정 사항을 기여해 주시기 바랍니다. 버그 보고도 환영하지만, 이 프로젝트는 어떠한 종류의 보증도 없이 자원 봉사자들에 의해 운영되므로 직접 수정 사항을 제공할 수 없는 경우 조금만 기다려주세요.
