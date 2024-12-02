---
title: npm 패키지 매니저 소개
layout: learn
authors: flaviocopes, MylesBorins, LaRuaNa, jgb-solutions, amiller-gh, ahmadawais
---

# npm 패키지 매니저 소개

## npm 소개

`npm`은 Node.js의 기본 패키지 매니저입니다.

2022년 9월 기준으로 npm 레지스트리에 210만 개 이상의 패키지가 등록되어 있어, 거의 모든 용도의 패키지를 찾을 수 있습니다.

처음에는 Node.js 패키지의 의존성을 다운로드하고 관리하기 위해 시작되었지만, 현재는 프론트엔드 JavaScript에서도 널리 사용되고 있습니다.

> [**Yarn**](https://yarnpkg.com/en/)과 [**pnpm**](https://pnpm.io)은 npm CLI의 대안으로, 한번 사용해보세요.

## 패키지

`npm`은 프로젝트의 종속성을 설치, 업데이트 및 관리합니다. 종속성은 라이브러리 및 패키지와 같은 사전 빌드된 코드이며, Node.js 애플리케이션이 작동하는 데 필요합니다.

### 모든 종속성 설치

프로젝트에 `package.json` 파일이 있으면 다음 명령어를 실행하세요.

```bash
npm install
```

`node_modules` 폴더에 프로젝트에 `package.json` 파일에 나열된 모든 종속성을 설치하고, 해당 폴더가 존재하지 않으면 생성합니다.

### 단일 패키지 설치

특정 패키지를 설치하려면 다음 명령어를 실행하세요.

```bash
npm install <package-name>
```

npm 버전 5 이상에서는 이 명령어가 `<패키지-이름>`을 package.json 파일의 dependencies에 자동으로 추가합니다. 버전 5 이전에서는 `--save` 플래그를 사용해야 했습니다.

자주 사용하는 플래그는 다음과 같습니다.

- `--save-dev`: 패키지를 설치하고 package.json 파일의 *devDependencies*에 추가합니다.
- `--no-save`: 패키지를 설치하지만 package.json 파일의 *dependencies*에 추가하지 않습니다.
- `--save-optional`: 패키지를 설치하고 package.json 파일의 *optionalDependencies*에 추가합니다.
- `--no-optional`: 선택적 종속성의 설치를 방지합니다.

아래와 같이, 플래그의 축약형도 사용할 수 있습니다.

- \-S: `--save`
- \-D: `--save-dev`
- \-O: `--save-optional`

*devDependencies*와 *dependencies*의 차이점은 다음과 같습니다. *devDependencies*는 테스트 라이브러리와 같은 개발 도구를 포함하며, *dependencies*는 프로덕션에 번들로 포함됩니다.

*optionalDependencies*의 차이점은 해당 종속성의 설치가 실패해도 전체 설치가 실패하지 않는다는 점입니다. 그러나 프로그램은 해당 종속성이 없을 때 이를 처리할 책임이 있습니다. [optional dependencies](https://docs.npmjs.com/cli/v7/configuring-npm/package-json#optionaldependencies)에 대해 자세히 읽어보세요.

### 패키지 업데이트

패키지 업데이트도 간단하게 할 수 있습니다. 다음 명령어를 실행하세요.

```bash
npm update
```

`npm`은 버전 제약 조건을 만족하는 새로운 버전을 모든 패키지에서 확인합니다.

특정 패키지만 업데이트하려면 다음 명령어를 사용하세요.

```bash
npm update <package-name>
```

## 버전 관리

npm은 **버전 관리** 기능을 제공하여 패키지의 특정 버전을 지정하거나 원하는 범위의 버전을 설치할 수 있습니다.

많은 경우, 라이브러리의 주요 릴리스가 다른 라이브러리와 호환되지 않을 수 있습니다. 또한, 최신 릴리스에 아직 수정되지 않은 버그가 있을 수도 있습니다.

특정 버전의 라이브러리를 지정하면 팀 전체가 동일한 버전을 사용하게 되어, package.json 파일이 업데이트될 때까지 모든 개발자가 동일한 환경에서 작업할 수 있습니다.

이러한 상황에서 버전 관리는 매우 유용하며, npm은 의미 체계 버전 관리(semver) 표준을 따릅니다.

특정 버전의 패키지를 설치하려면 다음 명령어를 실행하세요.

```bash
npm install <package-name>@<version>
```

## 작업 실행

`package.json` 파일의 `scripts` 섹션을 활용하면 다양한 명령어 작업을 정의하고 실행할 수 있습니다. 다음과 같이 명령어를 실행하세요.

```bash
npm run <task-name>
```

예를 들어, `package.json` 파일에 다음과 같이 스크립트를 정의할 수 있습니다.

```json
{
  "scripts": {
    "start-dev": "node lib/server-development",
    "start": "node lib/server-production"
  }
}
```

웹팩(Webpack)을 사용하는 경우, 자주 사용하는 명령어를 스크립트로 정의할 수 있습니다.

```json
{
  "scripts": {
    "watch": "webpack --watch --progress --colors --config webpack.conf.js",
    "dev": "webpack --progress --colors --config webpack.conf.js",
    "prod": "NODE_ENV=production webpack -p --config webpack.conf.js"
  }
}
```

이렇게 스크립트를 정의해두면, 긴 명령어를 매번 입력할 필요 없이 간단하게 실행할 수 있습니다.

```console
$ npm run watch
$ npm run dev
$ npm run prod
```