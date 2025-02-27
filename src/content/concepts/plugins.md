---
title: Plugins
sort: 4
contributors:
  - TheLarkInn
  - jhnns
  - rouzbeh84
  - johnstew
  - MisterDev
  - byzyk
  - chenxsan
---

**플러그인은** webpack의 [기본](https://github.com/webpack/tapable)입니다. webpack 자체는 webpack 설정에서 사용하는 것과 **동일한 플러그인 시스템으로** 구축되어 있습니다.

[로더](/concepts/loaders)가 할 수 없는 **다른 작업을** 수행할 목적으로 제공됩니다.

T> 플러그인에서 [`webpack-sources`](https://github.com/webpack/webpack-sources) 패키지를 사용할 때 `require('webpack-sources')` 대신 `require('webpack').sources`를 사용하면, 영구 캐시에 대한 버전 충돌을 방지합니다.

## Anatomy

webpack **플러그인은** [`apply`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply) 메서드를 가지고 있는 객체입니다. `apply` 메서드는 webpack 컴파일러에 의해 호출되며, **전체** 컴파일 라이프사이클에 접근 할 수 있습니다.

**ConsoleLogOnBuildWebpackPlugin.js**

```javascript
const pluginName = 'ConsoleLogOnBuildWebpackPlugin';

class ConsoleLogOnBuildWebpackPlugin {
  apply(compiler) {
    compiler.hooks.run.tap(pluginName, (compilation) => {
      console.log('The webpack build process is starting!!!');
    });
  }
}

module.exports = ConsoleLogOnBuildWebpackPlugin;
```

컴파일러 훅의 탭 메서드의 첫번째 파라미터는 플러그인 이름의 Camelcase 버전으로 작성해야 합니다. 모든 훅에서 재사용 할 수 있도록 하기 위해서 상수를 사용하는 것이 좋습니다.

## Usage

**플러그인은** 매개변수 및 옵션을 사용할 수 있으므로, webpack 설정에서 `plugins` 속성에 `새로운` 인스턴스로 전달해야 합니다.

webpack을 사용하는 방법에 따라 여러 가지 방법으로 플러그인을 사용 할 수 있습니다.

### Configuration

**webpack.config.js**

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin'); //npm 으로 설치됨
const webpack = require('webpack'); //빌트인 플러그인에 접근하기 위함
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    filename: 'my-first-webpack.bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: 'babel-loader',
      },
    ],
  },
  plugins: [
    new webpack.ProgressPlugin(),
    new HtmlWebpackPlugin({ template: './src/index.html' }),
  ],
};
```

`ProgressPlugin`은 컴파일하는 동안 리포트를 사용자 정의로 생성 할 수 있으며 `HtmlWebpackPlugin`은 `script`를 사용하여 `my-first-webpack.bundle.js` 파일을 포함하는 HTML 파일을 생성합니다.

### Node API

Node API를 사용할 때 설정의 `플러그인` 속성을 통해 플러그인을 전달 할 수도 있습니다.

**some-node-script.js**

```javascript
const webpack = require('webpack'); //webpack 런타임에 접근하기 위함
const configuration = require('./webpack.config.js');

let compiler = webpack(configuration);

new webpack.ProgressPlugin().apply(compiler);

compiler.run(function (err, stats) {
  // ...
});
```

T> 위에서 본 예제가 [webpack 런타임 자체](https://github.com/webpack/webpack/blob/e7087ffeda7fa37dfe2ca70b5593c6e899629a2c/bin/webpack.js#L290-L292)와 매우 유사한 걸 눈치채셨나요? [webpack 소스 코드](https://github.com/webpack/webpack)에는 사용자의 설정과 스크립트에 적용할 수 있는 훌륭하고 많은 사용 사례가 숨어 있습니다!
