---
title: Node.js에서 파일 디스크립터로 작업하기
layout: learn
authors: flaviocopes, MylesBorins, fhemberger, LaRuaNa, ahmadawais, clean99, vaishnav-mk
---

# Node.js에서 파일 디스크립터로 작업하기
> ❗️ *번역 날짜: 2024년 12월 23일* <br>
> 공식 문서 원문은 아래를 참고하세요.<br>
>[Working with file descriptors in Node.js](https://nodejs.org/en/learn/manipulating-files/working-with-file-descriptors-in-nodejs)

파일시스템에 있는 파일과 상호 작용하려면 먼저 파일 디스크립터를 가져와야 합니다.

파일 디스크립터는 `fs` 모듈에서 제공하는 `open()` 메서드를 사용하여 파일을 열면 반환되는 숫자(fd)인 열린 파일에 대한 참조입니다. 이 번호(`fd`)는 운영 체제에서 열려 있는 파일을 고유하게 식별합니다:

```cjs
const fs = require('node:fs');

fs.open('/Users/joe/test.txt', 'r', (err, fd) => {
  // fd는 파일 디스크립터입니다
});
```

```mjs
import fs from 'node:fs';

fs.open('/Users/joe/test.txt', 'r', (err, fd) => {
  // fd는 파일 디스크립터입니다
});
```

`fs.open()` 호출의 두 번째 매개변수로 사용한 `r`을 주목하세요.

이 플래그는 읽기를 위해 파일을 연다는 의미입니다.

**일반적으로 사용하는 다른 플래그는 다음과 같습니다:**

| 플래그 | 설명                                                                                                    | 파일이 없으면 생성됩니다 |
| ------ | ------------------------------------------------------------------------------------------------------- | ----------------------- |
| r+     | 이 플래그는 읽기 및 쓰기를 위해 파일을 엽니다.                                                       | ❌                      |
| w+     | 이 플래그는 읽기 및 쓰기를 위해 파일을 열고 스트림을 파일 시작 부분에 배치합니다.                   | ✅                      |
| a      | 이 플래그는 쓰기를 위해 파일을 열고 스트림을 파일 끝에 배치합니다.                                   | ✅                      |
| a+     | 이 플래그는 읽기 및 쓰기를 위해 파일을 열고 스트림을 파일 끝에 배치합니다.                           | ✅                      |

콜백으로 제공하는 대신 파일 설명자를 반환하는 `fs.openSync` 메서드를 사용하여 파일을 열 수도 있습니다:

```cjs
const fs = require('node:fs');

try {
  const fd = fs.openSync('/Users/joe/test.txt', 'r');
} catch (err) {
  console.error(err);
}
```

```mjs
import fs from 'node:fs';

try {
  const fd = fs.openSync('/Users/joe/test.txt', 'r');
} catch (err) {
  console.error(err);
}
```

파일 기술자를 얻으면 어떤 방법을 선택하든 `fs.close()` 호출 및 파일 시스템과 상호 작용하는 다른 많은 작업과 같이 파일 기술자가 필요한 모든 작업을 수행할 수 있습니다.

또한 `fs/promises` 모듈에서 제공하는 promise 기반 `fsPromises.open` 메서드를 사용하여 파일을 열 수도 있습니다.

`fs/promises` 모듈은 Node.js v14부터 사용할 수 있습니다. v14 이전, v10 이후에는 `require('fs').promises` 대신 사용할 수 있습니다. v10 이전, v8 이후에는 `util.promisify`를 사용하여 `fs` 메서드를 promise 기반 메서드로 변환할 수 있습니다.

```cjs
const fs = require('node:fs/promises');
// 또는 v14 이전 버전은 const fs = require('fs').promises.
async function example() {
  let filehandle;
  try {
    filehandle = await fs.open('/Users/joe/test.txt', 'r');
    console.log(filehandle.fd);
    console.log(await filehandle.readFile({ encoding: 'utf8' }));
  } finally {
    if (filehandle) await filehandle.close();
  }
}
example();
```

```mjs
import fs from 'node:fs/promises';
// 또는 v14 이전 버전은 const fs = require('fs').promises.
let filehandle;
try {
  filehandle = await fs.open('/Users/joe/test.txt', 'r');
  console.log(filehandle.fd);
  console.log(await filehandle.readFile({ encoding: 'utf8' }));
} finally {
  if (filehandle) await filehandle.close();
}
```

다음은 `util.promiseify`의 예시입니다:

```cjs
const fs = require('node:fs');
const util = require('node:util');

async function example() {
  const open = util.promisify(fs.open);
  const fd = await open('/Users/joe/test.txt', 'r');
}
example();
```

```mjs
import fs from 'node:fs';
import util from 'node:util';

async function example() {
  const open = util.promisify(fs.open);
  const fd = await open('/Users/joe/test.txt', 'r');
}
example();
```

`fs/promises` 모듈에 대한 자세한 내용은 [fs/promises API](https://nodejs.org/api/fs.html#promise-example)에서 확인하시기 바랍니다.