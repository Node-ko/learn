---
title: Node.js 파일 상태 정보 (file stats)
layout: learn
authors: flaviocopes, ZYSzys, MylesBorins, fhemberger, LaRuaNa, ahmadawais, clean99, ovflowd, vaishnav-mk
---

> ❗️ _번역 날짜: 2024년 12월 21일_ <br />
> 공식 문서 원문은 아래를 참고하세요.<br /> > [Node.js file stats](https://nodejs.org/en/learn/manipulating-files/nodejs-file-stats)

# Node.js 파일 상태 정보 (file stats)

Node.js를 사용할 때 우리가 확인할 수 있는 모든 파일은 세부 정보를 가지고 있습니다. 특히 [`fs` module](https://nodejs.org/api/fs.html) 이 제공하는 `stat()` 메서드를 통해 이런 정보들을 확인할 수 있습니다.

이 메서드는 호출할 때 파일 경로를 전달하며, Node.js가 파일의 세부 정보를 가져오면 **에러 메시지** 와 **파일 상태 정보** 두개의 매개변수를 포함한 콜백 함수가 호출됩니다.

```js
const fs = require('node:fs');

fs.stat('/Users/joe/test.txt', (err, stats) => {
  if (err) {
    console.error(err);
  }
  // 이제 `stats` 에 있는 파일 상태 정보에 대한 접근이 가능합니다.
});
```

```js
import fs from 'node:fs';

fs.stat('/Users/joe/test.txt', (err, stats) => {
  if (err) {
    console.error(err);
  }
  // `stats` 에 있는 파일 상태 정보에 대한 접근이 가능합니다.
});
```

Node.js는 파일 상태 정보가 준비될 때까지 스레드를 차단하는 동기식 메서드도 제공합니다.

```js
const fs = require('node:fs');

try {
  const stats = fs.statSync('/Users/joe/test.txt');
} catch (err) {
  console.error(err);
}
```

```js
import fs from 'node:fs';

try {
  const stats = fs.statSync('/Users/joe/test.txt');
} catch (err) {
  console.error(err);
}
```

파일 정보는 stats 변수에 포함되어 있습니다. stats를 사용해 어떤 정보를 추출할 수 있을까요?

**다음을 포함한 다양한 정보가 있습니다.**

- 파일이 디렉터리인지 파일인지 확인: `stats.isFile()`, `stats.isDirectory()`
- 파일이 심볼릭 링크인지 확인: `stats.isSymbolicLink()`
- 파일 크기를 바이트 단위로 확인: `stats.size`.

여기에 더 고급 메서드도 있지만, 일상적인 프로그래밍에서 자주 사용하는 것은 위와 같습니다.

```js
const fs = require('node:fs');

fs.stat('/Users/joe/test.txt', (err, stats) => {
  if (err) {
    console.error(err);
    return;
  }

  stats.isFile(); // true
  stats.isDirectory(); // false
  stats.isSymbolicLink(); // false
  stats.size; // 1024000 //= 1MB
});
```

```js
import fs from 'node:fs';

fs.stat('/Users/joe/test.txt', (err, stats) => {
  if (err) {
    console.error(err);
    return;
  }

  stats.isFile(); // true
  stats.isDirectory(); // false
  stats.isSymbolicLink(); // false
  stats.size; // 1024000 //= 1MB
});
```

`fsPromises.stat()` 모듈에서 제공하는 promise 기반의 `fs/promises` 메서드를 사용할 수도 있습니다.

```js
const fs = require('node:fs/promises');

async function example() {
  try {
    const stats = await fs.stat('/Users/joe/test.txt');
    stats.isFile(); // true
    stats.isDirectory(); // false
    stats.isSymbolicLink(); // false
    stats.size; // 1024000 //= 1MB
  } catch (err) {
    console.log(err);
  }
}
example();
```

```js
import fs from 'node:fs/promises';

try {
  const stats = await fs.stat('/Users/joe/test.txt');
  stats.isFile(); // true
  stats.isDirectory(); // false
  stats.isSymbolicLink(); // false
  stats.size; // 1024000 //= 1MB
} catch (err) {
  console.log(err);
}
```

`fs` 모듈에 대한 자세한 내용은 [공식 문서](https://nodejs.org/api/fs.html)에서 확인할 수 있습니다.
