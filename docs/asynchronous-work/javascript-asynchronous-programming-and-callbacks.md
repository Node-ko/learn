---
title: JavaScript 비동기 프로그래밍 및 콜백
layout: learn
authors: flaviocopes, MylesBorins, LaRuaNa, amiller-gh, ahmadawais, ovflowd
---

# JavaScript 비동기 프로그래밍 및 콜백
> ❗️ *번역 날짜: 2024년 12월 17일* <br>
> 공식 문서 원문은 아래를 참고하세요.<br>
> [JavaScript Asynchronous Programming and Callbacks](https://nodejs.org/en/learn/asynchronous-work/javascript-asynchronous-programming-and-callbacks)

## 프로그래밍 언어의 비동기성

컴퓨터는 본래 비동기적으로 설계되었습니다.

비동기적이란, 일이 주 프로그램 흐름과 독립적으로 일어날 수 있다는 의미입니다.

현재의 소비자용 컴퓨터에서는 모든 프로그램이 특정 시간 동안 실행되고, 그 후 다른 프로그램이 실행을 계속할 수 있도록 실행을 멈춥니다. 이 과정은 너무 빠르게 순환되기 때문에 우리가 인식하기 어렵습니다. 우리는 컴퓨터가 여러 프로그램을 동시에 실행한다고 생각하지만, 이는 환상에 불과합니다 (멀티프로세서 머신을 제외하고).

프로그램은 내부적으로 _인터럽트_ 를 사용하여 시스템의 주의를 끌기 위해 프로세서에 신호를 보냅니다.

이제 그 내부적인 동작에 대해서는 깊이 들어가지 않겠습니다만, 프로그램이 비동기적으로 실행되고, 필요할 때까지 실행을 멈추는 것이 정상이라는 점을 기억해두세요. 이렇게 하면 그동안 다른 작업을 처리할 수 있습니다. 예를 들어, 네트워크로부터 응답을 기다리는 프로그램은 요청이 완료될 때까지 프로세서를 멈출 수 없습니다.

일반적으로 프로그래밍 언어는 동기적이며, 일부 언어는 비동기성을 처리할 수 있는 방법을 언어 자체나 라이브러리를 통해 제공합니다. C, Java, C#, PHP, Go, Ruby, Swift, Python은 기본적으로 동기적입니다. 이들 중 일부는 비동기 작업을 처리하기 위해 스레드를 사용하거나 새로운 프로세스를 생성합니다.

## JavaScript

JavaScript는 기본적으로 **동기적**이며 단일 스레드로 실행됩니다. 즉, 코드가 새로운 스레드를 생성하거나 병렬로 실행될 수 없습니다.

코드는 순차적으로, 하나씩 실행됩니다. 예를 들어:

```js
const a = 1;
const b = 2;
const c = a * b;
console.log(c);
doSomething();
```

하지만 JavaScript는 브라우저 내에서 시작되었고, 초기에는 `onClick`, `onMouseOver`, `onChange`, `onSubmit` 등과 같은 사용자 동작에 응답하는 것이 주요 역할이었습니다. 동기적인 프로그래밍 모델로 어떻게 이런 작업을 처리할 수 있었을까요?

그 답은 바로 환경에 있습니다. **브라우저**는 이러한 기능을 처리할 수 있는 API 집합을 제공하여 이를 가능하게 했습니다.

최근에는 Node.js가 비동기 I/O 환경을 도입하여 파일 접근, 네트워크 호출 등으로 이 개념을 확장했습니다.

## 콜백

사용자가 버튼을 언제 클릭할지 알 수 없습니다. 그래서 **클릭 이벤트에 대한 이벤트 핸들러를 정의**합니다. 이 이벤트 핸들러는 이벤트가 트리거될 때 호출될 함수를 받습니다:

```js
document.getElementById('button').addEventListener('click', () => {
  // 클릭한 항목
});
```

이를 소위 **콜백**이라고 합니다.

콜백은 다른 함수에 값으로 전달되는 간단한 함수로, 이벤트가 발생할 때만 실행됩니다. JavaScript는 일급 함수(first-class functions)를 지원하기 때문에 함수는 변수에 할당하거나 다른 함수로 전달할 수 있습니다. (이를 **고차 함수**라고 합니다)

페이지가 준비되었을 때만 콜백 함수가 실행되도록 `window` 객체에 `load` 이벤트 리스너를 사용하여 모든 클라이언트 코드를 감싸는 것이 일반적입니다:

```js
window.addEventListener('load', () => {
  // window 로드 완료
  // 원하는 작업을 수행하세요
});
```


콜백은 DOM 이벤트뿐만 아니라 모든 곳에서 사용됩니다.

그 중 하나의 일반적인 예는 타이머를 사용하는 경우입니다:

```js
setTimeout(() => {
  // 2초 후에 실행됩니다
}, 2000);
```

XHR 요청도 콜백을 받습니다. 이 예제에서는 특정 이벤트가 발생할 때 호출될 함수를 속성에 할당하는 방식으로 콜백을 사용합니다 (이 경우 요청의 상태가 변경될 때):

```js
const xhr = new XMLHttpRequest();
xhr.onreadystatechange = () => {
  if (xhr.readyState === 4) {
    xhr.status === 200 ? console.log(xhr.responseText) : console.error('error');
  }
};
xhr.open('GET', 'https://yoursite.com');
xhr.send();
```

### 콜백에서 오류 처리하기

콜백에서 오류를 처리하는 방법은 무엇일까요? Node.js에서 채택한 매우 일반적인 전략 중 하나는 콜백 함수의 첫 번째 매개변수로 오류 객체를 사용하는 **에러 우선 콜백**입니다.

오류가 없으면 그 객체는 `null`입니다. 만약 오류가 있으면 오류에 대한 설명과 기타 정보가 포함된 객체가 전달됩니다.

```js
const fs = require('node:fs');

fs.readFile('/file.json', (err, data) => {
  if (err) {
    // 오류 처리
    console.log(err);
    return;
  }

  // 오류 없음, 데이터 처리
  console.log(data);
});
```

### 콜백의 문제점

콜백은 간단한 경우에는 정말 유용합니다!

하지만 매번 콜백을 사용하면 중첩이 추가되고, 콜백이 많아질수록 코드가 빠르게 복잡해지기 시작합니다:

```js
window.addEventListener('load', () => {
  document.getElementById('button').addEventListener('click', () => {
    setTimeout(() => {
      items.forEach(item => {
        // 여기에 당신의 코드를 작성하세요
      });
    }, 2000);
  });
});
```

이건 단순히 4단계 중첩된 코드일 뿐이지만, 더 많은 중첩을 본 적이 있고, 그건 정말 재미없습니다.

이 문제는 어떻게 해결할 수 있을까요?

### 콜백의 대안

ES6부터 JavaScript는 콜백을 사용하지 않고 비동기 코드를 처리할 수 있는 여러 가지 기능을 도입했습니다: Promises (ES6)와 Async/Await (ES2017)입니다.