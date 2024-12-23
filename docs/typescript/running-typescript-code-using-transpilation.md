---
title: TypeScript 코드 사용하기
layout: learn
authors: AugustinMauroy
---

# TypeScript 코드 사용하기
> ❗️ *번역 날짜: 2024년 12월 16일* <br>
> 공식 문서 원문은 아래를 참고하세요.<br>
>[Running TypeScript code using transpilation](https://nodejs.org/en/learn/typescript/transpile#running-typescript-code-using-transpilation)

Transpilation은 소스 코드를 한 언어에서 다른 언어로 변환하는 과정입니다. TypeScript의 경우, TypeScript 코드를 JavaScript 코드로 변환하는 과정입니다. 이는 브라우저와 Node.js가 TypeScript 코드를 직접 실행할 수 없기 때문입니다.

## TypeScript 코드를 JavaScript로 컴파일하기

TypeScript 코드를 실행하는 가장 일반적인 방법은 먼저 JavaScript로 컴파일하는 것입니다. 이를 위해 TypeScript 컴파일러 `tsc`를 사용할 수 있습니다.

**Step 1:** `example.ts`라는 파일에 TypeScript 코드를 작성합니다.

<!--
  Maintainers note: this code is duplicated in the previous article, please keep them in sync
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

**Step 2:** 패키지 매니저를 사용하여 TypeScript를 로컬에 설치합니다:

이 예제에서는 npm을 사용하겠습니다. npm에 대한 자세한 내용은 [npm 패키지 매니저 소개][npm 패키지 매니저 소개]를 참고하세요.

```bash displayName="Install TypeScript locally"
npm i -D typescript # -D는 --save-dev의 줄임말입니다.
```

**Step 3:** `tsc` 명령어를 사용하여 TypeScript 코드를 JavaScript로 컴파일합니다:

```bash
npx tsc example.ts
```
> **NOTE:** `npx`는 패키지를 전역적으로 설치하지 않고도 Node.js 패키지를 실행할 수 있는 도구입니다.

`tsc`는 TypeScript 컴파일러로, TypeScript 코드를 JavaScript로 변환합니다.
이 명령어는 `example.js`라는 새로운 파일을 생성하며, 이를 Node.js로 실행할 수 있습니다.
이제 TypeScript 코드를 컴파일하고 실행하는 방법을 알았으니, TypeScript의 타입 검사 기능을 살펴보겠습니다!

**Step 4:** Node.js를 사용하여 JavaScript 코드를 실행합니다:

```bash
node example.js
```

터미널에서 TypeScript 코드의 출력을 확인할 수 있습니다.

## 타입 오류가 있는 경우

TypeScript 코드에 타입 오류가 있는 경우, TypeScript 컴파일러는 이를 잡아내고 코드를 실행하지 못하도록 합니다. 예를 들어, `justine`의 `age` 속성을 문자열로 변경하면 TypeScript는 오류를 발생시킵니다:

우리는 이 코드를 다음과 같이 수정하여 의도적으로 타입 오류를 도입할 것입니다:

```ts
type User = {
  name: string;
  age: number;
};

function isAdult(user: User): boolean {
  return user.age >= 18;
}

const justine: User = {
  name: 'Justine',
  age: 'Secret!',
};

const isJustineAnAdult: string = isAdult(justine, "I shouldn't be here!");
```

TypeScript는 이에 대해 다음과 같이 말합니다:

```console
example.ts:12:5 - error TS2322: Type 'string' is not assignable to type 'number'.

12     age: 'Secret!',
       ~~~

  example.ts:3:5
    3     age: number;
          ~~~
    The expected type comes from property 'age' which is declared here on type 'User'

example.ts:15:7 - error TS2322: Type 'boolean' is not assignable to type 'string'.

15 const isJustineAnAdult: string = isAdult(justine, "I shouldn't be here!");
         ~~~~~~~~~~~~~~~~

example.ts:15:51 - error TS2554: Expected 1 arguments, but got 2.

15 const isJustineAnAdult: string = isAdult(justine, "I shouldn't be here!");
                                                     ~~~~~~~~~~~~~~~~~~~~~~


Found 3 errors in the same file, starting at: example.ts:12
```

TypeScript는 타입 오류를 잡아내기 위해 매우 유용합니다. 이는 TypeScript가 개발자들에게 인기 있는 이유 중 하나입니다.


[npm 패키지 매니저 소개]: https://tastekim.gitbook.io/nodejs-ko/learn/getting-started/an-introduction-to-the-npm-package-manager