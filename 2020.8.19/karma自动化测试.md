### karma + Mocha 单元测试

1. Karma（[ˈkɑrmə] 卡玛）是一个测试运行器，它可以呼起浏览器，加载测试脚本，然后运行测试用例
2. Mocha（[ˈmoʊkə] 摩卡）是一个单元测试框架/库，它可以用来写测试用例
3. Sinon（西农）是一个 spy / stub / mock 库，用以辅助测试


### 步骤
1. 安装各种工具
··
npm i -D karma karma-chrome-launcher karma-mocha karma-sinon-chai mocha sinon sinon-chai karma-chai karma-chai-spies
···
2. 创建karma