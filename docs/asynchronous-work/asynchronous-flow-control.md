---
title: 비동기 흐름 제어
layout: learn
authors: aug2uag, ovflowd
---

# 비동기 흐름 제어
> ❗️ *번역 날짜: 2024년 12월 16일* <br>
> 공식 문서 원문은 아래를 참고하세요.<br>
>[Asynchronous flow control](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control)

> 이 글의 자료는 [Mixu의 Node.js 책](http://book.mixu.net/node/ch7.html)에서 많이 영향을 받았습니다.

JavaScript의 핵심 설계 원칙은 메인 스레드가 멈추지 않는 것입니다. 메인 스레드가 막히면 사용자가 보는 UI가 멈추고, 사용자 입력이나 다른 이벤트들이 처리되지 못해 데이터가 유실될 수 있기 때문입니다.

이러한 제약으로 인해 JavaScript에서는 콜백 패턴이라는 독특한 프로그래밍 스타일이 등장하게 되었습니다.

그러나 콜백 패턴은 복잡한 비동기 로직을 다룰 때 문제가 될 수 있습니다. 특히 여러 콜백이 중첩되면서 발생하는 "콜백 지옥(callback hell)" 현상은 코드의 가독성을 해치고 유지보수와 디버깅을 어렵게 만듭니다.

```js
async1(function (input, result1) {
  async2(function (result2) {
    async3(function (result3) {
      async4(function (result4) {
        async5(function (output) {
          // output을 사용하여 무언가를 수행합니다
        });
      });
    });
  });
});
```

물론, 실제 상황에서는 `result1`, `result2` 등을 처리하기 위한 추가 코드가 있을 것이며, 이는 코드의 길이와 복잡성을 증가시킵니다. 이는 예제보다 훨씬 더 복잡해 보일 것입니다.

**이런 상황에서 *함수*가 유용하게 사용될 수 있습니다. 복잡한 작업은 여러개의 함수들로 나누어 처리할 수 있습니다:**

1. 시작 함수(initiator style) / 입력
2. 미들웨어(middleware)
3. 종료 함수(terminator)

**"시작 함수 / 입력"은 순서의 첫 번째 함수입니다. 이 함수는 작업에 대한 원본 입력을 받을 것입니다. 작업은 실행 가능한 함수 시퀀스이며, 원본 입력은 주로 다음과 같습니다:**

1. 전역 환경의 변수
2. 인수 없이 또는 인수와 함께 직접 호출
3. 파일 시스템 또는 네트워크 요청에서 얻은 값

네트워크 요청은 외부 네트워크에서 시작된 요청, 같은 네트워크의 다른 애플리케이션에서 시작된 요청, 또는 같은 네트워크의 애플리케이션에서 시작된 요청일 수 있습니다.

**"미들웨어"는 다른 함수를 반환하고, "종료 함수"는 콜백을 호출합니다. 다음은 네트워크 또는 파일 시스템 요청의 흐름을 보여줍니다. 여기서 대기 시간은 0입니다. 이 값들은 모두 메모리에 있기 때문입니다.**

```js
function final(someInput, callback) {
  callback(`${someInput} and terminated by executing callback `);
}

function middleware(someInput, callback) {
  return final(`${someInput} touched by middleware `, callback);
}

function initiate() {
  const someInput = 'hello this is a function ';
  middleware(someInput, function (result) {
    console.log(result);
    // 콜백을 통해 결과를 반환합니다
  });
}

initiate();
```

## 상태 관리

함수는 상태에 따라 다를 수 있습니다. 상태 의존성은 함수의 입력 또는 다른 변수가 외부 함수에 의존할 때 발생합니다.

**이러한 상태 관리에는 두 가지 주요 전략이 있습니다:**

1. 함수에 변수를 직접 전달하고,
2. 캐시, 세션, 파일, 데이터베이스, 네트워크 또는 다른 외부 소스에서 변수 값을 얻습니다.

참고, 전역 변수를 언급하지 않았습니다. 전역 변수를 사용한 상태 관리는 종종 문제가 되며, 복잡한 프로그램에서는 가능하면 피해야 합니다.

## 흐름 제어

객체가 메모리에 있으면 반복이 가능하며, 흐름 제어에 변경이 없습니다:

```js
function getSong() {
  let _song = '';
  let i = 100;
  for (i; i > 0; i -= 1) {
    _song += `${i} beers on the wall, you take one down and pass it around, ${
      i - 1
    } bottles of beer on the wall\n`;
    if (i === 1) {
      _song += "Hey let's get some more beer";
    }
  }

  return _song;
}

function singSong(_song) {
  if (!_song) throw new Error("song is '' empty, FEED ME A SONG!");
  console.log(_song);
}

const song = getSong();
// 이 작업은 작동하지 않습니다
singSong(song);
```

데이터가 메모리 외부에 있으면 반복이 더 이상 작동하지 않습니다:

```js
function getSong() {
  let _song = '';
  let i = 100;
  for (i; i > 0; i -= 1) {
    /* eslint-disable no-loop-func */
    setTimeout(function () {
      _song += `${i} beers on the wall, you take one down and pass it around, ${
        i - 1
      } bottles of beer on the wall\n`;
      if (i === 1) {
        _song += "Hey let's get some more beer";
      }
    }, 0);
    /* eslint-enable no-loop-func */
  }

  return _song;
}

function singSong(_song) {
  if (!_song) throw new Error("song is '' empty, FEED ME A SONG!");
  console.log(_song);
}

const song = getSong('beer');
// 이 작업은 작동하지 않습니다
singSong(song);
// Uncaught Error: song is '' empty, FEED ME A SONG!
```

왜 이런 일이 발생했을까요? `setTimeout`은 비동기 작업을 처리하기 위해 JavaScript 엔진의 이벤트 큐(작업 대기열)에 콜백 함수를 등록합니다. 이때 타이머가 0ms로 설정되어 있더라도, 콜백 함수는 현재 실행 중인 코드가 모두 완료된 후에야 실행됩니다. 그 사이에 `getSong` 함수는 이미 빈 문자열을 반환해버렸기 때문에 우리가 원하는 결과를 얻을 수 없는 것입니다.

이러한 상황은 파일 시스템 작업이나 네트워크 요청과 같은 다른 비동기 작업에서도 동일하게 발생합니다. JavaScript는 싱글 스레드로 동작하기 때문에 이러한 시간이 걸리는 작업들을 기다리며 메인 스레드를 멈출 수 없습니다. 대신 콜백 함수를 사용해 비동기 작업이 완료된 후에 실행될 코드를 지정하는 방식으로 처리합니다.

다음 3가지 패턴을 사용하여 대부분의 작업을 수행할 수 있습니다:

1. **순차적으로:** 함수는 엄격한 순차적 순서로 실행됩니다. 이는 `for` 루프와 가장 유사합니다.

```js
// 다른 곳에서 정의되고 실행 준비가 된 작업
const operations = [
  { func: function1, args: args1 },
  { func: function2, args: args2 },
  { func: function3, args: args3 },
];

function executeFunctionWithArgs(operation, callback) {
  // 함수 실행
  const { args, func } = operation;
  func(args, callback);
}

function serialProcedure(operation) {
  if (!operation) process.exit(0); // 완료
  executeFunctionWithArgs(operation, function (result) {
    // 콜백 후 계속
    serialProcedure(operations.shift());
  });
}

serialProcedure(operations.shift());
```

2. **전체 병렬:** 순서가 문제가 아닌 경우, 예를 들어 1,000,000명의 이메일 수신자에게 이메일을 보내는 경우.

```js
let count = 0;
let success = 0;
const failed = [];
const recipients = [
  { name: 'Bart', email: 'bart@tld' },
  { name: 'Marge', email: 'marge@tld' },
  { name: 'Homer', email: 'homer@tld' },
  { name: 'Lisa', email: 'lisa@tld' },
  { name: 'Maggie', email: 'maggie@tld' },
];

function dispatch(recipient, callback) {
  // `sendEmail`은 가상의 SMTP 클라이언트입니다
  sendMail(
    {
      subject: 'Dinner tonight',
      message: 'We have lots of cabbage on the plate. You coming?',
      smtp: recipient.email,
    },
    callback
  );
}

function final(result) {
  console.log(`Result: ${result.count} attempts \
      & ${result.success} succeeded emails`);
  if (result.failed.length)
    console.log(`Failed to send to: \
        \n${result.failed.join('\n')}\n`);
}

recipients.forEach(function (recipient) {
  dispatch(recipient, function (err) {
    if (!err) {
      success += 1;
    } else {
      failed.push(recipient.name);
    }
    count += 1;

    if (count === recipients.length) {
      final({
        count,
        success,
        failed,
      });
    }
  });
});
```

3. **제한된 병렬:** 제한된 병렬, 예를 들어 1,000,000명의 수신자에게 성공적으로 이메일을 보내는 경우.

```js
let successCount = 0;

function final() {
  console.log(`dispatched ${successCount} emails`);
  console.log('finished');
}

function dispatch(recipient, callback) {
  // `sendEmail`은 가상의 SMTP 클라이언트입니다
  sendMail(
    {
      subject: 'Dinner tonight',
      message: 'We have lots of cabbage on the plate. You coming?',
      smtp: recipient.email,
    },
    callback
  );
}

function sendOneMillionEmailsOnly() {
  getListOfTenMillionGreatEmails(function (err, bigList) {
    if (err) throw err;

    function serial(recipient) {
      if (!recipient || successCount >= 1000000) return final();
      dispatch(recipient, function (_err) {
        if (!_err) successCount += 1;
        serial(bigList.pop());
      });
    }

    serial(bigList.pop());
  });
}

sendOneMillionEmailsOnly();
```

각각은 자체 사용 사례, 이점, 그리고 더 자세히 실험하고 읽을 수 있는 문제가 있습니다. 가장 중요한 것은 작업을 모듈화하고 콜백을 사용하는 것입니다! 의심스러운 경우 모든 것을 미들웨어로 취급하세요!
