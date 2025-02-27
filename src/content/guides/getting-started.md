---
title: Getting Started
description: Learn how to bundle a JavaScript application with webpack 5.
sort: 1
contributors:
  - bebraw
  - varunjayaraman
  - cntanglijun
  - chrisVillanueva
  - johnstew
  - simon04
  - aaronang
  - TheDutchCoder
  - sudarsangp
  - Vanguard90
  - chenxsan
  - EugeneHlushko
  - ATGardner
  - ayvarot
  - bjarki
  - ztomasze
  - Spiral90210
  - byzyk
  - wizardofhogwarts
  - myshov
  - anshumanv
---

webpack은 JavsScript 모듈을 컴파일할 때 사용됩니다. [설치하면](/guides/installation), [CLI](/api/cli) 또는 [API](/api/node)로 webpack과 상호 작용할 수 있습니다. 아직 webpack이 익숙하지 않은 경우 [핵심 개념](/concepts)과 [비교 내용](/comparison)을 통해 커뮤니티의 다른 도구보다 왜 webpack을 사용해야 할지 알아보세요.

W> webpack 5를 실행하기 위한 최소 Node.js 버전은 10.13.0(LTS) 입니다.

## Basic Setup

먼저 디렉터리를 생성합니다. 그 다음 npm을 초기화하고, [webpack을 로컬로 설치](/guides/installation/#local-installation)한 후 [`webpack-cli`](https://github.com/webpack/webpack-cli)(커맨드-라인에서 webpack을 실행할 때 사용되는 도구)를 설치합니다.

```bash
mkdir webpack-demo
cd webpack-demo
npm init -y
npm install webpack webpack-cli --save-dev
```

가이드 전반에 걸쳐 **`diff`** 블록을 사용하여 디렉터리, 파일 코드의 변경을 보여줍니다. 예를 들면,

```diff
+ 이것은 코드에 복사할 새로운 라인 입니다.
- 그리고 이것은 코드에서 삭제될 라인 입니다.
  그리고 이것은 손대지 말아야 할 라인 입니다.
```

이제 다음의 디렉터리 구조와 파일, 콘텐츠를 생성합니다.

**project**

```diff
  webpack-demo
  |- package.json
+ |- index.html
+ |- /src
+   |- index.js
```

**src/index.js**

```javascript
function component() {
  const element = document.createElement('div');

  // 이 라인이 동작하려면 현재 스크립트를 통해 포함된 Lodash가 필요합니다.
  element.innerHTML = _.join(['Hello', 'webpack'], ' ');

  return element;
}

document.body.appendChild(component());
```

**index.html**

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Getting Started</title>
    <script src="https://unpkg.com/lodash@4.17.20"></script>
  </head>
  <body>
    <script src="./src/index.js"></script>
  </body>
</html>
```

또한 패키지를 `private`로 표기하고 `main` 항목을 제거하기 위해 `package.json` 파일을 조정해야 합니다. 이것은 실수로 코드가 출시되는 것을 방지하기 위한 것입니다.

T> `package.json`의 내부 동작을 더 알고 싶으면 [npm 문서](https://docs.npmjs.com/files/package.json)를 읽어 보시기 바랍니다.

**package.json**

```diff
 {
   "name": "webpack-demo",
   "version": "1.0.0",
   "description": "",
-  "main": "index.js",
+  "private": true,
   "scripts": {
     "test": "echo \"Error: no test specified\" && exit 1"
   },
   "keywords": [],
   "author": "",
   "license": "ISC",
   "devDependencies": {
     "webpack": "^5.4.0",
     "webpack-cli": "^4.2.0"
   }
 }
```

예시에서 `<script>` 태그 사이에는 암시적인 의존성이 있습니다. `index.js` 파일은 실행되기 전에 페이지에 포함되는 `lodash`와 연관이 있습니다. 이는 `index.js`가 `lodash`의 필요성을 명시적으로 선언하지 않았기 때문입니다. 단지 전역 변수인 `_`가 존재하는지 추정할 뿐입니다.

이러한 방식으로 JavaScript 프로젝트를 관리하는 것은 문제가 있습니다.

- 해당 스크립트가 외부 라이브러리에 의존한다는 것이 명확하지 않습니다.
- 의존성을 잃어버렸거나 잘못된 순서로 포함되었으면 애플리케이션이 제대로 작동하지 않습니다.
- 의존성이 포함되었지만 사용되지 않는 경우에도 브라우저는 필요 없는 코드를 강제로 다운로드합니다.

대신 webpack을 사용하여 이러한 스크립트를 관리할 수 있습니다.

## Creating a Bundle

먼저 디렉터리 구조를 약간 수정하여 "배포" 코드(`./dist`)를 "소스" 코드(`./src`)와 분리합니다. "소스" 코드는 우리가 작성하고 편집하는 코드입니다. "배포" 코드는 빌드 과정을 통해 최소화하고 최적화되어 궁극적으로 브라우저에서 로드될 `출력물` 입니다. 다음과 같이 디렉터리 구조를 변경합니다.

**project**

```diff
  webpack-demo
  |- package.json
+ |- /dist
+   |- index.html
- |- index.html
  |- /src
    |- index.js
```

T> 자세히 관찰한 분들은 비록 `index.html`이 `dist` 디렉터리에 있지만, 수동으로 생성되었다는 것을 알아챘을 것입니다. 나중에 [다른 가이드에서](/guides/output-management/#setting-up-htmlwebpackplugin), `index.html`을 수동으로 수정하는 대신 자동으로 생성할 것입니다. 이 작업이 완료되면 `dist` 디렉터리를 비우고 모든 파일을 다시 생성해도 좋습니다.

`lodash`의 의존성을 `index.js`와 함께 번들링 하려면, 라이브러리를 로컬에서 설치해야 합니다.

```bash
npm install --save lodash
```

T> 프로덕션 번들에 번들로 제공될 패키지를 설치할 때 `npm install --save`를 사용해야 합니다. 개발 목적(e.g. a linter, testing libraries, etc.)으로 패키지를 설치한다면 `npm install --save-dev`를 사용해야 합니다. 자세한 내용은 [npm 문서를](https://docs.npmjs.com/cli/install) 참고하세요.

지금부터 스크립트로 `lodash`를 가져오겠습니다.

**src/index.js**

```diff
+import _ from 'lodash';
+
 function component() {
   const element = document.createElement('div');

-  // 이 라인이 동작하려면 현재 스크립트를 통해 포함된 Lodash가 필요합니다.
+  // 이제 Lodash를 스크립트로 가져왔습니다.
   element.innerHTML = _.join(['Hello', 'webpack'], ' ');

   return element;
 }

 document.body.appendChild(component());
```

이제 스크립트로 번들링 할 것이므로 `index.html`을 업데이트해야 합니다. 현재 `import`한 lodash `<script>`를 삭제하고 원래의 `./src` 파일 대신 다른  `<script>` 태그로 번들을 로드하도록 수정합니다.

**dist/index.html**

```diff
 <!DOCTYPE html>
 <html>
   <head>
     <meta charset="utf-8" />
     <title>Getting Started</title>
-    <script src="https://unpkg.com/lodash@4.17.20"></script>
   </head>
   <body>
-    <script src="./src/index.js"></script>
+    <script src="main.js"></script>
   </body>
 </html>
```

이 설정에서 `index.js`는 명시적으로 `lodash`가 있어야 하며, 이것을 `_`에 바인딩합니다.(글로벌 스코프의 오염 없음) 모듈에 필요한 의존성을 명시함으로써 webpack은 이 정보를 사용하여 디펜던시 그래프를 만들 수 있습니다. 그런 다음 그래프를 사용하여 스크립트가 올바른 순서로 실행되는 최적화된 번들을 생성합니다.

그럼 `npx webpack`을 실행해 보겠습니다. 이 스크립트는 `src/index.js`를 [엔트리 포인트](/concepts/entry-points)로 사용하고 [output](/concepts/output)으로 `dist/main.js`을 생성합니다. `npx` 명령어는 Node 8.2/npm 5.2.0 이상 버전에서 제공되며, 처음에 설치했던 webpack 패키지의 webpack 바이너리(`./node_modules/.bin/webpack`)를 실행합니다.

```bash
$ npx webpack
[webpack-cli] Compilation finished
asset main.js 69.3 KiB [emitted] [minimized] (name: main) 1 related asset
runtime modules 1000 bytes 5 modules
cacheable modules 530 KiB
  ./src/index.js 257 bytes [built] [code generated]
  ./node_modules/lodash/lodash.js 530 KiB [built] [code generated]
webpack 5.4.0 compiled successfully in 1851 ms
```

T> 출력은 약간 다를 수 있지만, 빌드에 성공했다면 계속 진행해도 좋습니다.

브라우저에서 `dist` 디렉터리의 `index.html`을 열어봅니다. 모든 것이 제대로 되었다면 `'Hello webpack'` 글자가 표시될 것입니다.

## Modules

[`import 문`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)과 [`export 문`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export)은 [ES2015](https://babeljs.io/docs/en/learn/)에서 표준화되었습니다. 현재는 대부분의 브라우저에서 지원되지만, 몇몇 브라우저에서는 새 구문을 인식하지 못합니다. 하지만 webpack은 바로 사용할 수 있도록 지원해주니 걱정하지 마세요.

보이지않는 곳에서 webpack이 실제로 코드를 "**트랜스파일**" 하여 이전 브라우저에서도 실행 할 수 있도록 합니다. `dist/main.js`을 보면 webpack이 어떻게 트랜스파일 하는지 볼 수 있을 것입니다. 매우 독창적입니다! `import`와 `export` 외에도 webpack은 다양한 모듈 구문을 지원합니다. 자세한 내용은 [Module API에서](/api/module-methods) 볼 수 있습니다.

webpack은 `import`와 `export` 문 이외는 코드를 변경하지 않습니다. 다른 [ES2015 기능](http://es6-features.org/)을 사용한다면 webpack의 [로더 시스템](/concepts/loaders/)인 [Babel](https://babeljs.io/)이나 [Bublé](https://buble.surge.sh/guide/)을 [트랜스파일러로 사용](/loaders/#transpiling)해야 합니다.

## Using a Configuration

버전 4부터 webpack은 어떠한 설정도 필요하지 않습니다. 하지만 대부분의 프로젝트는 좀 더 복잡한 설정이 필요하므로 webpack에서 [설정 파일](/concepts/configuration)을 제공합니다. 이것은 터미널에서 많은 명령어를 수동으로 입력하는 것보다 훨씬 효율적입니다. 다음과 같이 생성해 보겠습니다.

**project**

```diff
  webpack-demo
  |- package.json
+ |- webpack.config.js
  |- /dist
    |- index.html
  |- /src
    |- index.js
```

**webpack.config.js**

```javascript
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist'),
  },
};
```

이제 새로운 설정 파일을 이용하여 다시 빌드를 실행해 보세요.

```bash
$ npx webpack --config webpack.config.js
[webpack-cli] Compilation finished
asset main.js 69.3 KiB [compared for emit] [minimized] (name: main) 1 related asset
runtime modules 1000 bytes 5 modules
cacheable modules 530 KiB
  ./src/index.js 257 bytes [built] [code generated]
  ./node_modules/lodash/lodash.js 530 KiB [built] [code generated]
webpack 5.4.0 compiled successfully in 1934 ms
```

T> `webpack.config.js`가 있으면 `webpack` 명령은 기본으로 이것을 선택합니다. 여기서는 `--config` 옵션을 사용하여 어떠한 이름의 설정이던 사용할 수 있도록 합니다. 이것은 여러 개의 파일로 분할해야 하는 복잡한 설정에서 유용합니다.

설정 파일은 단순한 CLI 사용보다 훨씬 많은 유연성을 제공합니다. 로더의 규칙, 플러그인, 해석 옵션 및 기타 여러 향상된 기능을 지정할 수 있습니다. 더 자세한 것은 [설정 문서](/configuration)를 참고하세요.

## NPM Scripts

CLI에서 webpack의 로컬 사본을 실행하기 위해 약간의 단축 명령어를 설정 할 수 있습니다. [npm script](https://docs.npmjs.com/misc/scripts)를 추가하여 _package.json을_ 수정해 보겠습니다.

**package.json**

```diff
 {
   "name": "webpack-demo",
   "version": "1.0.0",
   "description": "",
   "private": true,
   "scripts": {
-    "test": "echo \"Error: no test specified\" && exit 1"
+    "test": "echo \"Error: no test specified\" && exit 1",
+    "build": "webpack"
   },
   "keywords": [],
   "author": "",
   "license": "ISC",
   "devDependencies": {
     "webpack": "^5.4.0",
     "webpack-cli": "^4.2.0"
   },
   "dependencies": {
     "lodash": "^4.17.20"
   }
 }
```

이제 이전에 사용한 `npx` 명령 대신 `npm run build` 명령을 사용할 수 있습니다. `scripts`에서는 `npx`와 동일한 방식으로 로컬에서 설치된 npm 패키지를 이름으로 참조할 수 있습니다. 이 규칙은 모든 컨트리뷰터가 동일한 공통의 스크립트 세트를 사용할 수 있도록 하므로 대부분의 npm 기반 프로젝트에서 표준입니다.

이제 다음 명령을 실행하고 스크립트의 별칭이 작동하는지 확인하세요.

```bash
$ npm run build

...

[webpack-cli] Compilation finished
asset main.js 69.3 KiB [compared for emit] [minimized] (name: main) 1 related asset
runtime modules 1000 bytes 5 modules
cacheable modules 530 KiB
  ./src/index.js 257 bytes [built] [code generated]
  ./node_modules/lodash/lodash.js 530 KiB [built] [code generated]
webpack 5.4.0 compiled successfully in 1940 ms
```

T> 사용자 지정 파라미터는 `npm run build` 명령과 파라미터 사이에 두 개의 대시(예.  `npm run build -- --color`)를 추가하여 webpack에 전달할 수 있습니다.

## Conclusion

이제 기본 빌드를 함께 완료하였습니다. 다음 가이드인 [`Asset Management로`](/guides/asset-management) 이동하여 webpack을 이용한 이미지나 폰트 같은 애셋 관리 방법을 알아보겠습니다. 이 시점에서 프로젝트는 아래와 같아야 합니다.

**project**

```diff
webpack-demo
|- package.json
|- webpack.config.js
|- /dist
  |- main.js
  |- index.html
|- /src
  |- index.js
|- /node_modules
```

T> npm 5+ 이상을 사용하는 경우에는 디렉터리에 `package-lock.json` 파일도 표시될 것입니다.

W> 신뢰할 수 없는 코드는 webpack으로 컴파일하지 마세요. 이로 인해 컴퓨터, 원격 서버 또는 애플리케이션 최종 사용자의 웹 브라우저에서 악성 코드가 실행될 수 있습니다.

webpack 디자인에 대해 자세히 알아보고 싶으면 [basic concepts](/concepts)과 [configuration](/configuration) 페이지를 확인하세요. 또한 [API](/api)에서 webpack이 제공하는 다양한 인터페이스를 자세히 살펴봅니다.
