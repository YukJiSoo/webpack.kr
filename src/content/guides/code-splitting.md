---
title: Code Splitting
sort: 5
contributors:
  - pksjce
  - pastelsky
  - simon04
  - jonwheeler
  - johnstew
  - shinxi
  - tomtasche
  - levy9527
  - rahulcs
  - chrisVillanueva
  - rafde
  - bartushek
  - shaunwallace
  - skipjack
  - jakearchibald
  - TheDutchCoder
  - rouzbeh84
  - shaodahong
  - sudarsangp
  - kcolton
  - efreitasn
  - EugeneHlushko
  - Tiendo1011
  - byzyk
  - AnayaDesign
  - wizardofhogwarts
  - maximilianschmelzer
  - smelukov
  - chenxsan
  - Adarah
related:
  - title: <link rel=”prefetch/preload”> in webpack
    url: https://medium.com/webpack/link-rel-prefetch-preload-in-webpack-51a52358f84c
  - title: Preload, Prefetch And Priorities in Chrome
    url: https://medium.com/reloading/preload-prefetch-and-priorities-in-chrome-776165961bbf
  - title: Preloading content with rel="preload"
    url: https://developer.mozilla.org/en-US/docs/Web/HTML/Preloading_content
---

T> 이 가이드는 [Getting Started](/guides/getting-started)에 제공된 예제를 기준으로 합니다. 해당 예제와 [Output Management](/guides/output-management/) 챕터를 미리 알아두도록 합니다.

코드 스플리팅은 webpack의 가장 매력적인 기능 중 하나입니다. 이 기능을 사용하여 코드를 다양한 번들로 분할하고, 요청에 따라 로드하거나 병렬로 로드할 수 있습니다. 더 작은 번들을 만들고 리소스 우선순위를 올바르게 제어하기 위해서 사용하며, 잘 활용하면 로드 시간에 큰 영향을 끼칠 수 있습니다.

일반적으로 코드 스플리팅은 세 가지 방식으로 접근할 수 있습니다.

- **Entry Points**: [`entry`](/configuration/entry-context) 설정을 사용하여 코드를 수동으로 분할합니다.
- **Prevent Duplication**: [Entry dependencies](/configuration/entry-context/#dependencies) 또는 [`SplitChunksPlugin`](/plugins/split-chunks-plugin/)을 사용하여 중복 청크를 제거하고 청크를 분할합니다.
- **Dynamic Imports**: 모듈 내에서 인라인 함수 호출을 통해 코드를 분할합니다.


## Entry Points

코드를 분할하는 가장 쉽고 직관적인 방법입니다. 그러나 다른 방법에 비해 수동적이며, 같이 살펴볼 몇 가지 함정이 있습니다. 메인 번들에서 다른 모듈을 어떻게 분리하는지 알아보겠습니다.

**project**

```diff
webpack-demo
|- package.json
|- webpack.config.js
|- /dist
|- /src
  |- index.js
+ |- another-module.js
|- /node_modules
```

**another-module.js**

```js
import _ from 'lodash';

console.log(_.join(['Another', 'module', 'loaded!'], ' '));
```

**webpack.config.js**

```diff
 const path = require('path');

 module.exports = {
-  entry: './src/index.js',
+  mode: 'development',
+  entry: {
+    index: './src/index.js',
+    another: './src/another-module.js',
+  },
   output: {
-    filename: 'main.js',
+    filename: '[name].bundle.js',
     path: path.resolve(__dirname, 'dist'),
   },
 };
```

다음과 같은 빌드 결과가 생성됩니다.

```bash
...
[webpack-cli] Compilation finished
asset index.bundle.js 553 KiB [emitted] (name: index)
asset another.bundle.js 553 KiB [emitted] (name: another)
runtime modules 2.49 KiB 12 modules
cacheable modules 530 KiB
  ./src/index.js 257 bytes [built] [code generated]
  ./src/another-module.js 84 bytes [built] [code generated]
  ./node_modules/lodash/lodash.js 530 KiB [built] [code generated]
webpack 5.4.0 compiled successfully in 245 ms
```

언급했듯이 이 접근 방식에는 몇 가지 함정이 있습니다.

- 엔트리 청크 사이에 중복된 모듈이 있는 경우 두 번들에 모두 포함됩니다.
- 코어 애플리케이션 로직을 통한 코드의 동적 분할에는 사용할 수 없으며 유연하지 않습니다.

이 중 첫 번째 항목을 통해 지금 예제의 문제를 알 수 있습니다. 왜냐하면 `./src/index.js`에서도 `lodash`를 가져오므로 양쪽 번들에서 중복으로 포함되기 때문입니다. 다음 섹션에서 중복된 것을 제거하겠습니다.

## Prevent Duplication

### Entry dependencies

[`dependOn` 옵션](/configuration/entry-context/#dependencies)을 사용하면 청크간 모듈을 공유할 수 있습니다.

**webpack.config.js**

```diff
 const path = require('path');

 module.exports = {
   mode: 'development',
   entry: {
-    index: './src/index.js',
-    another: './src/another-module.js',
+    index: {
+      import: './src/index.js',
+      dependOn: 'shared',
+    },
+    another: {
+      import: './src/another-module.js',
+      dependOn: 'shared',
+    },
+    shared: 'lodash',
   },
   output: {
     filename: '[name].bundle.js',
     path: path.resolve(__dirname, 'dist'),
   },
 };
```

단일 HTML 페이지에서 여러 엔트리 포인트를 사용하는 경우 `optimization.runtimeChunk: 'single'`도 필요합니다. 그렇지 않으면 [여기](https://bundlers.tooling.report/)에서 설명하는 문제가 발생할 수 있습니다.

**webpack.config.js**

```diff
 const path = require('path');

 module.exports = {
   mode: 'development',
   entry: {
     index: {
       import: './src/index.js',
       dependOn: 'shared',
     },
     another: {
       import: './src/another-module.js',
       dependOn: 'shared',
     },
     shared: 'lodash',
   },
   output: {
     filename: '[name].bundle.js',
     path: path.resolve(__dirname, 'dist'),
   },
+  optimization: {
+    runtimeChunk: 'single',
+  },
 };
```

다음은 빌드 결과입니다.

```bash
...
[webpack-cli] Compilation finished
asset shared.bundle.js 549 KiB [compared for emit] (name: shared)
asset runtime.bundle.js 7.79 KiB [compared for emit] (name: runtime)
asset index.bundle.js 1.77 KiB [compared for emit] (name: index)
asset another.bundle.js 1.65 KiB [compared for emit] (name: another)
Entrypoint index 1.77 KiB = index.bundle.js
Entrypoint another 1.65 KiB = another.bundle.js
Entrypoint shared 557 KiB = runtime.bundle.js 7.79 KiB shared.bundle.js 549 KiB
runtime modules 3.76 KiB 7 modules
cacheable modules 530 KiB
  ./node_modules/lodash/lodash.js 530 KiB [built] [code generated]
  ./src/another-module.js 84 bytes [built] [code generated]
  ./src/index.js 257 bytes [built] [code generated]
webpack 5.4.0 compiled successfully in 249 ms
```

보시다시피 `shared.bundle.js`, `index.bundle.js` 및 `another.bundle.js` 외에 또 다른 `runtime.bundle.js` 파일이 생성됩니다.

webpack은 하나의 페이지에 여러 엔트리 포인트를 허용하지만, 가능하다면 `entry: { page: ['./analytics', './app'] }`처럼 여러 개의 import가 포함된 엔트리 포인트 사용을 피해야 합니다. 이는 `async` 스크립트 태그를 사용할 때 최적화에 용이하며 일관된 순서로 실행할 수 있도록 합니다.

### `SplitChunksPlugin`

[`SplitChunksPlugin`](/plugins/split-chunks-plugin/)을 사용하면 기존 엔트리 청크 또는 완전히 새로운 청크로 공통 의존성을 추출할 수 있습니다. 이를 활용하여 이전 예제의 `lodash` 중복을 제거해 보겠습니다.

**webpack.config.js**

```diff
  const path = require('path');

  module.exports = {
    mode: 'development',
    entry: {
      index: './src/index.js',
      another: './src/another-module.js',
    },
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist'),
    },
+   optimization: {
+     splitChunks: {
+       chunks: 'all',
+     },
+   },
  };
```

[`optimization.splitChunks`](/plugins/split-chunks-plugin/#optimizationsplitchunks) 설정 옵션을 적용하면 `index.bundle.js`와 `another.bundle.js`에서 중복 의존성이 제거된 것을 확인 할 수 있습니다. 플러그인은 `lodash`가 별도의 청크로 분리되었고 메인 번들에서도 제거된 것을 알 수 있습니다. 잘 동작하는지 확인하기 위해 `npm run build`를 실행해 보겠습니다.

```bash
...
[webpack-cli] Compilation finished
asset vendors-node_modules_lodash_lodash_js.bundle.js 549 KiB [compared for emit] (id hint: vendors)
asset index.bundle.js 8.92 KiB [compared for emit] (name: index)
asset another.bundle.js 8.8 KiB [compared for emit] (name: another)
Entrypoint index 558 KiB = vendors-node_modules_lodash_lodash_js.bundle.js 549 KiB index.bundle.js 8.92 KiB
Entrypoint another 558 KiB = vendors-node_modules_lodash_lodash_js.bundle.js 549 KiB another.bundle.js 8.8 KiB
runtime modules 7.64 KiB 14 modules
cacheable modules 530 KiB
  ./src/index.js 257 bytes [built] [code generated]
  ./src/another-module.js 84 bytes [built] [code generated]
  ./node_modules/lodash/lodash.js 530 KiB [built] [code generated]
webpack 5.4.0 compiled successfully in 241 ms
```

다음은 코드 스플리팅을 위해 커뮤니티에서 제공하는 다른 유용한 플러그인과 로더입니다.

-[`mini-css-extract-plugin`](/plugins/mini-css-extract-plugin) : 메인 애플리케이션에서 CSS를 분리하는데 유용합니다.

## Dynamic Imports

webpack은 동적 코드 스플리팅에 두 가지 유사한 기술을 지원합니다. 첫 번째이자 권장하는 접근 방식은 [ECMAScript 제안을](https://github.com/tc39/proposal-dynamic-import) 준수하는 [`import()`구문](/api/module-methods/#import-1)을 사용하는 방식입니다. 기존의 webpack 전용 방식은 [`require.ensure`](/api/module-methods/#requireensure)를 사용하는 것입니다. 이 두 가지 중 첫 번째를 사용해 보겠습니다.

W> `import()` 호출은 내부적으로 [promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)를 사용합니다. 이전 브라우저(예: IE 11)에서 `import()`를 사용하는 경우 [es6-promise](https://github.com/stefanpenner/es6-promise)나 [promise-polyfill](https://github.com/taylorhakes/promise-polyfill)과 같은 폴리필을 사용하여 `Promise`를 지원하도록 해야 합니다.

시작하기 전에 위 예제의 설정에서 추가 [`entry`](/concepts/entry-points/) 및 [`optimization.splitChunks`](/plugins/split-chunks-plugin)를 제거하겠습니다. 다음 데모에는 필요하지 않습니다.

**webpack.config.js**

```diff
 const path = require('path');

 module.exports = {
   mode: 'development',
   entry: {
     index: './src/index.js',
-    another: './src/another-module.js',
   },
   output: {
     filename: '[name].bundle.js',
     path: path.resolve(__dirname, 'dist'),
   },
-  optimization: {
-    splitChunks: {
-      chunks: 'all',
-    },
-  },
 };
```

또한 현재 사용하지 않는 파일을 프로젝트에서 제거하겠습니다.

**project**

```diff
webpack-demo
|- package.json
|- webpack.config.js
|- /dist
|- /src
  |- index.js
- |- another-module.js
|- /node_modules
```

이제 정적으로 가져오던 `lodash`를 동적으로 가져와서 청크를 분리해보겠습니다.

**src/index.js**

```diff
-import _ from 'lodash';
-
-function component() {
+function getComponent() {
   const element = document.createElement('div');

-  // Lodash, now imported by this script
-  element.innerHTML = _.join(['Hello', 'webpack'], ' ');
+  return import('lodash')
+    .then(({ default: _ }) => {
+      const element = document.createElement('div');
+
+      element.innerHTML = _.join(['Hello', 'webpack'], ' ');

-  return element;
+      return element;
+    })
+    .catch((error) => 'An error occurred while loading the component');
 }

-document.body.appendChild(component());
+getComponent().then((component) => {
+  document.body.appendChild(component);
+});
```

`default`가 필요한 이유는 webpack 4 이후로 CommonJS 모듈을 가져올 때 더 이상 `module.exports` 값 으로 해석되지 않으며 대신 CommonJS 모듈에 대한 인공 네임 스페이스 객체를 생성하기 때문입니다. 그 이유에 대한 자세한 내용은 [webpack 4: import() 및 CommonJs](https://medium.com/webpack/webpack-4-import-and-commonjs-d619d626b655)를 참고하세요.

webpack을 실행하여 `lodash`가 별도의 번들로 분리되어 있는지 살펴보겠습니다.

```bash
...
[webpack-cli] Compilation finished
asset vendors-node_modules_lodash_lodash_js.bundle.js 549 KiB [compared for emit] (id hint: vendors)
asset index.bundle.js 13.5 KiB [compared for emit] (name: index)
runtime modules 7.37 KiB 11 modules
cacheable modules 530 KiB
  ./src/index.js 434 bytes [built] [code generated]
  ./node_modules/lodash/lodash.js 530 KiB [built] [code generated]
webpack 5.4.0 compiled successfully in 268 ms
```

`import()`는 promise를 반환하므로 [`async` 함수](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)와 함께 사용할 수 있습니다. 코드를 단순화하는 방법은 다음과 같습니다.

**src/index.js**

```diff
-function getComponent() {
+async function getComponent() {
   const element = document.createElement('div');
+  const { default: _ } = await import('lodash');

-  return import('lodash')
-    .then(({ default: _ }) => {
-      const element = document.createElement('div');
+  element.innerHTML = _.join(['Hello', 'webpack'], ' ');

-      element.innerHTML = _.join(['Hello', 'webpack'], ' ');
-
-      return element;
-    })
-    .catch((error) => 'An error occurred while loading the component');
+  return element;
 }

 getComponent().then((component) => {
   document.body.appendChild(component);
 });
```

T> 나중에 계산된 변수를 기반으로 특정 모듈을 가져와야 할 경우 `import()`에 [dynamic expression](/api/module-methods/#dynamic-expressions-in-import)을 사용 할 수 있습니다.

## Prefetching/Preloading modules

webpack 4.6.0+에서 프리페치 및 프리로드에 대한 지원이 추가되었습니다.

모듈을 가져올 때 인라인 지시문을 사용하면 webpack이 브라우저에 아래와 같은 "리소스 힌트"를 줄 수 있습니다.

- **prefetch** : 향후 일부 탐색에 리소스가 필요할 수 있습니다.
- **preload** : 현재 탐색 중에 리소스도 필요합니다.

간단한 프리페치의 예제를 들어보겠습니다. `HomePage` 컴포넌트에서 `LoginButton` 컴포넌트를 렌더링하고, 이 컴포넌트를 클릭하면 `LoginModal` 컴포넌트를 요청하여 로드하는 경우입니다.

**LoginButton.js**

```js
//...
import(/* webpackPrefetch: true */ './path/to/LoginModal.js');
```

이는 페이지 head에 `<link rel="prefetch" href="login-modal-chunk.js">`를 추가하고 브라우저에 `login-modal-chunk.js`를 유휴 시간에 미리 가져오도록 지시합니다.

T> webpack은 부모 청크가 로드된 후 프리페치 힌트를 추가합니다.

프리로드 지시문은 프리페치와 비교했을 때 여러 가지 차이점이 있습니다.

- 프리로드 청크는 부모 청크와 병렬로 로드를 시작합니다. 프리페치 청크는 부모 청크가 로드 완료된 후에 로드를 시작합니다.
- 프리로드 청크는 중간 우선순위를 가지며 즉시 다운로드됩니다. 프리페치 청크는 브라우저가 유휴 상태일 때 다운로드 됩니다.
- 프리로드 청크는 부모 청크에서 즉시 요청 되어야 합니다. 프리페치 청크는 나중에 언제라도 사용할 수 있습니다.
- 지원하는 브라우저에 차이가 있습니다.

간단한 프리로드의 예로는, 별도의 청크에 있어야 하는 큰 라이브러리에 항상 의존하는 `Component`를 생각해 볼 수 있습니다.

거대한 `ChartingLibrary`가 필요한 `ChartComponent`를 상상해 봅시다. 렌더링 될 때 `LoadingIndicator`를 표시하고 즉시 `ChartingLibrary`를 요청하여 불러옵니다.

**ChartComponent.js**

```js
//...
import(/* webpackPreload: true */ 'ChartingLibrary');
```

`ChartComponent`를 사용하는 페이지를 요청할 때 `<link rel="preload">`를 통해 charting-library-chunk도 요청됩니다. page-chunk가 더 작고 더 빨리 완료된다고 가정하면 이미 요청된 `charting-library-chunk`가 완료될 때까지 페이지에는 `LoadingIndicator`가 표시됩니다. 두 번이 아닌 한 번의 라운드 트립이 필요하므로 대기 시간이 긴 환경에서 로드 시간이 증가할 수 있습니다.

T> `webpackPreload`를 잘못 사용하면 실제로 성능이 저하 될 수 있으므로 사용 시 주의하세요.

## Bundle Analysis

코드 스플리팅을 시작하면 출력을 분석하여 어디서 모듈이 종료되었는지 확인하는 데 유용합니다. [공식 분석 도구](https://github.com/webpack/analyse)부터 시작하는 것이 좋습니다. 커뮤니티에서 지원하는 다른 옵션도 있습니다.

- [webpack-chart](https://alexkuz.github.io/webpack-chart/): webpack 통계를 위한 인터렉티브 원형 차트.
- [webpack-visualizer](https://chrisbateman.github.io/webpack-visualizer/): 번들을 시각화하고 분석하여 어떤 모듈이 공간을 차지하고 있고 어떤 모듈이 중복될 수 있는지 확인합니다.
- [webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer):  확대/축소 가능한 편리한 인터렉티브 트리 맵으로 번들 콘텐츠를 표현하는 플러그인 및 CLI 유틸리티입니다.
- [webpack bundle optimize helper](https://webpack.jakoblind.no/optimize): 이 도구는 번들을 분석하고 번들 크기를 줄이기 위한 실용적인 개선 사항을 제공합니다.
- [bundle-stats](https://github.com/bundle-stats/bundle-stats): 번들 보고서(번들 크기, 애셋, 모듈)를 생성하고 서로 다른 빌드 간의 결과를 비교합니다.

## Next Steps

실제 애플리케이션에서 어떻게 `import()`를 사용하는지 더 구체적으로 알고 싶다면 [Lazy Loading](/guides/lazy-loading/)의 예제를 확인하세요. 더 효율적인 코드 스플리팅 방법은 [Caching](/guides/caching/)을 참고하세요.
