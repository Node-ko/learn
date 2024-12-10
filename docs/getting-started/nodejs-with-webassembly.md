---
title: Node.js와 WebAssembly
layout: learn
authors: lancemccluskey, ovflowd
---

# Node.js와 WebAssembly

> ❗️ _번역 날짜: 2024년 12월 09일_ <br>
> 공식 문서 원문은 아래를 참고하세요.<br> > [Node.js with WebAssembly](https://nodejs.org/en/learn/getting-started/nodejs-with-webassembly)

**[WebAssembly](https://webassembly.org)**는 C/C++, Rust, AssemblyScript를 포함한 다양한 언어로부터 컴파일 가능한 고성능 어셈블리와 유사한 언어입니다. 현재 Chrome, Firefox, Safari, Edge, Node.js에서 지원됩니다!

WebAssembly 명세는 두 가지 파일 형식을 정의합니다. `.wasm` 확장자를 가진 바이너리 형식인 WebAssembly 모듈과 `.wat` 확장자를 가진 텍스트 표현 형식인 WebAssembly 텍스트 형식입니다.

## 주요 개념

- **Module** - 컴파일된 WebAssembly 바이너리, 즉 `.wasm` 파일입니다.
- **Memory** - 크기를 조정할 수 있는 ArrayBuffer입니다.
- **Table** - 메모리에 저장되지 않은 참조의 크기 조정 가능한 형식화 배열입니다.
- **Instance** - 메모리, 테이블 및 변수를 포함한 모듈의 인스턴스화입니다.

WebAssembly를 사용하려면 `.wasm` 바이너리 파일과 WebAssembly와 통신하기 위한 API 세트가 필요합니다. Node.js는 전역 `WebAssembly` 객체를 통해 필요한 API를 제공합니다.

```js
console.log(WebAssembly);
/*
Object [WebAssembly] {
  compile: [Function: compile],
  validate: [Function: validate],
  instantiate: [Function: instantiate]
}
*/
```

## WebAssembly 모듈 생성하기

WebAssembly 바이너리 파일을 생성하는 방법에는 여러 가지가 있습니다:

- WebAssembly (`.wat`)를 직접 작성하고 [wabt](https://github.com/webassembly/wabt)와 같은 도구를 사용하여 바이너리 형식으로 변환
- [emscripten](https://emscripten.org/)을 사용하여 C/C++ 애플리케이션으로부터 생성
- [wasm-pack](https://rustwasm.github.io/wasm-pack/book/)을 사용하여 Rust 애플리케이션으로부터 생성
- TypeScript와 유사한 경험을 원한다면 [AssemblyScript](https://www.assemblyscript.org/) 사용

> 이 도구들 중 일부는 바이너리 파일뿐만 아니라 JavaScript "글루" 코드 및 브라우저에서 실행하기 위한 HTML 파일도 생성합니다.

## 사용 방법

WebAssembly 모듈이 준비되면 Node.js의 `WebAssembly` 객체를 사용하여 이를 인스턴스화할 수 있습니다.

```js
// add.wasm 파일이 존재하며 두 인수를 더하는 단일 함수가 포함되어 있다고 가정
const fs = require('node:fs');

// readFileSync 함수로 "add.wasm" 파일의 내용을 읽습니다
const wasmBuffer = fs.readFileSync('/path/to/add.wasm');

// WebAssembly.instantiate 메서드를 사용하여 WebAssembly 모듈을 인스턴스화
WebAssembly.instantiate(wasmBuffer).then((wasmModule) => {
  // 내보낸 함수는 instance.exports 객체 아래에 위치
  const { add } = wasmModule.instance.exports;
  const sum = add(5, 6);
  console.log(sum); // 출력: 11
});
```

```mjs
// add.wasm 파일이 존재하며 두 인수를 더하는 단일 함수가 포함되어 있다고 가정
import fs from 'node:fs/promises';

// readFile 함수를 사용하여 "add.wasm" 파일 내용을 읽습니다
const wasmBuffer = await fs.readFile('path/to/add.wasm');

// WebAssembly.instantiate 메서드를 사용하여 WebAssembly 모듈을 인스턴스화
const wasmModule = await WebAssembly.instantiate(wasmBuffer);

// 내보낸 함수는 instance.exports 객체 아래에 위치
const { add } = wasmModule.instance.exports;

const sum = add(5, 6);

console.log(sum); // 출력: 11
```

## 운영 체제와 상호 작용

WebAssembly 모듈은 자체적으로 운영 체제 기능에 직접 접근할 수 없습니다. 운영 체제 기능에 접근하려면 [Wasmtime](https://docs.wasmtime.dev/)과 같은 서드파티 도구를 사용할 수 있습니다. `Wasmtime`은 [WASI](https://wasi.dev/) API를 활용하여 운영 체제 기능에 접근합니다.

## 리소스

- [WebAssembly 일반 정보](https://webassembly.org/)
- [MDN 문서](https://developer.mozilla.org/en-US/docs/WebAssembly)
- [WebAssembly를 직접 작성하기](https://webassembly.github.io/spec/core/text/index.html)
