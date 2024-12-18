---
title: Introduction to TypeScript
layout: learn
authors: sbielenica, ovflowd, vaishnav-mk, AugustinMauroy
---

# TypeScript 소개

> ❗️ _번역 날짜: 2024년 12월 18일_ <br>
> 공식 문서 원문은 아래를 참고하세요.<br> > [Introduction to TypeScript](https://nodejs.org/en/learn/typescript/introduction)

## TypeScript란 무엇인가?

**[TypeScript](https://www.typescriptlang.org)**는 Microsoft에서 유지 관리 및 개발하는 오픈 소스 언어입니다.

기본적으로 TypeScript는 JavaScript에 추가적인 구문을 추가하여 에디터와의 통합을 강화합니다. 이를 통해 에디터나 CI/CD 파이프라인에서 오류를 조기에 잡아내고 더 유지보수하기 쉬운 코드를 작성할 수 있습니다.

TypeScript의 다른 장점은 나중에 다루도록 하고, 지금은 몇 가지 예제를 살펴보겠습니다!

## 첫 번째 TypeScript 코드

다음 코드 스니펫을 살펴본 후, 함께 분석해 봅시다:

<!--
  유지관리자 참고: 이 코드는 다음 글에서도 중복됩니다. 동기화를 유지하세요.
-->

```ts
type User = {
  name: string;
  age: number;
};

function isAdult(user: User): boolean {
  return user.age >= 18;
}

const justine = {
  name: 'Justine',
  age: 23,
} satisfies User;

const isJustineAnAdult = isAdult(justine);
```

첫 번째 부분(`type` 키워드 사용)은 사용자들을 나타내는 사용자 지정 객체 타입을 선언하는 역할을 합니다. 이후 이 새로 생성된 타입을 활용하여 `User` 타입의 인수를 받아 `boolean`을 반환하는 함수 `isAdult`를 생성합니다. 그 후, 이전에 정의한 함수를 호출하기 위해 사용할 예제 데이터 `justine`을 생성합니다. 마지막으로, `justine`이 성인인지 여부에 대한 정보를 담은 새 변수를 생성합니다.

이 예제와 관련하여 알아야 할 추가적인 사항이 있습니다. 첫째, 선언된 타입을 따르지 않으면 TypeScript가 문제가 있음을 알려주고 잘못된 사용을 방지합니다. 둘째, 모든 것을 명시적으로 타입 지정할 필요는 없습니다. TypeScript는 타입을 추론해줍니다. 예를 들어, 변수 `isJustineAnAdult`는 명시적으로 타입을 지정하지 않아도 `boolean` 타입이고, `justine`은 우리가 이 변수를 `User` 타입으로 선언하지 않았더라도 함수의 유효한 인수로 간주됩니다.

## TypeScript 코드를 실행하는 방법

좋아요, 이제 TypeScript 코드를 작성했습니다. 그러면 어떻게 실행할까요?
TypeScript 코드를 실행하는 몇 가지 방법이 있으며, 다음 글에서 이를 모두 다룰 예정입니다.
