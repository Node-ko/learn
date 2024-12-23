---
title: Node.js로 폴더 작업하기
layout: learn
authors: flaviocopes, MylesBorins, fhemberger, liangpeili, LaRuaNa, ahmadawais, clean99
---

# Node.js로 폴더 작업하기

Node.js `fs` 코어 모듈은 폴더를 작업하는 데 유용한 많은 메서드를 제공합니다.

## 폴더가 존재하는지 확인하기

`fs.access()` (및 promise-based `fsPromises.access()` 대응)를 사용하여 폴더가 존재하고 Node.js가 권한을 통해 액세스할 수 있는지 확인합니다.

## 새 폴더 만들기

`fs.mkdir()` 또는 `fs.mkdirSync()` 또는 `fsPromises.mkdir()`을 사용하여 새 폴더를 만듭니다.

```cjs
const fs = require('node:fs');

const folderName = '/Users/joe/test';

try {
  if (!fs.existsSync(folderName)) {
    fs.mkdirSync(folderName);
  }
} catch (err) {
  console.error(err);
}
```

```mjs
import fs from 'node:fs';

const folderName = '/Users/joe/test';

try {
  if (!fs.existsSync(folderName)) {
    fs.mkdirSync(folderName);
  }
} catch (err) {
  console.error(err);
}
```

## 디렉토리 내용 읽기

`fs.readdir()` 또는 `fs.readdirSync()` 또는 `fsPromises.readdir()`을 사용하여 디렉토리 내용을 읽습니다.

이 코드는 폴더의 내용을 읽고 파일과 하위 폴더를 반환합니다:

```cjs
const fs = require('node:fs');

const folderPath = '/Users/joe';

fs.readdirSync(folderPath);
```

```mjs
import fs from 'node:fs';

const folderPath = '/Users/joe';

fs.readdirSync(folderPath);
```

전체 경로를 얻을 수 있습니다:

```js
fs.readdirSync(folderPath).map(fileName => {
  return path.join(folderPath, fileName);
});
```

결과를 파일로 필터링하고 폴더를 제외할 수 있습니다:

```cjs
const fs = require('node:fs');

const isFile = fileName => {
  return fs.lstatSync(fileName).isFile();
};

fs.readdirSync(folderPath)
  .map(fileName => {
    return path.join(folderPath, fileName);
  })
  .filter(isFile);
```

```mjs
import fs from 'node:fs';

const isFile = fileName => {
  return fs.lstatSync(fileName).isFile();
};

fs.readdirSync(folderPath)
  .map(fileName => {
    return path.join(folderPath, fileName);
  })
  .filter(isFile);
```

## 폴더 이름 변경

`fs.rename()` 또는 `fs.renameSync()` 또는 `fsPromises.rename()`을 사용하여 폴더 이름을 변경합니다. 첫 번째 매개변수는 현재 경로이고 두 번째 매개변수는 새 경로입니다:

```cjs
const fs = require('node:fs');

fs.rename('/Users/joe', '/Users/roger', err => {
  if (err) {
    console.error(err);
  }
  // done
});
```

```mjs
import fs from 'node:fs';

fs.rename('/Users/joe', '/Users/roger', err => {
  if (err) {
    console.error(err);
  }
  // done
});
```

`fs.renameSync()`는 동기 버전입니다:

```cjs
const fs = require('node:fs');

try {
  fs.renameSync('/Users/joe', '/Users/roger');
} catch (err) {
  console.error(err);
}
```

```mjs
import fs from 'node:fs';

try {
  fs.renameSync('/Users/joe', '/Users/roger');
} catch (err) {
  console.error(err);
}
```

`fsPromises.rename()`은 promise-based 버전입니다:

```cjs
const fs = require('node:fs/promises');

async function example() {
  try {
    await fs.rename('/Users/joe', '/Users/roger');
  } catch (err) {
    console.log(err);
  }
}
example();
```

```mjs
import fs from 'node:fs/promises';

try {
  await fs.rename('/Users/joe', '/Users/roger');
} catch (err) {
  console.log(err);
}
```

## 폴더 삭제

`fs.rmdir()` 또는 `fs.rmdirSync()` 또는 `fsPromises.rmdir()`을 사용하여 폴더를 삭제합니다.

```cjs
const fs = require('node:fs');

fs.rmdir(dir, err => {
  if (err) {
    throw err;
  }

  console.log(`${dir} is deleted!`);
});
```

```mjs
import fs from 'node:fs';

fs.rmdir(dir, err => {
  if (err) {
    throw err;
  }

  console.log(`${dir} is deleted!`);
});
```

내용이 있는 폴더를 삭제하려면 `fs.rm()`을 사용하고 옵션 `{ recursive: true }`을 사용하여 재귀적으로 내용을 삭제합니다.

`{ recursive: true, force: true }`는 폴더가 존재하지 않으면 예외를 무시하도록 합니다.

```cjs
const fs = require('node:fs');

fs.rm(dir, { recursive: true, force: true }, err => {
  if (err) {
    throw err;
  }

  console.log(`${dir} is deleted!`);
});
```

```mjs
import fs from 'node:fs';

fs.rm(dir, { recursive: true, force: true }, err => {
  if (err) {
    throw err;
  }

  console.log(`${dir} is deleted!`);
});
```