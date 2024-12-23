---
title: Node.js로 파일 쓰기
layout: learn
authors: flaviocopes, MylesBorins, fhemberger, LaRuaNa, ahmadawais, clean99, ovflowd, vaishnav-mk
---

# Node.js로 파일 쓰기
> ❗️ *번역 날짜: 2024년 12월 23일* <br>
> 공식 문서 원문은 아래를 참고하세요.<br>
>[Writing files with Node.js](https://nodejs.org/en/learn/manipulating-files/writing-files-with-nodejs)

## 파일 쓰기

Node.js에서 파일을 쓰는 가장 쉬운 방법은 `fs.writeFile()` API를 사용하는 것입니다.

```js
const fs = require('node:fs');

const content = 'Some content!';

fs.writeFile('/Users/joe/test.txt', content, err => {
  if (err) {
    console.error(err);
  } else {
    // file written successfully
  }
});
```

### 동기 파일 쓰기

대신, 동기 버전 `fs.writeFileSync()`를 사용할 수 있습니다:

```js
const fs = require('node:fs');

const content = 'Some content!';

try {
  fs.writeFileSync('/Users/joe/test.txt', content);
  // file written successfully
} catch (err) {
  console.error(err);
}
```

`fs/promises` 모듈에서 제공하는 promise-based `fsPromises.writeFile()` 메서드를 사용할 수도 있습니다:

```js
const fs = require('node:fs/promises');

async function example() {
  try {
    const content = 'Some content!';
    await fs.writeFile('/Users/joe/test.txt', content);
  } catch (err) {
    console.log(err);
  }
}

example();
```

기본적으로 이 API는 파일이 이미 존재하면 **파일 내용을 대체**합니다.

**플래그를 지정하여 기본값을 수정할 수 있습니다:**

```js
fs.writeFile('/Users/joe/test.txt', content, { flag: 'a+' }, err => {});
```

#### 가장 많이 사용되는 플래그는

| 플래그 | 설명                                                                                                                         | 파일이 없으면 생성됩니다 |
| ---- | -------------------------------------------------------------------------------------------------------------------------- | :-----------------------------------: |
| `r+` | 이 플래그는 파일을 **읽기**와 **쓰기**를 위해 엽니다.                                                                   |                  ❌                   |
| `w+` | 이 플래그는 파일을 **읽기**와 **쓰기**를 위해 엽니다. 또한 스트림을 파일의 **시작**으로 위치시킵니다. |                  ✅                   |
| `a`  | 이 플래그는 파일을 **쓰기**를 위해 엽니다. 또한 스트림을 파일의 **끝**으로 위치시킵니다.                       |                  ✅                   |
| `a+` | 이 플래그는 파일을 **읽기**와 **쓰기**를 위해 엽니다. 또한 스트림을 파일의 **끝**으로 위치시킵니다.       |                  ✅                   |

- [fs 문서](https://nodejs.org/api/fs.html#file-system-flags)에서 더 많은 정보를 찾을 수 있습니다.

## 파일에 내용 추가

파일에 내용을 추가하는 것은 새로운 내용으로 덮어쓰지 않고 기존 내용에 추가하고 싶을 때 유용합니다.

### 예제

파일 끝에 내용을 추가하는 유용한 방법은 `fs.appendFile()` (및 동기 버전 `fs.appendFileSync()`):

```js
const fs = require('node:fs');

const content = 'Some content!';

fs.appendFile('file.log', content, err => {
  if (err) {
    console.error(err);
  } else {
    // done!
  }
});
```

#### Promise 예제

`fsPromises.appendFile()` 예제:

```js
const fs = require('node:fs/promises');

async function example() {
  try {
    const content = 'Some content!';
    await fs.appendFile('/Users/joe/test.txt', content);
  } catch (err) {
    console.log(err);
  }
}

example();
```