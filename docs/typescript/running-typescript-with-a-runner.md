---
title: runner로 TypeScript 실행하기
layout: learn
authors: AugustinMauroy
---

# runner를 사용하여 TypeScript 실행하기

> ❗️ _번역 날짜: 2024년 12월 17일_ <br />
> 공식 문서 원문은 아래를 참고하세요.<br />
> [Running TypeScript with a runner](https://nodejs.org/en/learn/typescript/run#running-typescript-with-a-runner)

이전 글에서는 **트랜스파일링** (**transpilation**)을 사용해 TypeScript 코드를 실행하는 방법을 알아봤습니다. 이번 글에서는 어떻게 runner를 사용하여 TypeScript 코드를 실행하는지 배워보겠습니다.

## TypeScript 코드를 `ts-node` 를 이용해 실행하기

[ts-node](https://typestrong.org/ts-node/) 는 Node.js용 TypeScript 실행환경입니다. 이를 사용하면 코드를 먼저 컴파일하지 않고도 TypeScript 코드를 직접 Node.js에서 실행할 수 있습니다. 하지만 주의할 점은 `ts-node` 는 코드의 타입 체크를 수행하지 않는다는 것입니다. 따라서 코드를 배포하기 전에 `tsc` 를 사용해 타입 체크를 먼저 수행하고 `ts-node` 로 실행하는 것을 권장합니다.

**ts-node**를 사용하기 위해 먼저 이를 설치해야 합니다.

```bash
npm i -D ts-node
```

그리고 아래와 같이 TypeScript코드를 실행할 수 있습니다.

```bash
npx ts-node example.ts
```

## `tsx` 로 TypeScript 코드 실행하기

[tsx](https://tsx.is/) 는 또 다른 Node.js용 TypeScript 실행 환경입니다. `tsx` 를 사용하면 코드를 먼저 컴파일하지 않고도 TypeScript 코드를 직접 Node.js에서 실행할 수 있습니다. 하지만 tsx 역시 코드의 타입 체크를 수행하지 않습니다. 따라서 `tsc` 를 사용해 타입 체크를 먼저 수행하고 `tsx` 로 실행하는 것을 권장합니다.

`tsx` 를 사용하기 위해, 이를 먼저 이를 설치해야 합니다.

```bash
npm i -D tsx
```

그리고 아래와 같이 TypeScript코드를 실행할 수 있습니다.

```bash
npx tsx example.ts
```

## `node` 에서 `tsx` 등록하기

만약 `node` 에서 `tsx` 를 사용하고싶다면 `--import` 를 통해 `tsx` 를 등록할 수 있습니다.

```bash
node --import=tsx example.ts
```
