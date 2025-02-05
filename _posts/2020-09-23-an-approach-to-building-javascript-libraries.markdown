---
layout: post
title: "一种构建 JavaScript 库的方法"
date: 2020-09-23 15:08:06 +0800
comments: true
categories: [Archives, Web Development]
keywords: Library, Module, CommonJS, AMD, UMD, TypeScript, ECMAScript Harmony, rollup
description: An Approach to Building JavaScript Libraries.
---
最近手头有个项目需要构建 JavaScript 库，核心需求是库既能用于传统的多页面，也能用于 Angular 的单页面。目前网络上暂时好像没有这类详细教程，反而是一些理论阐述的文章较多，经过一番研究后，我得到了一个适合自己的方法。  

由于我希望库既能用于多页面，也能用于 Angular 单页面，所以需要支持 UMD 和 ES Harmony。由于 Angular 是使用 TypeScript 开发，最好还能提供用于 TypeScript 的声明文件。最原始的想法自然是手动按要求提供各种文件，但这样工作量比较大，也不容易扩展。那么还有什么容易的办法吗？有的，最核心的想法就是库的源码只写一份，然后用工具生成各种模块系统需要文件。具体的做法可能有差异，但理念是一样的。  

从我的需求出发，我最终选择用 TypeScript 来写库的源码，基于脱敏的考虑，这里选择 TypeScript 文档中的示例代码来演示。  

首先我们建立好库的源码目录结构并配置好源码管理：  

{% codeblock %}
simple-module-example
├── src
│   ├── LettersOnlyValidator.ts
│   ├── Validation.ts
│   ├── ZipCodeValidator.ts
│   └── index.ts
{% endcodeblock %}

然后使用 `npm init` 生成 `package.json` 文件：

{% codeblock %}
{
  "name": "simple-module-example",
  "version": "1.0.0",
  "description": "A simple module example.",
  "main": "index.js",
  "directories": {
    "test": "test"
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "module",
    "example"
  ],
  "author": "Meiliang Dong",
  "license": "MIT"
}

{% endcodeblock %}
<!--more-->
其次是按需求编辑好 `package.json` 文件。这是关键步骤，`package.json` 的 main 字段通常用于指向 UMD 版本的库；module 字段则用于指向 ES 版本的库。我们还需要配置构建脚本生成对应版本的库，最终的 `package.json` 文件内容如下： 

{% codeblock %}
{
    "name": "simple-module-example",
    "version": "1.0.0",
    "description": "A simple module example.",
    "main": "dist/umd/simple-module-example.js",
    "module": "dist/esm/index.js",
    "browser": "dist/umd/simple-module-example.js",
    "types": "dist/types/index.d.ts",
    "files": [
        "dist"
    ],
    "directories": {
        "test": "test"
    },
    "scripts": {
        "clean": "rm -rf ./dist",
        "build": "npm run clean && npm run build:es2015 && npm run build:esm && npm run build:cjs && npm run build:umd",
        "build:es2015": "tsc --module es2015 --target es2015 --outDir dist/es2015",
        "build:esm": "tsc --module es2015 --outDir dist/esm",
        "build:umd": "rollup dist/esm/index.js --format umd --name SimpleModuleExample --sourcemap --file dist/umd/simple-module-example.js",
        "build:umd:min": "cd dist/umd && uglifyjs --compress --mangle --source-map --screw-ie8 --comments --o simple-module-example.min.js -- simple-module-example.js && gzip simple-module-example.min.js -c > simple-module-example.min.js.gz"
    },
    "keywords": [
        "module",
        "example"
    ],
    "author": "Meiliang Dong",
    "license": "MIT"
}
{% endcodeblock %}

然后是编写 TypeScript 的配置文件 `tsconfig.json`， 先使用命令 `npm install @tsconfig/recommended --save-dev` 安装推荐的配置，之后根据需求定制，最终的内容如下：  

{% codeblock %}
{
    "extends": "@tsconfig/recommended/tsconfig.json",
    "compilerOptions": {
      "outDir": "./dist",
      "target": "es5",
      "sourceMap": true,
      "declaration": true,
      "declarationDir": "./dist/types"
    },
    "include": ["./src/**/*"]
  }
{% endcodeblock %}

其次是测试，测试是库开发的重要环节，它能帮我们验证库是否正常工作，后续迭代重构也要依赖它。通常的做法是使用测试框架，像 Angular 是 [Karma test runner](https://karma-runner.github.io/)  搭配[Jasmine test framework](https://jasmine.github.io/) ， 我们可以参考选择。  

我这里还玩了一下用 rollup 打包, 然后在浏览器里运行测试用例。首先在库工程目录外重新创建一个测试工程，然后使用 `npm link` 命令来安装我们的开发库，具体目录结构和文件内容如下：

{% codeblock %}
$mkdir test-simple-module-example
├── Test.ts
├── dist
│   ├── esm
│   └── test-simple-module-example.js
├── index.html
├── package-lock.json
├── package.json
├── rollup.config.js
└── tsconfig.json
{% endcodeblock %}

`Test.ts`

{% codeblock %}
import { StringValidator, ZipCodeValidator, LettersOnlyValidator } from "simple-module-example";

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validators: { [s: string]: StringValidator } = {};
validators["ZIP code"] = new ZipCodeValidator();
validators["Letters only"] = new LettersOnlyValidator();

// Show whether each string passed each validator
strings.forEach((s) => {
  for (let name in validators) {
    console.log(
      `"${s}" - ${
        validators[name].isAcceptable(s) ? "matches" : "does not match"
      } ${name}`
    );
  }
});
{% endcodeblock %}

`package.json`

{% codeblock %}
{
    "name": "test-simple-module-example",
    "version": "1.0.0",
    "description": "A test application for simple module example.",
    "main": ".dist/Test.js",
    "scripts": {
        "clean": "rm -rf ./dist",
        "build": "npm run clean && npm run build:esm && npm run build:bundle",
        "build:esm": "tsc --module es2015 --outDir dist/esm",
        "build:bundle": "rollup -c",
        "test": "echo \"Error: no test specified\" && exit 1"
    },
    "author": "Meiliang Dong",
    "license": "MIT",
    "dependencies": {
        "@tsconfig/recommended": "^1.0.1"
    },
    "devDependencies": {
        "@rollup/plugin-node-resolve": "^9.0.0"
    }
}
{% endcodeblock %}

`rollup.config.js`

{% codeblock %}
import resolve from '@rollup/plugin-node-resolve';

export default {
    input: 'dist/esm/Test.js',
    output: {
        file: 'dist/test-simple-module-example.js',
        format: 'umd'
    },
    plugins: [resolve()]
};
{% endcodeblock %}

`tsconfig.json`

{% codeblock %}
{
  "extends": "@tsconfig/recommended/tsconfig.json",
  "compilerOptions": {
    "outDir": "./dist",
    "target": "es5",
    "sourceMap": true,
    "moduleResolution": "Node"
  },
  "include": ["./*.ts"]
}
{% endcodeblock %}


最后是根据需要选择发布方式，例如发布到 npm 公有仓库，具体做法参考官方文档就好了。

###Reference  

* [Learn the basics of the JavaScript module system and build your own library](https://www.freecodecamp.org/news/anatomy-of-js-module-systems-and-building-libraries-fadcd8dbd0e/)  
* [TypeScript output es2015, esm (ES Modules), CJS, UMD, UMD + Min + Gzip.](https://gist.github.com/jayphelps/51bafb4505558736fdba0aaf8bfe69d3)  
* [It's Not Hard: Making Your Library Support AMD and CommonJS](http://ifandelse.com/its-not-hard-making-your-library-support-amd-and-commonjs/)  
* [Writing Modular JavaScript With AMD, CommonJS & ES Harmony](https://addyosmani.com/writing-modular-js/)  
* [如何写一个现代的JavaScript库](https://yanhaijing.com/javascript/2018/08/17/2020-js-lib/)  

