---
title: Module Federation
sort: 8
contributors:
  - sokra
  - chenxsan
  - EugeneHlushko
  - jamesgeorge007
  - ScriptedAlchemy
related:
  - title: 'Webpack 5 Module Federation: A game-changer in JavaScript architecture'
    url: https://medium.com/swlh/webpack-5-module-federation-a-game-changer-to-javascript-architecture-bcdd30e02669
  - title: 'Explanations and Examples'
    url: https://github.com/module-federation/module-federation-examples
  - title: 'Module Federation YouTube Playlist'
    url: https://www.youtube.com/playlist?list=PLWSiF9YHHK-DqsFHGYbeAMwbd9xcZbEWJ
---

## Motivation

여러 개의 개별 빌드가 단일 애플리케이션을 형성해야 합니다. 이러한 개별 빌드는 서로 의존성이 없어서, 개별적으로 개발하고 배포할 수 있습니다.

이것은 종종 Micro-Frontends라고 알려져 있지만, 그것에만 국한되지는 않습니다.

## Low-level concepts

로컬 모듈과 원격 모듈을 구별합니다. 로컬 모듈은 현재 빌드의 일부인 일반 모듈입니다. 원격 모듈은 현재 빌드의 일부가 아니며 런타임에 컨테이너에서 로드 되는 모듈입니다.

원격 모듈을 로드 하는 것은 비동기 작업으로 간주됩니다. 원격 모듈을 사용할 때 이러한 비동기 작업은 원격 모듈과 엔트리 포인트 사이에 있는 다음 청크 로드 작업에 배치됩니다. 청크 로드 작업 없이는 원격 모듈을 사용할 수 없습니다.

청크 로드 작업은 일반적으로 `import()`를 호출하는 것이지만, `require.ensure` 또는 `require([...])` 같은 이전 구조도 지원됩니다.

특정 모듈에 대한 비동기 접근을 노출하는 컨테이너 엔트리를 통해 컨테이너가 생성됩니다. 노출된 접근은 두 단계로 구분됩니다.

1. 모듈 로드(비동기)
2. 모듈 평가(비동기)

1단계는 청크 로드 중에 수행됩니다. 2단계는 다른 로컬 및 원격 모듈과 인터리브된 모듈 평가 중에 수행됩니다. 이렇게 하면, 모듈을 로컬에서 원격으로 또는 그 반대로 변환해도 평가 순서가 영향을 받지 않습니다.

컨테이너를 중첩할 수 있습니다. 컨테이너는 다른 컨테이너의 모듈을 사용할 수 있습니다. 컨테이너 간의 순환 의존성도 가능합니다.

### Overriding

컨테이너는 선택한 로컬 모듈을 "오버라이딩 가능하다"로 표시할 수 있습니다. 컨테이너 소비자는 컨테이너의 오버라이딩 가능한 모듈 중 하나를 대체할 수 있는 모듈인 "오버라이딩" 모듈을 제공할 수 있습니다. 컨테이너의 모든 모듈은 소비자가 제공할 때 로컬 모듈 대신 대체 모듈을 사용합니다. 소비자가 대체 모듈을 제공하지 않을 때, 컨테이너의 모든 모듈은 로컬 모듈을 사용합니다.

컨테이너는 소비자가 오버라이딩 할 때 다운로드가 필요 없는 방식으로 오버라이딩 가능한 모듈을 관리합니다. 이것은 일반적으로 별도의 청크로 배치하여 발생합니다.

반면에, 대체 모듈 공급자는 비동기 로딩 기능만 제공합니다. 이것은 컨테이너가 필요한 경우만 대체 모듈을 로드 할 수 있습니다. 공급자는 컨테이너에서 요청하지 않을 때 다운로드할 필요가 없는 방식으로 대체 모듈을 관리합니다. 이것은 일반적으로 별도의 청크로 배치하여 발생합니다.

"name"은 컨테이너에서 오버라이딩 가능한 모듈을 식별하는데 사용합니다.

오버라이드는 컨테이너가 모듈을 노출하는 것과 비슷한 방식으로 제공되며, 다음 두 단계로 구분됩니다.

1. 로딩(비동기)
2. 평가(비동기)

W> 중첩을 사용할 때, 하나의 컨테이너에게 오버라이드를 제공하면 중첩된 컨테이너에 있는 동일한 "name"을 가진 모듈들이 자동으로 오버라이드 됩니다.

컨테이너의 모듈이 로드 되기 전에 오버라이드를 제공해야 합니다. 초기 청크에 사용되는 오버라이딩 가능한 항목은, 오직 promise를 사용하지 않는 동기 모듈 오버라이드에 의해서만 오버라이드 될 수 있습니다. 일단 평가되면, 오버라이딩 가능한 항목은 더 이상 오버라이딩 할 수 없습니다.

## High-level concepts

각각의 빌드는 컨테이너의 역할을 하며 다른 빌드를 컨테이너로 소비하기도 합니다. 이렇게 하면 각각의 빌드가 해당 컨테이너에서 로드 하여 다른 노출된 모듈에 접근할 수 있습니다.

공유된 모듈은 오버라이딩 가능하고 중첩된 컨테이너에 오버라이드로 제공됩니다. 일반적으로 각 빌드에서 동일한 모듈(예, 동일한 라이브러리)을 가리킵니다.

`packageName` 옵션을 사용하면 `requiredVersion`을 찾도록 패키지 이름을 설정할 수 있습니다. 기본적으로 모듈 요청에 자동으로 추론되며, 자동 추론을 비활성화해야 하는 경우 `requiredVersion`을 `false`로 설정합니다.

## Building blocks

### `OverridablesPlugin` (low level)

이 플러그인은 특정 모듈을 "오버라이딩 가능하게" 만듭니다. 로컬 API(`__webpack_override__`)를 사용하면 오버라이드를 제공할 수 있습니다.

**webpack.config.js**

```js
const OverridablesPlugin = require('webpack/lib/container/OverridablesPlugin');
module.exports = {
  plugins: [
    new OverridablesPlugin([
      {
        // OverridablesPlugin으로 오버라이드 가능한 모듈을 정의합니다.
        test1: './src/test1.js',
      },
    ]),
  ],
};
```

**src/index.js**

```js
__webpack_override__({
  // 여기서 test1 모듈을 오버라이드 합니다.
  test1: () => 'I will override test1 module under src',
});
```

### `ContainerPlugin` (low level)

이 플러그인은 지정된 노출 모듈로 추가 컨테이너 엔트리을 만듭니다. 또한 내부적으로 `OverridablesPlugin`을 사용하고 `override` API를 컨테이너 소비자에게 노출합니다.

### `ContainerReferencePlugin` (low level)

이 플러그인은 컨테이너에 대한 특정 참조를 외부로 추가하고 컨테이너에서 원격 모듈을 가져올 수 있도록 합니다. 또한 컨테이너의 `override` API를 호출하여 오버라이드를 제공합니다. 로컬 오버라이드 (빌드도 컨테이너인 경우 `__webpack_override__` 또는 `override` API을 통해) 그리고 지정된 오버라이드가 모든 참조된 컨테이너에게 제공됩니다.

### `ModuleFederationPlugin` (low level)

이 플러그인은 `ContainerPlugin`과 `ContainerReferencePlugin`을 결합합니다. 오버라이드 및 오버라이드 가능한 항목은 지정된 공유 모듈의 단일 목록으로 결합됩니다.

## Concept goals

- webpack이 지원하는 모든 모듈 유형을 노출하고 사용할 수 있어야 합니다.
- 청크 로드는 필요한 모든 것을 병렬 로드 해야 합니다(웹: 서버까지 한번의 라운드트립).
- 소비자에서 컨테이너까지 제어.
    - 오버라이딩 모듈은 단방향 작업입니다.
    - 형제 컨테이너는 서로의 모듈을 오버라이드 할 수 없습니다.
- 환경에 의존하지 않습니다.
    - 웹, Node.js, 등에서 사용 가능합니다.
- 공유의 상대 및 절대 요청
    - 사용하지 않더라도 항상 제공됩니다.
    - 상대 경로를 `config.context`에서 확인할 수 있습니다.
    - 기본적으로 `requiredVersion`을 사용하지 않습니다.
- 공유 모듈 요청
    - 사용할 때만 제공됩니다.
    - 빌드에서 사용된 모든 동일 모듈 요청과 일치합니다.
    - 일치하는 모든 모듈을 제공합니다.
    - 그래프의 이 위치에 있는 package.json에서 `requiredVersion`을 추출합니다.
    - 중첩된 node_module이 있는 경우 여러 다른 버전을 제공하고 사용할 수 있습니다.
- 공유된 후행에 `/` 가 있는 모듈 요청은 이 접두사의 모든 모듈 요청과 일치합니다.

## Use cases

### Separate builds per page

단일 페이지 애플리케이션의 각 페이지는 별도의 빌드에서 컨테이너 빌드로부터 노출됩니다. 또한 애플리케이션 쉘은 모든 페이지를 원격 모듈로 참조하는 별도의 빌드입니다. 이렇게 하면 각 페이지를 별도로 배포할 수 있습니다. 경로가 업데이트되거나 새 경로가 추가되면 애플리케이션 쉘이 배포됩니다. 애플리케이션 쉘은 페이지 빌드에서 중복을 방지하기 위해 일반적으로 사용되는 라이브러리를 공유 모듈로 정의합니다.

### Components library as container

많은 애플리케이션은 각 컴포넌트가 노출된 컨테이너로 빌드 할 수 있는 공통 컴포넌트 라이브러리를 공유합니다. 각 애플리케이션은 컴포넌트 라이브러리 컨테이너의 컴포넌트를 사용합니다. 모든 애플리케이션을 다시 배포할 필요 없이 컴포넌트 라이브러리의 변경을 별도로 배포할 수 있습니다. 애플리케이션은 컴포넌트 라이브러리의 최신 버전을 자동으로 사용합니다.

## Dynamic Remote Containers

컨테이너 인터페이스는 `get` 및 `init` 메소드를 지원합니다.
`init`은 하나의 인수(공유 범위 객체)로 호출되는 `async` 호환 메소드입니다. 이 객체는 원격 컨테이너에서 공유 범위로 사용되며 호스트에서 제공된 모듈로 채워집니다.
원격 컨테이너를 런타임에 동적으로 호스트 컨테이너에 연결하는데 사용할 수 있습니다.

**init.js**

```js
(async () => {
  // 공유 범위를 초기화 합니다. 이 빌드와 모든 원격에서 제공된 모듈로 채웁니다
  await __webpack_init_sharing__('default');
  const container = window.someContainer; // 또는 다른 곳에서 컨테이너를 얻으십시오
  // 컨테이너 초기화, 공유 모듈 제공 가능합니다
  await container.init(__webpack_share_scopes__.default);
  const module = await container.get('./module');
})();
```

컨테이너는 공유 모듈을 제공하려고 시도하지만, 공유 모듈이 이미 사용된 경우, 경고 및 제공된 공유 모듈이 무시됩니다. 컨테이너는 여전히 대체 수단으로 사용할 수 있습니다.

이렇게 하면 다른 버전의 공유 모듈을 제공하는 A/B 테스트를 동적으로 로드 할 수 있습니다.

T> 원격 컨테이너를 동적으로 연결하기 전에 컨테이너를 로드 했는지 확인하세요.

예:

**init.js**

```js
function loadComponent(scope, module) {
  return async () => {
    // 공유 범위를 초기화 합니다. 이 빌드와 모든 원격에서 제공된 모듈로 채웁니다
    await __webpack_init_sharing__('default');
    const container = window[scope]; // 또는 다른 곳에서 컨테이너를 얻으십시오
    // 컨테이너 초기화, 공유 모듈 제공 가능합니다
    await container.init(__webpack_share_scopes__.default);
    const factory = await window[scope].get(module);
    const Module = factory();
    return Module;
  };
}

loadComponent('abtests', 'test123');
```

[전체 구현 보기](https://github.com/module-federation/module-federation-examples/tree/master/advanced-api/dynamic-remotes)

## Dynamic Public Path

### Offer a host API to set the publicPath

호스트가 원격 모듈의 메소드를 노출하여 런타임 원격 모듈의 publicPath를 설정하도록 허용할 수 있습니다.

이 접근 방식은 호스트 도메인의 하위 경로에 독립적으로 배포된 하위 응용 프로그램을 마운트 할 때 특히 유용합니다.

시나리오입니다.

`https://my-host.com/app/*`에 호스팅 된 호스트 앱과 `https://foo-app.com`에 호스팅 된 하위 앱이 있습니다.
하위 앱도 호스트에 마운트되므로, `https://foo-app.com`은 `https://my-host.com/app/foo-app`와 `https://my-host.com/app/foo/*`를 통해 접근할 수 있습니다. 요청은 프록시를 통해 `https://foo-app.com/*`로 리다이렉션 됩니다.

예제입니다.

**webpack.config.js (remote)**

```js
module.exports = {
  entry: {
    remote: './public-path',
  },
  plugins: [
    new ModuleFederationPlugin({
      name: 'remote', // 이 이름은 엔트리 이름과 일치해야 합니다.
      exposes: ['./public-path'],
      // ...
    }),
  ],
};
```

**public-path.js (remote)**

```js
export function set(value) {
  __webpack_public_path__ = value;
}
```

**src/index.js (host)**

<!-- eslint-skip -->

```js
const publicPath = await import('remote/public-path');
publicPath.set('/your-public-path');

//boostrap app  e.g. import('./boostrap.js')
```

### Infer publicPath from script

`document.currentScript.src`의 스크립트 태그에서 publicPath를 유추하고 런타임에 [`__webpack_public_path__`](/api/module-variables/#__webpack_public_path__-webpack-specific) 모듈 변수로 설정할 수 있습니다.

예제입니다.

**webpack.config.js (remote)**

```js
module.exports = {
  entry: {
    remote: './setup-public-path',
  },
  plugins: [
    new ModuleFederationPlugin({
      name: 'remote', // 이 이름은 엔트리 이름과 일치해야 합니다.
      // ...
    }),
  ],
};
```

**setup-public-path.js (remote)**

```js
// 자체 로직으로 publicPath를 파생하고 __webpack_public_path__ API로 설정하세요.
__webpack_public_path__ = document.currentScript.src + '/../';
```

T> 또한 자동으로 publicPath를 결정하는 [`output.publicPath`](/configuration/output/#outputpublicpath)에 사용할 수 있는 `'auto'` 값이 있습니다.

## Troubleshooting

**`Uncaught Error: Shared module is not available for eager consumption`**

애플리케이션은 전 방향 호스트로 작동하는 애플리케이션을 실행하고 있습니다. 선택할 수 있는 옵션이 있습니다.

모듈을 비동기 청크에 넣지 않고 동기식으로 제공하는 Module Federation 고급 API 내에서 의존성을 eager로 설정할 수 있습니다. 이를 통해 초기 청크에서 이러한 공유 모듈을 사용할 수 있습니다. 그러나 제공된 모든 모듈과 예비 모듈은 항상 다운로드 되므로 주의하세요. 쉘 같이 애플리케이션의 한 지점에서만 제공하는 것이 좋습니다.

비동기 경계를 사용하는 것을 추천합니다. 추가 왕복을 방지하고 일반적인 성능 향상을 위해 더 큰 청크의 초기화 코드를 분할합니다.

예를 들어, 엔트리는 다음과 같습니다.

**index.js**

```js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
ReactDOM.render(<App />, document.getElementById('root'));
```

`bootstrap.js` 파일을 만들고 엔트리의 콘텐츠을 이 파일로 이동한 다음, 부트스트랩을 엔트리로 가져오겠습니다.

**index.js**

```diff
+ import('./bootstrap');
- import React from 'react';
- import ReactDOM from 'react-dom';
- import App from './App';
- ReactDOM.render(<App />, document.getElementById('root'));
```

**bootstrap.js**

```diff
+ import React from 'react';
+ import ReactDOM from 'react-dom';
+ import App from './App';
+ ReactDOM.render(<App />, document.getElementById('root'));
```

이 메소드는 동작하지만 제약사항이나 단점이 있을 수 있습니다.

`ModuleFederationPlugin`을 통해 의존성을 `eager: true`로 설정합니다.

**webpack.config.js**

```js
// ...
new ModuleFederationPlugin({
  shared: {
    ...deps,
    react: {
      eager: true,
    },
  },
});
```

**`Uncaught Error: Module "./Button" does not exist in container.`**

`"./Button"`은 표시되지 않지만, 오류 메세지는 유사하게 표시됩니다. 이 문제는 일반적으로 webpack beta.16에서 webpack beta.17로 업그레이드 하는 경우에 나타납니다.

ModuleFederationPlugin 내에서 다음 위치에서 exposes를 변경합니다.

```diff
new ModuleFederationPlugin({
  exposes: {
-   'Button': './src/Button'
+   './Button':'./src/Button'
  }
});
```

**`Uncaught TypeError: fn is not a function`**

원격 컨테이너가 누락되었을 가능성이 있으므로 추가되었는지 확인하세요.
사용하려는 원격 컨테이너가 로드되었지만 이 오류가 계속 나타나면 호스트 컨테이너의 원격 컨테이너 파일도 HTML에 추가하세요.
