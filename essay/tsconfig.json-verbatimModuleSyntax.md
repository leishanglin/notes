# tsconfig.json 中 `verbatimModuleSyntax` 配置的作用

`verbatimModuleSyntax` 配置项用于控制 TypeScript 编译器如何处理模块语法：

```json
{
  "compilerOptions": {
    "verbatimModuleSyntax": true
  }
}
```

设为 `true` 后，TypeScript 输出 JavaScript 代码时，会尽量保持模块语法不变，无论是 export 还是 import，都会直接保留在生成的 JavaScript 代码中。

在大多数项目中，源码都会被一些现代化的构建工具进行处理，比如：Webpack、Rollup、ESBuild 等，这些构建工具可以处理并优化原始的 ES 模块语法，因此 TypeScript 没必要再额外地提前处理一次。

## 与 `target` 的区别

如果只是为了让 TypeScript 在编译时保留 ES 模块语法，那不一定非要启用 `verbatimModuleSyntax`，直接将 `target` 设为 `ES6` 或之后的版本就好：

```json
{
  "compilerOptions": {
    "target": "ES6"
  }
}
```

它跟启用 `verbatimModuleSyntax` 的区别是：`target` 只是在“输出”时决定输出为支持 ES 模块语法的那个版本，而 `verbatimModuleSyntax` 则是不处理源码中的 ES 模块语法，“原封不动”地输出。

所以，`verbatimModuleSyntax` 更加适合现代 JavaScript 构建工具。

## 启用 `verbatimModuleSyntax` 的前提

使用 `verbatimModuleSyntax` 有一个前提，那就是 `module` 得是 ES 模块：

```json
{
  "compilerOptions": {
    "module": "ES2015"
  }
}
```

如果设置成 `CommonJS`，在使用 ES 模块时就会报错：

```ts
// ESM syntax is not allowed in a CommonJS module when 'verbatimModuleSyntax' is enabled.ts(1286)
import { chunk } from "lodash";

chunk([1, 2, 3, 4], 2);
```

这很好理解，因为启用 `verbatimModuleSyntax` 的目的就是不转换 ES 模块语法，如果源码不使用 ES 模块语法写的，那自然就无法处理了。

一般来说，`module` 不用显式指定为 ES 模块，因为它默认就是。

## import type ...

在 TypeScript 中，import 语句不仅可以导入模块，还能导入类型，在启用 `verbatimModuleSyntax` 之后，由于 TypeScript 编译器不再处理 import、export 语法，所以导入类型时需要写成 `import type ...`，否则就会报错，这也是启用 `verbatimModuleSyntax` 之后，给开发者最直观的表现：

```ts
// import { TypeA } from "./moduleA"; // 报错
import type { TypeA } from "./moduleA"; // 不报错
```

## 总结

如果源代码用的是 ES 模块语法，并且项目打包使用了 Webpack、ESBuild、Rollup、Vite 等，建议都启用 `verbatimModuleSyntax`，这样编译速度更快，同时 `import type` 这种写法也会更加直观得表名，引入的到底是模块还是类型。