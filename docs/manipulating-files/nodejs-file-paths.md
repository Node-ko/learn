---
title: Node.js 파일 경로
layout: learn
authors: flaviocopes, MylesBorins, fhemberger, LaRuaNa, amiller-gh, ahmadawais
---

# Node.js 파일 경로
> ❗️ *번역 날짜: 2024년 12월 23일* <br>
> 공식 문서 원문은 아래를 참고하세요.<br>
>[Node.js File Paths](https://nodejs.org/en/learn/manipulating-files/nodejs-file-paths)

시스템의 모든 파일에는 경로가 있습니다. Linux와 macOS에서 경로는 다음과 같이 보일 수 있습니다: `/users/joe/file.txt` 와 같은 구조이지만, Windows 컴퓨터는 이와 다릅니다: `C:\users\joe\file.txt` 와 같은 구조입니다.

애플리케이션에서 경로를 사용할 때는 이 차이를 고려해야 하므로 주의를 기울여야 합니다.

이 모듈은 `const path = require('node:path');`를 사용하여 파일에 포함하면 해당 메서드를 사용할 수 있습니다.

## 경로에서 정보 가져오기

경로가 주어지면 이러한 방법을 사용하여 경로에서 정보를 추출할 수 있습니다:

- `dirname`: 파일의 상위 폴더를 가져옵니다.
- `basename`: 파일명 부분을 가져옵니다.
- `extname`: 파일 확장자를 가져옵니다.

### 예제

```cjs
const path = require('node:path');

const notes = '/users/joe/notes.txt';

path.dirname(notes); // /users/joe
path.basename(notes); // notes.txt
path.extname(notes); // .txt
```

```mjs
import path from 'node:path';

const notes = '/users/joe/notes.txt';

path.dirname(notes); // /users/joe
path.basename(notes); // notes.txt
path.extname(notes); // .txt
```

두 번째 인수를 `basename`으로 지정하여 확장자 없이 파일 이름을 가져올 수 있습니다:

```js
path.basename(notes, path.extname(notes)); // notes
```

## 경로로 작업하기

`path.join()`을 사용하여 두 개 이상의 경로 부분을 결합할 수 있습니다:

```js
const name = 'joe';
path.join('/', 'users', name, 'notes.txt'); // '/users/joe/notes.txt'
```

`path.resolve()`를 사용하여 상대 경로의 절대 경로를 계산할 수 있습니다:

```js
path.resolve('joe.txt'); // 내 홈 폴더에서 실행하는 경우 '/Users/joe/joe.txt'
```

이 경우 Node.js는 단순히 현재 작업 디렉터리에 `/joe.txt`를 추가합니다. 두 번째 매개변수 폴더를 지정하면 `resolve`는 첫 번째를 두 번째의 기본으로 사용합니다:

```js
path.resolve('tmp', 'joe.txt'); // 내 홈 폴더에서 실행하는 경우 '/Users/joe/tmp/joe.txt'
```

첫 번째 매개변수가 슬래시로 시작하면 절대 경로라는 뜻입니다:

```js
path.resolve('/etc', 'joe.txt'); // '/etc/joe.txt'
```

`path.normalize()`는 `.` 또는 `..`와 같은 상대 지정자나 이중 슬래시가 포함된 경우 실제 경로를 계산하는 또 다른 유용한 함수입니다:

```js
path.normalize('/users/joe/..//test.txt'); // '/users/test.txt'
```

**해결이나 정규화 모두 경로가 존재하는지 확인하지 않습니다**. 단지 얻은 정보를 기반으로 경로를 계산할 뿐입니다.