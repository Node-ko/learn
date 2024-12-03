---
title: ECMAScript 2015(ES6) 및 그 너머
layout: learn
authors: ovflowd
---

# ECMAScript 2015(ES6) 및 그 너머
> ❗️ *번역 날짜: 2024년 12월 03일* <br>
> 공식 문서 원문은 아래를 참고하세요.<br>
> [ECMAScript 2015 (ES6) and beyond](https://nodejs.org/en/learn/getting-started/ecmascript-2015-es6-and-beyond)

Node.js는 최신 버전의 [V8](https://v8.dev/)을 기반으로 구축되었습니다. 이 엔진의 최신 릴리스를 최신 상태로 유지함으로써 [JavaScript ECMA-262 specification](http://www.ecma-international.org/publications/standards/Ecma-262.htm)의 새로운 기능을 적시에 제공하고 성능 및 안정성을 지속적으로 개선하여 Node.js 개발자에게 제공할 수 있습니다.

모든 ECMAScript 2015(ES6) 기능은 **shipping**, **staged**, 그리고 **in progress** 기능의 세 가지 그룹으로 나뉩니다:

- V8이 안정적이라고 간주하는 모든 **shipping** 기능은 **Node.js에서 기본적으로** 켜져있으며 런타임 플래그가 **필요하지 않습니다**.
- V8 팀에서 안정적이라고 간주하지 않는 거의 완성된 기능인 **Staged** 기능에는 런타임 플래그인 `--harmony`가 필요합니다.
- **In progress** 기능은 각각의 하모니 플래그를 통해 개별적으로 활성화할 수 있지만, 테스트 목적이 아니라면 권장하지 않습니다. 참고: 이러한 플래그는 V8에 의해 노출되며 어떠한 지원 중단 예고 없이 변경될 수 있습니다.

### 어떤 기능이 어떤 Node.js 버전과 기본으로 제공되나요?

웹사이트 [node.green](https://node.green/)에서는 kangax의 호환 테이블을 기반으로 다양한 버전의 Node.js에서 지원되는 ECMAScript 기능에 대한 훌륭한 개요를 제공합니다.

### 어떤 기능이 개발 중인가요?

V8 엔진에는 새로운 기능이 지속적으로 추가되고 있습니다. 일반적으로 시기는 알 수 없지만 향후 Node.js 릴리스에 포함될 것으로 예상됩니다.

각 Node.js 릴리스에서 사용 가능한 _in progress_ 기능들을 `--v8-options` 인자를 통해 검색하여 나열할 수 있습니다. 이 기능들은 아직 완성되지 않거나 V8에서 문제가 있을 가능성이 있으니, 사용 시 주의하시기 바랍니다:

```bash
node --v8-options | grep "in progress"
```

### 저는 --harmony 플래그를 활용하도록 인프라를 설정했습니다. 이를 제거해야 할까요?

현재 Node.js에서 `--harmony` 플래그의 동작은 **staged** 기능만 활성화하는 것입니다. 결국, 이는 `--es_staging`의 동의어가 되었습니다. 앞서 언급했듯이, 이 기능들은 완료되었지만 아직 안정적이라고 간주되지 않은 상태입니다. 특히 프로덕션 환경에서는, 이러한 기능이 V8과 Node.js에서 기본으로 활성화될 때까지 이 실행 플래그를 제거하는 것이 더 안전할 수 있습니다. 이 플래그를 활성화한 상태로 유지한다면, V8이 표준을 더 엄격히 따르도록 동작을 변경할 경우, Node.js 업그레이드로 인해 코드가 깨질 가능성에 대비해야 합니다.

### 특정 버전의 Node.js와 함께 제공되는 V8 버전을 찾으려면 어떻게 해야 하나요?

Node.js는 `process` 전역 객체를 통해 특정 바이너리와 함께 제공되는 모든 종속성과 각 버전을 간단하게 나열할 수 있는 방법을 제공합니다. V8 엔진의 경우 터미널에 다음과 같이 입력하여 해당 버전을 검색하세요:

```bash
node -p process.versions.v8
```