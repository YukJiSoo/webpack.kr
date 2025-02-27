---
title: Dependency Management
sort: 12
contributors:
  - ndelangen
  - chrisVillanueva
  - sokra
  - byzyk
  - AnayaDesign
---

> es6 modules

> commonjs

> amd

## require with expression

요청에 표현식이 포함된 경우 컨텍스트가 생성되므로, 컴파일 시간에 **정확한** 모듈을 알 수 없습니다.

예를 들어, `.ejs` 파일을 포함하는 다음과 같은 폴더 구조가 있습니다.

```bash
example_directory
│
└───template
│   │   table.ejs
│   │   table-row.ejs
│   │
│   └───directory
│       │   another.ejs
```

다음의 `require()` 호출이 평가될 때

```javascript
require('./template/' + name + '.ejs');
```

webpack은 `require()` 호출을 구문 분석하고 일부 정보를 추출합니다.

```code
Directory: ./template
Regular expression: /^.*\.ejs$/
```

**컨텍스트 모듈**

컨텍스트 모듈이 생성됩니다. 정규 표현식과 일치하는 요청에 필요할 수 있는 **해당 디렉터리의 모든 모듈에** 대한 참조를 포함합니다. 컨텍스트 모듈은 요청을 모듈 id로 변환하는 맵을 포함합니다.

맵 예제:

```json
{
  "./table.ejs": 42,
  "./table-row.ejs": 43,
  "./directory/another.ejs": 44
}
```

컨텍스트 모듈은 또한 맵에 접근하기 위한 런타임 로직도 포함합니다.

이것은 동적으로 요청하는 기능이 지원되지만 모든 모듈이 번들에 포함되는 것을 의미합니다.

## `require.context`

`require.context()` 함수로 자신만의 컨텍스트를 만들 수 있습니다.

검색할 디렉터리, 하위 디렉터리를 검색해야 하는지 여부를 나타내는 플래그,
일치하는 파일의 정규식을 전달할 수 있습니다.

webpack은 빌드 하는 동안 코드에서 `require.context()`를 구문 분석합니다.

구문은 다음과 같습니다.

```javascript
require.context(
  directory,
  (useSubdirectories = true),
  (regExp = /^\.\/.*$/),
  (mode = 'sync')
);
```

예:

```javascript
require.context('./test', false, /\.test\.js$/);
// test 디렉터리에서 요청이 `.test.js`로 끝나는 파일이 있는 컨텍스트입니다.
```

```javascript
require.context('../', true, /\.stories\.js$/);
// 상위 폴더와 그 하위 폴더에서 `.stories.js`로 끝나는 파일이 있는 컨텍스트입니다.
```

W> `require.context`에 전달할 인수는 리터럴이어야 합니다!

### context module API

컨텍스트 모듈은 하나의 인수(요청)를 가지는 함수를 export 합니다.

export 한 함수는 `resolve`, `keys`, `id` 3가지 속성을 가집니다.

- `resolve`는 파싱된 요청의 모듈 id를 반환하는 함수입니다.
- `keys`는 컨텍스트 모듈이 처리할 수 있는 가능한 모든 요청의 배열을 반환하는 함수입니다.

이것은 디렉터리의 모든 파일을 요청하거나 패턴과 일치시키려는 경우에 유용할 수 있습니다. 예제는 다음과 같습니다.

```javascript
function importAll(r) {
  r.keys().forEach(r);
}

importAll(require.context('../components/', true, /\.js$/));
```

```javascript
const cache = {};

function importAll(r) {
  r.keys().forEach((key) => (cache[key] = r(key)));
}

importAll(require.context('../components/', true, /\.js$/));
// 빌드 시 캐시는 모든 필수 모듈로 채워집니다.
```

- `id`는 컨텍스트 모듈의 모듈 id입니다. 이것은 `module.hot.accept`에서 유용할 수 있습니다.
