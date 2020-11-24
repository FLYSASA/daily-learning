### 步骤：

[配置参考](https://vue-test-utils.vuejs.org/zh/installation/testing-single-file-components-with-karma.html)
#### 1. 创建karma.conf.js
```js
var webpackConfig = require('@vue/cli-service/webpack.config.js')

module.exports = function (config) {
  config.set({
    frameworks: ['mocha'],

    files: [
      'tests/**/*.spec.js'
    ],

    preprocessors: {
      '**/*.spec.js': ['webpack', 'sourcemap']
    },

    webpack: webpackConfig,

    reporters: ['spec'],
    autoWatch: true,

    browsers: ['ChromeHeadless']
  })
}
```

#### 2. 安装依赖
```
npm i -D karma karma-chrome-launcher karma-mocha karma-sourcemap-loader karma-spec-reporter karma-webpack chai sinon sinon-chai
```

#### 3. 改写脚本
```js
// package.json
"scripts": {
  "start": "npm run serve",
  "serve": "vue-cli-service serve --open",
  "build": "vue-cli-service build",
  "test": "karma start --single-run",
  "test:unit": "karma start",
  "test:unit:cli": "vue-cli-service test:unit",
  "watch": "vue-cli-service test:unit --watch"
},
```